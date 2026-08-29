# Contributing

Contributions should make the policy more reliable, easier to adopt, or easier to verify. This is a documentation-first project. Every change should have a clear reason and a reviewable scope.

## Before opening a change

- Read [`AGENTS.md`](AGENTS.md) and [`docs/research.md`](docs/research.md).
- Check whether the proposed rule already exists in a shorter form.
- Tie new constraints to a concrete failure mode or a clear maintenance problem.
- Assign an honest evidence grade: evidence-backed (controlled evaluation, large-scale empirical study, or multiple real-world incidents with sources), principled (sound design principle, internally consistent with evidence), or readability/pipeline (preference, not proven to affect agent performance).
- Keep empirical claims linked to their source and distinguish synthesis from measurement.
- Do not add speculative rules, decorative content, or unrelated cleanup.
- Do not present readability or pipeline preferences as empirical findings.

## Evidence bar

A new rule should meet one of these bars:

- **Evidence-backed:** the rule addresses a failure mode documented in a controlled evaluation, large-scale empirical study, or multiple publicly documented real-world incidents with traceable sources.
- **Principled:** the rule is a sound design principle consistent with the evidence, even if not directly tested as a policy intervention. The rule should explain why it follows from the repository's overall approach.
- **Readability or pipeline:** the rule is a preference for clarity, token economy, or toolchain compatibility. It should be labeled as such and not presented as an evidence-backed failure-prevention mechanism.

A rule that does not meet any of these bars, for example, a rule based on a hypothetical failure, a personal preference not grounded in clarity or compatibility, or a claim without a traceable source, should not be added.

## Pull request expectations

A pull request should explain what changed, why it belongs in the repository, and how it was checked. Policy changes should identify the behavior they are intended to constrain and the evidence grade. Documentation changes should keep examples and links current.

Keep the diff atomic. Do not commit generated scratch files, rendered PDFs, credentials, or changes outside the stated scope.

## Project Learnings entries

A new Project Learnings entry must be a real, verified failure, not a hypothetical. All of the following must be true:

1. An agent caused a concrete failure.
2. The failure was observed and verified by a human or by objective tooling.
3. The fix is known.
4. The lesson can be stated as one concise rule that prevents recurrence.
5. The failure is traceable to a public source (GitHub issue, forum post, vendor confirmation, published post-mortem, or reputable news report).

Do not populate `Project Learnings` with hypothetical incidents. Entries should follow the shape used in `AGENTS.md`:

```markdown
### YYYY-MM-DD, Short title

**Failure:** Concise description of the verified agent-caused failure.

**Lesson Learned:** The specific, absolute prohibition or guideline derived from the failure.

**Source:** Where this failure is publicly documented.
```

## Validation

Run the repository checks before requesting review:

```bash
git diff --check
```

The GitHub Actions validation workflow also checks required files and local Markdown links because this project contains Markdown and configuration rather than application source code.

## Host-tool and harness awareness

When proposing a rule that is intended as a hard boundary, consider whether it can actually be enforced by the model reading this file, or whether it requires harness-level enforcement. Rules that require the model to self-police a hard boundary (workspace confinement, destructive operation approval, dev/prod separation) should note that limitation and point toward harness-level enforcement where it exists.

For rules that are advisory only, dependent on the model reading and following this file, acknowledge that limitation plainly. The policy is instructions for the model, not a security boundary.
