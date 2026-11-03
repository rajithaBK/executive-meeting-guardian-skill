---
name: executive-meeting-guardian-learning
description: >
  Learn from Executive Meeting Guardian self-heal history and propose
  better org rules as a GitHub pull request for human review. Use when
  asked what the guardian has learned, why rooms keep needing fixes,
  which rules are wrong or noisy, or to improve the self-heal policy.
---

# Executive Meeting Guardian: learn and propose

## Purpose

Every self-heal outcome in Canada Collab's executive rooms is recorded in
GitHub. Turn that history into insight, and when the evidence supports it,
propose a change to the org rules as a pull request. A human reviews and
merges; Cisco Cloud Control applies the merged rules on the next room event.

You never change the rules directly and you never merge.

## Tools

Use the `guardian-github` Canvas integration:

| Operation | Use it to |
| --- | --- |
| `readGuardianData` (HISTORY query) | Load every event: file names carry time, room, issue, decision and outcome; the text has detail |
| `readGuardianData` (RULES query) | Load the current `org-rules.json`, the main commit and tree ids, and open pull requests |
| `createRulesTree`, `createRulesCommit`, `createProposalBranch`, `openRulesPullRequest` | Propose a change, in that order |

The rules this skill may change are the ones in `org-rules.json`: `volume`,
`baseline`, `diagnostic_heals`, `lock_seconds`, `policy_version`. Identity
(admins, space, rooms), the 5-second approval window, verification timing
and "never change a room in an active call" are not editable here.

## Analysis procedure

1. Load HISTORY. Say how many events, which rooms, the date range, and how
   many are `seeded` (demo history) versus live.
2. Convert times to the room's timezone (America/Toronto) before looking
   for time patterns.
3. Look for these patterns, and quantify each one:
   - **Recurring trigger**: the same issue in the same room at the same
     weekday and hour, three or more weeks running. This is usually an
     external cause (cleaning crew, AV vendor, a scheduled event), not a
     rule problem. Recommend fixing the cause; a rule change rarely helps.
   - **Tug-of-war**: the guardian restores a value and people change it
     back within 15 minutes, repeatedly. The rule is fighting how the room
     is really used. Candidate: adjust the range or target.
   - **Repeated declines**: admins decline the same fix three or more
     times. The rule is wrong for at least one room. Candidate: stop
     enforcing it, and say which rooms lose protection.
   - **Unverified heals**: fixes that did not verify. Candidate: none;
     recommend investigation of the device.
   - **Deferred in call**: fixes postponed because a meeting was running.
     Report only; this is the safety rule working.
4. For each pattern give: rooms, dates, count, likely cause, and the
   business impact in plain language.

## When to propose a rule change

Propose only when all of these hold:
- At least three supporting events across at least two different weeks.
- The change addresses a tug-of-war or repeated-decline pattern.
- The new values are within the safety bounds below.
- No open `guardian/` pull request already covers it (check RULES).

Prefer the smallest change that removes the pattern. Never propose more
than one pull request per pattern.

### Safety bounds (Cloud Control rejects anything else)

- `volume`: integers with 20 <= min < target < max <= 90.
- `baseline` and `diagnostic_heals` settings: only
  `Audio.DefaultVolume` (0-100), `Audio.Ultrasound.MaxVolume` (0-90),
  `Conference.AutoAnswer.Mode` (On/Off), `Standby.Control` (On/Off),
  `Conference.DefaultCall.Protocol` (Auto/Sip/Spark/H323).
- `lock_seconds`: 30-600.

Increase `policy_version` by 0.1 in every proposal.

## How to open the pull request

1. `readGuardianData` with the RULES query. Keep `main.oid`,
   `main.tree.oid` and `rules.text`.
2. Edit only the keys you are changing; keep everything else, including
   `about`, exactly as it is. Content must be valid JSON with 2-space
   indent and a trailing newline.
3. `createRulesTree` with `base_tree` = `main.tree.oid` and the one
   allowed path.
4. `createRulesCommit` with `parents` = [`main.oid`].
5. `createProposalBranch` with `refs/heads/guardian/<yyyymmdd>-<slug>`.
6. `openRulesPullRequest` into `main` with this body:

```
## What the guardian learned
<pattern, rooms, dates, counts; link nothing private>

## Proposed change
<key: old -> new>

## Expected effect
<what stops happening, estimated events avoided per month>

## Risk
<which rooms lose protection or get a different target>

## Rollback
Revert this pull request; Cloud Control picks up the previous rules on the next event.

Proposed by AI Canvas (executive-meeting-guardian-learning). A human must review and merge.
```

7. Report the pull request link and tell the user what happens after
   merge: the next room event loads the new rules, and the Webex space
   shows an "Org rules updated from GitHub" card listing the change.

## Reporting template

### What the guardian learned
Events analysed, rooms, period, seeded vs live.

### Patterns
One section per pattern with evidence and impact.

### Recommendations
Process fixes (no rule change) and rule changes.

### Pull request
Link, or why no rule change is justified yet.
