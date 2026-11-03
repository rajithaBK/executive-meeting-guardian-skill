# Using the guardian-github integration

- Follow the `executive-meeting-guardian-learning` skill.
- `readGuardianData` accepts only the two fixed queries listed in the spec: the first is RULES, the second is HISTORY.
  Send the query string exactly as listed.
- To propose a change, call in order: `createRulesTree`, `createRulesCommit`, `createProposalBranch`,
  `openRulesPullRequest`. Use ids from the RULES response; never invent shas.
- Only `skills/executive-meeting-guardian-org-rules/org-rules.json` may be changed, only on a `guardian/` branch,
  only through a pull request into `main`. There is no merge operation: a human merges in GitHub.
- Never include credentials or tokens in commit messages, file content or pull request text.
