# Prompts used (Hermes Agent instead of GitHub Copilot)

All lab tasks that the manual assigns to Copilot were executed through **Hermes Agent**
(CLI), which wrote the files, ran the tests, created the repo, and drove the CI
experiment. Every file it produced was reviewed and verified by actually running
`pytest` before committing.

| # | Lab | Prompt (as given in the manual) | How it was executed with Hermes |
|---|-----|--------------------------------|---------------------------------|
| 1 | 1 | `# apply_discount(price, percent): return discounted price rounded to 2 decimals; raise ValueError if percent not in 0..100` (type comment, press Enter) | Given as a spec to Hermes; it generated `src/pricing.py` (matched the reference implementation) |
| 2 | 1 | "Write pytest tests for this file including boundary conditions and invalid input. Put them in tests/test_pricing.py." | Asked to Hermes with `src/pricing.py` as context; every assertion was then checked by hand against the reference and verified by running `pytest -v` |
| 3 | 1 | "ask Copilot for three tests (add stock, remove stock, removing too much raises ValueError)" | Asked to Hermes for `src/inventory.py`; saved as `tests/test_inventory.py` and verified by running pytest |
| 4 | 0 / Copilot basics | Select the error, then ask "Explain this error and suggest a fix" | Applied to the red CI run: the failed pytest log (`docs/responses/red_run_cli_log.txt`) was the selected error; explanation + fix in `docs/responses/responses.md` |
| 5 | 2 | "Explain each line of this workflow file." | Asked to Hermes about `.github/workflows/ci.yml`; full line-by-line explanation in `docs/responses/responses.md` |

Note on screenshots: the run-history, run-summary and job-steps pages were captured
from a browser. The expanded **job log pane** requires signing in to github.com in the
browser; saving a GitHub login in the browser session was declined, so the exact error
text is provided verbatim from the authenticated CLI (`gh run view --log-failed`) in
`docs/responses/red_run_cli_log.txt` instead of a screenshot.
