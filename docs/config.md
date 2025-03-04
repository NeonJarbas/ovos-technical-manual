# Configuration

- [Reading Configuration](#reading-configuration)
- [Configuring Configuration](#configuring-configuration)
    * [protected_keys](#protected-keys)
    * [disable_user_config](#disable-user-config)
    * [disable_remote_config](#disable-remote-config)
- [Meta Configuration](#meta-configuration)
    * [ovos.conf](#ovosconf)

## Reading Configuration

The configuration files loaded are determined by `ovos.conf` as described in the next section and can be in either json or
yaml format.

if `Configuration()` is called the following configs would be loaded in this order:

- `{ovos-config-package}/mycroft.conf`
- `os.environ.get('MYCROFT_SYSTEM_CONFIG')` or `/etc/mycroft/mycroft.conf`
- `os.environ.get('MYCROFT_WEB_CACHE')` or `$XDG_CONFIG_PATH/mycroft/web_cache.json`
- `$XDG_CONFIG_DIRS/mycroft/mycroft.conf`
- `/etc/xdg/mycroft/mycroft.conf`
- `$XDG_CONFIG_HOME/mycroft/mycroft.conf` (default `~/.config/mycroft/mycroft.conf`)

When the configuration loader starts, it looks in these locations in this order, and loads ALL configurations. Keys that
exist in multiple configuration files will be overridden by the last file to contain the value. This process results in
a minimal amount being written for a specific device and user, without modifying default distribution files.

## Configuring Configuration

There are a couple of special configuration keys that change the way the configuration stack loads.

* `Default` config refers to the config specified at `default_config_path` in
  `ovos.conf` (#1 `{ovos-config-package}/mycroft.conf` in the stack above).
* `System` config refers to the config at `/etc/{base_folder}/{config_filename}` (#2 `/etc/mycroft/mycroft.conf` in the stack
  above).

### protected_keys

A `"protected_keys"` configuration section may be added to a `Default` or `System` Config file
(default `/etc/mycroft/mycroft.conf`). This configuration section specifies
other configuration keys that may not be specified in `remote` or `user` configurations.
Keys may specify nested parameters with `.` to exclude specific keys within nested dictionaries.
An example config could be:

```json
{
  "protected_keys": {
    "remote": [
      "gui_websocket.host",
      "websocket.host"
    ],
    "user": [
      "gui_websocket.host"
    ]
  }
}
```

This example specifies that `config['gui_websocket']['host']` may be specified in user configuration, but not remote.
`config['websocket']['host']` may not be specified in user or remote config, so it will only consider default
and system configurations.

### disable_user_config

If this config parameter is set to True in `Default` or `System` configuration,
no user configurations will be loaded (no XDG configuration paths).

### disable_remote_config

If this config parameter is set to True in `Default` or `System` configuration,
the remote configuration (`web_cache.json`) will not be loaded.


## Meta Configuration

The `ovos_config` package determines which config files to load based on `ovos.conf`. This file is optional and does **NOT** need to exist

while `mycroft.conf` configures the voice assistant, `ovos.conf` configures the library

all XDG paths across OpenVoiceOS packages build their paths taking `"base_folder"` from `ovos.conf` into consideration

`ovos.conf` decides what files are loaded by the `Configuration` class described above, as an end user or skill developer you should never have to worry about this

`get_ovos_config` will return default values that load `mycroft.conf` unless otherwise configured.


`ovos.conf` files are loaded in the following order, with later files taking priority over earlier ones in the list:

- `/etc/OpenVoiceOS/ovos.conf`
- `$XDG_CONFIG_DIRS/OpenVoiceOS/ovos.conf`
- `/etc/xdg/OpenVoiceOS/ovos.conf`
- `$XDG_CONFIG_HOME/OpenVoiceOS/ovos.conf`  (default `~/.config/OpenVoiceOS/ovos.conf`)


A simple `ovos_config` should have a structure like:

```json
{
  "base_folder": "mycroft",
  "config_filename": "mycroft.conf",
  "default_config_path": "<Absolute Path to ovos-config>/mycroft.conf",
  "module_overrides": {},
  "submodule_mappings": {}
}
```

### Config in downstream packages

`ovos.conf` allows downstream voice assistants such as neon-core to change their config files to `neon.yaml`

```json
{
  "base_folder": "mycroft",
  "config_filename": "mycroft.conf",
  "default_config_path": "<Absolute Path to ovos-config>/mycroft.conf",
  
  "module_overrides": {
    "neon_core": {
      "base_folder": "neon",
      "config_filename": "neon.yaml",
      "default_config_path": "/etc/example/config/neon.yaml"
    }
  },
  
  "submodule_mappings": {
    "neon_messagebus": "neon_core",
    "neon_speech": "neon_core",
    "neon_audio": "neon_core",
    "neon_gui": "neon_core"
  }
}
```

> *Note*: `default_config_path` should always be an absolute path. any manual override must specify an absolute path to a json or yaml config file.


Using the above example, if `Configuration()` is called from `neon-core`, the following configs would be loaded in this
order:

- `/etc/example/config/neon.yaml`
- `os.environ.get('MYCROFT_SYSTEM_CONFIG')` or `/etc/neon/neon.yaml`
- `os.environ.get('MYCROFT_WEB_CACHE')` or `$XDG_CONFIG_PATH/neon/web_cache.json`
- `$XDG_CONFIG_DIRS/neon/neon.yaml`
- `/etc/xdg/neon/neon.yaml`
- `$XDG_CONFIG_HOME/neon/neon.yaml` (default `~/.config/neon/neon.yaml`)

A call to `get_ovos_config` from `neon_core` or `neon_messagebus` will return a configuration like:

```json
{
  "base_folder": "neon",
  "config_filename": "neon.yaml",
  "default_config_path": "/etc/example/config/neon.yaml",
  "module_overrides": {
    "neon_core": {
      "base_folder": "neon",
      "config_filename": "neon.yaml",
      "default_config_path": "/etc/example/config/neon.yaml"
    }
  },
  "submodule_mappings": {
    "neon_messagebus": "neon_core",
    "neon_speech": "neon_core",
    "neon_audio": "neon_core",
    "neon_gui": "neon_core"
  }
}
```

If `get_ovos_config` was called from `ovos_core` with the same configuration file as the last example,
the returned configuration would be:

```json
{
  "base_folder": "mycroft",
  "config_filename": "mycroft.conf",
  "default_config_path": "<Path to ovos-config>/mycroft.conf",
  "module_overrides": {
    "neon_core": {
      "base_folder": "neon",
      "config_filename": "neon.yaml",
      "default_config_path": "/etc/example/config/neon.yaml"
    }
  },
  "submodule_mappings": {
    "neon_messagebus": "neon_core",
    "neon_speech": "neon_core",
    "neon_audio": "neon_core",
    "neon_gui": "neon_core"
  }
}
```

Both projects could be installed side by side and each would load their corresponding config files