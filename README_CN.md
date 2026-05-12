     1|# CloakBrowser MCP Server
     2|
     3|[English](README.md) | [中文](README_CN.md)
     4|
     5|> 基于 [Model Context Protocol](https://modelcontextprotocol.io/) 的隐身浏览器自动化服务，由 [CloakBrowser](https://github.com/CloakHQ/CloakBrowser) 驱动。
     6|
     7|一个即插即用的 MCP 服务器，封装了 CloakBrowser 的隐身 Chromium —— 通过 **57 个源码级 C++ 指纹补丁**（不是 JS 注入）实现浏览器伪装。通过所有 30 项反 bot 检测（reCAPTCHA v3 评分 0.9、Cloudflare Turnstile: PASS、FingerprintJS: PASS）。
     8|
     9|## ✨ 特性
    10|
    11|- **22 个浏览器工具** —— 导航、点击、输入、截图、控制台、JS 执行、表单填充、拖拽等
    12|- **默认隐身** —— `navigator.webdriver = false`，真实 Chrome TLS 指纹，无 CDP 检测
    13|- **人机行为模拟** —— `humanize=True` 启用贝塞尔鼠标曲线、逐字符键盘节奏
    14|- **代理支持** —— HTTP / SOCKS5，支持 GeoIP 自动地理定位
    15|- **会话持久化** —— 保存/加载 cookies 和 localStorage
    16|- **兼容所有 MCP 客户端** —— Hermes Agent、Claude Desktop、Cursor 等
    17|
    18|## 🚀 快速开始
    19|
    20|### 安装
    21|
    22|```bash
    23|pip install mcp-cloakbrowser
    24|```
    25|
    26|### 运行
    27|
    28|```bash
    29|# 作为 stdio MCP 服务器启动
    30|mcp-cloakbrowser
    31|
    32|# 或者直接运行
    33|python -m cloakbrowser_mcp.server
    34|```
    35|
    36|### 配合 Hermes Agent 使用
    37|
    38|编辑 `~/.hermes/config.yaml`，添加以下配置：
    39|
    40|```yaml
    41|mcp_servers:
    42|  cloakbrowser:
    43|    command: "python"
    44|    args: ["-m", "cloakbrowser_mcp.server"]
    45|    timeout: 120
    46|```
    47|
    48|重启 Hermes Agent 后，工具会自动注册为 `mcp_cloakbrowser_browser_*` 前缀。
    49|
    50|### 配合 Claude Desktop 使用
    51|
    52|编辑 `claude_desktop_config.json`：
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
    64|## 📋 工具列表
    65|
    66|| 工具名 | 说明 | 参数 |
    67||--------|------|------|
    68|| `browser_launch` | 启动隐身 CloakBrowser 实例 | headless, humanize, proxy, fingerprint_seed, geoip, locale |
    69|| `browser_close` | 关闭浏览器并清理资源 | 无 |
    70|| `browser_navigate` | 导航到 URL，返回页面快照 | url, wait_until, humanize |
    71|| `browser_snapshot` | 获取页面可访问性树（含 ref ID） | full |
    72|| `browser_click` | 点击元素 | ref, button, click_count, humanize |
    73|| `browser_type` | 在输入框中输入文本 | ref, text, submit, humanize |
    74|| `browser_press` | 按下键盘按键 | key, humanize |
    75|| `browser_scroll` | 滚动页面 | direction (up/down) |
    76|| `browser_back` | 浏览器后退 | 无 |
    77|| `browser_forward` | 浏览器前进 | 无 |
    78|| `browser_console` | 获取控制台日志或执行 JS | expression, clear |
    79|| `browser_get_images` | 列出页面所有图片 | 无 |
    80|| `browser_screenshot` | 截取 PNG 截图 | save_path, full_page |
    81|| `browser_wait_for` | 等待元素或文本出现 | selector, text, timeout |
    82|| `browser_evaluate` | 执行 JavaScript 表达式 | expression |
    83|| `browser_get_content` | 获取页面/元素的文本或 HTML | selector, format |
    84|| `browser_extract_links` | 提取页面所有链接 | 无 |
    85|| `browser_fill_form` | 批量填充表单字段 | fields, submit_ref |
    86|| `browser_hover` | 鼠标悬停元素 | ref |
    87|| `browser_select_option` | 选择下拉框选项 | ref, values |
    88|| `browser_drag` | 拖拽元素到另一位置 | source_ref, target_ref |
    89|| `browser_save_storage_state` | 保存 cookies/localStorage | path |
    90|| `browser_load_storage_state` | 加载 cookies/localStorage | path |
    91|| `browser_info` | 获取当前页面 URL、标题、视口 | 无 |
    92|
    93|## 💡 使用示例
    94|
    95|### 基本导航与交互
    96|
    97|```python
    98|# 启动浏览器（隐身模式 + 人机模拟）
    99|await call_tool("browser_launch", {"headless": True, "humanize": True})
   100|
   101|# 导航到目标页面
   102|await call_tool("browser_navigate", {"url": "https://example.com"})
   103|
   104|# 获取页面快照，查看可交互元素
   105|snapshot = await call_tool("browser_snapshot", {})
   106|# 输出示例: [@e1] <a>链接文字, [@e2] <input>[type: text]...
   107|
   108|# 点击链接
   109|await call_tool("browser_click", {"ref": "@e1"})
   110|
   111|# 在搜索框输入文字并回车
   112|await call_tool("browser_type", {"ref": "@e2", "text": "你好世界", "submit": True})
   113|
   114|# 截图保存
   115|await call_tool("browser_screenshot", {"save_path": "result.png"})
   116|```
   117|
   118|### 填写登录表单
   119|
   120|```python
   121|await call_tool("browser_fill_form", {
   122|    "fields": [
   123|        {"ref": "@e1", "value": "用户名"},
   124|        {"ref": "@e2", "value": "密码123"},
   125|    ],
   126|    "submit_ref": "@e3",  # 点击登录按钮
   127|})
   128|```
   129|
   130|### 高级：自定义指纹 + 代理
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
   143|### 保存/恢复登录会话
   144|
   145|```python
   146|# 登录后保存会话
   147|await call_tool("browser_save_storage_state", {"path": "session.json"})
   148|
   149|# 下次使用时恢复会话
   150|await call_tool("browser_load_storage_state", {"path": "session.json"})
   151|```
   152|
   153|## 🏗️ 架构
   154|
   155|```
   156|MCP 客户端 (Hermes / Claude / Cursor 等)
   157|    │ stdio (JSON-RPC)
   158|    ▼
   159|mcp-cloakbrowser 服务器
   160|    │
   161|    ▼
   162|CloakBrowser (Playwright 兼容 API)
   163|    │
   164|    ▼
   165|隐身 Chromium (57 个 C++ 源码补丁)
   166|```
   167|
   168|服务器维护一个浏览器单例。所有工具操作当前活跃页面。首次调用工具时会自动启动浏览器（无需手动 launch）。
   169|
   170|## 🔧 与 Hermes Agent 内置浏览器工具的区别
   171|
   172|| 特性 | Hermes 内置 browser_* | CloakBrowser MCP |
   173||------|----------------------|------------------|
   174|| 反 bot 检测 | ❌ 原生 Playwright，容易被检测 | ✅ 57 个 C++ 补丁，通过全部检测 |
   175|| navigator.webdriver | true（可被检测） | false（完全隐藏） |
   176|| TLS 指纹 | Playwright 默认 | 真实 Chrome |
   177|| 人机模拟 | ❌ 无 | ✅ 贝塞尔曲线鼠标 + 逐字符输入 |
   178|| 代理 + GeoIP | ❌ 不支持 | ✅ HTTP/SOCKS5 + 地理定位 |
   179|| 指纹种子 | ❌ 固定 | ✅ 每次随机或自定义 |
   180|| 会话持久化 | ❌ 不支持 | ✅ 保存/加载 cookies |
   181|
   182|## 📁 项目结构
   183|
   184|```
   185|mcp-cloakbrowser/
   186|├── src/
   187|│   └── cloakbrowser_mcp/
   188|│       ├── __init__.py          # 版本信息
   189|│       ├── server.py            # MCP 服务器入口（工具注册）
   190|│       ├── tools.py             # 22 个工具的实现
   191|│       └── browser_manager.py   # 浏览器生命周期管理（单例）
   192|├── examples/
   193|│   └── hermes_config.yaml       # Hermes Agent 配置示例
   194|├── skills/
   195|│   └── mcp-cloakbrowser/
   196|│       └── SKILL.md             # 配套 Hermes Skill 文件
   197|├── pyproject.toml               # 打包配置
   198|├── README.md                    # 英文文档
   199|├── README_CN.md                 # 中文文档（本文件）
   200|└── LICENSE                      # MIT 许可证
   201|```
   202|
   203|## 🛠️ 开发
   204|
   205|```bash
   206|git clone https://github.com/MiwooMiwoo/mcp-cloakbrowser.git
   207|cd mcp-cloakbrowser
   208|pip install -e ".[dev]"
   209|```
   210|
   211|## 📄 许可证
   212|
   213|MIT
   214|