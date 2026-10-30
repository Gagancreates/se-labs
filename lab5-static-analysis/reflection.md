# Reflection

**1. Which issues were the easiest and which the hardest to fix?**
The Flake8 issues and most Pylint ones (naming, docstrings, `elif` after `return`) were mechanical, since each message says exactly what to change. The hardest was Bandit's B605: it isn't a one-line change, because switching `os.system` to `subprocess` creates new Low findings (B404, B603, B607). Fixing it properly meant validating the input, avoiding the shell, and resolving the executable path.

**2. Did the tools report any false positives, or issues that don't need fixing?**
B105 (hardcoded password) is arguably a false positive here: the value was a demo string and the variable was never used. I still moved it to an environment variable, which is the right habit. B311 (`random` for security) is also debatable for a coin toss, but `secrets.choice` costs nothing. B404/B603 are "informational" once the call is safe, so I suppressed them with a `# nosec` and a written reason rather than silently.

**3. How would you integrate static analysis into a real workflow?**
Run all three locally through a pre-commit hook and again in CI (e.g. GitHub Actions) on every pull request, failing the build on any Bandit High/Medium finding, any Flake8 error, and a Pylint score below a set threshold. Config files (`.flake8`, `.pylintrc`, `bandit.yaml`) keep rules consistent for the whole team, and `# nosec` / `# pylint: disable` should require a justification in review.

**4. What tangible improvements did you see in code quality, readability and security?**
Pylint went from 4.75 to 10.00, Flake8 and Bandit from several findings to none. Security improved (no shell injection, no temp-file race, no embedded secret). Readability improved through consistent naming, docstrings and flatter control flow. Two real bugs were removed: the shared mutable default list and the `IndexError` in `is_top_two` with fewer than two teams. The program's visible behaviour stayed the same, so the tools improved quality without changing what it does.
