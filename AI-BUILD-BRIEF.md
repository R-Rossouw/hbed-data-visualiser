# AI build brief — hbed-data-visualiser Phase 2

*Hand this file to the AI agent (Claude builder agent, Codex, or any capable coding agent) that will implement Phase 2. It defines the working contract. The owner is Caroline (not a programmer — explain decisions at product-owner level, show working).*

## Kickoff prompt (copy-paste to the agent)

> Read, in order: `README.md`, `TODO.md`, `PLAN-V2.md`, `SPEC-V2.md`, then `index.html`.
> Implement **one feature at a time** from SPEC-V2 in its stated order (P1-1 first). After each feature: run the fixture checks in `test.html`, verify v1 views still work, commit with a clear message, and STOP for review before the next feature.
> Hard rules: client-side only, no dependencies, no build step, no network calls; never commit a CSV or any real site/train identifier; the `LIMITS` defaults are untouchable; if the spec is ambiguous or conflicts with the code, reply `ESCALATE:` with the question instead of guessing.

## Working rules for the agent

1. **One feature per session/PR.** Small, reviewable, always-working increments.
2. **Acceptance criteria are the definition of done** — each SPEC-V2 feature lists them. Build the synthetic fixtures FIRST, watch them fail, then implement.
3. **Don't refactor v1 beyond what the feature needs.** Splitting `index.html` into ES modules is allowed when a P1-3 or later feature genuinely needs it; do it as its own commit with zero behaviour change.
4. **Confidentiality:** this is a PUBLIC repo. Synthetic data only. If you ever find a real export or an internal system/site identifier in the working tree, stop and flag it — do not commit.
5. **Escalate, don't guess:** anything ambiguous → `ESCALATE:` + the specific question.

## Review checklist (for the human or reviewing agent, after each feature)

- [ ] Feature meets every acceptance bullet in SPEC-V2
- [ ] `test.html` fixture checks all PASS
- [ ] v1 views unchanged (thermal strip, ΔT, profiles, scatter, ranking, print)
- [ ] No new dependencies, no network calls (check DevTools network tab: zero requests after load)
- [ ] `git diff` contains no CSV, no real identifiers
- [ ] Advisory wording present on pattern-read and what-if outputs
