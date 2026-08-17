# Contributing

Contributions should make the policy more reliable, easier to adopt, or easier to verify. This is a documentation-first project. Every change should have a clear reason and a reviewable scope.

## Before opening a change

- Read [`AGENTS.md`](AGENTS.md) and [`docs/research.md`](docs/research.md).
- Check whether the proposed rule already exists in a shorter form.
- Tie new constraints to a concrete failure mode or a clear maintenance problem.
- Keep empirical claims linked to their source and distinguish synthesis from measurement.
- Do not add speculative rules, decorative content, or unrelated cleanup.

## Pull request expectations

A pull request should explain what changed, why it belongs in the repository, and how it was checked. Policy changes should identify the behavior they are intended to constrain. Documentation changes should keep examples and links current.

Keep the diff atomic. Do not commit generated scratch files, rendered PDFs, credentials, or changes outside the stated scope.

## Validation

Run the repository checks before requesting review:

```bash
git diff --check
```

The GitHub Actions validation workflow also checks required files and local Markdown links because this project contains Markdown and configuration rather than application source code.

## Project learnings

Do not populate `Project Learnings` with hypothetical incidents. Add an entry only after an agent-caused failure has been verified, corrected, and reduced to a concise preventive rule.
