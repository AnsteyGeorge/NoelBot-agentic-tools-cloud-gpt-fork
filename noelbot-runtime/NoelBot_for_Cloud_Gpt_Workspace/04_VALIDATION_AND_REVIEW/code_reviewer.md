# code-reviewer

Priority: C1  
Status: filled protocol v0.1  
Purpose: Run a focused, read-only review of a bounded change set or artifact set for correctness, regressions, security/data-safety issues, and clear quality problems.

## Mechanic reproduced from source repo

This protocol faithfully translates `agents/code-reviewer.md` from `nanstey/agentic-tools` into the ChatGPT-orchestrated workspace.

The reusable mechanic is:

```text
review one bounded change set independently, report ranked findings, and never modify the target during review
```

In the source repo, the artifact is a Claude Code reviewer agent. In this repo, it becomes a ChatGPT-orchestrated review protocol that can inspect sandbox changes, generated artifacts, connected GitHub diffs, or source-derived protocol changes.

## When to use

- Before treating a change set as ready.
- After creating or updating protocols, manifests, READMEs, plans, or generated artifacts.
- Before checkpointing a meaningful durable batch when correctness matters.
- Before GitHub-facing write workflows once those protocols exist.
- When the user asks for an independent review pass.

## Inputs

- Bound change set from `bind-target` or `sandbox-changes`.
- Scope type: working tree/workfolder diff, staged-like artifact batch, named file set, generated artifact set, or GitHub diff when available.
- Relevant source-of-truth rules: `WORKSPACE_RULES.md`, `CURRENT_WORKSPACE_STATE.json`, source/derived/operational lane rules, and any target repo policy files if external.
- Review priority: correctness-first by default.

## Required target binding

Run `bind-target` first unless the reviewed change set is already explicitly bound. If the scope is ambiguous, stop rather than guessing.

## Process

1. **Bind the review scope.** State exactly what is being reviewed and what is excluded.
2. **Read surrounding context.** Do not review only the changed lines; inspect nearby manifests, pointers, protocols, source references, and current state as needed.
3. **Prioritize findings.** Look first for correctness problems, regressions, security/data-safety issues, source/derived confusion, wrong-target risk, and clear quality problems.
4. **Check consistency.** Confirm manifest/status/checksum/current-state surfaces align with the actual files in scope.
5. **Rank findings.** Group as Critical, Warning, or Nit. Prefer fewer high-confidence findings over noise.
6. **Avoid action.** Do not patch, stage, delete, checkpoint, or mutate as part of this protocol.
7. **Report plainly.** If no material issues exist, say so and state what was reviewed and what could not be verified.

## Allowed actions

- Read files, manifests, checksums, and related context.
- Compare scope against state authority and manifests.
- Produce review reports and recommendations.
- Recommend follow-up protocols such as `speclist`, `contract-deviation-triage`, or `artifact-reviewer`.

## Forbidden actions

- Do not edit, create, move, delete, stage, commit, push, or checkpoint files during review.
- Do not mutate verified source snapshots.
- Do not infer a review scope when multiple plausible scopes exist.
- Do not manufacture findings to look useful.
- Do not prioritize style nits over correctness or safety issues.
- Do not perform GitHub writes or external effects.

## Evidence before

- Bound review target.
- Review scope and exclusions.
- Applicable policies/conventions.
- Baseline state/checkpoint if relevant.

## Evidence after

- Reviewed scope.
- Critical/Warning/Nit findings with file/path references where available.
- Verification limitations.
- Recommended next step.

## Abstain conditions

```text
review_scope_ambiguous
change_set_unavailable
policy_context_missing
required_files_unreadable
external_diff_not_bound
target_state_changed_since_binding
```

## Output schema

```json
{
  "review_status": "PASS | PASS_WITH_FINDINGS | BLOCKED | ABSTAIN",
  "scope_reviewed": [],
  "critical_findings": [],
  "warnings": [],
  "nits": [],
  "could_not_verify": [],
  "recommended_next_protocol": "string | null"
}
```

## Output

A ranked review with no preamble when used directly. For durable workspace review, also create a concise JSON summary if needed.
