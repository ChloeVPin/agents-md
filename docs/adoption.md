# Adoption guide

Use `AGENTS.md` as a small repository contract, not as a replacement for a project handbook.

## Before copying

Identify the facts an agent cannot safely infer from the repository alone:

- the commands that build, test, lint, and type-check the project
- the configuration files that define important conventions
- architecture or workflow constraints that are easy to violate
- any required human approval boundary

Do not add general style advice that the agent can discover from existing code or a formatter configuration.

## Adapt the project context

Copy [`AGENTS.md`](../AGENTS.md) into the target repository root. Replace only the `Project Context` values first. Every command must exist, be safe to run, and produce an observable result in that project.

The policy's commands are examples. A copied file with stale commands is worse than a shorter file with accurate commands.

## Keep the policy lean

Prefer a small rule that blocks a known failure mode over a long explanation of the failure mode. Put rationale, examples, and source notes in project documentation. Revisit the file when the repository's commands or architecture change.

Avoid turning the file into:

- a full style guide
- a product requirements document
- a changelog
- a list of hypothetical failures
- a set of configuration flags invented for agent convenience

## Use the verification loop

For each task, the agent should:

1. State a short plan and concrete success criteria.
2. Inspect the current files and tools before editing.
3. Make only the approved changes.
4. Run the complete relevant validation suite.
5. Report exact output, including partial results when a harness fails.
6. Stop and ask when a missing fact makes a correct change impossible.

If the environment cannot run a required check, provide the exact copy-pasteable command and the expected success criteria. Do not describe an unrun check as passed.

## Maintain project learnings

Add a learning only after all of the following are true:

1. An agent caused a concrete failure.
2. The failure was observed and verified by a human or by objective tooling.
3. The fix is known.
4. The lesson can be stated as one concise rule that prevents recurrence.

Use this shape:

```markdown
### 2026-08-17 - Prevent silent validation failures

Failure: The agent reported success after a harness stopped before running the complete suite.

Lesson Learned: Never claim full validation without checking the final exit status and the complete result output.
```

Do not add a rule for a failure that has not happened in the project.

## Host compatibility

The policy only has an effect when the host agent reads repository instruction files. Support, precedence, file naming, and reload behavior vary by tool. Confirm the host's documented behavior and keep the project's own validation infrastructure authoritative.
