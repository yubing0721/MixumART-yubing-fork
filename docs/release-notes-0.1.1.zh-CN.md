# MixumART 0.1.1 — macOS Apple Silicon + Windows x86_64

状态：预发布。目标日期：2026-11-14。上一版本：0.1.0（预览版）。

> 英文原文：[release-notes-0.1.1.md](release-notes-0.1.1.md)。

## 重点特性

- **完整宿主与远程窗口双向剪贴板（P0-4a 与 B 路线）。** 桌面端与环境窗口之间已全面打通双向剪贴板同步：
  - **B 路线合成器：** Rust/Smithay 合成器完整接入 MAP 剪贴板协议，通告剪贴板能力，响应 `clipboard.set` 指令写入 Wayland 服务端选区，并通过 MAP socket 向上游流式推送容器客户端的剪贴板变动。
  - **桌面 SurfaceView：** 粘贴操作（Cmd/Ctrl+V）直接通过 `remote_clipboard_set` 写入活动窗口选区并触发目标程序粘贴，完整保留多行与长文本，消除逐键模拟按键带来的巨大延迟。
  - **回声抑制机制：** 遵循 RFC 0004 规范实现回声过滤队列，防止宿主主动写入的剪贴板内容在反射回调时错误覆盖宿主原有系统剪贴板。
  - **键码映射补充：** 补充了修饰键符号表（如 `Control_L`），使宿主端能正确向合成器发送组合修饰键事件。
- **桌面端 Agent 安装管理界面。** 桌面应用新增统一的可视化 Agent 管理面板，支持一键安装并监控 Claude Code、Codex、Gemini CLI 与 OpenCode：
  - 安装规则适配 npm 12 默认策略，并通过 `--version` 进行严格的有效性验证。
  - 在切换远程节点或切回本地环境时自动刷新各 Agent 的安装及运行状态。
  - 任务安装日志按目标节点进行世代隔离，避免切换节点时的日志竞争与乱序回写。
  - 遇到安装异常时展示底层真实失败原因。
- **系统权限提权可靠性增强（UAC 与 Sudo）。**
  - **Windows UAC 取消判定修复：** 修复了此前 PowerShell 提权失败却仍误报成功的问题，现可准确识别用户主动取消 UAC 弹窗的场景并友好提示。
  - **控制台字符集防乱码保护：** 提权脚本改用 ASCII 标记结合 Base64 编码的 UTF-8 载荷传递数据，规避了中文 Windows 下 GBK 控制台编码导致的错误解析失败。
  - **安装弹窗样式修正：** 修复了 Setup 弹窗的 CSS 变量引用错误，使模态提权卡片正常显示背景衬底。
  - **远程特权审批穿透：** 来自 agentd 的特权执行审批请求可实时穿透推送到桌面端弹窗，便于用户即时审核放行。

## 合成器与显示

- 在 `crates/mixum-env-compositor` 中实现 MAP RFC 0004 合成器规范。
- 容器客户端选区通过 socketpair 异步拉取并封装为 MAP 剪贴板事件上报。
- 注入选区标记有 `SelectionOrigin`，确保后续容器内粘贴操作直接读取注入内容。
- SurfaceView 界面工具栏提供明确的读/写剪贴板按钮及快捷键支持。

## 桌面与初始化

- 桌面端集成专属 Agent 安装管理标签页。
- 统一 `mixum setup` CLI 与桌面初始化向导的底层调度核心。
- 支持远程特权审批信道与状态流式监听。
- 优化了节点切换时的生命周期管理与连接稳定性。

## 验证结论

- 桌面端 Rust 库测试通过：`cargo test -p mixum-env-desktop --lib`。
- 合成器键码及剪贴板逻辑测试通过：`cargo test -p mixum-env-compositor`。
- 前端生产模式构建通过：`npm run build`（TypeScript + Vite）。
- B 路线端到端集成测试通过：`tests/e2e/b-clipboard.mjs`。

## 贡献者

- **yubing0721** ([@yubing0721](https://github.com/yubing0721)) — B 路线剪贴板、宿主 SurfaceView 剪贴板集成、UAC 取消处理修复、Agent 安装面板与规则适配、远程特权弹窗
- **AshReo** ([@ashreo](https://github.com/ashreo)) — 共享初始化核心、运行环境架构设计、打包发布
