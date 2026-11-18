---
name: adb-wifi-watchdog
description: Make a phone switch wireless debugging back on by itself (after a reboot, a Wi-Fi drop, or Android turning it off) with a Tasker profile that checks it's on the home network without needing Location. Use on "wireless debugging keeps turning off", "I keep losing ADB to my phone", "auto-enable wireless debugging", "keep adb wifi on".
---

# tasker-adb: adb-wifi-watchdog

Android turns wireless debugging off when Wi-Fi drops, and after a reboot, so a host that manages the
phone over wireless ADB loses it until someone flips the switch. This skill sets up Tasker on the
phone to flip it back on when it's at home. Build it with skill `drive-tasker`.

## Needs

- Tasker, with `WRITE_SECURE_SETTINGS` granted over adb (once, survives reboots):
  `adb shell pm grant net.dinglisch.android.taskerm android.permission.WRITE_SECURE_SETTINGS`
- A **home check URL** on the LAN: a page only the user's network serves, plus a **marker** string
  in it (for example a self-hosted wiki's title). Ask the user which; don't invent one. A private IP
  alone isn't proof of home, because other networks reuse `10.0.0.x` and `192.168.1.x`.
- The phone already paired with the host (pairing survives toggles and reboots).

## Why not the obvious routes

- Tasker `Wifi Connected` state: needs Location on, and never fired with Location off, with or
  without an SSID/IP filter (Android 16, Tasker 6.7.9).
- Apps that trust networks by SSID (e.g. KeepADB) can't read the SSID with Location off either
  (logs `identity_unavailable`).
- Users often keep Location off for battery or privacy, so this design doesn't rely on it.

## The profile

Profile: Time context, **Every 10 minutes**, From/To unticked. Task (in this order):

1. Custom Setting: Type `Global`, Name `adb_wifi_enabled`, Value empty, Read Setting To `%adbw`
2. Stop, If `%adbw` ~ `1` (already on: costs one settings read)
3. Wait 5 seconds
4. HTTP Request: GET `<home URL>`, timeout 10 s, Continue Task After Error ticked
5. Custom Setting: Type `Global`, Name `adb_wifi_enabled`, Value `1`, If `%http_data` ~ `*<marker>*`

Battery cost: one settings read every 10 minutes, plus the HTTP request only when debugging is off.

## Host side

Wireless debugging comes back on a **new random port**. The host has to find it: scan the phone's
30000-49999 range for the adb port (`nmap`), or use an autoconnect timer such as `adb-autoconnect` in
the `adb-ops` plugin. Optional: once connected, `adb tcpip 5555` pins a fixed port that survives
further toggles until reboot. That worked over wireless on one Android 16 phone and failed on the same
model a month earlier, so only do it where tested. With the watchdog in place, a failed pin recovers
on its own.

## Verify

1. `adb shell settings put global adb_wifi_enabled 0` (a `tcpip 5555` connection survives this).
2. Press Play on the task: within ~15 s, `settings get global adb_wifi_enabled` should read `1`.
3. Leave Tasker, set it to 0 again, and wait up to 10 minutes for the timer to set it back to 1.
4. Note the result with the date and the phone model.

Validated 2026-11-18 on an Android 16 phone with Tasker 6.7.9: Play restored the setting in under 15 s; with Tasker closed, the timer restored it about 8 minutes after it was switched off.
