# MixumART 0.1.1 — macOS Apple Silicon + Windows x86_64

Status: pre-release. Target date: 2026-11-14. Previous release: 0.1.0 (preview).

> Chinese translation: [release-notes-0.1.1.zh-CN.md](release-notes-0.1.1.zh-CN.md).

## Highlights

- **Full Host and Remote Window Clipboard (P0-4a & Route B).** Bidirectional clipboard
  synchronization is now fully supported across desktop and environment surfaces:
  - **Route B Compositor:** The Smithay compositor announces clipboard capabilities,
    accepts `clipboard.set` requests to inject Wayland server-side selections, and
    forwards container client selection changes over MAP socket streams.
  - **Desktop SurfaceView:** Pasting (Cmd/Ctrl+V) writes directly to the active window
    selection via `remote_clipboard_set` and signals the target application, preserving
    multiline text and eliminating slow thousands-of-events keystroke synthesis.
  - **Echo Suppression:** RFC 0004 echo suppression queue prevents host-initiated
    clipboard writes from triggering feedback loops back to the host system clipboard.
  - **Keysym Table:** Added modifier keysym mappings (such as `Control_L`) to the compositor,
    allowing synthetic modifier sequences on host platforms.
- **Agent Installation UI in Desktop.** The desktop application features an integrated
  management interface for installing and monitoring Claude Code, Codex, Gemini CLI, and
  OpenCode:
  - Recipes match npm 12 lifecycle script rules and verify installations via `--version`.
  - Dynamic agent status refresh when toggling between remote nodes and local instances.
  - Isolated log streams prevent background task race conditions and clear state across node switches.
  - Rich error reporting presents exact failure reasons when package installations fail.
- **Robust System Elevation (UAC & Sudo).**
  - **Windows UAC Cancellation:** Fixed PowerShell elevation handling where user-cancelled
    UAC prompts previously caused silent false-positive successes. Cancelling the elevation
    dialog is now accurately detected and handled.
  - **Console Codepage Safeguard:** PowerShell output uses ASCII markers with base64-encoded
    UTF-8 payloads, preventing localized error messages (e.g., GBK on Chinese Windows) from
    corrupting cancellation parsing.
  - **Setup Modal Styling:** Resolved CSS variable references so modal elevation cards render
    with proper contrast instead of transparent surfaces.
  - **Remote Privilege Forwarding:** Remote privilege approval notifications from agentd
    are piped directly into desktop approval dialogues for seamless one-click review.

## Compositor & Display

- Implemented MAP RFC 0004 compositor contract in `crates/mixum-env-compositor`.
- Container client selections are read asynchronously using socketpairs and published as
  MAP clipboard records.
- Injected selections are tagged with `SelectionOrigin` to ensure subsequent container paste
  operations correctly resolve from injected data rather than previous owners.
- SurfaceView includes explicit read and write clipboard toolbar actions alongside standard
  keyboard shortcuts.

## Desktop & Setup

- Added dedicated agent installation tab in the desktop UI.
- Unified initialization logic between `mixum setup` CLI and the desktop onboarding view.
- Added support for remote privilege approval routing and real-time approval status streaming.
- Improved node switching stability and lifecycle management.

## Validation

- Tested desktop Rust library tests: `cargo test -p mixum-env-desktop --lib`.
- Compositor keysym and clipboard logic verified: `cargo test -p mixum-env-compositor`.
- Frontend production build passed cleanly: `npm run build` (TypeScript + Vite).
- B-route end-to-end test suite: `tests/e2e/b-clipboard.mjs`.

## Contributors

- **yubing0721** ([@yubing0721](https://github.com/yubing0721)) — Route B clipboard, host SurfaceView clipboard integration, UAC cancellation fix, agent install UI & recipes, remote privilege dialog
- **AshReo** ([@ashreo](https://github.com/ashreo)) — shared setup core, runner architecture, packaging
