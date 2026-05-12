     1|# CloakBrowser MCP Server
     2|
     3|[English](README.md) | [中文](README_CN.md)
     4|
     5|> Stealth browser automation via [Model Context Protocol](https://modelcontextprotocol.io/), powered by [CloakBrowser](https://github.com/CloakHQ/CloakBrowser).
     6|
     7|A drop-in MCP server that wraps CloakBrowser's stealth Chromium with **57 source-level C++ fingerprint patches** — not JS injection. Passes all 30/30 bot detection tests (reCAPTCHA v3 score: 0.9, Cloudflare Turnstile: PASS, FingerprintJS: PASS).
     8|
     9|## Features
    10|
    11|- **22 browser tools** — navigate, click, type, screenshot, console, evaluate JS, form filling, drag & drop, and more
    12|- **Stealth by default** — `navigator.webdriver = false`, real Chrome TLS fingerprint, no CDP detection
    13|- **Human-like behavior** — `humanize=True` enables Bézier mouse curves, per-character keyboard timing
    14|- **Proxy support** — HTTP & SOCKS5 with GeoIP auto-detection
    15|- **Session persistence** — save/load cookies and localStorage
    16|- **Compatible with any MCP client** — Hermes Agent, Claude Desktop, Cursor, etc.
    17|
    18|## Quick Start
    19|
    20|### Install
    21|
    22|```bash
    23|pip install mcp-cloakbrowser
    24|```
    25|
    26|### Run
    27|
    28|```bash
    29|# As a stdio MCP server
    30|mcp-cloakbrowser
    31|
    32|# Or directly
    33|python -m cloakbrowser_mcp.server
    34|```
    35|
    36|### Use with Hermes Agent
    37|
    38|Add to `~/.hermes/config.yaml`:
    39|
    40|```yaml
    41|mcp_servers:
    42|  cloakbrowser:
    43|    command: "python"
    44|    args: ["-m", "cloakbrowser_mcp.server"]
    45|    timeout: 120
    46|```
    47|
    48|Restart Hermes Agent. Tools will be registered as `mcp_cloakbrowser_browser_*`.
    49|
    50|### Use with Claude Desktop
    51|
    52|Add to `claude_desktop_config.json`:
    53|
    54|```json
    55|{
    56|  "mcpServers": {
    57|    "cloakbrowser": {
    58|      "command": "mcp-cloakbrowser"
    59|    }
    60|  }
    61|}
    62|```
    63|
    64|## Available Tools
    65|
    66|| Tool | Description |
    67||------|-------------|
    68|| `browser_launch` | Launch a stealth CloakBrowser instance |
    69|| `browser_close` | Close the browser and clean up |
    70|| `browser_navigate` | Navigate to a URL, return compact snapshot |
    71|| `browser_snapshot` | Get accessibility tree with ref IDs |
    72|| `browser_click` | Click element by ref (e.g. `@e5`) |
    73|| `browser_type` | Type text into input field by ref |
    74|| `browser_press` | Press keyboard key (Enter, Tab, Escape...) |
    75|| `browser_scroll` | Scroll page up/down |
    76|| `browser_back` | Navigate back in history |
    77|| `browser_forward` | Navigate forward in history |
    78|| `browser_console` | Get console logs or evaluate JS |
    79|| `browser_get_images` | List all images with URLs and alt text |
    80|| `browser_screenshot` | Take PNG screenshot |
    81|| `browser_wait_for` | Wait for element or text to appear |
    82|| `browser_evaluate` | Evaluate JavaScript expression |
    83|| `browser_get_content` | Get text/HTML of page or element |
    84|| `browser_extract_links` | Extract all links as JSON |
    85|| `browser_fill_form` | Fill multiple form fields at once |
    86|| `browser_hover` | Hover over element by ref |
    87|| `browser_select_option` | Select options in `<select>` elements |
    88|| `browser_drag` | Drag element to another element |
    89|| `browser_save_storage_state` | Save cookies/localStorage to file |
    90|| `browser_load_storage_state` | Load cookies/localStorage from file |
    91|| `browser_info` | Get current page URL, title, viewport |
    92|
    93|## Tool Usage Examples
    94|
    95|### Navigate and Interact
    96|
    97|```python
    98|# Launch browser
    99|await call_tool("browser_launch", {"headless": True, "humanize": True})
   100|
   101|# Navigate to a page
   102|await call_tool("browser_navigate", {"url": "https://example.com"})
   103|
   104|# Get snapshot to see interactive elements
   105|snapshot = await call_tool("browser_snapshot", {})
   106|# Shows: [@e1] <a>Link text, [@e2] <input>[type: text]...
   107|
   108|# Click a link
   109|await call_tool("browser_click", {"ref": "@e1"})
   110|
   111|# Type into search box
   112|await call_tool("browser_type", {"ref": "@e2", "text": "hello world", "submit": True})
   113|
   114|# Take screenshot
   115|await call_tool("browser_screenshot", {})
   116|```
   117|
   118|### Fill a Login Form
   119|
   120|```python
   121|await call_tool("browser_fill_form", {
   122|    "fields": [
   123|        {"ref": "@e1", "value": "username"},
   124|        {"ref": "@e2", "value": "password123"},
   125|    ],
   126|    "submit_ref": "@e3",
   127|})
   128|```
   129|
   130|### Advanced: Custom Fingerprint & Proxy
   131|
   132|```python
   133|await call_tool("browser_launch", {
   134|    "headless": True,
   135|    "humanize": True,
   136|    "proxy": "socks5://user:pass@proxy:1080",
   137|    "fingerprint_seed": "my-unique-seed-123",
   138|    "geoip": True,
   139|    "locale": "zh-CN",
   140|})
   141|```
   142|
   143|### Save/Restore Session
   144|
   145|```python
   146|# Save session after login
   147|await call_tool("browser_save_storage_state", {"path": "session.json"})
   148|
   149|# Later: restore session
   150|await call_tool("browser_load_storage_state", {"path": "session.json"})
   151|```
   152|
   153|## Architecture
   154|
   155|```
   156|MCP Client (Hermes/Claude/etc.)
   157|    │ stdio (JSON-RPC)
   158|    ▼
   159|mcp-cloakbrowser server
   160|    │
   161|    ▼
   162|CloakBrowser (Playwright-compatible API)
   163|    │
   164|    ▼
   165|Stealth Chromium (57 C++ patches)
   166|```
   167|
   168|The server maintains a single browser instance (singleton pattern). All tools operate on the current page. The browser is auto-launched on first tool call if not explicitly launched.
   169|
   170|## Development
   171|
   172|```bash
   173|git clone https://github.com/MiwooMiwoo/mcp-cloakbrowser.git
   174|cd mcp-cloakbrowser
   175|pip install -e ".[dev]"
   176|```
   177|
   178|## License
   179|
   180|MIT
   181|