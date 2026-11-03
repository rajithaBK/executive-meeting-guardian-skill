---
name: executive-meeting-guardian-customer-context
description: >
  Canada Collab's business knowledge about its meeting rooms: which rooms
  matter and why, who uses them, fallback rooms, the partners we meet and
  how their side joins (Teams, Webex, Zoom, lobby rules), event profiles
  (board meeting, earnings call, customer review), change freezes, the
  Quebec French-language rule and the customer security baseline. Use
  whenever a question names a person, partner, event, site, freeze, tier
  or "our standard", or asks which room to use, what happens if a room
  fails, or whether a change is allowed.
---

# Executive Meeting Guardian: customer context (Canada Collab)

Control Hub and Cloud Control know the devices. They do not know the
business: that the HWS Board 75 is the boardroom, that Northwind Bank blocks
anonymous Teams guests, that Montreal rooms must speak French, or that
nothing may change during board week. This skill supplies that knowledge
from `customer-context.yaml`. Always report its `version`.

Use it with `executive-meeting-guardian` (readiness procedure and report)
and `executive-meeting-guardian-org-rules` (self-heal rules).

## How to use the context

1. **Resolve names.** Map what the user says to a room `id` and its Control
   Hub `device` name: "the boardroom" is `toronto-boardroom` on
   `HWS Board 75`; "the CEO's room" is `toronto-executive-room` on
   `HWS Desk Pro G2`. Map a partner name to its `partners` entry and a
   meeting type to its `event_profiles` entry. If the name is ambiguous,
   list the candidates instead of guessing.
2. **Get live evidence** for the resolved devices from the Collaboration
   Control Hub agent or `COLLAB - Assess Room Readiness`. Never answer a
   readiness question from this file alone; it says what should be true,
   the device says what is true.
3. **Apply the customer layer** on top of the readiness result:
   - Tier: use the tier's `impact_if_failed` for the business consequence
     and its `teams_join_test_required` for the Teams gate.
   - Partner: compare the partner `platform` with the room's
     `platforms_supported`. A platform the room does not support is FAIL,
     not WARNING. Explain the partner's `their_tenant_rules` and repeat
     the `join_advice` to the meeting owner.
   - Event profile: add every `extra_checks` item to the report. Items you
     cannot verify are UNKNOWN.
   - Site: in Quebec (`montreal`) check `UserInterface.Language` against
     `language_rule.required`. English-only is FAIL for client and
     executive rooms.
   - Security baseline: compare each key; a mismatch in an executive room
     is FAIL.
   - Intended mode: when `intended_mode` is set and the device software
     does not match (RoomOS reported where Microsoft Teams Rooms is
     intended), report the drift. Do not change the mode; it needs a
     human and a re-registration.
4. **Fallback.** For executive and client rooms, assess the `fallback`
   room too and name it in the answer. A fallback that is itself not ready
   does not count.
5. **Change control.** Before proposing any change, check
   `change_control.freezes` against today's date in America/Toronto and
   the room ids in `scope`. During a freeze only the listed `allowed`
   actions may be proposed. Outside a freeze, non-urgent changes go to
   the Saturday maintenance window.
6. **Who to tell.** Use the event profile `notify` list and the site's
   `onsite_support` (team, hours, dispatch minutes). Outside support hours,
   say so and do not promise a dispatch.

## Rules

- Never name a fallback, partner workaround or freeze that is not in the
  file. If the context has no answer, say it is not defined for Canada
  Collab and who owns it (`owner`).
- Personal desks and phones in `personal_devices` are never offered as a
  meeting room or fallback.
- A partner on a platform no room supports (Litware Capital on Zoom) gets
  "not joinable from any Canada Collab room" plus the `join_advice`. Do
  not suggest enabling a new platform as a quick fix.
- Times in answers are in the site's timezone with a label (for example
  "9:00 AM ET").
- Everything in this file is customer preference; label it as Canada
  Collab policy, not a Cisco requirement.
