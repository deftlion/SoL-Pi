# Pi extension API usage

Inventory of Pi APIs used by production code under `src/sol-pi/`. This is a source-usage inventory, not a compatibility validation. Line numbers are navigation hints and may drift as code changes.

## ExtensionAPI methods

| API                           | Purpose                                                            | Example location                                                                                                                                                     |
| ----------------------------- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pi.on()`                     | Register lifecycle and transformation handlers                     | `src/sol-pi/index.ts:37`                                                                                                                                             |
| `pi.registerTool()`           | Extend `edit` and `write`; add `obs_recall` and `update_plan`      | `src/sol-pi/extensions/action-fusion/index.ts:102`, `src/sol-pi/extensions/observation-pack/index.ts:72`, `src/sol-pi/extensions/online-context-compact/tools.ts:57` |
| `pi.appendEntry()`            | Persist reducer journal events and online-compaction state         | `src/sol-pi/extensions/evidence-preserving-reducer/journal.ts:18`, `src/sol-pi/extensions/online-context-compact/state.ts:150`                                       |
| `pi.sendMessage()`            | Resume the parent task with a custom message after compaction      | `src/sol-pi/extensions/online-context-compact/extension.ts:374`                                                                                                      |
| `pi.getSettings()` (optional) | Read effective compaction settings, with fallback when unavailable | `src/sol-pi/extensions/online-context-compact/extension.ts:197`                                                                                                      |

### Events registered through pi.on()

| Area                              | Events                                                        |
| --------------------------------- | ------------------------------------------------------------- |
| Startup                           | `session_start`                                               |
| Context transformation            | `context`                                                     |
| Tool-result reduction             | `tool_result`                                                 |
| Request and conversation tracking | `before_provider_request`, `message_end`, `input`, `turn_end` |
| Compaction orchestration          | `agent_settled`, `session_before_compact`, `session_compact`  |
| Branch navigation                 | `session_before_tree`, `session_tree`                         |
| Cleanup                           | `session_shutdown`                                            |

Most registrations are in `src/sol-pi/extensions/online-context-compact/extension.ts:246–496`. Observation packing uses `context` at `src/sol-pi/extensions/observation-pack/index.ts:144`; evidence-preserving reduction uses `tool_result` at `src/sol-pi/extensions/evidence-preserving-reducer/index.ts:205`.

## ExtensionContext APIs

| Area                         | Members used                                                                                          |
| ---------------------------- | ----------------------------------------------------------------------------------------------------- |
| Runtime and trust            | `cwd`, `mode`, `isProjectTrusted()`                                                                   |
| Session identity and history | `sessionManager.getSessionDir()`, `sessionManager.getSessionId()`, `sessionManager.getBranch()`       |
| Context measurement          | `getSystemPrompt()`, `getContextUsage()`, `model`, `model.contextWindow`                              |
| Execution control            | `signal`, `abort()`, `isIdle()`, `compact()`                                                          |
| UI                           | `ui.notify()`, `ui.setStatus()`                                                                       |
| Nested model calls           | `modelRegistry.find()`, `modelRegistry.complete()`; fallback to `modelRegistry.getApiKeyAndHeaders()` |

Key locations:

- Session paths: `src/sol-pi/runtime-paths.ts:14`
- Trust checks: `src/sol-pi/index.ts:33`
- Context accounting: `src/sol-pi/extensions/online-context-compact/extension.ts:209`
- Native compaction: `src/sol-pi/extensions/online-context-compact/extension.ts:423`
- Model registry compatibility path: `src/sol-pi/extensions/evidence-preserving-reducer/provider.ts:115–150`
- Notifications and status: `src/sol-pi/tui.ts:68–78`

## Other exported Pi helpers

These depend on Pi's exported library surface rather than methods on `ExtensionAPI`.

- **Built-in tools:** `createEditToolDefinition()`, `createWriteToolDefinition()`, `createBashToolDefinition()`. Action Fusion wraps their execution and rendering.
- **Configuration:** `getAgentDir()`, `CONFIG_DIR_NAME`, `SettingsManager.create()`, plus `drainErrors()`, `getShellPath()`, and `getShellCommandPrefix()`.
- **Compaction and projection:** `estimateTokens()`, `findCutPoint()`, `sessionEntryToContextMessages()`, and optional `buildSessionProjection()`.
- **Terminal rendering:** `Text`, `Container`, and the `Component` type from `@earendil-works/pi-tui`, plus Pi's `Theme` type.
- **Legacy provider fallback:** `complete` from `@earendil-works/pi-ai/compat`.

## Non-usage note

`ctx.executeTool()` is mentioned in a comment but is not called by production code. Action Fusion invokes the built-in tool definitions directly.
