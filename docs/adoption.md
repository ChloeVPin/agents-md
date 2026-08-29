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

## Evidence grades

The policy marks each rule group with an evidence grade. When adapting the file, keep the grades honest:

- **Evidence-backed** rules are grounded in controlled evaluation, large-scale empirical study, or multiple real-world incident reports with publicly documented sources. These are the rules most likely to generalize across projects.
- **Principled** rules are sound design principles consistent with the evidence, but not individually proven as failure-prevention mechanisms. They are still mandatory — the grade is about how we know, not about whether to enforce.
- **Readability and pipeline** preferences are not proven to affect agent performance. They are included because they are harmless, often useful, and consistent with the document's style of direct, functional communication. Do not present them as empirical findings when adapting the file.

## Use the verification loop

For each task, the agent should:

1. State a short plan and concrete success criteria.
2. Inspect the current files and tools before editing.
3. Make only the approved changes.
4. Run the complete relevant validation suite.
5. Report exact output, including partial results when a harness fails.
6. Stop and ask when a missing fact makes a correct change impossible.

If the environment cannot run a required check, provide the exact copy-pasteable command and the expected success criteria. Do not describe an unrun check as passed.

## After any write, verify

After any operation that modifies filesystem state, issue a read-back verification command confirming the expected state before reporting success. A write without a subsequent read is an unverified claim.

This is one of the most consequential rules in the policy, grounded in the MAST finding that verification failures (FM-3.2, FM-3.3) appear even in successful runs, and in the Gemini CLI incident where an agent misread a failed directory creation as success and overwrote all but one file without ever verifying.

## Separate hard boundaries from model instructions

AGENTS.md is instructions for the model, not a security boundary. Two of the format's own design rules make this unavoidable:

1. Explicit user chat prompts override everything.
2. Nearest-file-wins: a nested AGENTS.md displaces the root policy for its subtree.

These are correct, deliberate behaviors for a capability file. They are not bugs. But they mean that this file's prohibitions are enforceable only when the host agent reads and follows them.

For hard boundaries — workspace confinement, destructive operation approval, dev/prod environment separation, access gating — the enforcement must sit in the harness, not the model. When adapting this policy to a project, identify which boundaries must be hard and which are advisory. Do not assume that listing a boundary here makes it hard.

The emerging community convention for harness-level enforcement is a separate file (e.g., `agentaccess.txt`) that is evaluated by the harness before anything in the tree is read, including AGENTS.md itself. That file is deliberately disjoint from this one: it never enters model context, and it gates the reading rather than instructing the model. Investigate whether your host tool supports harness-level enforcement and configure it for the boundaries that must be hard.

## Destructive operations require out-of-band approval

Do not rely on a permission system or safety mode that cannot see the blast radius of a destructive command before approval. An agent running with permissions enabled is not necessarily safe — the Claude Code #10077 incident had permissions enabled, not dangerously-skip-permissions, and still executed recursive rm -rf from root.

For any destructive operation (recursive delete, root-targeted operations, database DROP/DELETE), require explicit out-of-band human approval regardless of any permission setting. Do not disable, bypass, or work around per-command approval, file deletion protection, or external filesystem protection.

## Loops need termination predicates

Any loop — whether a single agent retrying a task or multiple agents calling each other — must have a measurable, decidable termination predicate defined before execution begins. "The agent is satisfied" is not a termination predicate. Every agent invocation must have a per-agent budget cap and the pipeline must have a hard maximum cost or iteration count.

Observability without enforcement is not safety. The LangChain A2A incident ran for 264 hours and accrued $47,000 in costs because the team had dashboards but no hard caps.

## Maintain project learnings

Add a learning only after all of the following are true:

1. An agent caused a concrete failure.
2. The failure was observed and verified by a human or by objective tooling.
3. The fix is known.
4. The lesson can be stated as one concise rule that prevents recurrence.

Use this shape:

```markdown
### YYYY-MM-DD — Short title

**Failure:** Concise description of the verified agent-caused failure.

**Lesson Learned:** The specific, absolute prohibition or guideline derived from the failure.

**Source:** Where this failure is publicly documented.
```

Do not add a rule for a failure that has not happened in the project. The current entries in `AGENTS.md` are drawn from externally verified incidents in the agentic coding space — not hypotheticals. When this repository has its own verified failure, replace the placeholder with that incident.

## Host compatibility

The policy only has an effect when the host agent reads repository instruction files. Support, precedence, file naming, and reload behavior vary by tool. Confirm the host's documented behavior and keep the project's own validation infrastructure authoritative.

If your host tool loads nested AGENTS.md files with nearest-file-wins semantics, be aware that a nested file can displace the root policy for its subtree. This is a known property of the format. For hard boundaries, use harness-level enforcement rather than relying on the file nesting behavior.

The Configuration Effectiveness proposal (GitHub issue #213 on the format spec repository) distinguishes four failure categories that affect whether a rule actually takes effect:

- **Load failure:** the rule never reached the model (missing, shadowed, import failed)
- **Interpretation failure:** the rule loaded but was not followed
- **Applicability failure:** the rule loaded, but the target file or scope did not match
- **Maintenance failure:** the rule is stale, duplicated, or conflicting

When a rule does not appear to be working, diagnose which category applies before rewriting the rule.
