# Executive Meeting Guardian skills

Agent skills for Cisco Cloud Control AI Canvas (GitHub Skills integration) for executive meeting rooms
(Room Bar, Board Pro, Desk Pro) used for Webex and Microsoft Teams meetings.

| Path | What it is |
| --- | --- |
| `skills/executive-meeting-guardian/SKILL.md` | Readiness assessment for one device: procedure, remediation policy, report format |
| `skills/executive-meeting-guardian/executive-rooms.policy.yaml` | The readiness policy (thresholds and checks; no static room list) |
| `skills/executive-meeting-guardian-org-rules/SKILL.md` | Org rules for self-heal: status-change triggers (connected -> connected with issues / disconnected), admins, Webex space, Accept / Decline with 5 second auto-accept, decline suppression, device input |
| `skills/executive-meeting-guardian-org-rules/org-rules.json` | Proposed rule values used by the learning skill. Self-heal itself reads `emg_org_rules` in Cloud Control, not this file |
| `skills/executive-meeting-guardian-briefings/SKILL.md` | Cross-device questions answered from live Control Hub, Cloud Control and ServiceNow data |
| `skills/executive-meeting-guardian-learning/SKILL.md` | Learn from self-heal history and propose rule changes as pull requests |
| `integrations/guardian-github.openapi.yaml` | The `guardian-github` Canvas integration: read rules and history, open rule pull requests |

Nothing here names a room or a device. Device names come from Control Hub at run time.

## Self-heal and this repo

Self-heal runs entirely in Cisco Cloud Control (Control Hub status watch for every device, no macro, Webex cards,
ServiceNow) and reads its rules from the Cloud Control variable `emg_org_rules`. Nothing in this repository is
read at run time; the skills here explain the rules and the rooms to AI Canvas. The learning skill and the
`guardian-github` integration are kept for experiments and are not part of the self-heal path.

## Questions to ask in AI Canvas

Each answer combines live Control Hub data with this org's readiness policy, self-heal rules and guardian history,
which Cloud Control and Control Hub do not hold on their own. Use the device names exactly as Control Hub shows them.

| Ask | What it uses |
| --- | --- |
| "Which room devices are not meeting-ready right now, and which of those can the guardian fix on its own?" | Control Hub status and error codes, org rules (fixable, read-only, no rule) |
| "Run a readiness check on HWS Board 75. Is it ready for a Teams meeting?" | Readiness workflow with Device = HWS Board 75; Teams gate (configuration is not proof) |
| "Why didn't the guardian fix HWS Room Bar?" | Control Hub lag, declined fixes, call state, read-only settings, re-notify window, account key health |
| "Which devices keep flipping between Connected and Connected with issues today, and why?" | Guardian cards and per-device status; ultrasound interference between nearby devices |
| "Which devices drift from our readiness policy?" | Readiness checks per device: volume range, auto answer, default volume, standby |
| "Is it safe to change HWS Desk Pro G2 right now?" | Fresh active-call read, open proposal, self-heal lock |
| "What did the guardian fix today, how long did each fix take, and what is still open in ServiceNow?" | Decisions, verified heals, incidents per device |
| "Which devices are offline, since when, and do they have an incident?" | Control Hub last seen, `emg-` incidents per device |
