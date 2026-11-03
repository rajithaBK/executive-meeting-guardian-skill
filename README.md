# Executive Meeting Guardian skills

Agent skills for Cisco Cloud Control AI Canvas (GitHub Skills integration) for executive meeting rooms
(Room Bar, Board Pro, Desk Pro) used for Webex and Microsoft Teams meetings.

| Path | What it is |
| --- | --- |
| `skills/executive-meeting-guardian/SKILL.md` | Readiness assessment: procedure, remediation policy, report format |
| `skills/executive-meeting-guardian/executive-rooms.policy.yaml` | The customer readiness policy |
| `skills/executive-meeting-guardian-org-rules/SKILL.md` | Canada Collab org rules for self-heal: status-change triggers (connected -> connected with issues / disconnected), admins, Webex space, Accept / Decline with 5 second auto-accept, decline suppression |
| `skills/executive-meeting-guardian-org-rules/org-rules.json` | Proposed rule values used by the learning skill. Self-heal itself reads `emg_org_rules` in Cloud Control, not this file |
| `skills/executive-meeting-guardian-customer-context/SKILL.md` | How to apply Canada Collab's business knowledge on top of live device evidence |
| `skills/executive-meeting-guardian-customer-context/customer-context.yaml` | Rooms and tiers, fallbacks, partners and their join rules, event profiles, change freezes, Quebec French rule, security baseline, on-site support |
| `skills/executive-meeting-guardian-briefings/SKILL.md` | Playbooks for go/no-go, partner meeting, "what if", change safety, compliance, reliability and weekly digest questions |
| `skills/executive-meeting-guardian-learning/SKILL.md` | Learn from self-heal history and propose rule changes as pull requests |
| `integrations/guardian-github.openapi.yaml` | The `guardian-github` Canvas integration: read rules and history, open rule pull requests |

## Self-heal and this repo

Self-heal runs entirely in Cisco Cloud Control (Control Hub status watch for every device, no macro, Webex cards,
ServiceNow) and reads its rules from the Cloud Control variable `emg_org_rules`. Nothing in this repository is
read at run time; the skills here explain the rules and the rooms to AI Canvas. The learning skill and the
`guardian-github` integration are kept for experiments and are not part of the self-heal path.

## Questions to ask in AI Canvas

Control Hub and Cloud Control know the devices; these questions also need the customer's business
context, so they only work with the skills above.

| Ask | What makes it customer specific |
| --- | --- |
| "Will tomorrow's board meeting in the Toronto Boardroom work, and is the fallback room ready?" | Event profile extra checks, fallback room, notify list, 7-day guardian history |
| "Can we meet Northwind Bank on Teams from the CEO's room? What will their side do to us?" | Partner tenant rules: lobby on, anonymous join blocked |
| "Litware Capital sent a Zoom invite for the board observer call. Which room should I book?" | No room supports Zoom: the answer is "none" plus the workaround |
| "Fabrikam Quebec wants to meet in French next week. Which Montreal room is ready and compliant?" | Quebec French-language rule, partner language, Montreal support hours |
| "What happens if the boardroom fails 10 minutes before the board meeting? Write the script for the executive assistant." | Fallback, self-heal limits, on-site dispatch time, phone fallback |
| "Can I push a new volume baseline to the boardroom on Nov 25?" | Q4 fiscal close freeze; maintenance window |
| "Our 'MTR' rooms: are they really running Microsoft Teams Rooms like our standard says?" | Intended mode versus the RoomOS the devices report |
| "Which executive rooms break our security standard right now?" | Customer security baseline (auto answer, encryption, HTTPS, standby) |
| "Which rooms kept breaking this month, and what did it cost us in executive meeting risk?" | Guardian history and ServiceNow incidents weighted by business tier |
| "Give me next week's executive meeting room digest for the CEO's office." | Upcoming events, freezes, partner risk and open items, in plain language |
