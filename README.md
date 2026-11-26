# tasker-adb

Operate [Tasker](https://tasker.joaoapps.com/) on an Android phone from Claude Code. Claude reads the
screen as an element list, taps and types through Tasker's own editor, then tests the result. It drives
the screen through a [mobile-mcp](https://github.com/mobile-next/mobile-mcp) server when one is connected,
and falls back to ADB (`uiautomator` + `input`) through the bundled `phone-ui` helper.
No root, no plugins on the phone, no hand-written XML.

## Install

```
/plugin install tasker-adb --marketplace danielrosehill/Claude-Code-Plugins
```

## Skills

| Skill | Use |
|---|---|
| `drive-tasker` | Build or edit profiles, tasks and actions in Tasker's editor, then test them. Holds the screen map and the gotchas (keyboard covering fields, paste-before-selection, changes applying only after leaving Tasker, Location-gated contexts). |
| `adb-wifi-watchdog` | Recipe: Tasker switches wireless debugging back on by itself when the phone is at home, without needing Location. For anyone managing a phone over wireless ADB who keeps losing it. |

## Requirements

- Recommended: a mobile-mcp server connected to the phone, for reading the screen and tapping.
- `adb` connected to the phone (USB or wireless), and `python3` on the same machine.
- Tasker installed on the phone. Settings actions need `WRITE_SECURE_SETTINGS`:
  `adb shell pm grant net.dinglisch.android.taskerm android.permission.WRITE_SECURE_SETTINGS`
- Android 13+ for `phone-ui clear` (it uses `input keycombination`).

## Helper

`scripts/phone-ui` does `dump`, `checked`, `tap`, `long`, `swipe`, `text`, `clear`, `key`, `start`
and `focus`. Run it with no arguments for usage. Set `ADB_SERIAL` when more than one device is
attached.

## Cautions

It drives the real screen. Whoever holds the phone sees every tap, and if they touch it mid-run the
taps land somewhere else. Claude dumps the screen before each step for that reason. The phone's
screen timeout can lock it mid-edit.

## History

Written 2026-11-18 while building a wireless-debugging watchdog on an Android 16 phone with Tasker
6.7.9. Import XML and Tasker's Location-based Wi-Fi state were tried first and ruled out (see the skills).
