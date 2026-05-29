# halflist-action

GitHub Action for testing MCP servers in CI. Runs conformance checks, security scans, and latency benchmarks using [mcp-halflist](https://github.com/abhishekhsingh/mcp-halflist).

Three lines to test your MCP server:

```yaml
- uses: abhishekhsingh/halflist-action@v1
  with:
    command: "python3 my_server.py"
```

## Quick Start

```yaml
# .github/workflows/mcp-test.yml
name: MCP Server Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: abhishekhsingh/halflist-action@v1
        with:
          command: "python3 my_server.py"
```

## Full Example with JUnit Reporting

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: abhishekhsingh/halflist-action@v1
        with:
          command: "python3 my_server.py"
          mode: audit
          junit: 'true'
          args: "--timeout 60 --verbose"

      - uses: dorny/test-reporter@v1
        if: always()
        with:
          name: MCP Server Report
          path: halflist-results.xml
          reporter: java-junit
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `command` | Server command for stdio transport | |
| `url` | Server URL for HTTP transport | |
| `mode` | `check`, `bench`, or `audit` | `check` |
| `args` | Additional CLI arguments | |
| `version` | mcp-halflist version to install | latest |
| `python-version` | Python version | `3.12` |
| `junit` | Generate JUnit XML report | `false` |
| `args-file` | Path to tool arguments JSON file | |
| `config` | Path to halflist.toml config file | |
| `fail-on-warn` | Fail the action if warnings are found | `false` |

Either `command` or `url` is required.

## Outputs

| Output | Description |
|--------|-------------|
| `result` | `pass` or `fail` |
| `score` | Conformance score (0-100, available when `junit` or `fail-on-warn` is enabled) |
| `report` | Path to JUnit XML file (when `junit` is enabled) |

## Examples

### stdio transport

```yaml
- uses: abhishekhsingh/halflist-action@v1
  with:
    command: "python3 my_server.py"
```

### HTTP transport

```yaml
- uses: abhishekhsingh/halflist-action@v1
  with:
    url: "http://localhost:8000/mcp"
    mode: check
```

### Full audit with benchmarks

```yaml
- uses: abhishekhsingh/halflist-action@v1
  with:
    command: "python3 my_server.py"
    mode: audit
```

### Custom tool arguments

```yaml
- uses: abhishekhsingh/halflist-action@v1
  with:
    command: "python3 my_server.py"
    args-file: "test-args.json"
```

### With halflist.toml config

```yaml
- uses: abhishekhsingh/halflist-action@v1
  with:
    config: "halflist.toml"
```

When using a config file with `server.command` or `server.url`, the `command`/`url` inputs can be omitted if the config provides them. Pass `command` or `url` anyway to keep the workflow self-documenting.

### Pin a specific version

```yaml
- uses: abhishekhsingh/halflist-action@v1
  with:
    command: "python3 my_server.py"
    version: "1.0.0"
```

### Fail on warnings

```yaml
- uses: abhishekhsingh/halflist-action@v1
  with:
    command: "python3 my_server.py"
    fail-on-warn: 'true'
```

### Use outputs in later steps

```yaml
- uses: abhishekhsingh/halflist-action@v1
  id: halflist
  with:
    command: "python3 my_server.py"
    junit: 'true'

- run: echo "Score: ${{ steps.halflist.outputs.score }}"
```

## License

MIT
