---
tags:
  - ai-skills
  - ui-ux
  - frontend
  - claude-code
  - cursor
---

# UI UX Pro Max Skill 安装使用指南

> **一句话总结**：`ui-ux-pro-max-skill` 是一个专为 AI 编程助手设计的 UI/UX 设计智能知识库，通过为 AI 注入专业的设计规范、调色板系统与布局模式，彻底消除生成界面时的“AI 味 / 塑料感”。

---

## 1. Key Takeaways

* **解决的核心痛点**：防止 AI 大模型直接生成样式平淡、边距混乱、色彩不协调的前端界面（AI Slop）。
* **内置设计资源**：包含 160+ 行业设计推理规则、170+ 调色板方案、数十种现代 UI 风格（Bento Grid、Glassmorphism、Minimalism 等）与无障碍 (A11y) 规范。
* **推荐安装机制**：优先通过 `npm install -g uipro-cli` 并结合 `uipro init` 进行安装。
* **配置范围建议**：个人开发推荐 `uipro init --ai claude --global`（全局生效）；团队仓库推荐在项目根目录运行 `uipro init --ai all` 并提交配置文件至 Git。

---

## 2. 核心作用与适用场景

### 2.1 核心作用
1. **设计决策增强**：AI 在接收到“生成某界面”指令时，会先通过内部逻辑匹配该行业的 UI 最佳实践（如配色、字体排版、控件间距）。
2. **多技术栈适配**：无缝支持 React、Next.js、Vue、Svelte、Tailwind CSS、shadcn/ui、SwiftUI、Flutter 等主流框架。
3. **消除代码模版感**：让 AI 输出的界面不仅功能可用，更具备现代高质感的视觉体验。

### 2.2 适用场景
* **全栈 / 后端开发者**：在没有独立 UI 设计师配合时，独立产出符合高审美标准的前端应用。
* **MVP / 演示原型快速构建**：在 Hackathon 或创业初期快速搭建商业级质感的展示界面。
* **专业数据控制台**：Web3 交易终端、车联网监控大屏、音频工作站、SaaS 仪表盘等复杂交互界面。

---

## 3. 安装方式总览与原理

### 3.1 项目级安装 vs 全局级安装

| 维度 | 项目级安装 (`uipro init --ai <tool>`) | 全局级安装 (`uipro init --ai <tool> --global`) |
| :--- | :--- | :--- |
| **生效范围** | 仅对当前项目目录生效 | 对当前 Mac 上的所有项目全局生效 |
| **配置文件存储** | 当前项目根目录（如 `./.claude/`, `./.cursor/`） | 用户主目录（如 `~/.claude/`, `~/.cursor/`） |
| **Git 版本控制** | 可随项目代码提交到 Git 仓库，团队共享 | 仅个人本地环境生效，不随项目代码分发 |
| **推荐场景** | **团队协作项目、开源代码库** | **个人独立开发、快捷创建 Demo** |

### 3.2 底层运行原理
1. **规则注入**：`uipro-cli` 脚手架在执行 `init` 时，会将包含 `SKILL.md` 及设计规则索引的文件夹复制到目标 AI 助手的特定配置路径中。
2. **自动识别**：以 Claude Code 为例，每次启动时都会自动解析读取 `.claude/skills/` 目录下的 Skill 定义文件；Cursor 则解析 `.cursor/rules/`。
3. **上下文驱动**：当在 Prompt 中提及“UI/UX”、“设计”或相关控件需求时，AI 会唤醒该 Skill 知识库中的模板与规范。

---

## 4. 不同 Agent 配置命令与差异

| AI 编程工具 | 项目级安装命令 | 全局级全局安装命令 | 自动生成的配置目录 |
| :--- | :--- | :--- | :--- |
| **Claude Code** | `uipro init --ai claude` | `uipro init --ai claude --global` | `./.claude/skills/` |
| **Cursor** | `uipro init --ai cursor` | `uipro init --ai cursor --global` | `./.cursor/rules/` |
| **GitHub Copilot** | `uipro init --ai copilot` | `uipro init --ai copilot --global` | `./.github/copilot-instructions.md` |
| **支持所有工具** | `uipro init --ai all` | `uipro init --ai all --global` | 同时生成上述全部配置文件 |

---

## 5. 实例：推荐创建与实战步骤

### 5.1 新建测试工程并配置环境

在 Mac 终端中运行以下命令，创建一个基于 **Vite + React + Tailwind CSS v3** 的标准测试项目：

```bash
# 1. 切换到工作区并创建项目
cd /Users/yuanjianwei/workspace
npm create vite@latest ui-ux-demo -- --template react-ts
cd ui-ux-demo

# 2. 安装项目依赖与 Tailwind CSS v3
npm install
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init -p

# 3. 全局安装 CLI 并初始化 Skill
npm install -g uipro-cli
uipro init --ai all
```

*(运行 `uipro init --ai all` 后，项目根目录下将自动产生 `.claude/` 和 `.cursor/` 等配置文件夹)*

