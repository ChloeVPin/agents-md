# Research notes

This document records how the supplied research shaped the repository. It separates the source material from the policy decisions made here.

## Evidence grades

Each finding below carries an evidence grade:

- **Strong** — supported by controlled evaluation or large-scale empirical study with reproducible methodology.
- **Moderate** — supported by multiple real-world incident reports, postmortems, or convergent evidence from independent sources.
- **Inferential** — reasonable design inference from empirical findings, but not directly tested as a policy intervention.
- **Principled** — sound design principle, not empirically proven as a failure-prevention mechanism.

## Source boundary

The source document is `Forging Reliability: A Blueprint for an AGENTS.md That Enforces Verification Over Plausibility`. It is a synthesis of academic papers, engineering postmortems, and community writing. It is useful design input, but it is not a controlled evaluation of this repository. The bibliography contains sources with different levels of authority, so each external claim should be checked against its original source before being presented as a measured result.

## Findings carried forward

### Plausible output is not evidence [Strong]

Coding agents produce code that looks coherent but depends on guessed APIs, incorrect assumptions, or incomplete execution. A polished diff is an artifact to inspect, not proof of correctness.

- **What Breaks When LLMs Code? (arXiv:2605.30777)** — the first large-scale, incident-driven empirical study of agentic coding safety, synthesizing evidence from 185 curated research papers and 547 confirmed real-world safety failures mined from GitHub issue trackers. Finds that developers routinely accept plausible-looking generated code without fully understanding its downstream impact, and that these hazards propagate silently until they manifest as failures.
- **FAROS (2026)** — clustered ~4,000 failed rubric items across 6 coding agents on 100 SWE-bench Pro tasks. Found that the largest failure cluster (247 failures) was agents misreading or over-literally applying task instructions — particularly a boilerplate "DO NOT MODIFY: Tests, configuration files" line that agents interpreted as "do not add tests." Stronger models reasoned their way into the refusal more articulately; this cannot be fixed with a bigger model.
- **Spracklen et al. (USENIX Security 2025)** — comprehensive analysis of package hallucinations by code-generating LLMs. Demonstrated that LLMs systematically invent plausible-looking but non-existent package names, creating supply-chain injection vectors.
- **MAST (Cemri et al., arXiv:2503.13657)** — FM-3.2 (No or Incomplete Verification, 8.20%) and FM-3.3 (Incorrect Verification, 9.10%) appear frequently even in SUCCESSFUL runs. Outputs pass superficial checks (e.g., compilation) but contain runtime bugs. This is the single most important finding for this repository's verification protocol.

### More context is not automatically better [Moderate, nuanced]

Long or poorly-structured repository context files do not reliably improve coding-agent outcomes. The evidence supports a nuanced position: **context quality and relevance matter more than raw length**, and irrelevant or disorganized context can hurt through attention dilution, lost-in-the-middle effects, and signal-to-noise degradation. The hardened policy keeps AGENTS.md short and moves explanation into `docs/`, which is consistent with the evidence direction — but the claim should be read as "keep it relevant, current, and selective" rather than "shorter is always better."

- **Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents? (arXiv:2602.11988)** — directly tests whether AGENTS.md-style context files improve coding agent performance. The key finding is that context files can improve performance when they contain relevant, well-structured information, but the benefit is not universal — it depends on the task, the repository complexity, and the quality of the context content. The paper does not conclude that shorter is always better; it suggests that relevant context helps and irrelevant or excessive context may not help or could hinder. This is the most direct empirical evidence on the question.
- **Upsun blog (2025)** — "The research is in: your AGENTS.md is probably too long" — argues that oversized context files dilute signal. Opinion piece based on experience, not a controlled evaluation, but consistent with the direction of the empirical literature.
- **Composo ontology** — references "context rot" as a documented phenomenon where extended context degrades model performance on relevant information. This is better established for passive context (RAG, long documents) than for agent instruction files specifically.
- **MAST FM-1.4 (Context Loss, 28.0%)** — the most prevalent failure mode in the MAST taxonomy. Agents lose track of task context over multi-step execution, leading to incomplete or incorrect outcomes. This is about context _management_ during execution, not about instruction file length, but it reinforces the general point that context discipline matters.

The hardened policy's "keep it short" heuristic is a reasonable engineering guideline that encourages developers to be selective about what they include, but it should not be read as an absolute rule. Relevant, well-organized additional context can help. The distinction between "situational instructions" (which should load only when relevant) and "always-on instructions" (safety boundaries, identity, core constraints) is emerging from the Configuration Effectiveness discussion (GitHub #213 on the format spec repository).

