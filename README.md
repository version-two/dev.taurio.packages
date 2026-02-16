# Taurio Packages

Package definitions for [Taurio](https://github.com/version-two/dev.taurio.app). Each entry in `packages.json` describes how to download, install, configure, and manage a service or tool.

## Structure

`packages.json` is a JSON array of package objects. The Taurio app fetches this file on startup and seeds it into the local package registry.

## Path Variables

All paths use the `{{TAURIO_ROOT}}` placeholder instead of hardcoded directories. Taurio expands this at runtime to the actual install location (default `C:\Taurio`). Never use absolute paths - always use `{{TAURIO_ROOT}}`.

---

## Package Fields

### Top-level

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | `string` | yes | Unique identifier (e.g. `"php-8.4"`, `"mysql"`, `"composer"`) |
| `name` | `string` | yes | Display name shown in UI |
| `version` | `string` | yes | Semantic version of the bundled software |
| `category` | `string` | yes | `"service"` (long-running process) or `"tool"` (CLI utility) |
| `tags` | `string[]` | no | Freeform tags for filtering (e.g. `["database", "sql"]`) |
| `preselect_default` | `boolean` | yes | Whether this package is checked by default during onboarding |
| `icon` | `string` | no | Icon identifier used by the UI |
| `description` | `string` | no | Short description shown in package listings |
| `sources` | `Source[]` | yes | Download sources (see below) |
| `install` | `InstallStep[]` | yes | Steps to install the package |
| `uninstall` | `UninstallStep[]` | yes | Steps to uninstall the package |
| `shims` | `Shim[]` | no | CLI shims to create in `{{TAURIO_ROOT}}\shims\` |
| `config` | `Config` | no | Runtime/service configuration written on install |
| `commands` | `Commands` | no | Start/stop/reload commands for services |
| `overrides` | `Overrides` | no | User-editable config fields (ports, paths, etc.) |
| `ui` | `UI` | no | Management panels shown on the package detail page |
| `scripts` | `Script[]` | no | Executable CLI scripts with parameters |
| `intelligence` | `Intelligence` | no | Auto-detection and recommendation hints |

---

### Sources

Download locations for the package files.

```json
{
  "source_type": "url",
  "url": "https://example.com/package.zip",
  "sha256": null
}
```

| Field | Type | Description |
|-------|------|-------------|
| `source_type` | `string` | Always `"url"` |
| `url` | `string` | Direct download URL |
| `sha256` | `string \| null` | Optional SHA-256 hash for verification |

---

### Install Steps

Executed in order during installation. Reference sources by index: `"source:0"`, `"source:1"`, etc.

#### `extract` - Unzip an archive to a directory

```json
{
  "step_type": "extract",
  "from": "source:0",
  "to": "{{TAURIO_ROOT}}\\bin\\nginx\\nginx-1.27.4"
}
```

#### `copy_source` - Copy a single downloaded file

```json
{
  "step_type": "copy_source",
  "from": "source:0",
  "to": "{{TAURIO_ROOT}}\\bin\\composer\\composer.phar"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `step_type` | `string` | `"extract"` or `"copy_source"` |
| `from` | `string` | Source reference (`"source:N"`) |
| `to` | `string` | Destination path (use `{{TAURIO_ROOT}}`) |

### Uninstall Steps

#### `remove_path` - Delete a file or directory

```json
{
  "step_type": "remove_path",
  "path": "{{TAURIO_ROOT}}\\bin\\nginx\\nginx-1.27.4"
}
```

---

### Shims

CLI shims created in `{{TAURIO_ROOT}}\shims\` so tools are available on PATH.

```json
{
  "name": "php",
  "target": "{{TAURIO_ROOT}}\\bin\\php\\php-8.4\\php.exe"
}
```

With arguments (e.g. composer wraps php + phar):

```json
{
  "name": "composer",
  "target": "{{TAURIO_ROOT}}\\bin\\php\\php-8.4\\php.exe",
  "args": ["{{TAURIO_ROOT}}\\bin\\composer\\composer.phar"]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `name` | `string` | Command name (creates `name.cmd` shim) |
| `target` | `string` | Path to the actual executable |
| `args` | `string[]` | Optional arguments prepended before user args |

---

### Config

Written once during installation. Tells Taurio how to register the package.

```json
{
  "tool_type": "php_runtime",
  "app_config": { ... },
  "config_files": [ ... ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `tool_type` | `string` | Registration type (see values below) |
| `app_config` | `object` | Type-specific configuration data |
| `config_files` | `ConfigFile[]` | Config files to create/populate on install |

#### `tool_type` values

| Value | Description |
|-------|-------------|
| `"php_runtime"` | Registers as a PHP runtime in `php_runtimes` |
| `"nginx"` | Registers as the nginx server |
| `"generic_service"` | Registers as a generic managed service |
| `"mysql"` | MySQL-specific registration |
| `"redis"` | Redis-specific registration |
| `"acme_client"` | ACME/Let's Encrypt client |

#### `app_config` for `php_runtime`

```json
{
  "version": "8.4",
  "php_path": "{{TAURIO_ROOT}}\\bin\\php\\...\\php.exe",
  "php_cgi_path": "{{TAURIO_ROOT}}\\bin\\php\\...\\php-cgi.exe",
  "fcgi_port": 9000,
  "fcgi_mode": "pool"
}
```

| `fcgi_mode` | Description |
|-------------|-------------|
| `"pool"` | Multi-worker FastCGI pool managed by taurio-fpm (fastest, recommended) |
| `"fcgi"` | Single persistent php-cgi process |
| `"off"` | CLI only, no FastCGI |

#### `app_config` for `generic_service`

```json
{
  "service_name": "mysql",
  "command": "{{TAURIO_ROOT}}\\bin\\mysql\\...\\mysqld.exe",
  "args": ["--defaults-file=..."],
  "working_dir": "{{TAURIO_ROOT}}\\bin\\mysql\\..."
}
```

#### Config Files

Config files that Taurio writes during installation (e.g. php.ini, my.ini).

```json
{
  "path": "{{TAURIO_ROOT}}\\bin\\php\\...\\php.ini",
  "separator": "=",
  "comment_prefix": ";",
  "settings": {
    "memory_limit": "512M",
    "max_execution_time": "300"
  },
  "install_overrides": { },
  "append_lines": ["extension=curl", "extension=mbstring"]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `path` | `string` | Path to the config file (use `{{TAURIO_ROOT}}`) |
| `separator` | `string` | Key-value separator: `"="`, `" = "`, `":"`, `" "` |
| `comment_prefix` | `string` | Comment character: `";"`, `"#"`, `"//"` |
| `settings` | `object` | Key-value pairs to write |
| `install_overrides` | `object` | Optional settings applied only on first install |
| `append_lines` | `string[]` | Optional raw lines appended to the end of the file |

---

### Commands

Start/stop/reload commands for services.

```json
{
  "start": {
    "command": "{{TAURIO_ROOT}}\\bin\\...\\mysqld.exe",
    "args": ["--defaults-file=..."],
    "working_dir": "{{TAURIO_ROOT}}\\bin\\...",
    "label": "Start MySQL"
  },
  "stop": {
    "command": "{{TAURIO_ROOT}}\\bin\\...\\mysqladmin.exe",
    "args": ["-u", "root", "shutdown"],
    "label": "Stop MySQL"
  },
  "reload": null,
  "quick_actions": []
}
```

| Field | Type | Description |
|-------|------|-------------|
| `start` | `CommandDef \| null` | Start command |
| `stop` | `CommandDef \| null` | Stop command |
| `reload` | `CommandDef \| null` | Reload/restart command |
| `quick_actions` | `QuickAction[]` | Additional action buttons shown in UI |

#### Quick Actions

```json
{
  "id": "reset-root-password",
  "label": "Reset Root Password",
  "command": "{{TAURIO_ROOT}}\\bin\\...\\mysql.exe",
  "args": ["-u", "root", "-e", "ALTER USER 'root'@'localhost' IDENTIFIED BY '{{password}}'"],
  "params": [
    {
      "name": "password",
      "label": "New Password",
      "type": "password",
      "required": true
    }
  ]
}
```

---

### Overrides

User-configurable fields that persist across updates. Values are stored in per-package override files.

```json
{
  "fields": [
    {
      "key": "port",
      "label": "Port",
      "type": "number",
      "default": "3306",
      "config_file": "{{TAURIO_ROOT}}\\bin\\...\\my.ini",
      "config_key": "port"
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `key` | `string` | Override key name |
| `label` | `string` | Display label in settings UI |
| `type` | `string` | Input type (see below) |
| `default` | `string \| null` | Default value |
| `config_file` | `string \| null` | Config file to update when value changes |
| `config_key` | `string \| null` | Key in the config file to update |

#### Override field types

| Type | Description |
|------|-------------|
| `"number"` | Numeric input |
| `"text"` | Text input |
| `"password"` | Masked password input |
| `"select:val1=Label 1,val2=Label 2"` | Dropdown selector. Format: comma-separated `value=label` pairs after `select:` prefix |

---

### UI Panels

Management panels shown on the package detail page.

```json
{
  "panels": [
    { "id": "config", "type": "config-editor", "label": "Configuration", "config": { ... } },
    { "id": "logs", "type": "log-viewer", "label": "Logs", "config": { ... } }
  ]
}
```

#### Panel types

| Type | Config | Description |
|------|--------|-------------|
| `config-editor` | `{ "file": "path", "separator": "=", "comment_prefix": ";" }` | INI/conf file editor |
| `log-viewer` | `{ "file": "path", "lines": 200 }` | Tail log file viewer |
| `db-manager` | `{ "engine": "mysql", "host": "127.0.0.1", "port_override_key": "port", "cli_path": "path" }` | Database management panel |
| `action-list` | `{ "actions": [{ "id": "...", "label": "...", "script_id": "..." }] }` | List of script-backed action buttons |
| `tls-manager` | *(none)* | TLS certificate management |
| `php-extensions` | *(none)* | PHP extension toggle panel |
| `php-settings` | *(none)* | PHP runtime settings editor |
| `php-xdebug` | *(none)* | Xdebug configuration panel |

---

### Scripts

Executable CLI scripts with parameter substitution. Referenced by `action-list` panels or shown directly.

```json
{
  "id": "create-database",
  "label": "Create Database",
  "type": "cli",
  "command": "{{TAURIO_ROOT}}\\bin\\...\\mysql.exe",
  "args": ["-u", "root", "-e", "CREATE DATABASE `{{db_name}}`"],
  "params": [
    { "name": "db_name", "label": "Database Name", "type": "text", "required": true }
  ],
  "confirm": true,
  "confirm_message": "Create database '{{db_name}}'?"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique script identifier |
| `label` | `string` | Display label |
| `type` | `string` | Always `"cli"` |
| `command` | `string` | Executable path |
| `args` | `string[]` | Arguments with `{{param}}` placeholders |
| `params` | `Param[]` | Parameter definitions |
| `confirm` | `boolean` | Show confirmation dialog before running |
| `confirm_message` | `string \| null` | Custom confirmation message (supports `{{param}}` placeholders) |

#### Parameter types

| Type | Description |
|------|-------------|
| `"text"` | Text input |
| `"password"` | Masked password input |
| `"number"` | Numeric input |

---

### Intelligence

Hints for automatic project detection and recommendations.

```json
{
  "recommend_for": ["php", "laravel", "symfony"],
  "companion_packages": ["mysql", "redis"],
  "companion_tools": ["composer", "phpstan"]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `recommend_for` | `string[]` | Project types this package is recommended for |
| `companion_packages` | `string[]` | Packages commonly used together |
| `companion_tools` | `string[]` | Tools commonly used together |

---

## Editing Guidelines

1. **Keep `id` stable** - changing an `id` breaks existing installations and override files
2. **Use `{{TAURIO_ROOT}}`** - never hardcode absolute paths; use `{{TAURIO_ROOT}}` which expands at runtime
3. **Reference sources by index** - install steps use `"source:0"`, `"source:1"`, etc.
4. **Test downloads** - verify all `url` values resolve to valid downloads
5. **Validate JSON** - the file must be valid JSON; use a linter before committing
