# Hardened AGENTS.md

Correctness over plausible output.

`AGENTS.md` is a compact, verification-first operating contract for coding agents. It sets hard boundaries around scope, assumptions, error handling, and human approval. It also requires the agent to run the project's own validation tools before making a success claim.

This repository makes that contract easy to inspect, adapt, and maintain. It contains the policy, the reasoning behind it, an adoption guide, and a small validation layer for the repository itself.

> An instruction file can improve discipline. It cannot replace tests, review, or technical judgment.

## Why this exists

Coding agents can produce output that looks finished while relying on guessed APIs, unverified project state, or incomplete checks. The result is a plausible diff that fails when it meets the real codebase.

The policy addresses that failure pattern with five commitments:

1. Define scope and success before editing.
2. Use sharp constraints for predictable failure modes.
3. Make the project's tools the source of truth.
4. Keep human approval in control of commits and acceptance.
5. Record only confirmed, agent-caused failures as new project learnings.

## The operating loop

```text
Plan and define success
        |
Make only approved edits
        |
Run the project's validation tools
        |
Report exact observed output
        |
Human reviews the diff and accepts the result
```

The loop is deliberately ordinary. Reliability comes from making each transition explicit and observable.

## Quick start

Copy the policy into the root of a project that uses coding agents:

```bash
cp /path/to/agents-md/AGENTS.md /path/to/your-project/AGENTS.md
```

Then replace the `Project Context` values with the target project's actual commands, configuration files, and architecture constraints. Keep the values short, concrete, and verified. Do not copy the example commands unchanged into a project that does not provide them.

The rest of the file is the reusable policy. Review it with the people who own the repository, then run the normal build, test, lint, and type-check commands for that project.

## What is in this repository

- [`AGENTS.md`](AGENTS.md): the reusable policy artifact.
- [`docs/research.md`](docs/research.md): the research boundary, design rationale, and reference trail.
- [`docs/adoption.md`](docs/adoption.md): how to adapt the file without turning it into a second handbook.
- [`CONTRIBUTING.md`](CONTRIBUTING.md): review and contribution expectations.
- [`.github/workflows/validate.yml`](.github/workflows/validate.yml): the same checks in GitHub Actions.

## Validation

This is a documentation repository, so its GitHub Actions workflow checks required files and local Markdown links. It does not pretend to compile a product that is not here.

```bash
git diff --check
```

The workflow is the authoritative repository check on pushes and pull requests.

## Limits

The policy is not a sandbox, a security boundary, a test runner, or a guarantee that an agent will obey every instruction. Host tools differ in whether and when they load repository context files. The project files, validation tools, CI, and human review remain authoritative.

The companion research argues for a lean instruction file because extra context can dilute the signal and increase cost. This repository keeps rationale in documentation so the operating contract stays small enough to inspect during every task.

## Research and lineage

The design is based on the supplied research synthesis and its bibliography. The research notes distinguish evidence from interpretation and link to the primary or representative sources that shaped the policy.

- [Read the research notes](docs/research.md)
- [Read the adoption guide](docs/adoption.md)
- [See the earlier agents-md lineage cited by the research](https://github.com/TheRealSeanDonahoe/agents-md/blob/main/README.md)

## Contributing

Changes should make the policy more reliable, easier to adopt, or easier to verify. A new rule should have a concrete failure mode behind it. Start with [`CONTRIBUTING.md`](CONTRIBUTING.md).

If this repository changes how you review agent work, star it so other teams can find the pattern.

## License

Released under the [MIT License](LICENSE).
