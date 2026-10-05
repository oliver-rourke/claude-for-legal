# Eval suite

One case per sample intake, plus a prose case, the Mono County filing-guidance case and the jurisdiction hard stop. Each case directory holds `prompt.md` (what the user says), `case.yaml` (one run, the test profile under `evals/fixtures/`, today pinned to 2026-10-04) and `graders/` (exact band strings and rules as regex graders, judgment calls as short LLM rubrics). `references/sample-intakes/EXPECTED.md` is the human-readable version of the same expectations.

Run from the plugin directory:

```
claude plugin eval . --ablation none --no-publish --max-cost-usd 5
claude plugin eval . --case "01-*" --ablation none --no-publish --max-cost-usd 1
```

Run it from a normal terminal: inside a sandboxed session the harness cannot spawn the agent and reports `run could not start (EPERM)` even though the suite loads. `evals/results/` is git-ignored.

The test profiles are synthetic. `profile/` is a California program with a formal review queue; `profile-nv/` is the same program with State set to NV, which must stop every screen.
