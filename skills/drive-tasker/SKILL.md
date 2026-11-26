---
name: drive-tasker
description: Build or edit Tasker profiles, tasks and actions on an Android phone by driving Tasker's own editor (mobile-mcp element list and taps, ADB fallback), and test them. Use on "make a Tasker profile", "add a Tasker task", "change my Tasker action", "set up Tasker to do X", or when an automation needs Tasker and the user isn't at the phone.
---

# tasker-adb: drive-tasker

Tasker has no API for creating profiles from outside, and hand-written import XML is easy to get
wrong (action codes and arg layouts aren't documented). Driving Tasker's editor is reliable:
read the screen, tap, type, then read the screen again. It works while the user watches, and they can
take over at any point.

**Driver: a mobile-mcp server first** ([mobile-next/mobile-mcp](https://github.com/mobile-next/mobile-mcp)),
when one is connected to the phone. Its tools act on the accessibility tree: `mobile_list_elements_on_screen`
(text, resource id, coordinates, `ref`), `mobile_click_on_screen_at_coordinates` (x,y or a `ref`),
`mobile_long_press_on_screen_at_coordinates`, `mobile_type_keys`, `mobile_swipe_on_screen`,
`mobile_press_button` (BACK, HOME, ENTER), `mobile_launch_app`, `mobile_get_foreground_app`. Take a
screenshot only when the element list comes back empty (canvas UIs).

Fallback, and for adb-only jobs: `${CLAUDE_PLUGIN_ROOT}/scripts/phone-ui` (run it with no arguments for
usage). It needs `adb` connected to the phone (USB or wireless) and `python3` on the machine running it.
If adb lives on another host, copy the script there and run it over ssh. Set `ADB_SERIAL=<ip:port|serial>`
when more than one device is attached. Keep it (or plain `adb`) for what mobile-mcp has no tool for:
`clear` (select-all uses `input keycombination`), `checked` (switch state), `pm grant`, `settings`,
`dumpsys`, explicit intents (`am start`). `uiautomator dump` can return stale XML on some screens;
prefer mobile-mcp's element list when both are available.

**Secrets** (passwords, tokens in HTTP Request actions): type them with `adb shell input text` from the
machine holding the secret, never through mobile-mcp, and don't read the screen back through mobile-mcp
while the secret shows in an unmasked field. Confirm which field has focus before typing (move with
`KEYCODE_TAB`).

## Loop

1. Open Tasker: `mobile_launch_app` `net.dinglisch.android.taskerm`, or `phone-ui start
   net.dinglisch.android.taskerm`. Not `monkey`: Tasker has a second launcher entry ("Secondary App")
   that monkey can pick, and it opens a different screen. Check with `mobile_get_foreground_app`.
2. List the elements (`mobile_list_elements_on_screen`, fallback `phone-ui dump`), then act on the
   coordinates or ref it gives. Never tap from memory: dialogs shift the layout.
3. After each action, list the elements again and check the result before the next step.

## Screens and controls (Tasker 6.7, phone in portrait)

| Where | What |
|---|---|
| Main | tabs `PROFILES` / `TASKS` / `SCENES` / `VARS` (`tab_text`), each profile's switch is `button_enabled`, add is `button_add` (bottom right) |
| Profile | context row `context_text_one` (tap to edit the condition), task row `task_text_one_line_one` |
| New profile | `button_add` → tip dialog (`CREATE`) → context type list (Application, Day, Event, Location, State, Time) → editor → back → pick a task |
| Task Edit | actions `actionnum_text` / `actionname_text` with `argN_text` summaries; `button_add_action` (bottom right); Play `image_left_one` (bottom left); `Cancel` in the top bar discards every edit since the task was opened |
| Add action | category list with a `filter_text` box: tap it, type the action name (`Custom Setting`, `HTTP Request`, `Wait`, `Stop`), tap the match |
| Action Edit | labelled `edittext` fields, `continue_on_error_checkbox`, condition `condition_add` → `lhs` / `op` / `rhs`; the top-left arrow (`Navigate up`) saves |

## Gotchas (each one cost a retry)

- **Tip dialogs** come up the first time you open most screens. Tap `STOP REMINDING` (`button3`) when
  it's offered, otherwise `OK`/`NO`.
- **The keyboard covers the lower fields.** After typing in a field, press BACK (`mobile_press_button`
  BACK or `phone-ui key 4`) to close the keyboard, then list the elements before tapping the next field.
  Otherwise the tap goes to the keyboard, and the next text ends up in the field you just filled. Press
  BACK only while the keyboard is open: with no keyboard up it leaves the screen, and can leave Tasker.
- **Replace a field's text** with tap → `phone-ui clear` → type. Typing into a non-empty field appends.
- **Number pickers** (Time context From/To) don't reliably take typed values. Untick From/To when
  they aren't needed: "Every N minutes" alone is accepted.
- **Reordering:** dragging doesn't work over `input`. Use cut and paste instead. Long-press an
  action to select it (the top bar shows Copy/Cut/Paste), Cut, then long-press the action that should
  come *after* it and Paste. **Paste inserts before the selected action.** Select one action per
  long-press and check the top bar each time; a missed selection pastes a duplicate group. If the
  list goes wrong, `Cancel` and redo.
- **New actions are appended at the end.** To put one first, add it, cut it, select action 1, paste.
- **Changes take effect only after leaving Tasker.** Back out of the main screen (BACK, then
  HOME) before testing a profile's trigger. Play inside Task Edit runs the task right away.
- **Check switches with `phone-ui checked button_enabled`**: the dump doesn't show switch state. Profiles
  can be off; an imported or old profile that "never works" may just be disabled.
- **Location-gated contexts:** Tasker's `Wifi Connected` state (with or without an SSID/IP filter)
  didn't fire with Location off (tested on Android 16). Use a Time context plus a
  reachability check in the task instead (see skill `adb-wifi-watchdog`).
- **Settings actions** (`Custom Setting`, global/secure) need `WRITE_SECURE_SETTINGS`. Grant it once:
  `adb shell pm grant net.dinglisch.android.taskerm android.permission.WRITE_SECURE_SETTINGS`.
- **The screen locks** on the phone's own timeout. The element list then only shows the lock screen. Ask
  the user to unlock, then launch Tasker again to return; Tasker keeps unsaved editor state.

## Verify

Test the result itself, not the editor. Set up the state the task should fix, press Play (or wait for
the trigger after leaving Tasker), then read the outcome over adb (`settings get …`, `dumpsys …`).
Report what you confirmed and what you didn't.
