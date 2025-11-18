# Chloroplast Docs Action

A reusable composite GitHub Action for building documentation with the [Chloroplast .NET tool](https://www.nuget.org/packages/Chloroplast.Tool).

## Features

- 🔧 **Easy to use**: Only requires config file path and output path as mandatory inputs
- 🛠️ **Flexible setup**: Supports both dotnet tool manifest and direct NuGet installation
- 🎯 **Smart defaults**: Uses latest .NET LTS SDK and latest Chloroplast tool version by default  
- 📝 **Comprehensive logging**: Detailed output for debugging and monitoring
- ⚙️ **Configurable**: Optional parameters for working directory, tool version, and extra CLI arguments

## Quick Start

### Basic Usage

```yaml
- uses: actions/checkout@v4
- name: Build Docs with Chloroplast
  uses: joelmartinez/chloroplast-docs-action@main
  with:
    rootpath: docs
    outpath: tmp/docsout
```

### With dotnet-tools.json Manifest

If your repository has a `dotnet-tools.json` file that includes the Chloroplast tool, the action will automatically restore and use it:

```yaml
- uses: actions/checkout@v4
- name: Build Docs with Chloroplast
  uses: joelmartinez/chloroplast-docs-action@main
  with:
    rootpath: docs
    outpath: docs-output
    manifest_directory: ./tools  # Directory containing dotnet-tools.json
```

### Advanced Configuration

```yaml
- uses: actions/checkout@v4
- name: Build Docs with Chloroplast
  uses: joelmartinez/chloroplast-docs-action@main
  with:
    rootpath: docs
    outpath: build/documentation
    sdk_version: '8.0.x'
    version: '0.7.139'  # Specific Chloroplast tool version
    working_directory: ./src
    extra_args: '--verbose --theme modern'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `rootpath` | Path to the Chloroplast site where the SiteConfig.yml config file exists. | ✅ Yes | - |
| `outpath` | Path where to put the generated documentation output | ✅ Yes | - |
| `sdk_version` | .NET SDK version to use | No | `8.0.x` |
| `version` | Chloroplast tool version to install (if not using manifest) | No | `latest` |
| `working_directory` | Working directory for running commands | No | `.` |
| `manifest_directory` | Directory containing dotnet-tools.json manifest | No | `.` |
| `extra_args` | Additional CLI arguments for chloroplast build command | No | `` |

## How It Works

1. **Setup .NET SDK**: Installs the specified .NET SDK version (defaults to latest LTS 8.0.x)
2. **Tool Detection**: Checks for existing `dotnet-tools.json` manifest in the specified directory
3. **Tool Installation**: 
   - If manifest exists: Restores tools using `dotnet tool restore`
   - If no manifest: Installs Chloroplast tool globally from NuGet
4. **Documentation Generation**: Runs `chloroplast build --config <config> --output <outpath>` with any additional arguments
5. **Verification**: Confirms output was generated successfully

## Examples

### Example 1: Simple Documentation Build

Create a workflow that builds documentation on every push to main:

```yaml
name: Build Documentation

on:
  push:
    branches: [ main ]

jobs:
  build-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Documentation
        uses: joelmartinez/chloroplast-docs-action@main
        with:
          rootpath: docs
          outpath: public/docs
      
      - name: Upload Documentation
        uses: actions/upload-artifact@v4
        with:
          name: documentation
          path: public/docs
```

### Example 2: Using Tool Manifest

If you have a `dotnet-tools.json` file like this:

```json
{
  "version": 1,
  "isRoot": true,
  "tools": {
    "chloroplast.tool": {
      "version": "1.2.3",
      "commands": ["chloroplast"]
    }
  }
}
```

Your workflow can use it:

```yaml
name: Build Docs with Manifest

on: [push, pull_request]

jobs:
  docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docs
        uses: joelmartinez/chloroplast-docs-action@main
        with:
          rootpath: docs
          outpath: dist/documentation
          # The action will automatically detect and use dotnet-tools.json
```

### Example 3: Multi-Platform Build

```yaml
name: Cross-Platform Documentation

on:
  workflow_dispatch:

jobs:
  build-docs:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Documentation
        uses: joelmartinez/chloroplast-docs-action@main
        with:
          rootpath: docs
          outpath: docs-${{ matrix.os }}
          sdk_version: '8.0.x'
          extra_args: '--verbose'
      
      - name: Upload OS-specific docs
        uses: actions/upload-artifact@v4
        with:
          name: docs-${{ matrix.os }}
          path: docs-${{ matrix.os }}
```

## Troubleshooting

### Common Issues

1. **Config file not found**: Ensure the `config` path is relative to either the `working_directory` or the repository root. The action will automatically check both locations.
2. **Tool installation fails**: Check that the specified `version` exists on NuGet
3. **Permission errors**: Make sure the `outpath` directory is writable

### Debug Output

The action provides detailed logging. Check the workflow logs for:
- Environment information (including working directory and workspace paths)
- Config file path resolution details
- Tool installation status  
- Command execution details
- Output verification results

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the repository for details.
