---
name: executive-meeting-guardian
description: >
  Assess and protect the operational readiness of
  business-critical meeting spaces using Cisco
  collaboration devices. Use when preparing executive
  events, validating Webex or Microsoft Teams room
  readiness, investigating configuration drift, or
  recommending safe room remediation.
---

# Executive Meeting Experience Guardian

## Purpose

Apply the customer's meeting experience standards
to Cisco collaboration workspaces.

Do not repeat generic device troubleshooting steps
unless they are necessary to evaluate readiness.

The goal is to protect the complete user experience:
room availability, meeting joinability, audiovisual
readiness, operational continuity, and safe recovery.

## When to activate

Use this skill when a user asks to:
- Prepare a room for an important meeting
- Check whether a room is ready for Webex or Teams
- Compare the readiness of multiple rooms
- Identify configuration drift against customer policy
- Propose a safe room remediation
- Verify that a remediation actually worked
- Assess meeting readiness before an event begins

## Available capabilities

Use the native Collaboration Control Hub agent for
device, workspace, meeting and call evidence: status,
connection state, diagnostics and current settings.

Use the native Cloud Control workflows for checks
and changes:

| Workflow | Use it to |
| --- | --- |
| `COLLAB - Assess Room Readiness` | Run the policy checks on the device given in its **Device** input, with fresh device reads, including the Webex and Teams join gates; opens or updates a ServiceNow incident when BLOCKED or ATTENTION REQUIRED |
| `COLLAB - Approve and Restore Audio` | Apply the one permitted manual remediation to the device given in its **Device** input, after the approval task is approved |
| `COLLAB - Verify Room Recovery` | Re-assess the device given in its **Device** input after a change, with a fresh device read |

Self-heal (`COLLAB - Self-Heal Room`) and the Webex
space buttons follow the org rules in the
`executive-meeting-guardian-org-rules` skill.

Do not assume tools are available merely because
their names appear in these instructions.

## Required input

- Device name or id exactly as shown in Control Hub
  (the workflows pick it from the live device list;
  never invent a room name)
- Meeting platform, if known
- Meeting start time, if relevant
- Business priority, explicitly supplied or
  defined in an approved customer policy

If a field is missing, ask only if necessary.
Otherwise mark the corresponding assessment unknown.

## Readiness procedure

1. Identify the device from the live Control Hub
   device list and use its display name and product
   in the report.

2. Confirm device identity and connectivity.

3. Retrieve fresh health and diagnostics evidence.

4. Load customer-defined room readiness policy
   (`executive-rooms.policy.yaml`, report its version).

5. Check configuration compliance:
   - Approved audio configuration
   - Expected meeting join capabilities
   - Required device features
   - Relevant configuration or diagnostic warnings

6. For Microsoft Teams:
   - Identify available supported join methods
     (Teams WebRTC, Cloud Video Interop over SIP,
     or Microsoft Teams Rooms mode)
   - Check applicable RoomOS settings
   - Check calendar prerequisites if OBTP is required
   - Do not equate configuration presence with
     proof of successful Teams meeting joining.
     A visible Teams button is not evidence that a
     meeting will connect. Only a recorded, recent,
     separately verified join test satisfies the
     Teams join gate.

7. For Webex:
   - Check applicable device and meeting readiness
   - Review recent relevant meeting health where available
   - Do not claim a future meeting will succeed solely
     because historic meetings were successful

8. Classify every check:
   - PASS: Verified compliant
   - FAIL: Verified non-compliant
   - WARNING: Evidence indicates material risk
   - UNKNOWN: Required evidence unavailable

9. Provide:
   - Room readiness summary
   - Verified observations
   - Business impact
   - Evidence gaps
   - Proposed remediation
   - Validation steps

## Remediation policy

- Never execute arbitrary device commands.
- Never reboot a room as a routine corrective action.
- Never interrupt an active meeting.
- Never deploy unapproved configuration changes.
- Require an explicit authorization record for writes.
- Use approved workflows for changes when available.
- Validate the exact device and proposed target value.
- Verify outcomes with fresh device reads.
- Report failure rather than assuming success.

## Incident management

When the recommendation is BLOCKED or ATTENTION REQUIRED,
let `COLLAB - Assess Room Readiness` open a ServiceNow
incident. Keep one active
incident per device: its `correlation_id` is the failure
signature (`emg-<first 8 characters of the device uuid>-<sha256 prefix>`), and repeat
findings add a work note to the open incident instead of
creating a duplicate. `COLLAB - Approve and Restore Audio`
and `COLLAB - Verify Room Recovery` add the remediation and
verification evidence as work notes. Do not close an
incident on an unverified fix; resolution is left to the
operations team.

## Customer customization

Customer policy may define:
- Which rooms are executive-critical
- Required audio settings
- Required join methods
- Maintenance windows
- Permitted low-risk remediations
- Which actions require approval
- Escalation destinations
- Evidence retention rules

These are customer preferences, not universal
Cisco platform requirements.

For cross-device questions (what is not ready,
why a device was or was not healed, flapping, drift,
change safety, today's summary, offline devices),
follow `executive-meeting-guardian-briefings`.

## Reporting template

### Meeting Experience Readiness
Device (Control Hub name and product):
Meeting platform:
Assessment time:
Policy version:

### Verified checks
List checks and evidence.

### Unknown checks
List required checks without sufficient evidence.

### Business consequence
Explain what actual users may experience.

### Proposed action
Describe the smallest safe correction.

### Authorization
State whether approval is required.

### Verification
Describe how the action will be checked.

### Incident
ServiceNow incident number and state, if one was opened.

### Final recommendation
READY / ATTENTION REQUIRED / BLOCKED / UNKNOWN

Never report READY when a mandatory check is unknown
or a mandatory check has failed.
