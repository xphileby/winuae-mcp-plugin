# winuae-mcp — Claude Code plugin

Drive a real Amiga from Claude. This plugin bundles:

- **`bin/winuae64.exe`** — WinUAE (x64, optimized) with an embedded MCP/JSON-RPC
  server (~46 tools: input synthesis, screenshots, CPU/memory inspection,
  breakpoints, serial I/O, disk & save-state management, machine control).
- **`server/mcp-winuae-bridge.py`** — a resilient stdio↔TCP relay registered as
  the `winuae` MCP server. Start Claude and the emulator in any order; the
  server stays available and reports emulator reachability via the
  `winuae_status` tool.
- **`skills/winuae`** — a skill teaching Claude the operating patterns for
  gaming, programming/testing, and 68k debugging on the emulated Amiga.

## Install

```
/plugin marketplace add xphileby/winuae-mcp-plugin
/plugin install winuae-mcp@winuae-mcp
```

Requirements: Windows x64, Python 3.8+ available as `py` (the Windows
launcher).

## Use

You must supply your own Amiga **Kickstart ROM** and a `.uae` config (ROMs are
copyrighted; none are bundled). Then either ask Claude to launch the emulator,
or run it yourself:

```powershell
& "$env:USERPROFILE\.claude\plugins\<...>\winuae-mcp\bin\winuae64.exe" -f my.uae
```

The emulator listens on TCP 7843 (all interfaces, **no auth** — see Security).
Ask Claude things like:

- "Boot the Workbench HDF and open the Prefs window with the mouse"
- "Insert game.adf, hard-reset, turbo through loading, screenshot the title"
- "Mount C:\proj\build as a volume and run mytool from a Shell, capture SER: output"
- "Set a breakpoint at 0xF8159C and show me the registers when it hits"

## Security

The embedded server binds `0.0.0.0:7843` without authentication. Anyone who
can reach the port controls the emulator (input, memory, files visible to the
guest). Use on trusted networks, or firewall the port / rebuild with loopback
binding.

## Source & license

WinUAE is GPL-licensed; this distribution includes a modified build
("mcpbridge"). Complete corresponding source for the included binary:
https://github.com/xphileby/WinUAE-mcpbridge (fork of
https://github.com/tonioni/WinUAE, base tag 6030). If the source repo is not
accessible to you, request access via an issue here and a source archive will
be provided.

Plugin scaffolding (relay, skill, manifests): GPL-2.0-or-later, same as the
emulator.
