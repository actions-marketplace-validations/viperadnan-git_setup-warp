# Setup WARP by Cloudflare

A GitHub Action to set up [Cloudflare WARP](https://developers.cloudflare.com/warp-client/) on GitHub Actions runners. Routes all runner traffic (including Docker containers) through Cloudflare's network with automatic retries and connectivity verification.

## Features

- **Two modes** &mdash; WARP client (recommended) or WireGuard
- **Docker support** &mdash; automatically configures Docker containers to route through WARP
- **Automatic retries** &mdash; configurable retry attempts with exponential backoff
- **Connectivity verification** &mdash; verifies WARP tunnel is active before proceeding
- **DNS configuration** &mdash; sets Docker daemon DNS to Cloudflare (1.1.1.1) and Google (8.8.8.8)
- **Dual-stack** &mdash; supports IPv4-only, IPv6-only, or dual-stack

## Usage

### Basic

```yaml
steps:
  - uses: viperadnan-git/setup-warp@v1
```

### With options

```yaml
steps:
  - name: Set up WARP
    uses: viperadnan-git/setup-warp@v1
    with:
      mode: client        # or 'wireguard'
      stack: dual         # or 'ipv4', 'ipv6'
      retries: 3
      configure_docker: true

  - name: Run something through WARP
    run: curl https://cloudflare.com/cdn-cgi/trace
```

### Docker containers

When `configure_docker` is enabled (default), Docker containers automatically route through WARP:

```yaml
steps:
  - uses: viperadnan-git/setup-warp@v1

  - run: |
      docker run --rm alpine sh -c "wget -qO- https://cloudflare.com/cdn-cgi/trace"
      # Output will show warp=on
```

This works by:
1. Configuring Docker daemon DNS to use Cloudflare/Google DNS
2. Installing a transparent wrapper that injects `--network host` into `docker run` commands

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `mode` | WARP mode: `client` or `wireguard` | `client` |
| `stack` | IP stack: `ipv4`, `ipv6`, or `dual` | `dual` |
| `retries` | Number of setup retry attempts | `3` |
| `configure_docker` | Configure Docker to route through WARP | `true` |

## Modes

### Client mode (recommended)

Uses the official Cloudflare WARP client. Registers fresh credentials on each run, making it more reliable than shared WireGuard keys.

```yaml
- uses: viperadnan-git/setup-warp@v1
  with:
    mode: client
```

### WireGuard mode

Uses a WireGuard tunnel with pre-configured Cloudflare WARP credentials. Lighter weight but uses shared keys.

```yaml
- uses: viperadnan-git/setup-warp@v1
  with:
    mode: wireguard
```

## How it works

1. **Installs WARP** &mdash; downloads and installs the Cloudflare WARP client or WireGuard tools
2. **Connects** &mdash; establishes the WARP tunnel (client mode registers fresh credentials each run)
3. **Verifies** &mdash; confirms connectivity and that `warp=on` via Cloudflare's trace endpoint
4. **Configures Docker** &mdash; updates Docker daemon DNS and installs a wrapper for container networking
5. **Retries** &mdash; if any step fails, tears down and retries with exponential backoff
6. **Debug dump** &mdash; on final failure, prints network interfaces, routes, DNS config, and WARP status

## Compatibility

- **Runners**: Ubuntu (20.04, 22.04, 24.04)
- **Architecture**: amd64 (auto-detected via `dpkg --print-architecture`)

## Credits

Inspired by [fscarmen/warp-on-actions](https://github.com/fscarmen/warp-on-actions). Rewritten with Docker networking fixes, robust error handling, retry logic, and connectivity verification.

## License

[MIT](LICENSE)
