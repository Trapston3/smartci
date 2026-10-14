# Hermes responses

## 1. Generated `src/pricing.py` (Lab 1)

Hermes produced exactly the reference implementation:

```python
def apply_discount(price, percent):
    if not 0 <= percent <= 100:
        raise ValueError('percent must be between 0 and 100')
    return round(price * (1 - percent / 100), 2)


def add_tax(price, rate=0.18):
    return round(price * (1 + rate), 2)
```

## 2. Every assertion checked by hand (Lab 1)

| Assertion | Hand check | Result |
|---|---|---|
| `apply_discount(100, 10) == 90.0` | 100 × (1 − 0.10) = 90.0 | OK |
| `apply_discount(100, 0) == 100.0` | 100 × (1 − 0) = 100.0 | OK |
| `apply_discount(100, 100) == 0.0` | 100 × (1 − 1) = 0.0 | OK |
| `apply_discount(100, 101)` raises ValueError | 101 > 100 → raise | OK |
| `add_tax(100) == 118.0` | 100 × 1.18 = 118.0 | OK |
| `Cart().total() == 0` | sum of no items = 0; apply_discount(0,0)=0; add_tax(0)=0 | OK |
| `Cart().add('pen',50,2); total(discount=10) == 106.2` | subtotal 100 → discount 90 → tax 90×1.18 = 106.2 | OK |
| inventory add/remove/over-remove | add 5→5; remove 2→3; remove 5 of 2 → ValueError | OK |

Real run (`.venv`, Python 3.12.10): **9 passed in 0.08s** — no hallucinated assertions.

## 3. Red-run error explained + fix (Copilot-basics replacement)

The selected error (from `docs/responses/red_run_cli_log.txt`):

```
E       assert 100.0 == 101.0
tests/test_pricing.py:11: AssertionError
FAILED tests/test_pricing.py::test_discount_boundaries - assert 100.0 == 101.0
```

Explanation: the test asserted `apply_discount(100, 0) == 101.0`, i.e. it claimed a
0% discount should produce 101.0. `apply_discount(100, 0)` correctly returns
100 × (1 − 0/100) = 100.0, so the *assertion* (not the function) was wrong. The fix
was reverting the assertion to the correct expected value `100.0`. Commit
`6619ef0` introduced the break; commit `9ebe14d` (Revert) fixed it, and CI went
green again (run CI #4).

## 4. "Explain each line of this workflow file" (`.github/workflows/ci.yml`)

| Line | Meaning |
|---|---|
| `name: CI` | Label shown in the GitHub Actions tab. |
| `on: [push, pull_request]` | The **event** that triggers the workflow: every push and every opened/updated pull request. |
| `jobs:` | A workflow contains one or more jobs; they run in parallel by default. |
| `build:` | Our job id (a name we chose). |
| `runs-on: ubuntu-latest` | The **runner**: GitHub provisions a fresh Ubuntu VM for this job, which is why every run reinstalls dependencies (slow) and why caching exists. |
| `steps:` | Ordered list of steps inside the job. |
| `- uses: actions/checkout@v4` | Ready-made action that downloads the repo onto the runner. |
| `with: fetch-depth: 0` | Downloads the full git history (needed in Lab 4 for `git diff`). |
| `- uses: actions/setup-python@v5` | Ready-made action that installs the given Python on the runner. |
| `with: python-version: '3.12'` | Pins Python 3.12 to match our local venv. |
| `- run: pip install -r requirements.txt` | A shell step: installs pytest + requests on the fresh VM. |
| `- name: Run tests` | Human-readable label for the test step. |
| `run: \|` | Multi-line shell script follows. |
| `set -o pipefail` | Makes the step fail if pytest fails, even though its output is piped to tee (without it the exit code of `tee` would mask the failure). |
| `pytest 2>&1 \| tee build.log` | Runs the tests, shows output live **and** saves it to `build.log`. |

## 5. What is the use of a red run? (Lab 2 question)

A red run is CI doing its job: it is **objective, enforced evidence that this exact
commit is broken**, caught before a human ever ran the code.

- It converts "tests failed somewhere on a laptop" into a public, permanent record
  attached to the commit/PR — the team cannot miss it, and the branch is known-bad.
- It pinpoints *where* to look: the failed step (`Run tests`), the failing test
  (`test_discount_boundaries`), and the exact assertion (`100.0 == 101.0`) are in the
  log, so debugging starts at the answer.
- It **blocks bad code from being trusted**: red = do not merge/deploy. The repo has
  a truthful history: green before, red for the broken commit, green after the fix.
- In this experiment the red run caught a wrong *expectation* in a test rather than a
  bug in the app code — also valuable: tests are code too, and CI guards them the
  same way.
