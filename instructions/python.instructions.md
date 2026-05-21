---
applyTo:
  - "**/*.py"
---

# Python-specific review standards

## Required

- Type hints on every public function (parameters and return).
- `from __future__ import annotations` at the top of new modules.
- Async code uses `httpx.AsyncClient`. Never `requests` (it's sync, will block the loop).
- Logging via `structlog.get_logger(__name__)`. Never plain `logging` in new code.
- Public dataclasses use `pydantic.BaseModel` or `dataclasses.dataclass(frozen=True)`.

## Disallowed

- `print()` outside CLI entrypoints (files under `cli/` or `if __name__ == "__main__"` blocks).
- Bare `except:` clauses. Catch specific exceptions.
- `assert` for runtime validation in non-test code — assertions are stripped under `-O`.
- Mutable default arguments: `def f(x=[])`, `def f(x={})`.
- `datetime.utcnow()` and `datetime.utcfromtimestamp()` — deprecated. Use
  `datetime.now(timezone.utc)` and `datetime.fromtimestamp(t, tz=timezone.utc)`.
- `os.path` in new code. Prefer `pathlib.Path`.
- `requirements.txt` edits without a corresponding `pyproject.toml` or lockfile update.

## Patterns to flag

- F-strings inside SQL queries, shell commands, or log messages with user input.
- Direct dict access on user-controlled input (`d["k"]`) — prefer `.get()` with explicit handling.
- Sync file I/O (`open()`, `Path.read_text()`) inside `async def` functions.
- Catching `Exception` and swallowing it without logging.
- Threading primitives (`threading.Lock`) inside async code — use `asyncio.Lock`.

## Tests

- New functions in `src/` should have corresponding tests in `tests/`.
- Test file naming: `test_<module>.py`.
- Use `pytest` fixtures over `setUp`/`tearDown` methods.
- Async tests must use `pytest-asyncio` markers.
