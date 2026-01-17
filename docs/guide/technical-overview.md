# 技术概览

本页聚焦 Gemini Voyager 的技术栈、构建方式与浏览器扩展架构，并进一步披露各功能的实现细节，帮助开发者快速理解项目组成与关键实现。

## 技术栈

- **语言与框架**：TypeScript + React 19，用于扩展 UI 与交互实现。
- **构建体系**：Vite 7 + CRXJS 插件完成多浏览器扩展打包。
- **样式体系**：Tailwind CSS 与工具类合并（tailwind-merge）。
- **文档系统**：VitePress 作为文档站点生成器。
- **测试工具**：Vitest 用于单测与覆盖率。

## 构建与开发脚本

项目使用 `bun` 或 `npm` 运行开发与构建任务：

- `dev:*`：`nodemon` 监听与热重建，适配 Chrome/Firefox/Safari。
- `build:*`：针对不同浏览器输出扩展构建产物。
- `docs:*`：VitePress 文档站点的开发、构建与预览。
- `test:*`：Vitest 运行、UI 模式与覆盖率。

## 扩展入口与初始化策略

内容脚本入口位于 `src/pages/content/index.tsx`，该模块负责：

- 识别 Gemini、AI Studio 与自定义网站域名。
- 针对前台与后台标签页做错峰初始化，避免多标签恢复时触发频繁请求。
- 以「重型功能（时间轴/文件夹）」与「轻量功能」两段延迟顺序启动。

扩展页面（Popup/Options）与后台服务 worker 则分别处理设置、权限与跨域请求（如导出图片时的背景抓取）。

## 功能实现技术细节（逐项披露）

下面按功能拆解实现方式与关键模块，便于定位源码与二次开发。

### 1) 时间轴（Timeline）

- 入口：`startTimeline()` 在内容脚本初始化时启动。
- 核心：`TimelineManager` 负责注入时间轴 DOM（bar、track、tooltip）、监听滚动/鼠标/快捷键，并将节点定位到对话转折点。
- 数据：星标与折叠状态写入本地存储（localStorage），位置与设置写入同步存储（chrome.storage.sync）。
- 交互：支持右键节点层级、折叠/展开、拖拽定位条，以及快捷键导航。

### 2) 文件夹（Folder）

- Gemini 侧：`FolderManager` 注入侧边栏 UI、监听拖拽、维护树形层级与对话引用。
- AI Studio 侧：`AIStudioFolderManager` 针对 prompts/library 页面渲染简化版文件夹 UI。
- 存储适配：通过 `FolderStorageAdapter` 在 localStorage 与 browser.storage.local 之间做适配与迁移，保证 Safari 兼容。
- 导入/导出：支持 JSON 格式导入、合并与覆盖策略。

### 3) 灵感库（Prompt Library）

- 入口：`startPromptManager()` 内容脚本启动。
- 能力：提示词列表、标签筛选、搜索、复制、导入/导出、备份打包。
- 存储：使用 `StorageService` 读写，并在迁移时将 localStorage 数据同步到 chrome.storage.local。
- 自定义网站：当域名命中自定义站点时，仅启用 Prompt Manager（避免注入其他 Gemini 功能）。

### 4) 对话导出（Chat Export）

- 入口：`startExportButton()` 在对话页面注入导出按钮与对话框。
- 抽取逻辑：通过多套选择器收集用户/助手消息，去重并根据 offset 生成稳定 turnId；结合时间轴星标数据标记重点回合。
- 输出格式：`ConversationExportService` 使用策略模式输出 JSON / Markdown / PDF；Markdown 处理图片资源并可打包 ZIP。

### 5) Deep Research 导出

- 入口：`startDeepResearchExport()` 检测 `deep-research-immersive-panel` 并注入菜单按钮。
- 抽取：`extractThinkingPanels()` 遍历 `thinking-panel`，解析 thought 与 browse-chip（引用链接）结构。
- 输出：格式化为 Markdown 并触发文件下载。

### 6) Mermaid 图表渲染

- 入口：`startMermaid()` 在内容脚本中启用。
- 识别：检测代码块关键字，过滤未完成或过短的 Mermaid 内容。
- 交互：提供「源码/渲染」切换按钮与全屏查看（拖拽/缩放）。

### 7) 引用回复（Quote Reply）

- 入口：`startQuoteReply()` 注入选择按钮与样式。
- 逻辑：监听选区变化，确保选区位于正文区域且不在输入框内；点击按钮将文本格式化为 Markdown blockquote 并插入输入框。

### 8) 公式复制（Formula Copy）

- 入口：`startFormulaCopy()` 初始化单例服务。
- 逻辑：事件委托监听点击的 KaTeX/MathJax 元素，读取 `data-math` 并按配置格式（LaTeX/UnicodeMath/无美元符）复制。
- 存储：复制格式使用 chrome.storage.sync 持久化。

### 9) NanoBanana 水印去除

- 入口：`startWatermarkRemover()` 在 Gemini 对话页启用。
- 引擎：`WatermarkEngine` 基于 reverse alpha blending 重建原图像素；根据图片尺寸识别 48/96 水印版本。
- 集成：MutationObserver 监听图片，处理后替换下载逻辑，并通过自定义事件与 fetch 拦截器沟通状态。

### 10) 输入框折叠

- 入口：`startInputCollapse()` 注入样式并定位输入容器。
- 逻辑：检测输入框是否为空，空状态时添加折叠 class，仅显示占位符；点击或输入时恢复展开。

### 11) 对话宽度调整

- 入口：`startChatWidthAdjuster()` 读取 `geminiChatWidth`，在 CSS 中覆盖主容器与气泡最大宽度。
- 兼容：支持旧的像素设置并自动迁移为百分比。

---

> 以上模块均在 `src/pages/content` 或 `src/features` 中实现。若需要二次开发，可从内容脚本入口开始逐模块调试与扩展。
