# Research notes

This document records how the supplied research shaped the repository. It separates the source material from the policy decisions made here.

## Source boundary

The source document is `Forging Reliability: A Blueprint for an AGENTS.md That Enforces Verification Over Plausibility`. It is a synthesis of academic papers, engineering postmortems, and community writing. It is useful design input, but it is not a controlled evaluation of this repository. The bibliography contains sources with different levels of authority, so each external claim should be checked against its original source before being presented as a measured result.

## Findings carried forward

### Plausible output is not evidence

The research describes a recurring failure pattern in which an agent produces code that looks coherent but depends on guessed APIs, incorrect assumptions, or incomplete execution. A polished diff is an artifact to inspect, not proof of correctness.

### More context is not automatically better

The research reviews evaluations that question whether long repository context files improve coding-agent outcomes. Its practical conclusion is narrower than a universal verdict: context should be short, current, and limited to information the agent cannot reliably discover on its own. A stale or oversized file can compete with the rules that matter.

### Binary boundaries reduce room for self-justification

The policy uses explicit prohibitions for recurring failure modes such as scope drift, speculative features, silent error handling, invented artifacts, and unsupported claims. These rules are easier to check than a general instruction to be careful.

### Verification is a workflow, not a closing sentence

The research treats verification as a required loop: state success criteria, make the approved edit, run the available checks, read their output, and report what actually happened. When a check cannot run, the agent should provide an exact command and expected result rather than implying that the check passed.

### Learnings should be reactive

The `Project Learnings` section is intentionally empty apart from its schema. A new rule belongs there only after a real agent-caused failure has been observed, corrected, and converted into a concise preventive rule. Hypothetical failures create noise and make the file stale.

## Repository decisions

| Research concern | Repository decision |
| --- | --- |
| Context collapse | Keep `AGENTS.md` short and move explanation into `docs/`. |
| Plausible-but-broken changes | Require tool-driven validation and exact reporting. |
| Scope drift | Prohibit unrelated edits and invented configurability. |
| Self-deception | Forbid claims about files, APIs, errors, or tests without tool evidence. |
| Stale rules | Add project learnings only after verified incidents. |
| Loss of human control | Keep commits, acceptance, and pull-request approval with the human developer. |

## What this project does not claim

- It does not claim that every AGENTS.md improves task success.
- It does not claim that prohibitions replace engineering judgment.
- It does not claim that one policy fits every repository or host tool.
- It does not treat the research bibliography as a single peer-reviewed evidence base.

## Reference trail

These are representative links from the supplied research. They are included as a starting point for verification, not as a substitute for reading the sources.

- [Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?](https://arxiv.org/abs/2602.11988)
- [Why Do Multi-Agent LLM Systems Fail?](https://arxiv.org/pdf/2503.13657)
- [The research is in: your AGENTS.md is probably too long](https://developer.upsun.com/posts/ai/agents-md-less-is-more)
- [No Cheating: Isolated Specification Testing with Claude Code](https://www.codecentric.de/en/knowledge-hub/blog/dont-let-your-ai-cheat-isolated-specification-testing-with-claude-code)
- [AGENTS.md from oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent/blob/dev/AGENTS.md)
- [Agentic Code Review](https://addyosmani.com/blog/agentic-code-review/)

## Reading guide

The source PDF is the design brief for this repository. `AGENTS.md` is the resulting policy artifact. `docs/adoption.md` explains how to adapt it to a real codebase without copying its example context blindly.
