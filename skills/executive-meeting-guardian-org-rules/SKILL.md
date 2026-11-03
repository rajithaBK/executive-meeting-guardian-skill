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

These are the customer's rules, not Cisco defaults. Self-heal runs entirely in
Cisco Cloud Control and reads its rules from the global variable
`emg_org_rules` (admins, space, rooms, triggers, fixes, approval window). It
does not read this repository; this skill explains the rules to AI Canvas.

Use this skill together with `executive-meeting-guardian`, which covers the
readiness procedure and report format.

## Org

| Rule | Value |
| --- | --- |
| Organization | Canada Collab |
| Admins (only people who can accept, decline or use the control panel) | rajithak@canadacollab.ca, rkanthet@cisco.com |
| Webex space for all Guardian messages | Executive Meeting Guardian |
| Devices in scope | Every room device (Room Bar, Board Pro, Desk Pro, Room Kit, Desk) and phone in the org. Peripherals (navigators, cameras, microphones, monitors) follow their room device |

Anyone who is not an admin and presses a Guardian button gets a
"Not authorized" reply and nothing changes.

## What triggers self-heal

Self-heal starts when a device's Control Hub status changes:

| Change | What happens |
| --- | --- |
| Connected -> **Connected with issues** | Self-heal proposal for that device in the space; auto-accepted after 5 s; fix applied and verified |
| Connected -> **Disconnected** | "Device disconnected" card and a ServiceNow incident for that device (a disconnected device cannot be healed remotely) |
| Disconnected -> Connected | "Device back online" card and a work note on its incident |

No macro is used. `COLLAB - Watch Control Hub Status` reads the Control Hub
status of every room device and phone every 20 seconds. Control Hub shows a
change 40 seconds to about 3 minutes after it happens, so a proposal usually
appears 15 seconds to 3 minutes after the change. Devices that were already
disconnected or connected with issues when the watch started do not raise
anything until they change again. Changed devices are queued and
`COLLAB - Self-Heal Room` handles them one at a time. An issue already being
handled, or declined for that device, is never proposed twice.

Volume or setting changes that do not change the Control Hub status do not
trigger self-heal; the readiness check and `COLLAB - Approve and Restore
Audio` cover those.

## Self-heal fixes

| Rule id | Diagnostic (Control Hub error code) | Fix |
| --- | --- | --- |
| `ultrasound_disabled` | `UltrasoundConfigSettings` | `Audio.Ultrasound.MaxVolume` 70 |
| `internal_speaker_disabled` | `AudioInternalSpeakerDisabled` | `Audio.Output.InternalSpeaker.Mode` On |

Any other warning, error or critical diagnostic is not healed. It is reported
in the space and opened as a ServiceNow incident by the workflow. The same
happens when the setting is read-only on that device (for example a Room Bar
in Microsoft Teams Rooms mode reports `CONFIG_MANAGED_BY_DIFFERENT_AUTHORITY`)
and for phones, which report their Control Hub error codes.

Guard rails, all mandatory:

- Never change a device that is in a call, or whose call state cannot be read.
  The check is repeated immediately before the change; if busy, the heal is
  deferred, the space is told why, and it is proposed again after the call.
- One device at a time (queue plus a lock that expires after 120 seconds).
- Verify 6 seconds after the change with a fresh device read. Report PASS only
  when the device returns the expected value.
- If the fix cannot be verified, open a ServiceNow incident (one per device;
  repeats add a work note).
- Never reboot, factory reset, erase configuration or run arbitrary commands.
- Only the fixes in the table above. Anything else needs the approval workflow
  `COLLAB - Approve and Restore Audio` or a human.

## Approval flow

1. A device changes to "connected with issues" (for example ultrasound set to
   0 in Control Hub). When Control Hub shows it, `COLLAB - Watch Control Hub
   Status` queues the device and `COLLAB - Self-Heal Room` starts.
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

When asked why a device was not healed, check in this order: whether
Control Hub has shown the change yet, a decline in `emg_heal_suppressed`, an
active call, a read-only setting, then an unverified result or an incident.

## Triggers

From AI Canvas, run these Cloud Control workflows (Workflows in the composer,
or ask in natural language):

| Ask | Workflow |
| --- | --- |
| "Check the Toronto executive room" | `COLLAB - Assess Room Readiness` |
| "Verify the room recovered" | `COLLAB - Verify Room Recovery` |
| "Restore the audio, with approval" | `COLLAB - Approve and Restore Audio` |

From the Webex space, the admin control panel card has: Readiness check,
Verify, Simulate 'connected with issues', Show org rules and Reset declined
fixes. Simulate is for demos on test devices and is refused while the room is
in a call.

Automatic triggers:

- The **EMG Control Hub status watch** rule runs
  `COLLAB - Watch Control Hub Status` every minute (three reads, 20 s apart).
- The watch hands changed devices to `COLLAB - Self-Heal Room` through the
  **Executive Meeting Guardian control** webhook (rule **EMG self-heal
  (Control Hub status)**).
- Webex button presses (`attachmentActions`) start
  `COLLAB - Guardian Card Actions`.
- The **Executive Room Readiness Schedule** runs the readiness check on
  weekdays at 7:00 AM and every 2 hours to 5:00 PM Eastern.

## Changing the rules

An admin changes `emg_org_rules` in Cloud Control (or `policies/executive-rooms.yaml`
in the main repo, then regenerates). Changes apply to the next event; no
re-import is needed.

## Messages

All Guardian messages are adaptive cards: a header with the room and time, a
one-line outcome, and facts (before, after, who decided, how long from
detection to heal). Keep replies in the proposal's thread so the space stays
readable. Do not paste raw JSON or tokens into the space.
