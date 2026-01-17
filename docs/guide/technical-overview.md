# 技术概览

本页聚焦 Gemini Voyager 的技术栈、构建方式与浏览器扩展架构，帮助开发者快速理解项目组成与关键实现。

## 项目定位

Gemini Voyager 是一款基于浏览器扩展（Manifest V3）的功能增强工具，通过内容脚本注入与扩展页面 UI，为 Gemini/AI Studio 提供时间轴、文件夹、提示词库、导出等能力。

## 技术栈

- **语言与框架**：TypeScript + React 19，面向现代浏览器的 UI 与交互开发。
- **构建体系**：Vite 7 + CRXJS 插件用于扩展打包；多浏览器配置文件覆盖 Chrome/Firefox/Safari 构建。
- **样式体系**：Tailwind CSS 与工具类合并方案（tailwind-merge）。
- **文档系统**：VitePress 用于站点化文档输出。
- **测试工具**：Vitest 作为单元测试与覆盖率运行器。

## 构建与开发脚本

项目使用 `bun` 或 `npm` 运行开发与构建任务：

- `dev:*`：以 `nodemon` 监听并重启构建，适配 Chrome/Firefox/Safari。
- `build:*`：针对不同浏览器输出扩展构建产物。
- `docs:*`：VitePress 文档站点的开发、构建与预览。
- `test:*`：Vitest 测试、UI 模式与覆盖率。

## 浏览器扩展架构

Manifest V3 架构要点如下：

- **后台服务**：使用 `service_worker` 作为后台入口，承担扩展的后台逻辑。
- **内容脚本**：在 Gemini 与 AI Studio 域名上注入脚本与样式，扩展页面体验。
- **扩展页面**：包含 Popup 与 Options 页面，分别用于快速操作与设置管理。
- **权限体系**：使用 `storage`、`identity`、`scripting` 权限，并声明 Gemini/AI Studio 相关域名的访问权限；同时提供可选的任意站点权限以支持自定义站点能力。

## 与文档输出的关系

技术文档采用 VitePress 维护在 `docs/` 目录，提供中英文双语入口；你可以直接使用 `docs:dev` 启动预览并进行扩展维护或二次开发文档编写。
