---
name: executive-meeting-guardian-briefings
description: >
  Answer business questions about Canada Collab meetings that need both live
  device evidence and customer context: "will the board meeting work",
  "can we meet Northwind from the CEO's room", "what if the boardroom fails
  10 minutes before", "is it safe to change rooms this week", "are Montreal
  rooms compliant", "which rooms keep breaking and what does it cost us",
  and the weekly executive digest. Use for go/no-go, partner meeting,
  fallback, change-freeze, compliance and trend questions.
---

# Executive Meeting Guardian: briefings

Each playbook below combines three sources:

| Source | Gives |
| --- | --- |
| Collaboration Control Hub agent, `COLLAB - Assess Room Readiness`, `COLLAB - Verify Room Recovery` | Live device status, diagnostics, settings, call state |
| `executive-meeting-guardian-customer-context` | Rooms, tiers, partners, event profiles, freezes, language rule, security baseline, fallbacks, support |
| Cloud Control variables `emg_room_status`, `emg_heal_decisions`, `emg_heal_suppressed`, `emg_last_assessment`, and ServiceNow incidents (`correlation_id` starts with `emg-`) | What the guardian already saw, healed, declined or escalated |

If a source is not available in this session, mark the dependent checks
UNKNOWN and say which source was missing. Never fill a gap from memory.

Always end with one verdict line: **GO**, **GO WITH CONDITIONS**,
**NO-GO** or **UNKNOWN**, then the single next action and its owner.

## 1. Event go/no-go

Questions like "Will tomorrow's board meeting work?" or "Is the CEO's room
ready for the earnings call at 4 PM?"

1. Resolve the event profile, its rooms and fallback rooms.
2. Run the readiness check on every room and fallback room (fresh reads).
3. Add each profile `extra_checks` item; unverifiable items are UNKNOWN.
4. Check change control: is a freeze active between now and the event?
5. Check guardian history for each room in the last 7 days: heals,
   declines, unverified fixes, open incidents.
6. Verdict: NO-GO if a mandatory check fails on the main room and the
   fallback is not ready. GO WITH CONDITIONS if the main room fails but the
   fallback is ready, or only advisory checks fail.
7. List who is notified (`notify`) and the on-site support window.

## 2. Partner meeting check

Questions like "Can we meet Northwind Bank from the Toronto Executive
Room?" or "Which room should I book for Litware?"

1. Resolve the partner. Compare its platform with each candidate room's
   `platforms_supported`. Rooms that cannot join are excluded, with the
   reason.
2. For each remaining room, run readiness with the partner's platform
   (Teams gate for Teams partners).
3. Explain the partner's tenant rules in plain language (lobby, anonymous
   join, meeting lock) and give the `join_advice`.
4. If the partner language is `fr`, prefer Montreal rooms that pass the
   language rule.
5. Recommend one room and one fallback, or say no room can join.

## 3. Failure rehearsal ("what if")

Questions like "What happens if the boardroom fails 10 minutes before the
board meeting?"

Do not break anything. Describe the plan from evidence:
1. Readiness of the fallback room right now.
2. What self-heal would do for the likely failures (org rules: ultrasound,
   internal speaker) and what it would not (disconnect, read-only setting,
   active call), with the time it typically takes (15 s to 3 min to see the
   change, about 15 s more to heal).
3. Who is called (on-site support, dispatch minutes, hours) and whether the
   event falls outside support hours.
4. The phone fallback for the event profile, if any, and its status.
5. A 3-step script for the executive assistant.

## 4. Change safety

Questions like "Can I push the new volume baseline to the boardroom
today?"

1. Check active and upcoming freezes for the room ids.
2. Check the room's calendar evidence if available, and the call state now.
3. Check open guardian incidents on the room.
4. Answer ALLOWED / ALLOWED IN MAINTENANCE WINDOW / NOT ALLOWED, with the
   rule that decides it. Never perform the change from this playbook; hand
   off to the approval workflow when allowed.

## 5. Compliance sweep

Questions like "Are our Montreal rooms compliant?" or "Which executive
rooms break our security standard?"

1. Select rooms by site or tier.
2. Read `UserInterface.Language` (Quebec), the `security_baseline` keys and
   the device software (intended mode) for each.
3. One row per room: each rule PASS / FAIL / UNKNOWN with the value read.
4. Proposed fixes go through the approval workflow and respect freezes.
   Mode drift (RoomOS where Microsoft Teams Rooms is intended) is a project
   for a human, not a fix.

## 6. Reliability and cost

Questions like "Which rooms keep breaking, and what does it cost us?"

1. Load guardian history (`emg_heal_decisions`, ServiceNow `emg-`
   incidents) for the period asked, default 30 days.
2. Per room: number of issues, healed automatically, declined, unverified,
   disconnects and total time not ready.
3. Weight by tier `impact_if_failed`; rank executive rooms first.
4. Name recurring patterns (same issue, same room, same weekday and hour)
   and suggest the cause, not just the fix. Hand rule changes to
   `executive-meeting-guardian-learning`.

## 7. Weekly executive digest

Questions like "Give me this week's executive meeting room digest."

One page, plain language, for a non-technical reader:
- Headline: are executive rooms ready for next week (verdict per room).
- Upcoming events from `event_profiles` and freezes in the next 14 days.
- What the guardian fixed by itself, with time to heal.
- What needs a human (open incidents, read-only settings, mode drift,
  compliance FAILs), each with an owner.
- Partner meetings at risk (platform not supported, strict lobby).

## Guard rails

- Read-only by default. Any change goes through `COLLAB - Approve and
  Restore Audio` or the self-heal rules, with approval and a verified
  readback.
- Never change or test a room that is in a call.
- Never reboot, factory reset or run arbitrary commands.
- Never invent partners, events, fallbacks or dates that are not in the
  customer context.
