# Executive Meeting Guardian skills

Agent skills for Cisco Cloud Control AI Canvas (GitHub Skills integration) for executive meeting rooms
(Room Bar, Board Pro, Desk Pro) used for Webex and Microsoft Teams meetings.

| Path | What it is |
| --- | --- |
| `skills/executive-meeting-guardian/SKILL.md` | Readiness assessment: procedure, remediation policy, report format |
| `skills/executive-meeting-guardian/executive-rooms.policy.yaml` | The customer readiness policy |
| `skills/executive-meeting-guardian-org-rules/SKILL.md` | Canada Collab org rules for self-heal: status-change triggers (connected -> connected with issues / disconnected), admins, Webex space, Accept / Decline with 5 second auto-accept, decline suppression |
| `skills/executive-meeting-guardian-org-rules/org-rules.json` | Proposed rule values used by the learning skill. Self-heal itself reads `emg_org_rules` in Cloud Control, not this file |
| `skills/executive-meeting-guardian-learning/SKILL.md` | Learn from self-heal history and propose rule changes as pull requests |
| `integrations/guardian-github.openapi.yaml` | The `guardian-github` Canvas integration: read rules and history, open rule pull requests |

## Self-heal and this repo

Self-heal runs entirely in Cisco Cloud Control (Control Hub status watch for every device, no macro, Webex cards,
ServiceNow) and reads its rules from the Cloud Control variable `emg_org_rules`. Nothing in this repository is
read at run time; the skills here explain the rules and the rooms to AI Canvas. The learning skill and the
`guardian-github` integration are kept for experiments and are not part of the self-heal path.
