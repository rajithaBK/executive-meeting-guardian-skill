# Executive Meeting Guardian skill

Agent skill for Cisco Cloud Control AI Canvas (GitHub Skills integration). It tells the assistant how to assess
executive meeting rooms (Room Bar, Board Pro, Desk Pro) for Webex and Microsoft Teams readiness, how to propose the one
permitted remediation, and when to open or update a ServiceNow incident.

- `skills/executive-meeting-guardian/SKILL.md`: the skill.
- `skills/executive-meeting-guardian/executive-rooms.policy.yaml`: the customer readiness policy the skill refers to.
- `skills/executive-meeting-guardian-org-rules/SKILL.md`: the Canada Collab org rules for self-heal (admins, Webex
  space, thresholds, Accept / Decline with 5 second auto-accept, decline suppression, triggers).

The skill drives these Cloud Control workflows: `COLLAB - Assess Room Readiness`,
`COLLAB - Approve and Restore Audio`, `COLLAB - Verify Room Recovery`, `COLLAB - Self-Heal Room` and
`COLLAB - Guardian Card Actions`. Changes need an approval (the approval task, or an admin's Accept / auto-accept in the
Webex space for the self-heal fixes the org rules allow) and a verified readback, and a room is never changed while it is in a meeting.
