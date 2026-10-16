# Netbird UI

### Objective
Build the native NetBird desktop client GUI application (`netbird-ui.app`) from local source code in [`client/ui/`](file:///Users/pavkry/Documents/_projects/git/hub/techiknow/netbird/client/ui) and install it to `/Applications` to connect to your private peer-to-peer mesh.

---

### Current Environment & Prerequisites Check

| Component | Status / Version | Action Needed |
| :--- | :--- | :--- |
| **Node.js** | `v22.14.0` | Ready (meets ≥ 20 requirement) |
| **pnpm** | `10.13.1` | Ready |
| **Go** | `1.23.1` detected | Needs Go ≥ 1.25 (will auto-download `go1.25.12` via Go toolchains or upgrade via `brew upgrade go`) |
| **Wails v3 CLI** | Missing from `$PATH` | Needs `go install github.com/wailsapp/wails/v3/cmd/wails3@latest` |
| **Task runner** | Missing from `$PATH` | Needs `go install github.com/go-task/task/v3/cmd/task@latest` (or `brew install go-task`) |
| **CGO / Xcode** | macOS Command Line Tools | Required for CGO compilation |

---

### Step-by-Step Execution Plan

#### Phase 1: Toolchain Setup
1. Install `task` and `wails3` CLI tools:
   ```zsh
   go install github.com/go-task/task/v3/cmd/task@latest
   go install github.com/wailsapp/wails/v3/cmd/wails3@latest
   export PATH="$(go env GOPATH)/bin:$PATH"
   ```
*(Note: These commands require network access outside the sandbox to download Go modules).*

#### Phase 2: Build the Frontend & macOS App Bundle
Run the packaging task inside [`client/ui`](file:///Users/pavkry/Documents/_projects/git/hub/techiknow/netbird/client/ui):
```zsh
cd client/ui
task package
```
This automatically runs:
1. `wails3 generate bindings`: Generates TypeScript interfaces from Go service models into `frontend/bindings/`.
2. `pnpm install` & `pnpm run build`: Compiles the React UI into `frontend/dist`.
3. `go build`: Compiles the macOS native executable with embedded webview assets and CGO.
4. Assembles the `.app` structure at `client/ui/bin/netbird-ui.app` with `Info.plist`, icons (`icons.icns`), and ad-hoc code signing (`codesign --force --deep --sign -`).

#### Phase 3: Install & Launch (Outside Sandbox Write)
1. Copy the packaged bundle to macOS Applications:
   ```zsh
   cp -R client/ui/bin/netbird-ui.app /Applications/NetBird.app
   ```
2. Launch NetBird UI:
   ```zsh
   open /Applications/NetBird.app
   ```
3. Configure your private mesh connection:
   * In the NetBird UI Settings panel, configure your custom Management URL (e.g. `https://<your-management-domain>:443`).
   * The UI will communicate with your NetBird client daemon via `/var/run/netbird.sock`.

---

Please let me know if you approve this plan to proceed with the toolchain installation and build.