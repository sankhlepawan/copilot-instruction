# Copilot code review custom instructions

Drop these files into your repo at the paths shown. GitHub Copilot reads them
automatically when reviewing PRs.

```
.github/
├── copilot-instructions.md              # repo-wide, applies to every PR
└── instructions/
    ├── python.instructions.md           # applied to **/*.py
    ├── infra.instructions.md            # applied to Dockerfile, *.tf, k8s/**, helm/**
    └── security.instructions.md         # applied to auth/, security/, middleware/
```

## How it works

- Copilot reads `.github/copilot-instructions.md` first — these are global to the repo.
- Then it reads each `*.instructions.md` whose `applyTo:` globs match the files
  in the PR. Path-scoped rules layer on top of repo-wide ones.
- Instructions are read from the **base branch** of the PR (the target, not the
  source) — so changes to instructions only take effect after they merge.

## Hard limits to remember

- **4,000 character cap per file** for Copilot code review. Anything past that
  is silently ignored. Split into multiple path-scoped files rather than one
  giant repo-wide file.
- **Non-deterministic.** Copilot may not follow every instruction every time.
  Treat these as strong guidance, not a hard contract. If you need deterministic
  enforcement (block-merge-on-violation), use a CI scanner + gate stage.
- **Per-PR cost.** Each Copilot review on a PR consumes one Premium Request Unit
  from the reviewer's (or PR author's, for auto-reviews) monthly quota. From
  June 2026, runs also consume GitHub Actions minutes.

## Toggling on / off

Repo Settings → Code & automation → Copilot → Code review →
"Use custom instructions when reviewing pull requests". On by default.

## Testing your instructions

1. Open a draft PR that intentionally violates one of your rules
   (e.g. add a `print()` in production code, or a Dockerfile with `FROM ubuntu:latest`).
2. Add Copilot as a reviewer.
3. Confirm the comment cites your rule. If it doesn't, your instruction is
   probably either over the 4000-char limit, scoped to the wrong path glob,
   or phrased too vaguely.

## What instruction phrasing works best

- **Specific** beats general: "Flag `requests` library in async code, prefer `httpx`"
  beats "use modern HTTP libraries".
- **Actionable** beats descriptive: "Flag `eval` usage" beats "be careful with dynamic code".
- **Concrete severity labels** in the prompt help Copilot prioritize:
  `[must-fix]`, `[should-fix]`, `[nit]`.
- **Tell it what to skip**, not just what to flag. Otherwise you get noise on
  formatter-handled things.

## How this fits with the custom agentic pipeline

These two systems are complementary:

| Concern | Copilot custom instructions | Custom agentic pipeline |
|---|---|---|
| Best for | Style, conventions, lightweight best-practice nudges | Blocking security findings, dep CVEs, IaC misconfig |
| Enforcement | Suggestions only, never blocks merge | Can fail the pipeline / block merge |
| Determinism | Non-deterministic LLM output | Deterministic scanners + structured LLM JSON |
| Customization ceiling | 4000 chars per file, prompt-shaped | Full system prompt + custom Semgrep rules |
| Cost | Per-PR PRU + Actions minutes (from June 2026) | ~$0.02–0.10 per MR in API calls |

Use Copilot custom instructions as the chatty in-line reviewer that catches
style and convention drift. Use the custom pipeline as the gate that enforces
the rules you actually want to block on. They don't conflict — developers see
Copilot inline suggestions plus a single consolidated pipeline review.

## Resources

- Official docs: <https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot>
- Tutorial on effective instructions: <https://docs.github.com/copilot/tutorials/customize-code-review>
- Community examples: <https://github.com/github/awesome-copilot>


- learning
https://dev.to/vevarunsharma/injecting-ai-agents-into-cicd-using-github-copilot-cli-in-github-actions-for-smart-failures-58m8
