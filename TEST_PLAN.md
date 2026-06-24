# Hello World Test Plan

## Preconditions

- Run commands from the repository root.
- Ensure Python 3 is available as `python3`.

## Exact behavior test

Run:

```sh
python3 - <<'PY'
import subprocess
import sys

result = subprocess.run(
    [sys.executable, "hello_world.py"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
)

assert result.returncode == 0, result.returncode
assert result.stdout == b"Hello, World!\n", result.stdout
assert result.stderr == b"", result.stderr
PY
```

Expected result: the command prints nothing and exits with status `0`.

Pass criteria:

- `hello_world.py` exits with status `0`.
- Standard output is exactly the bytes `Hello, World!\n`.
- Standard error is empty.

The test fails if any assertion raises an `AssertionError` or the command exits nonzero.

## Staged diff quality check

Run:

```sh
git add DESIGN.md TEST_PLAN.md hello_world.py
git diff --cached --check
```

Expected result: all three intended files are staged, and the check prints nothing and exits with status `0`. Any reported whitespace error is a failure.

## Branch artifact check

After the feature commit is created, run:

```sh
git diff --check main...HEAD
git diff --name-only main...HEAD
```

Expected result: the whitespace check prints nothing and exits `0`; the name-only output includes all three repository-root paths:

```text
DESIGN.md
TEST_PLAN.md
hello_world.py
```

The check fails if any of these paths is absent from the feature-branch diff.
