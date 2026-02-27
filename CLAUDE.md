# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 提供在本仓库中工作时的指导。

## 项目概述

**OpenClaw** 是一个个人 AI 助手网关，连接各种消息平台（WhatsApp、Telegram、Slack、Discord、Signal、iMessage、Google Chat、Microsoft Teams 等）并将消息路由到 AI 智能体。它包含：

- **网关 (Gateway)**：WebSocket 控制平面 (`src/gateway/`)，管理会话、频道、工具和事件
- **CLI**：命令行界面 (`src/cli/`、`src/commands/`)
- **频道 (Channels)**：消息平台集成 (`src/telegram/`、`src/discord/`、`src/slack/`、`src/signal/`、`src/imessage/`、`src/web/`、`src/channels/`)
- **扩展 (Extensions)**：插件系统 (`extensions/*`，37 个工作区包)
- **应用 (Apps)**：原生应用 (`apps/macos/`、`apps/ios/`、`apps/android/`)
- **技能 (Skills)**：社区贡献的包 (`skills/`)

## 构建、测试和开发命令

**前置要求**：Node.js ≥22.12.0，pnpm 10.23.0

```bash
# 安装
pnpm install

# 开发（自动重载）
pnpm dev                              # 以开发模式运行 CLI
pnpm openclaw <command>               # 使用 tsx 运行 CLI（直接运行 TypeScript）
pnpm gateway:watch                    # 网关开发监视模式

# 构建
pnpm build                            # 完整构建（TypeScript -> dist/）
pnpm ui:build                         # 构建 Web UI

# 测试
pnpm test                             # 运行所有测试（Vitest，并行）
pnpm test:coverage                    # 运行测试并生成覆盖率报告
pnpm test:unit                        # 仅运行单元测试
pnpm test:e2e                         # 运行端到端测试
pnpm test:live                        # 运行实时测试（需要真实 API 密钥）

# 代码检查和格式化
pnpm check                            # 完整检查（格式化 + TypeScript + 代码检查）
pnpm lint                             # Oxlint（类型感知）
pnpm lint:fix                         # 自动修复代码检查问题
pnpm format                           # 检查格式化（oxfmt）
pnpm format:fix                       # 修复格式化
pnpm tsgo                             # 仅进行 TypeScript 类型检查
```

**运行单个测试**：`pnpm test -- src/path/to/file.test.ts`

## 高层架构

### 网关架构
网关 (`src/gateway/`) 是一个作为控制平面的 WebSocket 服务器：
- `server.ts` - 主 WebSocket 服务器入口点
- `client.ts` - 客户端连接处理
- `protocol/` - 协议定义和消息类型
- `server-methods/` - RPC 方法实现（配置、发送、技能、对话、网页、向导）

### 频道系统
频道是消息平台集成：
- 核心频道：`src/telegram/`、`src/discord/`、`src/slack/`、`src/signal/`、`src/imessage/`、`src/web/`
- 频道抽象：`src/channels/`（消息、安全、设置等适配器接口）
- 扩展：`extensions/*`（基于插件的频道，如 Matrix、Zalo、Teams 等）

修改共享频道逻辑（路由、白名单、配对、命令门控）时，需考虑**所有**内置 + 扩展频道。

### 插件 SDK
扩展使用插件 SDK (`src/plugin-sdk/`) 进行类型导入：
- 导入路径：`openclaw/plugin-sdk`（通过 jiti 别名解析到 `./src/plugin-sdk/index.ts`）
- 扩展是 `extensions/*` 中的工作区包
- 将插件专用依赖项保留在扩展的 `package.json` 中；除非核心使用，否则不要添加到根目录

### 智能体运行时
- `src/agents/` - 带有工具定义的 AI 智能体运行时
- 通过 `@mariozechner/pi-*` 包进行 Pi 集成
- 用于扩展智能体功能的技能系统

### 配置和存储
- 配置存储在 `~/.openclaw/`
- 会话存储在 `~/.openclaw/sessions/`
- 凭证存储在 `~/.openclaw/credentials/`

## 代码风格和约定

- **语言**：TypeScript（ESM），严格模式，避免使用 `any`
- **代码检查/格式化**：Oxlint 和 Oxfmt（提交前运行 `pnpm check`）
- **文件大小**：保持文件在 700 行以内；在提高清晰度时进行拆分/重构
- **命名**：产品使用 **OpenClaw**，CLI/路径/配置键使用 `openclaw`
- **注释**：为复杂或非明显的逻辑添加简要注释

## 测试指南

- **框架**：使用 V8 覆盖率的 Vitest
- **命名**：`*.test.ts`（与源代码同位置），e2e：`*.e2e.test.ts`，实时：`*.live.test.ts`
- **覆盖率阈值**：行/函数/语句 70%，分支 55%
- **工作线程**：最多 16 个（在 `vitest.config.ts` 中配置）
- **实时测试**：`CLAWDBOT_LIVE_TEST=1 pnpm test:live`（需要真实 API 密钥）

## 关键文件和位置

- 入口点：`src/entry.ts`、`src/index.ts`、`src/runtime.ts`
- CLI 接线：`src/cli/`
- 命令：`src/commands/`
- 媒体管道：`src/media/`
- 基础设施：`src/infra/`
- 类型：`src/types/`

## 多智能体安全

- **不要**创建/应用/删除 git stash 条目，除非明确要求
- **不要**切换分支，除非明确要求
- **不要**创建/删除/修改 git worktree 检出，除非明确要求
- 当用户说 "push" 时，你可以执行 `git pull --rebase` 来集成最新更改
- 当用户说 "commit" 时，仅提交你的更改
- 报告时专注于你的编辑；除非真正受阻，否则避免免责声明

## 文档

- 文档托管在 Mintlify 上，地址为 `docs.openclaw.ai`
- 内部文档链接：根相对路径，不带 `.md`/`.mdx` 扩展名（例如 `[Config](/configuration)`）
- README 使用绝对文档 URL（`https://docs.openclaw.ai/...`）
- 文档内容必须是通用的：不包含个人设备名称/主机名/路径

## 发布渠道

- **stable**：标记的发布版本（`vYYYY.M.D`），npm dist-tag `latest`
- **beta**：预发布标签（`vYYYY.M.D-beta.N`），npm dist-tag `beta`
- **dev**：`main` 分支的最新提交

## 平台特定说明

**macOS/iOS**：
- SwiftUI：优先使用 `@Observable`、`@Bindable`，而非 `ObservableObject`/`@StateObject`
- 使用 `Observation` 框架
- 不要通过 SSH 重新构建 macOS 应用
- 日志：`./scripts/clawlog.sh` 查询统一日志

**移动端**：
- 使用模拟器前，检查是否有连接的真实设备（iOS + Android）
- "重启应用" 意味着重新构建（重新编译/安装）并重新启动，而不仅仅是终止/启动

## 重要模式

- **CLI 进度**：使用 `src/cli/progress.ts`（`osc-progress` + `@clack/prompts`）；不要手动创建旋转器
- **状态输出**：保持表格 + ANSI 安全换行（`src/terminal/table.ts`）
- **工具模式**：避免在工具输入模式中使用 `Type.Union`；使用 `stringEnum`/`optionalStringEnum`。避免使用原始 `format` 属性名。
- **错误处理**：带有 `formatUncaughtError` 的自定义错误类

## 依赖项

- 补丁依赖项（`pnpm.patchedDependencies`）必须使用精确版本（不含 `^`/`~`）
- 永远不要更新 Carbon 依赖项（`@buape/carbon`）
- 补丁依赖项需要明确批准
