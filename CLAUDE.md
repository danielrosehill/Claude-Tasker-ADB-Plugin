# Claude-Tasker-ADB-Plugin (plugin `tasker-adb`)

Public plugin. **No personal details in this repo**: no names, phone models tied to a person, hostnames,
LAN IPs, URLs of anyone's home services, or account ids. Examples use placeholders (`<home URL>`,
`192.0.2.10`). Grep for them before every push.

- Release tier: **small**. Tags and titles are `v1.0`, `v1.1`, … (a breaking change goes to `v2.0`).
  `plugin.json` `version` uses `1.N.0`, matching the marketplace entry in
  `danielrosehill/Claude-Code-Plugins`.
- Skills live flat in `skills/`; the screen helper is `scripts/phone-ui`.
- Every gotcha in `skills/drive-tasker/SKILL.md` was hit for real. Add new ones with the Tasker version
  and Android version, and record what didn't work as well as what did.
