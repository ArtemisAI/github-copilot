# Change Request: VS Code Remote Tunnel Manager Improvements

## Objective

Modernize and harden the `vscode-server-deploy-script` project so that it:

1. Provides a **persistent, auto-healing VS Code tunnel** that can be managed by the user.
2. Supports **Microsoft account login** as the primary authentication provider (instead of GitHub-only).
3. Uses a **`.env` configuration file** for tunable settings (tunnel name, provider, install path, update channel, log file, etc.).
4. Improves **security** (download validation, safer process management, least-privilege install options).
5. Brings the **documentation up to date** with the latest VS Code Tunnels and CLI guidance, including links to the curated docs in `.tools/Code_Tunnel`.

All changes should maintain compatibility with typical Linux server environments and be straightforward to extend.

---

## Scope of Work

### 1. Configuration via `.env`

Introduce a `.env` file (and optional sample `.env.example`) to configure the tunnel manager instead of hard-coded values inside `tunnel-manager.sh`.

**New configuration variables (suggested, adjust as needed):**

- `TUNNEL_NAME` – default tunnel name; if empty, the script generates a random name.
- `TUNNEL_PROVIDER` – authentication provider; default: `microsoft`. Allowed values: `microsoft`, `github`.
- `TUNNEL_VISIBILITY` – optional visibility (e.g., `org`, `private`) if supported by the current CLI.
- `VSCODE_CLI_DOWNLOAD_URL` – default `https://update.code.visualstudio.com/latest/cli-alpine-x64/stable`.
- `VSCODE_CLI_INSTALL_PATH` – default `/usr/bin` (but allow non-root paths, e.g., `$HOME/.local/bin`).
- `VSCODE_CLI_BINARY_NAME` – default `code`.
- `VSCODE_CLI_ARCHIVE` – default `vscode_cli_alpine_x64_cli.tar.gz`.
- `VSCODE_LOG_FILE` – default `vscode-server.log`.
- `AUTO_UPDATE` – default `false`; when `true`, periodically check for CLI updates.
- `RESTART_ON_FAILURE` – default `true`; when `true`, enable auto-heal loop.
- `HEALTHCHECK_INTERVAL_SECONDS` – default e.g. `60`.
- `MAX_RESTARTS_PER_HOUR` – to avoid restart storms (optional).

**Implementation details:**

- Add a small `.env` loader at the top of `tunnel-manager.sh` (POSIX-compatible):
  - If `.env` exists in the script directory, `set -a` / `source .env` / `set +a`, or read line-by-line and `export` key/value pairs while ignoring comments.
  - Validate required variables and set sensible defaults if they are missing.
- Add `.env.example` with commented documentation for each variable.
- Ensure that `.env` is listed in `.gitignore` so real deployment config is not committed.

---

### 2. Authentication and provider handling (Microsoft-first)

Update the script so that it explicitly supports Microsoft login and can be configured via `.env`.

**Current state:**

- Script calls `code tunnel --name … --accept-server-license-terms` directly, relying on first-run device code flow.
- No explicit `code tunnel user login` step and no control over provider.

**Required changes:**

1. **Pre-login command support:**
   - Implement a function `login_user()` that runs **once** (or as needed):
     - For `TUNNEL_PROVIDER=microsoft`:
       - `code tunnel user login --provider microsoft`
     - For `TUNNEL_PROVIDER=github`:
       - `code tunnel user login --provider github` (or the default provider if supported).
   - Run `login_user()` before starting the tunnel if:
     - There is no existing login, or
     - An explicit `login` subcommand is requested (`./tunnel-manager.sh login`).
   - Ensure CLI output from login is also logged to `VSCODE_LOG_FILE` with appropriate sensitivity notes.

2. **New CLI subcommand:**
   - Add `login` and `status` commands to `tunnel-manager.sh`:
     - `./tunnel-manager.sh login` – run `login_user()` for the configured provider.
     - `./tunnel-manager.sh status` – report whether the tunnel is running and basic tunnel info.

3. **Provider validation:**
   - Validate `TUNNEL_PROVIDER` at startup; if not one of allowed values, exit with a clear message.

4. **Documentation updates:**
   - Update the project README to explain how to:
     - Configure `TUNNEL_PROVIDER`.
     - Use `./tunnel-manager.sh login` to authenticate with Microsoft.
     - Use `./tunnel-manager.sh start` to start the tunnel once login is complete.

---

### 3. Persistence and auto-healing (continuous tunnel)

The script must be able to keep the tunnel alive and attempt automatic recovery if the process fails.

**Current state:**

- Uses `nohup code tunnel … &` to start the tunnel once.
- No monitoring, no restart, no health check, no limiting of restarts.

