# Hardened AGENTS.md

A verification-first repository instruction file for coding agents. It defines how an agent establishes scope, makes bounded assumptions, runs project checks, and reports what the evidence proves.

## What it provides

The policy asks an agent to:

- define an observable outcome before editing;
- use the repository and its tools as the source of truth;
- keep uncertainty visible instead of turning guesses into facts;
- preserve human control over commits and acceptance;
- verify changes at the boundary where they matter.

The policy is an instruction file, not a security boundary. Host permissions, workspace confinement, approval, and destructive-operation controls must be enforced by the agent harness.

## Use it

Copy the policy into a project, then replace its project context with that project's actual commands and constraints.

```sh
cp AGENTS.md /path/to/project/AGENTS.md
```

Do not copy example commands into a project that does not provide them. Read the complete file before adopting it.

## Repository contents

- [AGENTS.md](AGENTS.md) is the policy artifact.
- [docs/research.md](docs/research.md) records the evidence and limits behind the policy.
- [docs/adoption.md](docs/adoption.md) explains how to adapt it.
- [CONTRIBUTING.md](CONTRIBUTING.md) describes changes and review.
- [.github/workflows/validate.yml](.github/workflows/validate.yml) runs the repository checks.

## Validate

This repository contains documentation and validation scripts rather than a product runtime.

```sh
git diff --check
```

The workflow also checks required files and local Markdown links.

## Research boundary

The research documents distinguish evidence-backed findings from design principles and readability preferences. Read the sources and the policy together; neither replaces project tests, code review, or technical judgment.

## License

MIT. See LICENSE.