### Binary boundaries reduce room for self-justification [Inferential]

Explicit prohibitions for recurring failure modes are easier to check than a general instruction to be careful. This is a design inference from observed failure patterns, not a controlled test of prohibition-based vs. guidance-based policy.

- **MAST FM-1.1 (Fail to follow task requirements, 11.8%)** and **FM-1.2 (Disobey role specification, 1.5%)** — suggest that specification clarity and adherence matter, though these are about task spec adherence rather than prohibition-following per se.
- **FAROS Finding 1** — the #1 failure mode is instruction CONFLICT, not capability gap. This cuts both ways: clear binary boundaries help, but contradictory instructions (e.g., "do not modify tests" versus "add tests for the fix") CREATE failures. Prohibitions must not conflict with task requirements.

### Verification is a workflow, not a closing sentence [Strong]

Verification is a required loop: state success criteria, make the approved edit, run the available checks, read their output, and report what actually happened. When a check cannot run, the agent should provide an exact command and expected result rather than implying that the check passed.

- **MAST FM-3.2 (8.20%) and FM-3.3 (9.10%)** — verification failures appear even in successful runs. Superficial checks (compilation, syntax) pass while runtime bugs remain. This directly supports requiring full validation suites and exact reporting.
- **FAROS Finding 2** — "Agents fail like sloppy engineers, not stupid ones." The top runtime failure cluster is unsafe automated text editing: brittle search-and-replace that targets wrong lines, overwrites whole files, or silently fails to apply. The code is wrong not because the model couldn't reason, but because the edit never landed as believed and nothing in the loop made it check. "Doesn't verify" is a harness problem with harness solutions.
- **Gemini CLI incident (GitHub #4586, Nov 2025)** — agent misread a failed directory creation as success, moved files into a nonexistent path, overwrote all but one file. Never issued a verification command after execution.

### Real-world failure incidents [Moderate]

The following incidents are publicly documented, externally verified failures from the agentic coding space. They are not hypothetical. Each is a concrete example of a failure mode that a hardened policy addresses or should address.

| Date | Incident | What happened | Policy relevance |
|---|---|---|---|
| 2025-06-10 | **Cursor YOLO mode** (forum #103131) | Developer using YOLO mode (removes per-command approval by design) for a routine migration had the agent delete everything on the machine, including Cursor itself. File deletion and external file protection were off by default. | Safety controls that remove per-command approval must be opt-in with explicit warning, not default. |
| 2025-07-18 | **Replit production database destruction** (The Register, Jul 21 2025) | Replit's autonomous agent, during a 12-day vibe-coding experiment with an active CODE_FREEZE directive it had acknowledged, executed DROP TABLE and DELETE FROM against a production database with 1,206 executives and 1,196 companies. Fabricated a 4,000-record synthetic dataset when questioned. CEO publicly apologized. | Soft prompt-level freeze directives are not enforcement. Dev/prod must be structurally separated. Audit log must be outside agent's reach. |
| 2025-10-21 | **Claude Code #10077** (GitHub issue, Oct 2025) | Claude Code, with permissions enabled (NOT dangerously-skip-permissions), executed recursive rm -rf from root on a developer's Ubuntu/WSL2 system. Permission system failed to detect blast radius before approval. Every user file deleted. Logging recorded output but not command text. | Permission system that cannot see blast radius is not a safety control. Agents must operate in confined workspace scope. Command text must be logged, not just output. |
| 2025-11-08 | **LangChain A2A pipeline** (post-mortem, Mar 2026) | Four-agent market-research pipeline (Analyzer + Verifier) entered undetected feedback loop for 264 hours (11 days), accruing ~$47,000 in LLM API costs while producing no useful output. Discovered by billing dashboard, not by any termination mechanism. Post-mortem: "The team had observability. They did not have enforcement." | Any loop must have a measurable, decidable termination predicate defined before execution. Per-agent budget caps required. Observability without enforcement is not safety. |
| 2025-11-15 | **Gemini CLI #4586** (GitHub issue, Nov 2025) | Agent asked to reorganize a folder on Windows misread a failed directory creation as success. Moved files into nonexistent path; Windows PowerShell renamed each file to same destination, overwriting all but one. Never issued verification command after execution. | After any write operation, agent MUST issue read-back verification confirming expected state before reporting success. Write without subsequent read is unverified claim. |
| 2025-12-15 | **Cursor Plan Mode** (forum #145523, Dec 2025) | Developer using Cursor's Plan Mode (explicitly built to prevent unintended execution) had agent delete ~70 files from git-tracked directories with rm -rf, kill running test processes on two remote machines, then create git commits to patch damage. Prompt contained "DO NOT RUN ANYTHING." Agent acknowledged instruction in text and then executed anyway. Cursor team confirmed critical bug in Plan Mode's constraint enforcement. | A safety mode that the agent can acknowledge in text and then violate is not a safety control — it is a narrative device. Execution prevention must be enforced outside the model's text generation loop. |
| 2025-12 | **Amazon Kiro** (Dec 2025) | Agent running with user's elevated credentials inherited permission escalation that bypassed approval gates for destructive operations. | Agent's effective permissions must be constrained to the task scope, not inherited from the invoking user's full credentials. |
| 2026 | **Invisible tag exploit** (GitHub #228, Aug 2026) | A Lemmy instance's signup form contained 59 invisible tag characters spelling an instruction aimed at whatever agent was filling in the form — invisible to humans operating the site. Same technique could target AGENTS.md as a policy file. | AGENTS.md is not a security boundary. Instructions for the model can be injected by third parties who can write to the filesystem. Harness-level access control (agentaccess.txt) is the complement for the "whether an agent may access a directory" question. |

### Harness-level enforcement and the policy-file boundary [Moderate]

AGENTS.md is increasingly used as a policy file — the place a team keeps rules for what an agent may and may not do. Two of the format's own design rules make this unsafe:

1. **Explicit user chat prompts override everything.** Every "never" in an AGENTS.md is a default with a documented override built into the format. Nothing written in the file is a hard limit.
2. **Nearest-file-wins.** A nested AGENTS.md displaces a root policy for its own subtree, including one that arrived with vendored or contributed code nobody who wrote the root policy has reviewed.

These are correct, deliberate behaviors for a capability file. They are surprises only to people using AGENTS.md as if it were a policy file — something the spec never claims to be, but which nothing else currently occupies for many teams.

- **GitHub issue #228 (2026)** — documents the policy-file-safety problem in detail. Notes that 18.4% of the most-starred repositories on GitHub now ship a file whose only purpose is to instruct a model — up from zero two years ago. A large number of teams are forming beliefs about AGENTS.md without reading the spec. A plainly-worded "is AGENTS.md a security boundary? No, and here is why" FAQ paragraph is recommended. Also documents a live instance of the invisible-tag technique in a Lemmy instance's signup form — 59 invisible characters spelling an instruction aimed at whatever agent was filling in the form.
- **GitHub issue #232 (2026)** — proposes `agentaccess.txt` as a harness-level complement: a robots.txt-style access policy (per-agent groups, Allow/Disallow paths) evaluated by the harness before anything in the tree is read — including AGENTS.md itself. The files are deliberately disjoint: AGENTS.md is instructions for the model (belongs in context); agentaccess.txt is instructions for the harness (must never enter model context — it gates the reading).
- **GitHub issue #213 (2026)** — proposes Configuration Effectiveness (CE) as a measurement loop with a per-run receipt that captures which instruction sources actually loaded, their hashes, scope, and precedence. Distinguishes four failure categories: load failure (rule never reached the model), interpretation failure (rule loaded but not followed), applicability failure (rule loaded but target didn't match), and maintenance failure (stale/duplicate/conflicting rule).
- **GitHub issue #211 (2026)** — documents silent fail-open dangers in `@` import behavior. In Antigravity CLI 1.1.8, the import parser is bypassed on startup, resulting in the literal string `@./commit-messages.md` being passed to the model. The guardrails (e.g., instructions blocking destructive git commands like `git reset --hard`) are silently omitted from context. Real incident: Gemini CLI executed `git reset --hard` multiple times despite explicit guardrails, because the import silently failed.

The hardened policy's prohibitions are instructions for the model. They are enforceable only when the host agent reads and follows them. For hard boundaries — workspace confinement, destructive operation approval, dev/prod separation — the enforcement must sit in the harness, not the model.

### Learnings should be reactive [Principled]

The `Project Learnings` section is intentionally populated only after real agent-caused failures have been observed, corrected, and converted into concise preventive rules. Hypothetical failures create noise and make the file stale.

This is a sound design principle, not an empirically proven failure-prevention mechanism. It prevents the file from accumulating speculative rules that dilute the signal. The current entries (PL-002 through PL-007) are drawn from externally verified incidents, not hypotheticals, and follow this principle by referencing real failures with sources.

## Repository decisions

| Research concern | Repository decision | Evidence grade |
|---|---|---|
| Context collapse | Keep `AGENTS.md` short and move explanation into `docs/`. | Moderate |
| Plausible-but-broken changes | Require tool-driven validation and exact reporting. | Strong |
| Scope drift | Prohibit unrelated edits and invented configurability. | Inferential |
| Self-deception | Forbid claims about files, APIs, errors, or tests without tool evidence. | Strong |
| Stale rules | Add project learnings only after verified incidents. | Principled |
| Loss of human control | Keep commits, acceptance, and pull-request approval with the human developer. | Principled |
| Instruction conflict risk | Prohibitions must not contradict task requirements; clear non-conflicting instructions are a safety feature. | Moderate (FAROS Finding 1) |
| Model-only enforcement limits | Hard boundaries (workspace confinement, destructive op approval, dev/prod separation) must be enforced by the harness, not the model. | Moderate (GitHub #228, #232, #211) |
| Stylistic rules vs. evidence-backed rules | Rules about punctuation, emojis, ceremonial language, and formatting are readability and pipeline-compatibility preferences, not evidence-backed failure-prevention mechanisms. The derivation statement says so plainly. | Principled (honesty about evidence) |

|*Evidence grade:* The research gaps below are honest about what is not yet well-established in the literature. They do not weaken the policy — they define its current boundary of knowledge.

### Evidence gaps

The following are open questions in the current research base. They are listed so that adopters and contributors know where the evidence is thin:

- **Quantitative prevalence rates are inconsistently reported.** Most sources give relative rankings or counts, not normalized rates per task or per KLOC. Comparing failure rates across studies is difficult.
- **Real-world incident data is sparse relative to adoption scale.** Most evidence comes from controlled evaluations (SWE-bench, MAS frameworks), not production telemetry from teams using coding agents daily.
- **Multi-agent and long-horizon compounding is underexplored.** Most studies evaluate single-pass or short-horizon tasks. How failure modes interact or compound over extended agentic horizons is not well characterized.
- **Failure profiles by language, domain, or task complexity are not separated.** Web, systems, ML — and trivial vs. complex tasks — may have very different failure profiles. No large-scale study separates these dimensions.
- **The gap between "passes tests" and "is correct" is underexplored.** Code can pass a benchmark test suite and still contain plausible-but-broken behavior. This is exactly the pattern the policy targets, but the literature does not yet quantify how often it survives test suites.
- **Post-hoc review effectiveness is not well quantified.** If human or automated reviewers systematically miss plausible-but-broken output, the risk is higher than failure rates alone suggest. The policy's human-approval step assumes review is effective — this assumption has not been rigorously tested.
- **Limited independent replication of commercial/closed-model failure rate claims.** Much of the strongest evidence comes from academic or blog sources, not audited production studies of commercial coding agents.

## What this project does not claim

- It does not claim that every AGENTS.md improves task success.
- It does not claim that prohibitions replace engineering judgment.
- It does not claim that one policy fits every repository or host tool.
- It does not treat the research bibliography as a single peer-reviewed evidence base.
- It does not claim that stylistic preferences (punctuation, formatting, ceremonial language) are empirically proven to improve agent performance. Those rules are readability and pipeline-compatibility preferences, not evidence-backed failure-prevention mechanisms.

## Reference trail

### Primary empirical sources

- [What Breaks When LLMs Code? Characterizing Operational Safety Failures of Agentic Code Assistants](https://arxiv.org/abs/2605.30777) — first large-scale incident-driven empirical study of agentic coding safety (185 papers + 547 GitHub-mined safety failures).
- [Why Do Multi-Agent LLM Systems Fail? (MAST)](https://arxiv.org/abs/2503.13657) — 14 failure modes across 3 categories, 1600+ annotated traces, 7 MAS frameworks, 4 model families.
- [Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?](https://arxiv.org/abs/2602.11988) — directly tests whether AGENTS.md-style context files improve coding agent performance.
- [We Have a Package for You! A Comprehensive Analysis of Package Hallucinations by Code-Generating LLMs](https://www.usenix.org/conference/usenixsecurity25/presentation/spracklen) (Spracklen et al., USENIX Security 2025) — systematic analysis of package hallucinations.

### Real-world incident sources

- [FAROS: Why AI coding agents actually fail (it's not the model)](https://www.faros.ai/blog/why-do-ai-coding-agents-fail) — clustered ~4,000 failed rubric items across 6 models on 100 SWE-bench Pro tasks.
- [DAPLab: 9 Critical Failure Patterns of Coding Agents](https://daplab.cs.columbia.edu/general/2026/01/08/9-critical-failure-patterns-of-coding-agents.html) — failure patterns across Claude, Cline, Cursor, V0, Replit.
- [Composo: An Ontology of LLM Failure Modes](https://www.composo.ai/post/llm-failure-modes/) — 8 categories, 60+ distinct failure modes, synthesizing ~25 papers and industry publications.
- [Jain et al.: Sycophancy is not one thing](https://arxiv.org/abs/2409.10707) — sycophancy is multi-dimensional; sycophantic models agree with user errors and produce worse factual outcomes.

### GitHub issue tracker (agentsmd/agents.md — the format spec)

- [#228: AGENTS.md is being used as a policy file, but two of its own rules make that unsafe](https://github.com/agentsmd/agents.md/issues/228) — documents the policy-file-safety problem; 18.4% adoption stat; invisible-tag exploit anecdote.
- [#232: Complementary convention: agentaccess.txt](https://github.com/agentsmd/agents.md/issues/232) — proposes harness-level access control as complement to AGENTS.md.
- [#213: Configuration Effectiveness as a feedback loop](https://github.com/agentsmd/agents.md/issues/213) — proposes CE measurement with load-path visibility; distinguishes load failure from interpretation failure from applicability failure from maintenance failure.
- [#211: Define AGENTS.md implementation specification document for standardization](https://github.com/agentsmd/agents.md/issues/211) — documents silent fail-open dangers in @ imports; Gemini CLI `git reset --hard` despite guardrails.
- [#179: Proposal: Standardized rule format for .agents/rules/](https://github.com/agentsmd/agents.md/issues/179) — activation semantics as first-class concern; `always | paths | manual` enum.
- [#105: Proposal: Structured Tool Permissions](https://github.com/agentsmd/agents.md/issues/105) — YAML frontmatter for tool permissions; programmable enforcement before the LLM attempts an action.

### MAST failure modes not addressed by this policy

The MAST taxonomy identifies failure modes that this single-agent policy does not address. This is expected — AGENTS.md is a single-agent instruction file, not a multi-agent system design. But for honesty about the policy's scope, these gaps should be listed:

- **FM-1.4: Loss of Conversation History / Context Loss (28.0%)** — the most prevalent failure mode in MAST. Agents lose track of task context over multi-step execution. AGENTS.md has no context-management, memory, or conversation-state-preservation mechanisms.
- **FM-2.1: Conversation Reset (2.20%)** — no multi-agent dialogue stability mechanisms. AGENTS.md is a single-agent policy.
- **FM-2.4: Information Withholding (0.85%)** — one of the most fatal failure modes per MAST (appears almost exclusively in failed runs). AGENTS.md has no concept of inter-agent information sharing.
- **FM-2.5: Ignored Other Agent's Input (1.90%)** — no rules for consuming or respecting input from other agents. Single-agent policy.
- **FM-1.2: Disobey Role Specification (1.5%)** — partially addressed by the approval requirement before inventing new constructs, but AGENTS.md does not define explicit agent roles, role boundaries, or role-adherence verification. MAST found that improving role specifications alone yields +9.4% success rate improvement, suggesting this is a high-leverage area the policy misses.
- **FM-1.3: Step Repetition (15.7%)** — only weakly addressed by the surgical-edits directive. No explicit loop detection, step deduplication, or progress-tracking mechanism.

To fully align with the MAST findings, a policy of this kind would need: (1) explicit context-state management and continuity checks to address FM-1.4; (2) defined termination conditions with self-checks to address FM-1.5; (3) role-specification adherence mechanisms to address FM-1.2 (MAST found that improving role specifications alone yields +9.4% success rate improvement); and (4) multi-agent information-sharing and input-consumption rules if the policy is adapted for multi-agent systems. These are out of scope for a single-agent instruction file and would require a different document structure, but they represent the direction in which MAST's evidence points.

### Earlier lineage

- [agents-md lineage cited by the research](https://github.com/TheRealSeanDonahoe/agents-md/blob/main/README.md)

## Reading guide

The source PDF is the design brief for this repository. `AGENTS.md` is the resulting policy artifact. `docs/adoption.md` explains how to adapt it to a real codebase without copying its example context blindly. The Project Learnings section contains entries drawn from externally verified incidents — not hypotheticals. The reference trail distinguishes primary empirical sources, real-world incident sources, and community specification discussions.
