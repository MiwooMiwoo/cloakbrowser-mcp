# CloakBrowser MCP Server

[English](README.md) | [中文](README_CN.md)

> Stealth browser automation via [Model Context Protocol](https://modelcontextprotocol.io/), powered by [CloakBrowser](https://github.com/CloakHQ/CloakBrowser).

A drop-in MCP server that wraps CloakBrowser's stealth Chromium with **57 source-level C++ fingerprint patches** — not JS injection. Passes all 30/30 bot detection tests (reCAPTCHA v3 score: 0.9, Cloudflare Turnstile: PASS, FingerprintJS: PASS).

## Features

- **22 browser tools** — navigate, click, type, screenshot, console, evaluate JS, form filling, drag & drop, and more
- **Stealth by default** — `navigator.webdriver = false`, real Chrome TLS fingerprint, no CDP detection
- **Human-like behavior** — `humanize=True` enables Bézier mouse curves, per-character keyboard timing
- **Proxy support** — HTTP & SOCKS5 with GeoIP auto-detection
- **Session persistence** — save/load cookies and localStorage
- **Compatible with any MCP client** — Hermes Agent, Claude Desktop, Cursor, etc.

## Quick Start

### Install

```bash
pip install cloakbrowser-mcp
```

### Run

```bash
# As a stdio MCP server
cloakbrowser-mcp

# Or directly
python -m cloakbrowser_mcp.server
```

### Use with Hermes Agent

Add to `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  cloakbrowser:
    command: "python"
    args: ["-m", "cloakbrowser_mcp.server"]
    timeout: 120
```

Restart Hermes Agent. Tools will be registered as `mcp_cloakbrowser_browser_*`.

### Use with Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "cloakbrowser": {
      "command": "cloakbrowser-mcp"
    }
  }
}
```

## Available Tools

| Tool | Description |
|------|-------------|
| `browser_launch` | Launch a stealth CloakBrowser instance |
| `browser_close` | Close the browser and clean up |
| `browser_navigate` | Navigate to a URL, return compact snapshot |
| `browser_snapshot` | Get accessibility tree with ref IDs |
| `browser_click` | Click element by ref (e.g. `@e5`) |
| `browser_type` | Type text into input field by ref |
| `browser_press` | Press keyboard key (Enter, Tab, Escape...) |
| `browser_scroll` | Scroll page up/down |
| `browser_back` | Navigate back in history |
| `browser_forward` | Navigate forward in history |
| `browser_console` | Get console logs or evaluate JS |
| `browser_get_images` | List all images with URLs and alt text |
| `browser_screenshot` | Take PNG screenshot |
| `browser_wait_for` | Wait for element or text to appear |
| `browser_evaluate` | Evaluate JavaScript expression |
| `browser_get_content` | Get text/HTML of page or element |
| `browser_extract_links` | Extract all links as JSON |
| `browser_fill_form` | Fill multiple form fields at once |
| `browser_hover` | Hover over element by ref |
| `browser_select_option` | Select options in `<select>` elements |
| `browser_drag` | Drag element to another element |
| `browser_save_storage_state` | Save cookies/localStorage to file |
| `browser_load_storage_state` | Load cookies/localStorage from file |
| `browser_info` | Get current page URL, title, viewport |

## Tool Usage Examples

### Navigate and Interact

```python
# Launch browser
await call_tool("browser_launch", {"headless": True, "humanize": True})

# Navigate to a page
await call_tool("browser_navigate", {"url": "https://example.com"})

# Get snapshot to see interactive elements
snapshot = await call_tool("browser_snapshot", {})
# Shows: [@e1] <a>Link text, [@e2] <input>[type: text]...

# Click a link
await call_tool("browser_click", {"ref": "@e1"})

# Type into search box
await call_tool("browser_type", {"ref": "@e2", "text": "hello world", "submit": True})

# Take screenshot
await call_tool("browser_screenshot", {})
```

### Fill a Login Form

```python
await call_tool("browser_fill_form", {
    "fields": [
        {"ref": "@e1", "value": "username"},
        {"ref": "@e2", "value": "password123"},
    ],
    "submit_ref": "@e3",
})
```

### Advanced: Custom Fingerprint & Proxy

```python
await call_tool("browser_launch", {
    "headless": True,
    "humanize": True,
    "proxy": "socks5://user:pass@proxy:1080",
    "fingerprint_seed": "my-unique-seed-123",
    "geoip": True,
    "locale": "zh-CN",
})
```

### Save/Restore Session

```python
# Save session after login
await call_tool("browser_save_storage_state", {"path": "session.json"})

# Later: restore session
await call_tool("browser_load_storage_state", {"path": "session.json"})
```

## Architecture

```
MCP Client (Hermes/Claude/etc.)
    │ stdio (JSON-RPC)
    ▼
cloakbrowser-mcp server
    │
    ▼
CloakBrowser (Playwright-compatible API)
    │
    ▼
Stealth Chromium (57 C++ patches)
```

The server maintains a single browser instance (singleton pattern). All tools operate on the current page. The browser is auto-launched on first tool call if not explicitly launched.

## Development

```bash
git clone https://github.com/MiwooMiwoo/cloakbrowser-mcp.git
cd cloakbrowser-mcp
pip install -e ".[dev]"
```

## License

MIT
