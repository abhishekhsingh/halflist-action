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

One of `command`, `url`, or a `config` file whose `[server]` section sets `command` or `url` is required.

## Outputs

| Output | Description |
|--------|-------------|
| `result` | `pass` or `fail` |
| `score` | Conformance score (0-100). Parsed from the report when `fail-on-warn` is `true`; otherwise `100` when the run passes and `0` when it fails |
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

When the config file sets `server.command` or `server.url`, the `command` and `url` inputs can be omitted. If you do pass `command` or `url`, they override the config.

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

## Notes

- **`args` is split on whitespace.** There is no shell quoting, so a value that contains spaces (for example `--header "Authorization: Bearer x"`) cannot be passed through `args`. Put headers and secrets in a `halflist.toml` (`[server]` section, `${ENV_VAR}` expansion is supported) and pass it with `config`.
- **halflist runs in its own virtual environment.** The action never changes the Python packages used by your server, so servers built on any `mcp` SDK version work, and the `version` input can pin any halflist release. Your server command runs with the job's own Python. The `halflist` command is not added to the job's `PATH`; to run it in other steps, install it there with `pip install mcp-halflist`.
- **Do not pass `--format` in `args`.** The action sets the format itself for `junit` and `fail-on-warn`, and a later `--format` in `args` overrides it.
- **HTTP servers without authentication:** halflist starts an OAuth PKCE flow for any HTTP server when no `Authorization` header is set. In CI, add `args: "--no-auth"` for servers that need no authentication, or supply the header through `config`.
- **`fail-on-warn` does not print the per-check report.** It parses the JSON report instead, so the log shows only the score. Run the action without `fail-on-warn` to see the full output.
- **`mode: bench` does not fail on its own.** halflist exits `0` for `bench` even when tool calls fail. Use `check` or `audit` to gate a build.
- **Exit codes** are passed through from halflist: `1` for failed checks, `2` for a connection error, `3` for a configuration error.
- **Use `junit: 'true'` once per job.** The artifact is always named `halflist-junit-results` and the file is always `halflist-results.xml`.

## License

MIT

## Author

[Abhishekh Singh](https://abhishekhsingh.github.io/)
