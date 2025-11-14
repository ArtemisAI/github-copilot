# VS Code Remote Tunnel Manager - AI Agent Guidelines

## Project Architecture

This is a single-shell-script project (`tunnel-manager.sh`) that manages VS Code remote tunnels. The script handles:
- Downloading and installing the VS Code CLI binary
- Starting/stopping tunnel processes with background execution
- Authentication via GitHub/Microsoft providers
- Process monitoring and logging

**Key files:**
- `tunnel-manager.sh` - Main script with functions: `log()`, `killProcessByName()`, `downloadLatestVersion()`, `startTunnel()`, `stopTunnel()`
- `vscode-server.log` - All operations logged here, including auth URLs and process events
- `CHANGE_REQUEST.md` - Planned improvements (`.env` config, Microsoft auth, auto-healing)

## Critical Workflows

**Setup:**
```bash
chmod +x tunnel-manager.sh
```

**Start tunnel:**
```bash
./tunnel-manager.sh start [tunnel-name] [--update]
```
- Downloads CLI if missing, extracts to `/usr/bin`, starts tunnel with `nohup`
- Random name generated if none provided: `tunnel-$(cat /proc/sys/kernel/random/uuid | md5sum | cut -c 1-8)`

**Stop tunnel:**
```bash
./tunnel-manager.sh stop
```
- Kills all `code` processes via `ps -ef | grep code | awk '{print $2}' | xargs kill -9`

**Authentication:**
- First run triggers device code flow (GitHub by default)
- Visit `https://github.com/login/device` with provided code
- Subsequent runs reuse authentication

## Project Conventions

**Process Management:**
- Background execution: `nohup $command 2>&1 | while read line; do log "$line"; done &`
- Process killing: `ps -ef | grep $processName | grep -v grep | awk '{print $2}' | xargs kill -9`
- No PID files or graceful shutdowns

**Download/Install:**
- CLI URL: `https://update.code.visualstudio.com/latest/cli-alpine-x64/stable`
- Archive: `vscode_cli_alpine_x64_cli.tar.gz`
- Extraction: `sudo tar zxf $archive -C /usr/bin`
- Tools: Prefers `curl`, falls back to `wget`

**Configuration:**
- Hard-coded variables at script top (no external config yet)
- Log file: `vscode-server.log` with timestamps
- Install path: `/usr/bin` (requires sudo)

**Error Handling:**
- Basic checks: file existence, download success, extraction
- Logs failures but continues where possible
- No retry logic or validation

## Dependencies & Integration

**External Dependencies:**
- `curl` or `wget` for downloads
- `sudo` for CLI installation
- Linux/Unix environment with `/proc/sys/kernel/random/uuid`

**VS Code Integration:**
- CLI commands: `code tunnel --name $name --accept-server-license-terms`
- Auth providers: GitHub (current), Microsoft (planned in CHANGE_REQUEST.md)
- Connection: `vscode.dev` links or VS Code Command Palette → "Remote - Tunnels: Connect to remote..."

**Cross-Component Patterns:**
- All operations logged to single file with timestamps
- Process names used for identification (no unique IDs)
- Direct CLI execution without wrappers or error parsing

## Development Patterns

**Logging Pattern:**
```bash
log() {
    timestamp=$(date +"%Y-%m-%d %T")
    echo "$timestamp: $1" >>$logfile
}
```

**Background Process Pattern:**
```bash
nohup $command >> $logfile 2>&1 &
```

**Process Discovery Pattern:**
```bash
pids=$(ps -ef | grep $processName | grep -v grep | awk '{print $2}')
```

**Planned Changes (CHANGE_REQUEST.md):**
- `.env` configuration file
- Microsoft authentication provider
- Auto-healing supervisor loop
- PID-based process management
- Non-root installation paths</content>
<parameter name="filePath">c:\Users\Laptop\Desktop\Projects\Tools_for_Agents\github-copilot\.tools\Code_Tunnel\vscode-server-deploy-script\.github\copilot-instructions.md
