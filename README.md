# acad-mcp

Let Claude Code or Codex read, draw, and edit in AutoCAD. Version **0.16.0**.

**Claude Code is the tested client. Codex support is new and untested in AutoCAD.**

## Install with your AI assistant

Tell Claude Code or Codex: "Install acad-mcp from https://github.com/bai-ting-chen/acad-mcp-release".

Instructions for the assistant (Windows PowerShell):

1. Check `Get-Process -Name acad -ErrorAction SilentlyContinue`. If AutoCAD is open, ask the person to close it and wait for them before continuing. Keep the current AI session open.
2. Get the latest release and its single zip asset; do not clone or build the source repository:

```powershell
$ErrorActionPreference = 'Stop'
$release = Invoke-RestMethod 'https://api.github.com/repos/bai-ting-chen/acad-mcp-release/releases/latest'
$assets = @($release.assets | Where-Object { $_.name -like 'acad-mcp-v*.zip' })
if ($assets.Count -ne 1) { throw 'Expected one acad-mcp release zip.' }
$downloadFolder = Join-Path $env:TEMP ('acad-mcp-install-' + [Guid]::NewGuid().ToString('N'))
New-Item -ItemType Directory -Path $downloadFolder | Out-Null
$zipPath = Join-Path $downloadFolder $assets[0].name
Invoke-WebRequest -UseBasicParsing -Uri $assets[0].browser_download_url -OutFile $zipPath
$unpackedFolder = Join-Path $downloadFolder 'unpacked'
Expand-Archive -LiteralPath $zipPath -DestinationPath $unpackedFolder
```

3. Run the extracted installer without requesting interactive input:

```powershell
powershell -NoProfile -NonInteractive -ExecutionPolicy Bypass -File "$unpackedFolder\install.ps1"
$installExitCode = $LASTEXITCODE
```

4. Relay the installer's last lines: which client registrations succeeded, failed, or were skipped, and any next steps. If the exit code is nonzero, explain its message and resolve that problem before claiming installation succeeded.
5. Delete the download folder on success or failure. First verify its resolved path is the unique `acad-mcp-install-...` folder you created directly under `%TEMP%`, then remove that folder with `Remove-Item -LiteralPath $downloadFolder -Recurse -Force`. Preserve any unrelated files.
6. After success, tell the person: **Start AutoCAD 2024 (click Load Once), then start a new Claude Code / Codex session.** The current AI session does not gain the tools automatically.

## What you need

- Windows 64-bit and **AutoCAD 2024**. This is the AutoCAD version tested so far.
- **Claude Code or Codex**, installed and signed in. Claude Code is the tested client; Codex support is new.
- An internet connection for installation. The installer sets up uv and Python for your Windows user; you do not need developer tools or administrator access.

## Install

1. Download `acad-mcp-v0.16.0.zip` from the [Releases page](https://github.com/bai-ting-chen/acad-mcp-release/releases).
2. Extract the whole zip into a folder. Do not run the script from inside the zip.
3. Close AutoCAD. Your AI assistant can stay open.
4. Right-click `install.ps1` in the extracted folder and choose **Run with PowerShell**. Keep the other extracted files beside it.

If Windows prevents the script from running, open PowerShell in the extracted folder and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

If installation stops, read its last message, fix the problem, and run it again. An office network may block the downloads; ask your IT team if needed.

Start AutoCAD 2024 and open a drawing. When **Security - Unsigned Executable File** appears, click **Load Once**. This prompt can appear each time AutoCAD starts.

Read the installer's registration report. It uses the Claude Code CLI when available, otherwise Claude's user settings file, and registers with Codex when its CLI is on PATH. Other settings and servers are kept. If one client fails, the other successful registration stays in place.

Start a **new Claude Code or Codex session** after installation.

## First tries

Try these requests in your AI assistant:

- "ping acad-mcp"
- "What's in this drawing?"
- "Draw a 3 × 4 m rectangle on layer A-WALL. Check the drawing units first."

Use a practice drawing for your first tries. You can undo drawing edits in AutoCAD. Save your drawing when you want to keep the changes.

## Tools

| Tool | What it does |
|---|---|
| `ping` | Checks the connection and reports the acad-mcp version. |
| `autocad_instances` | Lists AutoCAD windows with the helper loaded. |
| `drawing_info` | Reads drawing units, layers, layouts, object counts, and file details. |
| `drawing_new` | Starts a blank drawing. |
| `drawing_open` | Opens a drawing by its full file path. |
| `drawing_close` | Closes a drawing, asking before discarding unsaved work. |
| `drawing_extents` | Finds the main drawing area and objects far away from it. |
| `query_entities` | Finds drawing objects by layer, type, or area. |
| `view_capture` | Shows Claude a picture of the whole drawing, an area, or the current view. |
| `force_flatten` | Projects supported geometry throughout the drawing onto Z=0, including inside blocks. |
| `scale1d` | Scales drawing objects along one direction. |
| `run_lisp` | Draws and edits using AutoLISP; a successful script is one Undo step. |

## Feedback

Send feedback **directly to Bai-Ting Chen**, by email or Teams. Quote **version 0.16.0** (or ask Claude to `ping` and copy the version), what you asked for, and what happened. Include a screenshot if useful. Avoid sending project details you are not allowed to share. Feedback is not collected through public GitHub Issues.

## Update

Ask your assistant to install the latest release again, or download and extract the newer release zip manually. Close AutoCAD, then run the new folder's `install.ps1`. It replaces the previous version. Start AutoCAD, choose **Load Once**, and start a new Claude Code or Codex session.

## Uninstall

Close AutoCAD. Open PowerShell in the extracted release folder and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\install.ps1 -Uninstall
```

This removes the acad-mcp registrations from the available clients, server, and AutoCAD helper. Other client settings and your drawings stay in place. Keep the client CLIs available on PATH so their registrations can be removed. uv and its downloaded Python stay installed for other programs that may use them. You may delete the extracted release folder afterward.
