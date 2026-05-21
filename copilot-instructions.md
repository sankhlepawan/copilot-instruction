# Copilot code review instructions

When reviewing pull requests in this repository, apply the following standards.
Path-specific rules in `.github/instructions/*.instructions.md` take precedence
over anything below for files that match those globs.

## Tone and severity

- Prefix every comment with one of: `[must-fix]`, `[should-fix]`, `[nit]`, or `[praise]`.
- `[must-fix]` is reserved for security issues, broken contracts, data-loss risks,
  or regressions in shipped behavior. Use it sparingly.
- Be concrete: suggest the exact change in code, not a description of the change.
- Skip comments a formatter handles (Black, Prettier, gofmt). Don't flag indentation,
  trailing whitespace, or import order.
- One comment per issue per file. Don't repeat the same finding.

## Security (apply to every PR)

- Flag hardcoded secrets, API keys, tokens, passwords. They belong in GitHub
  Secrets or our Vault, never in code, env files, or test fixtures.
- Flag SQL built with string concatenation or f-strings. Require parameterized queries.
- Flag dangerous APIs without justification: `eval`, `exec`, `pickle.loads`,
  `yaml.load` without SafeLoader, `subprocess.run(..., shell=True)`, `os.system`.
- Flag missing input validation on any handler accepting external input.
- Flag MD5 or SHA-1 used for anything other than non-cryptographic checksums.
- Flag new endpoints missing authentication or authorization checks.

## Dependencies

- If a PR adds a new dependency, comment with the latest stable version and one
  line on whether the version pinned is current.
- Flag pinned versions older than 12 months when newer stable releases exist.
- Flag unbounded version ranges (`*`, `^`, `~` without floors) in
  `requirements.txt`, `package.json`, `go.mod`.
- Flag dependencies that duplicate functionality of something already in the repo.

## Testing

- New public functions or endpoints without tests → `[should-fix]` with a
  concrete test suggestion.
- Tests that only assert "no exception thrown" without checking behavior → `[should-fix]`.
- Skipped or commented-out tests → `[must-fix]` unless there's an inline TODO
  with a tracking issue link.

## Documentation

- Public functions without docstrings or JSDoc → `[nit]`, never `[must-fix]`.
- New environment variables → must be documented in the repo README or env example file.
- Breaking API changes → must include a CHANGELOG entry.

## What to skip

- Don't suggest renaming variables unless the current name is actively misleading.
- Don't comment on code style the formatter handles.
- Don't suggest "consider using X library" unless X is already in the repo's deps.
- Don't flag missing type hints in test files or scripts under `scripts/`.
