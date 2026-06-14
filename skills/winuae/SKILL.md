---
name: winuae
description: Operate the WinUAE Amiga emulator through the winuae MCP server. Use whenever the user wants to run, play, test, debug, or automate anything on an emulated Amiga - games, AmigaOS, their own programs, disk images, or reverse engineering. Covers launching the emulator, screen-coordinate input, save states, mounting host folders, the 68k debugger, and serial I/O.
---

# Driving the Amiga emulator

The `winuae` MCP server exposes ~46 tools controlling a live WinUAE instance.
The MCP relay is always available, but the **emulator itself must be running**
for tools to act.

## Before anything else

1. Call `winuae_status`. If `connected: false`, the emulator is not running.
2. Launch it (the binary ships with this plugin):

   ```powershell
   & "<plugin-root>/bin/winuae64.exe" -f "path\to\config.uae"
   ```

   `<plugin-root>` is this plugin's install directory (the directory
   containing `bin/`). The user must supply their own `.uae` config and
   Kickstart ROM — ROMs are copyrighted and not bundled. A working config
   needs at least: `kickstart_rom_file`, a boot medium (`floppy0=` path to an
   .adf, or `hardfile2=`/`filesystem2=` for HDF/directory), and `use_gui=no`
   for headless start.
3. Wait for boot: `wait_for_idle {timeout_ms: 15000, quiet_ms: 300}` then
   `screenshot` to see where you are.

The MCP listener binds port 7843. First launch may show a Windows Firewall
prompt the user has to accept.

## Seeing and pointing

- `screenshot {scale:"native"}` returns the Amiga screen at native resolution.
  **Its pixels are the same coordinate space as the pointer tools** (Intuition
  screen pixels, (0,0) = top-left of the Amiga screen — note the screenshot
  includes a small overscan border around the screen, typically ~(54,28); use
  `get_pointer_pos` to calibrate if precision matters).
- `mouse_move {x, y}` is absolute and closed-loop (self-correcting, works in
  windowed/maximized/fullscreen). It BLOCKS until the pointer arrives.
  `get_pointer_pos` returns where Intuition thinks the pointer is.
- `mouse_click {button:0, count:2}` double-clicks (opens icons). Clicks are
  vsync-paced; they execute after any queued motion.
- Workflow for "click the X icon": screenshot → locate X visually → convert to
  guest coords → `mouse_move` → `mouse_click` → screenshot to verify.

## Typing

- `type_text` and `key_press` are serialized and blocking — call them
  back-to-back, no delays needed. `type_text` is US-keyboard ASCII; newline
  acts as Return.
- `key_press {key:"e", modifiers:["RAmiga"]}` = Amiga hotkeys (RAmiga+E opens
  Workbench's Execute Command dialog — the fastest way to run a one-liner).
- Open a Shell: `key_press RAmiga+E`, `type_text "newshell"`, `key_press
  Return`. Then type commands into the Shell with `type_text`.
- **Full key reference** (every named key, the numeric keypad, modifiers, and
  the raw `"0xNN"` form): see [keycodes.md](keycodes.md). Keys accept a
  symbolic name (e.g. `"f5"`, `"kp7"`, `"help"`, `"left"`) or a raw Amiga
  rawkey code as a hex string.

## Gaming

- Insert disks: `disk_insert {drive:0, path:"C:\\games\\game.adf"}` (df0:),
  `disk_eject`, `disk_list`. Reset to boot the disk: `reset {kind:"hard"}`.
- Joystick input: `inject_event` with the built-in input events, e.g.
  `{name:"JOY2_FIRE_BUTTON", value:"1"}` press / `"0"` release, and
  `JOY2_LEFT`, `JOY2_RIGHT`, `JOY2_UP`, `JOY2_DOWN` (port 2 is the usual
  joystick port; port 1 = mouse).
- Save-scumming: `save_state {path}` / `load_state {path}` snapshot the whole
  machine. (Known issue: after load_state the display can need a
  `reset {kind:"soft"}` or a moment to redraw.)
- `set_speed {mode:"turbo"}` fast-forwards through loading screens;
  `{mode:"balanced"}` restores normal speed. `pause`/`resume` freeze the game.
- `screenshot` + `wait_for_pixel`/`wait_for_region_change` synchronize on
  visual events (title screen appeared, level loaded).

## Programming & testing your own software

- `mount_dir {path:"C:\\myproject\\build"}` mounts a host folder as a guest
  volume (icon appears on Workbench within ~1s). Compile on the host,
  cross-build for m68k, then run the binary in the guest from that volume —
  no HDF rebuilding.
- Run it: open a Shell and `type_text "volumename:myprog"` + Return, or use
  RAmiga+E Execute.
- Serial I/O works with no host serial port: guest writes to `SER:` are
  captured by `serial_read`; `serial_write` feeds the guest's serial input
  (readable in the guest via `SER:`). Great for test harness output:
  have the program print results to SER: and assert on `serial_read`.
- `get_config`/`set_config` read/tune emulator options at runtime (exact .uae
  option names). `set_config {line:"cpu_speed=max"}` etc.

## Debugging & reverse engineering

- `breakpoint_set {addr:"0x..."}`: on hit the CPU parks at the exact PC and a
  `notifications/breakpoint` arrives. While stopped: `get_cpu_state`,
  `memory_read`, `disassemble`, and `debug` all work; queued tools
  (screenshot, input) time out until you `breakpoint_continue`.
  `breakpoint_clear` while stopped auto-resumes. Beware breakpoints on hot
  loops: they re-hit immediately after continue.
- `get_cpu_state` = D0-D7/A0-A7/PC/SR. `memory_read`/`memory_write` take
  `addr` as a "0xNN" string; data is base64. ExecBase pointer lives at 0x4.
- `find_in_memory {pattern_b64, start, end}` scans for byte patterns.
- `debug {command:"..."}` is a passthrough to WinUAE's full debugger CLI
  (`r` registers, `m addr` memory dump, `d addr` disassembly, `w` watchpoints
  and much more) — use it for anything without a dedicated tool.

## Notifications you may receive

`notifications/breakpoint` (hit, with pc), `notifications/screen_mode_changed`,
`notifications/paused|resumed|reset|config_changed|mousehack_changed`, and
from the relay: `notifications/winuae_connected|winuae_disconnected`.

## Gotchas

- Tools that need the emu thread (screenshot, input, mount_dir...) time out
  while the CPU is parked at a breakpoint or the emulator is paused — that is
  expected; resume first.
- `audio_record` returns an error if the host has no active sound device.
- `quit` cleanly exits the emulator; `winuae_status` then reports
  `connected:false` until it is relaunched.
- If a tool call returns "WinUAE is not running", relaunch the emulator
  (see top) — the MCP connection recovers automatically.
