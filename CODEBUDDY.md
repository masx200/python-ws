# CODEBUDDY.md

This file provides guidance to CodeBuddy Code when working with code in this
repository.

## Project Overview

This is a Python serverless proxy implementation supporting VLESS, Trojan, and
Shadowsocks protocols over WebSocket. Designed for PaaS platforms (e.g., Koyeb,
Railway, Render).

## Common Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the server locally
python3 app.py

# Run with custom environment variables
UUID=your-uuid DOMAIN=your-domain python3 app.py

# Build Docker image
docker build -t python-ws .

# Run Docker container
docker run -p 3000:3000 -e UUID=xxx -e DOMAIN=xxx python-ws
```

## Architecture

### Core Components

- **app.py** - Main application (~695 lines)
  - `ProxyHandler` class (lines 152-459): Handles protocol parsing and proxying
    - `handle_vless()`: VLESS protocol (lines 157-254)
    - `handle_trojan()`: Trojan protocol (lines 256-367)
    - `handle_shadowsocks()`: Shadowsocks protocol (lines 369-459)
  - `websocket_handler()` (lines 461-505): WebSocket connection handler
  - `http_handler()` (lines 507-536): HTTP endpoints for subscription
  - `run_nezha()` (lines 570-622): Nezha monitoring agent integration
  - `main()` (lines 646-693): Server initialization

- **index.html** - Frontend landing page served at `/`

### Protocol Flow

1. Client connects via WebSocket to `/{WSPATH}`
2. First message determines protocol (VLESS=0x00, Trojan, Shadowsocks)
3. `ProxyHandler` validates credentials and parses target address
4. Bidirectional data forwarding between WebSocket and TCP connection

### Key Environment Variables

| Variable     | Required | Default        | Description                     |
| ------------ | -------- | -------------- | ------------------------------- |
| UUID         | No       | auto-generated | Authentication UUID             |
| PORT         | No       | 3000           | Server port                     |
| DOMAIN       | Yes      | -              | Domain for subscription         |
| NEZHA_SERVER | No       | -              | Nezha v0/v1 server              |
| NEZHA_KEY    | No       | -              | Nezha authentication key        |
| SUB_PATH     | No       | sub            | Subscription path               |
| WSPATH       | No       | UUID[:8]       | WebSocket path                  |
| IP_PRIORITY  | No       | ipv4           | IP priority: ipv6, ipv4, random |

### Dependencies

- `aiohttp`: Async HTTP/WebSocket server
- `requests`: HTTP client (for IP detection)

## Development Notes

- The server uses `aiohttp.web` for async request handling
- Protocol handlers use `asyncio.open_connection` for TCP proxying
- DNS resolution via Google DNS (`dns.google/resolve`)
- ISP detection via `api.ip.sb/geoip` or `ip-api.com`
- Supports both direct IP and domain-based deployment
- Auto-cleanup of temporary files after 180 seconds
