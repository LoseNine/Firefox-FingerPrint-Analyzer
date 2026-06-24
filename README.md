<div align="center">

<img src="https://raw.githubusercontent.com/LoseNine/Firefox-FingerPrint-Analyzer/main/icon.png" width="96" height="96" alt="Ruyi Trace" />

# 如意 Trace · Ruyi Trace

[**🇨🇳 简体中文**](./README.md) &nbsp;|&nbsp; [🇬🇧 English](./README.en.md)

A research-grade, instrumented Firefox build for fingerprint risk-control analysis and one-click AI-assisted JavaScript reverse-engineering.

![Platform](https://img.shields.io/badge/platform-Windows%20x64-blue)
![Firefox](https://img.shields.io/badge/firefox-DOMTrace-orange)
![License](https://img.shields.io/badge/license-MPL--2.0-green)
![Status](https://img.shields.io/badge/status-research-yellow)

</div>

---

### 这是什么

**如意 Trace** 是一个面向**网站指纹风控研究**与**辅助 JS 逆向学术分析**的桌面工具，由两部分组成：

1. **如意定制 trace 内核**（已随包发布）。
2. **Electron 客户端**（`RuyiTrace.exe`）——一键启动定制内核、采集 NDJSON 日志、管理日志文件。

> ⚠️ **仅限学术研究、安全教育与防御性分析使用。** 请勿用于绕过任何网站的服务条款或业务风控。

### 核心价值

> **ruyiPage 抓轮廓 → 如意 Trace 采集运行时日志 → 把日志交给 AI → 自动得到补环境代码与指纹风控分析报告。**

由于探针位于 **C++ 内核层而非 JS 层**，页面脚本无法通过原型检测、`toString` 嗅探、或已知 hook 特征发现监听存在 —— 因此采集的 trace 数据可作为研究指纹检测策略的高保真基线。

### 推荐工作流

1. **先用 [ruyiPage](https://github.com/LoseNine/ruyipage) 自动化框架，并使用其 Release 中配套的火狐指纹浏览器分析目标网页**，获取网站加载的全部 JS 文件并抓取完整的网络数据包，建立网站的整体轮廓。
2. **再用本工具（如意 Trace）启动定制 Firefox 进行实际访问**，生成 NDJSON 运行时调用日志。
3. **把日志直接交给 AI**，让 AI 结合第 1 步拿到的 JS 文件和数据包，对相关文件进行定位、补环境与指纹风控分析。

### 快速上手

1. **保持完整目录结构**：`RuyiTrace.exe` 必须与 `firefox/` 子目录处于同一文件夹。
2. **双击运行 `RuyiTrace.exe`**。
3. 在窗口中：
   - 填写**启动页面**（默认 `https://xjbedu.site/`）
   - 选择**日志目录**（默认 `<exe目录>/log`）
   - 确认顶部"定制内核"徽章为绿色（**定制内核已就绪**）
4. 点击 **开始采集** —— 定制 Firefox 自动启动并加载目标页面，右上角胶囊开始计时。
5. 在浏览器中正常浏览 / 触发指纹检测脚本。
6. 完成后点击 **停止采集**，日志保存为 `trace_<时间戳>_<PID>.ndjson`。
7. 点击 **打开目录** 取走 NDJSON 日志文件。

### 把日志交给 AI

NDJSON 每行一条事件，结构如下：

```json
{
  "t": "call",
  "api": "CanvasRenderingContext2D.fillText",
  "args": ["BrowserLeaks,com", 4, 17],
  "stack": [
    {"file": "https://example.test/fp.js", "line": 42, "col": 17}
  ]
}
```

**推荐 AI 分析提示词模板：**

```
你是一名前端安全研究员。我已经做了如下前置工作：
1. 先使用 ruyiPage 自动化框架（https://github.com/LoseNine/ruyipage）
   ——并使用其 release 中配套的火狐指纹浏览器——分析了目标网页，
   获取了网站加载的全部 JS 文件，并抓取了完整的网络数据包，
   建立了网站整体轮廓。
2. 然后使用如意 Trace 工具采集了一份完整的 DOM/JS API 运行时调用日志（NDJSON）。

现在我把这份运行时日志直接交给你，请你结合第 1 步拿到的 JS 文件与网络数据包：
1. 在 NDJSON 中识别所有指纹采集点（Canvas / WebGL / WebRTC / Audio /
   Navigator / Screen / Crypto），并定位到具体是哪个 JS 文件的哪个函数。
2. 还原每个指纹函数的完整调用链与入口脚本。
3. 输出一份补环境（环境替换）JavaScript 代码模板，使脚本运行结果与真实浏览器一致。
4. 列出该网站采用的指纹风控策略与对抗优先级。

[在此粘贴 NDJSON 内容]
```

把日志贴给 ChatGPT、Claude、Gemini 等任意大模型即可获得：
- 📊 **指纹风控分析报告**（哪些 API 被读取、调用顺序、关键参数）
- 🛠️ **补环境代码模板**（Canvas / WebGL / Audio / Navigator 自动还原）
- 🔍 **完整的 JS 调用图谱**（从入口函数到底层 API 的完整路径）

### 文件结构

```
RuyiTrace/                  (≈ 427 MB)
├── RuyiTrace.exe           Electron 客户端 (≈ 87 MB)
├── README.md               本说明
└── firefox/                如意定制 trace 内核 (≈ 340 MB)
    ├── firefox.exe
    ├── RUYI_DOMTRACE.txt   ← 内核标志文件，请勿删除
    └── ... DLL / 资源
```

### 高级使用

**手动指定环境变量启动 Firefox（不通过客户端）：**

```cmd
set MOZ_DOM_TRACE=1
set MOZ_DOM_TRACE_FILE=D:\logs\trace.ndjson
set MOZ_DISABLE_LAUNCHER_PROCESS=1
firefox\firefox.exe -no-remote -new-instance https://example.com
```

**支持的环境变量：**

| 变量 | 用途 |
|------|------|
| `MOZ_DOM_TRACE=1` | 开启 trace |
| `MOZ_DOM_TRACE_FILE=<path>` | 输出路径，PID 自动追加 |
| `MOZ_DOM_TRACE_LIMIT=<n>` | 单进程行数上限 |
| `MOZ_DOM_TRACE_PTYPE=<list>` | 启用 trace 的进程类型（逗号分隔） |
| `MOZ_DISABLE_LAUNCHER_PROCESS=1` | Windows 下避免 launcher 提前退出 |

### 常见问题

**Q：为什么不能用 Puppeteer / Playwright？**
A：它们注入 JS 钩子，可被页面脚本探测（`toString`、原型链、`navigator.webdriver` 等）。如意 Trace 在 C++ 层埋点，从 JS 视角完全不可见。

**Q：日志文件特别大怎么办？**
A：单次会话可能产生数百 MB。建议：**分段读取**——把 NDJSON 按行切成多块（例如每 5–10 万行一段），分批投喂给 AI 分析；同时可以 (1) 短时间内只访问目标页 (2) 使用 `MOZ_DOM_TRACE_LIMIT` 限制单进程行数 (3) 用 `MOZ_DOM_TRACE_PTYPE` 只保留关心的进程类型。

**Q："定制内核"徽章显示红色？**
A：表示选中的 `firefox.exe` 旁边没有 `RUYI_DOMTRACE.txt` 标志文件 —— 你大概率指向了系统自带的官方 Firefox。请确保 `RuyiTrace.exe` 与 `firefox/` 子目录处在同一文件夹。

**Q：可以采集 HTTPS / Service Worker / Web Worker 内的调用吗？**
A：可以。`MOZ_DOM_TRACE_PTYPE` 自动传递到所有子进程。

### 免责声明

本项目修改了 Mozilla Firefox 源代码（MPL-2.0），二进制中**不包含 Mozilla 商标**。本项目**不隶属于 Mozilla 基金会**，不代表 Mozilla 的官方立场。"Mozilla" 与 "Firefox" 是 Mozilla 基金会的注册商标。

使用本工具产生的一切后果由使用者自行承担。请遵守你所在司法辖区的法律法规及目标网站的服务条款。

---

<div align="center">
<sub>如意 · 心想事成 — Ruyi · "as you wish"</sub><br/>
<sub>Built for researchers, by researchers.</sub>
</div>