**Required changes:**

1. **Introduce a monitoring loop (auto-heal mode):**

   Implement a function, e.g. `run_tunnel_supervisor()`, which:

   - Starts the tunnel process.
   - Waits for it to exit.
   - Logs exit code and timestamp.
   - If `RESTART_ON_FAILURE=true`:
     - Sleeps `HEALTHCHECK_INTERVAL_SECONDS` seconds.
     - Respects `MAX_RESTARTS_PER_HOUR` or similar throttle.
     - Restarts the tunnel, repeating the cycle.
   - If `RESTART_ON_FAILURE=false`, exits after the first failure.

   This can be implemented as a simple `while` loop inside the script, not requiring systemd but compatible with running under systemd or cron.

2. **Refactor start logic:**

   - Replace the current `nohup` pipeline with a more controlled start:
     - Start the tunnel via `code tunnel --name … --accept-server-license-terms` inside `run_tunnel_supervisor()`.
     - Capture the process PID and log it.
     - Route stdout/stderr to the log file directly (e.g., `>> "$VSCODE_LOG_FILE" 2>&1`).

3. **Health checks (optional but recommended):**

   - Implement a lightweight health check in `status`:
     - Check if the tracked PID is alive.
     - Optionally parse CLI/log output for known success markers.
   - Log when health checks detect a dead process and when a restart is triggered.

4. **Systemd integration (optional but recommended in docs):**

   - Provide an example `systemd` unit file in docs (not necessarily installed by default):
     - `ExecStart=/path/to/tunnel-manager.sh start`
     - `Restart=always`
     - `RestartSec=<interval>`
   - Reference this in the README so users who want a stronger supervisor can use it.

5. **Stop behavior:**

   - Instead of killing all `code` processes:
     - Track the PID (or PID file, e.g., `tunnel-manager.pid`).
     - On `stop`, read the PID and attempt a graceful shutdown:
       - `kill PID`; wait; then `kill -9 PID` if needed.
     - If PID file is missing or stale, fall back to a safe `pkill -f "code tunnel"` or similar with clear warnings.

---

### 4. Security and robustness improvements

Address known security and robustness issues:

1. **Safer process management:**

   - Replace `ps -ef | grep code | ... | kill -9` with PID-based management (PID file) as described above.
   - Only kill the specific tunnel process started by the script.
   - Avoid `kill -9` as the first resort—use graceful signals first.

2. **Download hardening:**

   - Keep using HTTPS but document the trust model explicitly.
   - Optional enhancement (recommended if feasible):
     - Allow specifying an expected checksum in `.env` (e.g. `VSCODE_CLI_SHA256`) and verify the downloaded tarball before extraction.
   - Log the CLI version after install, if `code --version` provides it.

3. **Install location flexibility (least privilege):**

   - Support non-root installation by using `VSCODE_CLI_INSTALL_PATH`:
     - Default may remain `/usr/bin` for backward compatibility.
     - If `VSCODE_CLI_INSTALL_PATH` is set to a path under `$HOME`, do not use `sudo` for extraction.
   - Document how to add the install path to `$PATH` in shell profiles.

4. **Input validation and quoting:**

   - Quote variables used in commands, especially `tunnelName` and file paths:
     - `--name "$tunnelName"`
     - `"$VSCODE_CLI_INSTALL_PATH/$VSCODE_CLI_BINARY_NAME"`
   - Validate that `tunnelName` does not contain problematic characters (spaces, quotes, etc.), or clearly document supported characters.

5. **Logging sensitivity:**

   - Document that `VSCODE_LOG_FILE` can contain device login URLs and codes.
   - Consider masking or not logging certain highly sensitive lines if they appear (optional but ideal).
   - Ensure log file permissions are reasonable (e.g. `600` by default in docs or script).

---

### 5. Code structure and CLI interface changes

Refine `tunnel-manager.sh` into clearly separated responsibilities and a stable user-facing interface.

**New/updated commands:**

- `./tunnel-manager.sh login` – Perform `code tunnel user login` using the configured provider.
- `./tunnel-manager.sh start [--once] [--update]` – Start the tunnel.
  - Default: start supervisor loop with auto-heal behavior.
  - `--once`: start the tunnel once without auto-restart.
  - `--update`: force CLI download/update before start (existing behavior, but now integrated with `.env` settings).
- `./tunnel-manager.sh stop` – Stop the supervised tunnel process cleanly.
- `./tunnel-manager.sh status` – Show whether the tunnel is running, PID, and key config values.
- `./tunnel-manager.sh help` – Print usage.

Update the usage/help text accordingly and keep it in sync with README examples.

**Refactoring suggestions:**

