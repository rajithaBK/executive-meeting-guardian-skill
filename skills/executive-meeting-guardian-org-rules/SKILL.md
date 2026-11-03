---
name: executive-meeting-guardian-org-rules
description: >
  Canada Collab org rules for Executive Meeting Guardian self-heal.
  Use when someone in AI Canvas asks what the org allows for executive
  rooms, wants to trigger a readiness check, verification or self-heal,
  asks why a room was (or was not) healed, or wants to change who can
  approve fixes, the thresholds, or the approval window.
---

# Executive Meeting Guardian: org rules (Canada Collab)

These are the customer's rules, not Cisco defaults. The enforced self-heal
rules (volume range, baseline, diagnostic fixes, lock) live in
`skills/executive-meeting-guardian-org-rules/org-rules.json` on `main` in this
repo. Cisco Cloud Control reads that file on every room event, checks it
against safety bounds, and keeps the last known good copy in the global
variable `emg_org_rules`. Identity (admins, space, rooms) and timing (the
5-second approval window, verify delay) stay in Cloud Control.

Use this skill together with `executive-meeting-guardian`, which covers the
readiness procedure and report format.

## Org

| Rule | Value |
| --- | --- |
| Organization | Canada Collab |
| Admins (only people who can accept, decline or use the control panel) | rajithak@canadacollab.ca, rkanthet@cisco.com |
| Webex space for all Guardian messages | Executive Meeting Guardian |
| Rooms in scope | Toronto Executive Room (Desk Pro G2) |

Anyone who is not an admin and presses a Guardian button gets a
"Not authorized" reply and nothing changes.

## Self-heal rules

Values below are policy 1.2. Always read `org-rules.json` for the values in force.

| Rule id | Detected when | Fix |
| --- | --- | --- |
| `audio_volume` | `Audio.Volume` outside 45–65 | `Audio.Volume.Set` Level 55 |
| `ultrasound_disabled` | Diagnostic `UltrasoundConfigSettings` (device shows "connected with issues") | `Audio.Ultrasound.MaxVolume` 70 |
| `configuration_drift` | A baseline setting differs: `Audio.DefaultVolume` 50, `Conference.AutoAnswer.Mode` Off, `Standby.Control` On | Restore the baseline value |

Any other diagnostic is not healed. It is reported in the space and opened as
a ServiceNow incident by the workflow.

Guard rails, all mandatory:

- Never change a room that is in a call, or whose call state cannot be read.
  The check is repeated immediately before the change; if busy, the heal is
  deferred and the space is told why.
- One self-heal per room at a time (60 second lock).
- Verify 6 seconds after the change with a fresh device read. Report PASS only
  when the device returns the expected value.
- If the fix cannot be verified, open a ServiceNow incident (one per room and
  failure signature; repeats add a work note).
- Never reboot, factory reset, erase configuration or run arbitrary commands.
- Only the fixes in the table above. Anything else needs the approval workflow
  `COLLAB - Approve and Restore Audio` or a human.

## Approval flow

1. The room macro reports the change within about two seconds and
   `COLLAB - Self-Heal Room` starts.
2. The first message is a **Self-heal proposal** adaptive card in the
   Executive Meeting Guardian space, with **Accept** and **Decline**.
3. If no admin answers within **5 seconds**, the fix is **auto-accepted** and
   applied.
4. **Accept** applies it straight away.
5. **Decline** means: do not heal, and do not propose this issue again. The
   issue is stored in `emg_heal_suppressed` and stays suppressed until an
   admin presses **Reset declined fixes** in the control panel.
6. The outcome (healed and verified, deferred, declined, or incident opened) is
   posted as a card in the proposal's thread.

When asked why a room was not healed, check in this order: a decline in
`emg_heal_suppressed`, an active call, the lock, then an unverified result
or an incident.

## Triggers

From AI Canvas, run these Cloud Control workflows (Workflows in the composer,
or ask in natural language):

| Ask | Workflow |
| --- | --- |
| "Check the Toronto executive room" | `COLLAB - Assess Room Readiness` |
| "Verify the room recovered" | `COLLAB - Verify Room Recovery` |
| "Restore the audio, with approval" | `COLLAB - Approve and Restore Audio` |

From the Webex space, the admin control panel card has: Readiness check,
Verify, Simulate volume drift, Simulate 'connected with issues', Show org
rules and Reset declined fixes. The two Simulate buttons are for demos on
test devices and are refused while the room is in a call.

Automatic triggers:

- Room macro events to the **Executive Meeting Guardian events** webhook
  start `COLLAB - Self-Heal Room` (self-heal within seconds).
- Webex button presses (`attachmentActions`) start
  `COLLAB - Guardian Card Actions`.
- The **Executive Room Readiness Schedule** runs the readiness check on
  weekdays at 7:00 AM and every 2 hours to 5:00 PM Eastern.

## Changing the rules

Rule changes are pull requests on `org-rules.json`; a human reviews and
merges. The `executive-meeting-guardian-learning` skill proposes them from
the self-heal history. After merge, the next room event loads the new rules
and the Webex space shows an "Org rules updated from GitHub" card. A change
that breaks the safety bounds is rejected, announced as "Org rules change
rejected", and the previous rules stay in force. Identity and timing changes
are made in Cloud Control by an admin.

## Messages

All Guardian messages are adaptive cards: a header with the room and time, a
one-line outcome, and facts (before, after, who decided, how long from
detection to heal). Keep replies in the proposal's thread so the space stays
readable. Do not paste raw JSON or tokens into the space.
