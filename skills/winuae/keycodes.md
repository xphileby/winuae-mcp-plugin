# Amiga keyboard codes (rawkey) for `key_down` / `key_up` / `key_press`

The keyboard tools accept either a **symbolic name** (case-insensitive, table
below) or a **raw Amiga rawkey code** as a hex string like `"0x42"`. The code
is fed straight to the emulated keyboard, so anything in the 0x00–0x7F rawkey
space can be sent even if it has no name here.

`type_text` is the better choice for ordinary text; use these key tools for
control keys, modifiers, shortcuts (e.g. `key_press {key:"e",
modifiers:["RAmiga"]}`), games, and the numeric keypad.

## Letters

| key | code | | key | code | | key | code |
|---|---|---|---|---|---|---|---|
| a | 0x20 | | j | 0x26 | | s | 0x21 |
| b | 0x35 | | k | 0x27 | | t | 0x14 |
| c | 0x33 | | l | 0x28 | | u | 0x16 |
| d | 0x22 | | m | 0x37 | | v | 0x34 |
| e | 0x12 | | n | 0x36 | | w | 0x11 |
| f | 0x23 | | o | 0x18 | | x | 0x32 |
| g | 0x24 | | p | 0x19 | | y | 0x15 |
| h | 0x25 | | q | 0x10 | | z | 0x31 |
| i | 0x17 | | r | 0x13 | | | |

(Letters are unshifted/lowercase. For uppercase use `key_press` with
`modifiers:["LShift"]`, or just use `type_text`.)

## Main-row digits & symbols

| key | code | aliases |
|---|---|---|
| 1..9 | 0x01..0x09 | |
| 0 | 0x0a | |
| `` ` `` | 0x00 | backtick, grave |
| - | 0x0b | minus |
| = | 0x0c | equal |
| \ | 0x0d | backslash |
| [ | 0x1a | leftbracket |
| ] | 0x1b | rightbracket |
| ; | 0x29 | semicolon |
| ' | 0x2b | quote, apostrophe |
| , | 0x38 | comma |
| . | 0x39 | period |
| / | 0x3a | slash |

Shifted symbols (`! @ # $ % ^ & * ( ) _ + { } : " < > ?` etc.) are produced by
holding `LShift` with the base key, or simply via `type_text`.

## Action / editing keys

| name | code | name | code |
|---|---|---|---|
| space | 0x40 | escape, esc | 0x45 |
| backspace | 0x41 | delete, del | 0x46 |
| tab | 0x42 | help | 0x5f |
| return, enter | 0x44 | | |

## Cursor keys

| name | code |
|---|---|
| up | 0x4c |
| down | 0x4d |
| right | 0x4e |
| left | 0x4f |

## Function keys

| name | code | name | code |
|---|---|---|---|
| f1 | 0x50 | f6 | 0x55 |
| f2 | 0x51 | f7 | 0x56 |
| f3 | 0x52 | f8 | 0x57 |
| f4 | 0x53 | f9 | 0x58 |
| f5 | 0x54 | f10 | 0x59 |

(Classic Amiga keyboards have F1–F10 only.)

## Numeric keypad

The Amiga keypad's rawkey codes differ from the main row. On the US keymap the
digit keys produce the same characters as the main-row digits.

| name | code | aliases |
|---|---|---|
| kp0 | 0x0f | numpad0 |
| kp1 | 0x1d | numpad1 |
| kp2 | 0x1e | numpad2 |
| kp3 | 0x1f | numpad3 |
| kp4 | 0x2d | numpad4 |
| kp5 | 0x2e | numpad5 |
| kp6 | 0x2f | numpad6 |
| kp7 | 0x3d | numpad7 |
| kp8 | 0x3e | numpad8 |
| kp9 | 0x3f | numpad9 |
| kpdot | 0x3c | kpperiod, numpaddot |
| kpenter | 0x43 | numericenter, keypadenter |
| kpminus | 0x4a | kpsub |
| kpdiv | 0x5c | kpdivide |
| kpmul | 0x5d | kpmultiply |
| kpadd | 0x5e | kpplus |
| kplparen | 0x5a | kpleftparen |
| kprparen | 0x5b | kprightparen |

## Modifiers

Use these in `key_press`'s `modifiers` array, or hold/release them yourself
with `key_down`/`key_up`.

| name | code | aliases |
|---|---|---|
| lshift | 0x60 | leftshift, shift |
| rshift | 0x61 | rightshift |
| capslock | 0x62 | caps |
| ctrl | 0x63 | control, lctrl |
| lalt | 0x64 | leftalt, alt |
| ralt | 0x65 | rightalt |
| lamiga | 0x66 | leftamiga, lmeta, lwin |
| ramiga | 0x67 | rightamiga, rmeta, rwin |

## Non-US keys

International keyboards have two extra keys with no US equivalent:

| name | code | note |
|---|---|---|
| nonus_hash | 0x2a | `#`/`~` key (UK, DE, ...) |
| nonus_backslash | 0x30 | `\`/`|` or `<`/`>` key next to Left Shift |

## Common shortcuts (examples)

- Reset (Ctrl-Amiga-Amiga): prefer the `reset` tool, but you can also
  `key_down ctrl`, `key_down lamiga`, `key_down ramiga`, then release.
- Workbench "Execute Command": `key_press {key:"e", modifiers:["RAmiga"]}`.
- Close window menu / shortcuts in apps: `key_press {key:"c",
  modifiers:["RAmiga"]}` (copy), `"x"` (cut), `"v"` (paste), etc.
- Break a Shell command: `key_press {key:"c", modifiers:["Ctrl"]}`.

## Sending an unnamed code

Any rawkey value works as a hex string regardless of this table, e.g.
`key_press {key:"0x5e"}` presses keypad `+`. Release bit handling is automatic;
you give the base code.