- Introduce functions for:
  - `load_env()` – load and validate configuration.
  - `download_cli()` – encapsulate download logic.
  - `install_cli()` – handle extraction and path/permissions.
  - `generate_tunnel_name()` – random name logic.
  - `start_tunnel_once()` – single-run start.
  - `run_tunnel_supervisor()` – loop for persistence/auto-heal.
  - `login_user()` – provider-based authentication.
  - `print_status()` – status reporting.

This will make the script easier to maintain and extend.

---

### 6. Documentation updates

The project documentation should be brought in line with the new behavior and with the external VS Code Tunnels documentation curated under `.tools/Code_Tunnel`.

**Specific documentation tasks:**

1. **Update existing `README.md`:**

   - Add a high-level architecture/behavior overview:
     - What the script does (install CLI, manage tunnels, supervise process).
     - Relationship to official VS Code Tunnels features.
   - Document all new `.env` variables with descriptions and defaults.
   - Document the new CLI commands (`login`, `start`, `stop`, `status`, `help`).
   - Provide step-by-step examples for:
     - Microsoft login:
       - Set `TUNNEL_PROVIDER=microsoft` in `.env`.
       - Run `./tunnel-manager.sh login` and follow the device code instructions.
       - Run `./tunnel-manager.sh start` to establish the tunnel.
     - GitHub login (optional path for users who prefer GitHub): set `TUNNEL_PROVIDER=github`.
   - Add a section explaining persistence and auto-heal:
     - How the supervisor loop works.
     - How to disable auto-heal (`RESTART_ON_FAILURE=false`).
     - How to integrate with `systemd` (include example unit file snippet).

2. **Add `.env.example`:**

   - Annotate each variable with comments describing its purpose and typical values.
   - Include reasonable defaults that work for a simple local install.

3. **Link to Code_Tunnel docs:**

   - In the README, add a section such as "Further Reading" linking to:
     - `.tools/Code_Tunnel/arm-vscode-tunnels-guide.md` (for ARM-specific CLI install guidance).
     - `.tools/Code_Tunnel/vscode-remote-tunnels-marketplace.md` (for extension behavior and UI-based connection).
     - `.tools/Code_Tunnel/stackoverflow-microsoft-account.md` (for Microsoft-provider login context).

4. **Create a high-level doc inside `.tools/Code_Tunnel` (optional but recommended):**

   - Add a `vscode-server-deploy-script.md` in `.tools/Code_Tunnel` summarizing the project:
     - Purpose, features, security considerations, limitations.
     - How it complements the ARM and marketplace docs.
   - Ensure `Code_Tunnel/README.md` links to this new file.

---

### 7. Backward compatibility considerations

When implementing these changes, please:

- Preserve basic behavior for existing users who simply run:
  - `./tunnel-manager.sh start my-tunnel`
  - `./tunnel-manager.sh stop`
- If `.env` is absent, fall back to built-in defaults that match current behavior as closely as possible:
  - Provider: `github` (to match existing GitHub-centric login), **or** switch default to `microsoft` if that’s acceptable and documented.
  - Install path: `/usr/bin`.
  - Log file: `vscode-server.log`.
  - No auto-heal loop unless explicitly enabled (or vice versa; choose and document).
- Clearly document any breaking changes in a `Changelog` section.

---

### 8. Acceptance criteria

The change request is considered complete when:

1. **Functional:**
   - A user can:
     - Configure the script via `.env` (including provider and tunnel name).
     - Run `./tunnel-manager.sh login` and log in with Microsoft as provider.
     - Run `./tunnel-manager.sh start` to start a tunnel that is supervised and auto-heals on failure.
     - Run `./tunnel-manager.sh status` to view current tunnel state.
     - Run `./tunnel-manager.sh stop` to cleanly stop the supervised tunnel.

2. **Security & robustness:**
   - Script no longer indiscriminately kills all `code` processes; it manages its own PID.
   - Install path can be configured to a non-root directory, and sudo is only used when needed.
   - Download behavior is as safe as reasonably possible (HTTPS, optional checksum, documented trust model).

3. **Documentation:**
   - `README.md` is updated with new features, `.env` configuration, and usage examples.
   - `.env.example` exists and is clear.
   - Optional `vscode-server-deploy-script.md` is added under `.tools/Code_Tunnel` and linked from `Code_Tunnel/README.md`.

4. **Code Quality:**
   - `tunnel-manager.sh` is organized into well-named functions with clear responsibilities.
   - Help output (`./tunnel-manager.sh help` or default usage) matches the documented interface.

Please implement these changes in a way that keeps the script POSIX-compatible (no Bash-only features unless the shebang is updated accordingly and documented).
