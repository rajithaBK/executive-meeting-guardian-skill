---
name: executive-meeting-guardian-briefings
description: >
  Answer cross-device questions about meeting readiness from live data only:
  which devices are not meeting-ready and which of those self-heal can fix,
  why a device was or was not healed, which devices keep flipping status and
  why, which devices drift from the readiness policy, whether a device is
  safe to change right now, what the guardian did today, and which devices
  are offline with or without an incident. Use for go/no-go, "why", drift,
  flapping, change-safety and daily summary questions.
---

# Executive Meeting Guardian: briefings

Every answer is built from live sources at the time of the question. Device
names, products and statuses come from Control Hub; never use a room name,
person, partner or event that is not in one of these sources.

| Source | Gives |
| --- | --- |
| Collaboration Control Hub agent | Every device: display name, product, type, connection status, error codes, last seen, workspace |
| `COLLAB - Assess Room Readiness` (input **Device**) | Fresh policy checks for one device: online, audio range, join method, Teams gate, cameras, baseline, diagnostics, active call |
| `COLLAB - Verify Room Recovery` (input **Device**) | Fresh re-check after a change, compared with the last remediation |
| Cloud Control variables | `emg_controlhub_status` (last status of every watched device and the self-heal queue), `emg_room_status` (per device: issues, open issues, last update), `emg_heal_decisions` (accept, decline, auto, with who and when), `emg_heal_suppressed` (declined fixes), `emg_heal_lock`, `emg_last_assessment`, `emg_last_remediation`, `emg_org_rules` |
| Webex space **Executive Meeting Guardian** | The proposal, result, disconnect and incident cards, with times |
| ServiceNow | Incidents whose `correlation_id` starts with `emg-<first 8 characters of the device uuid>-` |

Rules for every answer:

- Name devices exactly as Control Hub does, with the product:
  "HWS Board 75 (Cisco Board Pro 75)". Do not invent friendly room names.
- If a source is not available in this session, mark the dependent checks
  UNKNOWN and say which source was missing.
- Give times in US Eastern with the zone label (for example "11:14 AM EST").
- End with one verdict line (**GO**, **GO WITH CONDITIONS**, **NO-GO** or
  **UNKNOWN**) when the question asks about readiness, and with the single
  next action and who can take it.

## 1. What is not meeting-ready right now

Questions like "Which devices are not meeting-ready right now, and which of
those can the guardian fix on its own?"

1. List every room device (type `roomdesk`) from Control Hub that is not
   `connected`, with its status and error codes. Leave out peripherals
   (navigators, cameras, microphones, displays); they show up in their room
   device's error codes.
2. For each, map the error codes or diagnostics to the org rules in
   `emg_org_rules`:
   - in `diagnostic_heals`: self-heal fixes it automatically (say the setting
     and target value), unless the setting is read-only on that device;
   - not in the rules: report and ServiceNow incident, no automatic fix;
   - disconnected or offline: cannot be fixed remotely.
3. Add whether the fix is suppressed (`emg_heal_suppressed`) and whether an
   incident is open.

## 2. Why was (or wasn't) a device healed

Questions like "Why didn't the guardian fix HWS Room Bar?"

Check in this order and stop at the first that explains it:
1. Control Hub has not shown the change yet (it lags 40 seconds to about
   3 minutes), or the device never went back to `connected` before breaking
   again.
2. The fix was declined (`emg_heal_suppressed`).
3. The device was in a call (deferred; it is proposed again after the call).
4. The setting is read-only on that device (for example
   `CONFIG_MANAGED_BY_DIFFERENT_AUTHORITY`): it becomes a notice and an
   incident.
5. No rule exists for that diagnostic.
6. The issue was already reported in the last `renotify_minutes`.
7. The fix did not verify (an incident was opened).
8. Control Hub reads failed (HTTP 401 means the account key needs a new
   authorization).

## 3. Flapping devices

Questions like "Which devices keep flipping between Connected and Connected
with issues, and why?"

1. From the Webex space cards and `emg_room_status`, count status changes per
   device in the period asked (default today).
2. Name the diagnostic behind each flip. `UltrasoundInterference` on two or
   more devices at the same time usually means devices close enough to hear
   each other's ultrasound; say which devices reported it in the same window.
3. Recommend the cause fix (separate the devices, or lower ultrasound on one
   of them through an approved change), not a rule change.

## 4. Policy drift

Questions like "Which devices drift from our readiness policy?"

1. For each room device the user names, or for each device that is
   connected, run `COLLAB - Assess Room Readiness` with that device.
2. Report the "Known configuration maintained" and "Audio volume" checks per
   device with the values read (for example "Conference.AutoAnswer.Mode is On,
   expected Off").
3. Fixes go through `COLLAB - Approve and Restore Audio` (audio only) or a
   human; this playbook never changes a device.

## 5. Is it safe to change a device now

Questions like "Is it safe to change HWS Desk Pro G2 right now?"

Answer SAFE or NOT SAFE with the deciding fact:
- the active-call count from a fresh status read (unknown is NOT SAFE);
- an open self-heal proposal or the self-heal lock held for that device;
- the device's current Control Hub status.

## 6. What did the guardian do today

Questions like "What did the guardian fix today and what is still open?"

From the Webex space cards, `emg_heal_decisions` and ServiceNow:
- each fix: device, issue, decision (auto-accepted, accepted, declined) and
  by whom, time from Control Hub change to verified heal;
- each notice without a fix and its incident number;
- disconnects and reconnects;
- what is still open, with an owner.

## 7. Offline devices

Questions like "Which devices are offline, for how long, and do they have an
incident?"

List room devices and phones that are `disconnected` or `offline_expired`
with their last-seen time, how long ago that was, and whether an `emg-`
incident is open for them. Devices that were already offline when the
watcher started do not have an incident until they change; say so.

## Guard rails

- Read-only. Any change goes through the approval workflow or the self-heal
  rules, with approval and a verified readback.
- Never change or test a device that is in a call.
- Never reboot, factory reset or run arbitrary commands.
