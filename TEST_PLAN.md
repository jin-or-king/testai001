# Hello World Test Plan

## Preconditions

- Run commands from the repository root.
- Ensure Python 3 is available as `python3`.

## 精确行为测试 / Exact behavior test

Run:

```sh
python3 - <<'PY'
import re
import subprocess
import sys

result = subprocess.run(
    [sys.executable, "hello_world.py"],
    stdout=subprocess.PIPE,
    stderr=subprocess.PIPE,
)

assert result.returncode == 0, result.returncode
assert result.stderr == b"", result.stderr

lines = result.stdout.splitlines(keepends=True)
assert len(lines) == 2, result.stdout
assert lines[0] == b"Hello, World!\n", lines[0]
assert lines[1].endswith(b"\n"), lines[1]
assert re.match(rb"^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\n$", lines[1]), lines[1]
PY
```

Expected result: the command prints nothing and exits with status `0`.

Pass criteria:

- `hello_world.py` exits with status `0`.
- 标准输出第 1 行严格为 `Hello, World!\n`（与历史字节一致）。
- 标准输出第 2 行匹配正则 `^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}$` 并以换行结尾。
- 标准输出总行数为 2，且整体以换行结尾。
- Standard error is empty.

The test fails if any assertion raises an `AssertionError` or the command exits nonzero.

## Staged diff quality check

Run:

```sh
git add DESIGN.md TEST_PLAN.md PROVENANCE.md hello_world.py
git diff --cached --check
```

Expected result: all four intended files are staged, and the check prints nothing and exits with status `0`. Any reported whitespace error is a failure.

## Branch artifact check

After the feature commit is created, run:

```sh
git diff --check origin/main...HEAD
git diff --name-only origin/main...HEAD
```

Expected result: the whitespace check prints nothing and exits `0`; the name-only output includes all four repository-root paths:

```text
DESIGN.md
TEST_PLAN.md
PROVENANCE.md
hello_world.py
```

The check fails if any of these paths is absent from the feature-branch diff.
