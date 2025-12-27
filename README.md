# aqua-installer-cache

Cache aqua install directory using [actions/cache](https://github.com/actions/cache).

This GitHub Action caches the [aqua](https://aquaproj.github.io/) installation directory to avoid GitHub API rate limits and speed up your CI/CD workflows by preventing repeated downloads of tools.

## Features

- 🚀 Automatically caches aqua packages and registries
- 🛡️ Prevents GitHub API rate limit issues
- 🔍 Auto-detects aqua configuration files
- 🌐 Supports multiple aqua config files (including `AQUA_GLOBAL_CONFIG`)
- 💻 Works on Linux, macOS, and Windows runners
- 🎯 Creates optimized cache keys based on OS, architecture, and config file hash

## Usage

```yaml
- uses: zumix/aqua-installer-cache@v0
```

### Basic Example

```yaml
name: CI

on: pull_request

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
      
      - uses: zumix/aqua-installer-cache@v0
      
      - name: Install aqua
        uses: aquaproj/aqua-installer@11dd79b4e498d471a9385aa9fb7f62bb5f52a73c # v4.0.4
        with:
          aqua_version: v2.56.0
      
      - name: Run tests
        run: |
          # Your tools installed by aqua are now available
          your-tool --version
```

### With Custom Working Directory

```yaml
- uses: zumix/aqua-installer-cache@v0
  with:
    working-directory: ./path/to/project
```

## Inputs

| Name | Description | Required | Default |
| ------ | ------------- | ---------- | --------- |
| `working-directory` | The working directory where aqua config file is located | No | `${{ github.workspace }}` |
| `config-file` | Custom path to the aqua config file | No | Auto-detected |

When `config-file` is not specified, the action automatically searches for aqua config files in the order defined in [aqua's configuration file path documentation](https://aquaproj.github.io/docs/reference/config/#configuration-file-path).

## Outputs

| Name | Description |
| ------ | ------------- |
| `aqua-root-dir` | The aqua root directory path that was cached |
| `config-file-hash` | The SHA256 hash of the aqua config file(s) |
| `cache-hit` | Whether the cache was restored (`true` or `false`) |

## Cache Key Strategy

The action creates cache keys with the following format:

```text
aqua-installer-cache-{OS}-{ARCH}-{CONFIG_HASH}
```

- **OS**: Runner OS (Linux, macOS, Windows)
- **ARCH**: Runner architecture (X64, ARM64, etc.)
- **CONFIG_HASH**: SHA256 hash of all aqua config files

This ensures that:

- Different OS/architecture combinations have separate caches
- Cache is invalidated when config files change
- Fallback to restore partial matches if exact hash doesn't match

## How It Works

1. **Detect aqua root directory**:
   - Uses `AQUA_ROOT_DIR` environment variable if set
   - Otherwise uses OS-specific default paths:
     - Windows: `%LOCALAPPDATA%\aquaproj-aqua`
     - Linux/macOS: `$XDG_DATA_HOME/aquaproj-aqua` or `~/.local/share/aquaproj-aqua`

2. **Find config files**:
   - Uses specified `config-file` input if provided
   - Otherwise auto-detects aqua config file in working directory
   - Also includes any files specified in `AQUA_GLOBAL_CONFIG` environment variable

3. **Calculate hash**:
   - Computes SHA256 hash of all detected config files
   - Used as part of the cache key

4. **Cache/Restore**:
   - Attempts to restore cache using the generated key
   - Falls back to restore from partial key match if exact match not found
   - Saves cache for future runs

## License

[MIT](LICENSE)

## Related Projects

- [aqua](https://github.com/aquaproj/aqua) - Declarative CLI Version Manager
- [aqua-installer](https://github.com/aquaproj/aqua-installer) - GitHub Action to install aqua
