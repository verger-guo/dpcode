# Codex → Codex-Internal 迁移记录

> **日期**: 2026-04-14
> **分支**: main
> **目标**: 将所有 `codex` CLI 引用全面替换为 `codex-internal`，不再兼容旧的 `codex` 命令

---

## 背景

项目原先兼容 `codex` 和 `codex-cli` 两个命令名。现在统一切换到 `codex-internal` 作为唯一支持的 CLI binary，同时将 JSON-RPC 初始化参数 `clientInfo.name` 更改为 `codex-tui`。

---

## 变动文件清单

### 1. `packages/shared/src/terminalThreads.ts`

| 改动点 | 旧值 | 新值 |
|--------|------|------|
| `MANAGED_TERMINAL_COMMAND_NAME_BY_CLI_KIND.codex` | `"codex"` | `"codex-internal"` |
| `CODEX_COMMAND_NAMES` | `Set(["codex", "codex-cli"])` | `Set(["codex-internal"])` |

**影响**: 终端进程识别逻辑只会将命令名为 `codex-internal` 的进程识别为 Codex 终端会话。

### 2. `packages/shared/src/codexConfig.ts`

| 改动点 | 旧值 | 新值 |
|--------|------|------|
| `resolveCodexHome()` fallback 路径 | `~/.codex` | `~/.codex-internal` |

**影响**: 当未设置 `CODEX_HOME` 环境变量时，配置文件将从 `~/.codex-internal/` 目录读取。

### 3. `apps/server/src/codexAppServerManager.ts`

| 改动点 | 旧值 | 新值 |
|--------|------|------|
| `buildCodexInitializeParams().clientInfo.name` | `"t3code_desktop"` | `"codex-tui"` |
| `startSession` binary fallback | `"codex"` | `"codex-internal"` |
| `resumeSession` binary fallback | `"codex"` | `"codex-internal"` |
| discovery session `assertSupportedCodexCliVersion` + `spawn()` | `"codex"` | `"codex-internal"` |

**影响**: 所有 Codex app-server 子进程均使用 `codex-internal` 命令启动；JSON-RPC 握手时上报的客户端名称变为 `codex-tui`。

### 4. `apps/server/src/provider/Layers/ProviderHealth.ts`

| 改动点 | 旧值 | 新值 |
|--------|------|------|
| `runCodexCommand` 中 `ChildProcess.make()` | `"codex"` | `"codex-internal"` |
| "not installed" 错误消息 | `` `codex` `` | `` `codex-internal` `` |
| "not authenticated" 错误消息 | `` `codex login` `` | `` `codex-internal login` `` |
| auth probe 失败消息 | `` `codex login` `` | `` `codex-internal login` `` |
| `codexHome` fallback 路径 | `~/.codex` | `~/.codex-internal` |
| 上游输出匹配 | 保留 `"run \`codex login\`"` 匹配 | 新增 `"run \`codex-internal login\`"` 匹配 |

**影响**: 健康检查使用 `codex-internal --version` 和 `codex-internal auth --status` 进行探测；错误提示引导用户运行 `codex-internal login`。

### 5. `apps/server/src/git/Layers/CodexTextGeneration.ts`

| 改动点 | 旧值 | 新值 |
|--------|------|------|
| `ChildProcess.make()` 命令名 | `"codex"` | `"codex-internal"` |

**影响**: Git commit message 生成使用 `codex-internal exec` 命令。

### 6. `apps/server/src/telemetry/Identify.ts`

| 改动点 | 旧值 | 新值 |
|--------|------|------|
| `auth.json` 读取路径 | `~/.codex/auth.json` | `~/.codex-internal/auth.json` |

**影响**: 遥测身份识别从 `~/.codex-internal/auth.json` 读取认证信息。

### 7. `apps/server/src/terminal/managedTerminalWrappers.ts`

| 改动点 | 旧值 | 新值 |
|--------|------|------|
| zsh wrapper `unalias` 目标 | `codex` | `codex-internal` |
| zsh wrapper 函数名 | `codex()` | `codex-internal()` |
| wrapper 内 bin 查找/fallback | `codex` | `codex-internal` |

**影响**: 托管终端中的 shell wrapper 函数拦截 `codex-internal` 命令而非 `codex`。

### 8. `apps/web/src/routes/_chat.settings.tsx`

| 改动点 | 旧值 | 新值 |
|--------|------|------|
| Binary path 说明文案 | `Leave blank to use codex from your PATH.` | `Leave blank to use codex-internal from your PATH.` |

**影响**: Settings UI 中的提示文案正确反映默认 binary 名称。

---

## 格式化改动（无功能影响）

以下文件仅包含 `bun fmt` 自动格式化调整，无逻辑变更：

- `apps/server/src/git/Layers/GitCore.ts` — 代码换行调整
- `apps/web/src/components/BranchToolbar.tsx` — import 语句合并为单行
- `apps/web/src/components/Sidebar.tsx` — 函数签名换行调整
- `apps/web/src/components/chat/MessagesTimeline.tsx` — 长表达式换行调整

---

## 前置条件

使用此版本前，确保系统已安装 `codex-internal` CLI：

```bash
# 验证安装
which codex-internal
codex-internal --version

# 如需认证
codex-internal login
```

配置目录应位于 `~/.codex-internal/`（或通过 `CODEX_HOME` 环境变量指定自定义路径）。

---

## 环境变量

| 变量名 | 作用 | 默认值 |
|--------|------|--------|
| `CODEX_HOME` | Codex 配置/数据目录 | `~/.codex-internal` |
| `CODEX_BINARY_PATH` | 覆盖 binary 路径（Settings UI 中也可配置） | `codex-internal` |
