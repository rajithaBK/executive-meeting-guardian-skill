# Executive Meeting Guardian skills

Agent skills for Cisco Cloud Control AI Canvas (GitHub Skills integration) for executive meeting rooms
(Room Bar, Board Pro, Desk Pro) used for Webex and Microsoft Teams meetings.

| Path | What it is |
| --- | --- |
| `skills/executive-meeting-guardian/SKILL.md` | Readiness assessment: procedure, remediation policy, report format |
| `skills/executive-meeting-guardian/executive-rooms.policy.yaml` | The customer readiness policy |
| `skills/executive-meeting-guardian-org-rules/SKILL.md` | Canada Collab org rules for self-heal: admins, Webex space, Accept / Decline with 5 second auto-accept, decline suppression, triggers |
| `skills/executive-meeting-guardian-org-rules/org-rules.json` | **The enforced self-heal rules.** Cisco Cloud Control reads this file from `main` on every room event |
| `skills/executive-meeting-guardian-learning/SKILL.md` | Learn from self-heal history and propose rule changes as pull requests |
| `integrations/guardian-github.openapi.yaml` | The `guardian-github` Canvas integration: read rules and history, open rule pull requests |

## How a rule change flows

1. Every self-heal outcome is written by Cloud Control to the private repo `rajithaBK/emg-guardian-history`.
2. AI Canvas (learning skill + `guardian-github`) finds patterns and opens a pull request that edits `org-rules.json`.
3. A human reviews and merges. Canvas cannot merge.
4. On the next room event Cloud Control loads the new rules, checks them against safety bounds, posts
   "Org rules updated from GitHub" in the Webex space, and applies them. Out-of-bounds rules are rejected and the
   last known good rules stay in force.

## Safety bounds enforced by Cloud Control

- `volume`: integers, 20 <= min < target < max <= 90
- Settings in `baseline` and `diagnostic_heals`: `Audio.DefaultVolume` (0-100), `Audio.Ultrasound.MaxVolume` (0-90),
  `Conference.AutoAnswer.Mode` (On/Off), `Standby.Control` (On/Off), `Conference.DefaultCall.Protocol` (Auto/Sip/Spark/H323)
- `lock_seconds`: 30-600
- Not editable from GitHub: admins, Webex space, rooms, the 5 second approval window, verification timing, and
  "never change a room in an active call".
