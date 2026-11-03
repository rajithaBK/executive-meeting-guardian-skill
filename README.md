# Executive Meeting Guardian skill

Agent skill for Cisco Cloud Control AI Canvas (GitHub Skills integration). It tells the assistant how to assess
executive meeting rooms (Room Bar, Board Pro, Desk Pro) for Webex and Microsoft Teams readiness, how to propose the one
permitted remediation, and when to open or update a ServiceNow incident.

- `skills/executive-meeting-guardian/SKILL.md`: the skill.
- `skills/executive-meeting-guardian/executive-rooms.policy.yaml`: the customer readiness policy the skill refers to.

The skill drives these Cloud Control workflows: `COLLAB - Assess Room Readiness`,
`COLLAB - Approve and Restore Audio` and `COLLAB - Verify Room Recovery`. Changes always need an approval task and a
verified readback, and a room is never changed while it is in a meeting.