---

### 5.2 测试验证 Prompt 建议

在项目中启动 Cursor 或 Claude Code，复制并运行以下 Prompt 校验生成效果：

#### 🎨 测试主题：高端奢华独立音乐制作与音轨控制台（极简工业设计风）

> **推荐理由**：告别常见的数据表格，主打精致的音频波形图、滑块控制控件与精致的 Bento Grid 网格排版。

```text
请使用 UI/UX Pro Max 规范，帮我在当前 React + Tailwind 项目中设计并实现一个【AI 音乐创作与音轨合成工作站组件】。

视觉与设计要求：
1. 视觉风格：采用 Apple / Teenage Engineering 风格的高端极简工业暗黑风（Minimalist Dark Studio Style），搭配高饱和度的橙黄/琥珀色点缀。
2. 页面布局包括：
   - 顶部：音频主控条（播放/暂停、BPM 调节、Master 音量波形表）。
   - 侧边：音色库与素材采样资源树。
   - 中央主区域：多音轨编辑器视图（含有虚拟音频波形图 Waveform、轨道 EQ 调节旋钮与增益 Slider）。
   - 右侧：AI 智能生成参数调控面板与当前工程导出卡片。
```

---

## 6. 核心命令参数详细解释

* **`npm create vite@latest ui-ux-demo -- --template react-ts`**
  * `npm create`：npm 的脚手架快捷命令（相当于 `npx create-vite`）。
  * `vite@latest`：使用最新的 Vite 前端构建工具。
  * `ui-ux-demo`：指定的输出项目文件夹名称。
  * `--`：npm 的透传参数标号，指示将后面的选项直接传递给 `create-vite`。
  * `--template react-ts`：选用的预设模板，即 React + TypeScript。
* **`npm install -D tailwindcss@3 postcss autoprefixer`**
  * `-D`：将依赖包写入 `devDependencies`（开发环境依赖）。
  * `tailwindcss@3`：锁定安装 Tailwind CSS v3 版本，保证 `tailwind.config.js` 的完美兼容。
  * `postcss` & `autoprefixer`：CSS 预处理工具和浏览器厂商前缀自动补充插件。
* **`npx tailwindcss init -p`**
  * `npx`：运行本地或远程 Node 包命令。
  * `init -p`：同时生成 `tailwind.config.js` 和 `postcss.config.js`。
* **`npm install -g uipro-cli`**
  * `-g`：全局安装，使得 `uipro` 命令行工具可在任何路径直接调用。
* **`uipro init --ai all`**
  * `uipro init`：启动规则下发流程。
  * `--ai all`：指示同时为 Claude Code、Cursor、GitHub Copilot 等所有支持的 Agent 下发配置文件。

---

## 7. 常用维护与更新命令

在实际使用过程中，可以通过 `uipro-cli` 快捷管理 Skill 的生命周期：

```bash
# 检查 uipro-cli 版本及状态
uipro --version

# 更新 Skill 规则数据库至最新版本（保持设计规则库处于最新状态）
uipro update

# 从当前项目或全局中移除指定 AI 的 Skill 规则
uipro remove --ai claude          # 移除项目级配置
uipro remove --ai claude --global # 移除全局配置
```

---

## 8. 高级实战技巧与避坑指南

### 8.1 进阶 Prompt 技巧
* **显式指定图标与动画库**：在 Prompt 中加上“配合 `lucide-react` 图标库和 `framer-motion` 微交互”，AI 会自动生成带有平滑过渡与精致图标的组件。
* **深浅色模式与响应式强制约束**：在 Prompt 中加入“要求同时支持深色/浅色模式切换，并在移动端自动转换为底部抽屉（Drawer）或折叠面板（Accordion）”。
* **保护业务逻辑**：对现有组件重构时，输入“**仅重构视觉排版与 CSS 样式，请严格保持现有的 React state 与回调函数逻辑不变**”，防止 AI 篡改原有功能。

### 8.2 常见问题排查（Troubleshooting）
1. **样式未生效 / 看起来依然是纯文本**：
   * 检查 `tailwind.config.js` 中的 `content` 配置路径是否包含组件文件（如是否漏掉了 `./src/**/*.{js,ts,jsx,tsx}`）。
   * 检查 `src/index.css` 是否引入了 `@tailwind base; @tailwind components; @tailwind utilities;`。
2. **AI 工具没有自动感知 Skill 规则**：
   * **Cursor**：确认 `.cursor/rules/` 文件夹没有被根目录的 `.gitignore` 误过滤。在 Cursor 对话框中亦可键入 `@SKILL.md` 显式引用该规则。
   * **Claude Code**：运行 `claude` 命令前确保终端工作目录在项目根目录下，以便正确识别 `.claude/skills/`。

---

## 9. 参考来源

* [GitHub 官方仓库 (nextlevelbuilder/ui-ux-pro-max-skill)](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill)
* [NPM 官方包 uipro-cli](https://www.npmjs.com/package/uipro-cli)

