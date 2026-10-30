# Issues found and fixed

File: `tournament_scoreboard.py`. Line numbers refer to the **original** file.

Baseline: Pylint **4.75/10**, Bandit **1 High / 1 Medium / 2 Low**, Flake8 **4**.
After: Pylint **10.00/10**, Bandit **0**, Flake8 **0**. Program output unchanged.

| Issue | Type | Line(s) | Description | Fix Approach |
|-------|------|---------|-------------|--------------|
| B605 shell command injection (**High**) | Security | 71 | `os.system("scp table.json " + server + ...)` builds a shell command from the `server` argument, so `server = "x; rm -rf ~"` runs arbitrary commands. | Use `subprocess.run([...], check=True)` with an argument list and no shell. Validate `server` with a strict hostname regex (can't start with `-`, so no option injection). Resolve `scp` with `shutil.which`, which also clears the follow-up B607. |
| B306 insecure `mktemp` (**Medium**) | Security | 66 | `tempfile.mktemp()` only returns a name; another process can create or symlink that path before we open it (race condition). | Use `tempfile.mkstemp()`, which creates the file atomically with safe permissions; close the returned fd and return the path. |
| W0102 mutable default argument | Bug | 11-13 | `players=[]` is created once, so every team added without a player list shares the same list (adding a player to one team adds it to all). | Default to `None`; create a new list inside the function. Checked: appending to team A leaves team B empty. |
| B311 weak PRNG (Low) | Security | 61 | `random.choice` is predictable, not suitable where fairness/security matters. | Use `secrets.choice`. |
| B105 hardcoded password (Low) | Security | 6 | `REFEREE_PASSWORD = "referee-demo"` puts a credential in source control. | Read it from the `REFEREE_PASSWORD` environment variable. |
| R1714 / IndexError risk | Bug / Style | 58 | `teamName == table[0] or teamName == table[1]` crashes with `IndexError` when fewer than 2 teams exist. | `team_name in league_table()[:2]` (also what Pylint suggested). |
| R1705 no-else-return | Style | 17-22 | `elif`/`else` after `return` adds needless nesting. | Plain sequential `if` statements. |
| C0103 invalid-name (x17) | Style | many | camelCase functions/arguments (`addTeam`, `teamName`, `homeGoals`, ...) break PEP 8 snake_case. | Renamed all to snake_case (`add_team`, `team_name`, `home_goals`, ...). All call sites updated. |
| C0114 / C0116 missing docstrings (x12) | Docs | 1, 11, 16, ... | Module and every function lacked a docstring. | Added a one-line docstring to each. |
| Module-level `main()` call | Design | 92 | Importing the file runs the demo as a side effect. | Wrapped in `if __name__ == "__main__":`. |
| E203 | Style (Flake8) | 5 | `"win" : 3` whitespace before `:`. | `"win": 3` |
| E302 | Style (Flake8) | 60 | One blank line before `tossCoin`; PEP 8 wants two. | Added the blank line. |
| E231 | Style (Flake8) | 79 | `2,1` missing space after comma. | `2, 1` |
| E261 | Style (Flake8) | 87 | Inline comment needs two spaces before `#`. | Two spaces. |

## Verification

- Reports before and after are in `reports/before/` and `reports/after/` (latest copies also in the repo root).
- `python tournament_scoreboard.py` prints the same table, top-two result and match count as before (only the random coin toss differs).
- Two `# nosec` comments remain (B404 on `import subprocess`, B603 on the `subprocess.run` call). Both are justified inline: fixed, resolved binary, validated host, no shell.
