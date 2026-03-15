# Copilot Instructions

This file provides guidance to GitHub Copilot when working with code in this repository.

## Project Overview

A Python data analysis project examining Cookie Cats (mobile game) A/B test data. The core business question: **does moving the progression gate from level 30 to level 40 hurt player retention?**

Dataset (`data/cookie_cats.csv`, 90,189 rows):
- `userid` — unique player ID
- `version` — `gate_30` (control) or `gate_40` (treatment)
- `sum_gamerounds` — total rounds played in 14 days post-install
- `retention_1` / `retention_7` — bool, whether user returned on day 1 / day 7

Additional metric definitions:
- `sum_gamerounds` — number of game rounds a player played during the first 14 days after installation
- `retention_1` — whether the player returned and played 1 day after installation
- `retention_7` — whether the player returned and played 7 days after installation (same meaning as descriptions labeled `retention_2`)

## Main Commands

This project uses [uv](https://docs.astral.sh/uv/). Use `uv` for all dependency and environment management.

```bash
uv run python main.py        # Run scripts
uv add <package>             # Add a dependency
uv sync                      # Install dependencies from lock file
uv run jupyter notebook      # Launch notebooks
```

Python 3.12 (enforced via `.python-version`).

## Folder Structure

```text
.
├─ CLAUDE.md
├─ README.md
├─ main.py
├─ pyproject.toml
├─ uv.lock
├─ data/
│  └─ cookie_cats.csv
├─ notebooks/
│  └─ 01_EDA.ipynb
└─ .github/
   └─ copilot-instructions.md
```

## Notebook Structure

Notebooks live in `notebooks/`. Data paths use relative `../data/` references.

- `01_EDA.ipynb` — complete EDA: group balance check, `sum_gamerounds` distribution (heavy right skew, median ≈ 17), retention rates by group and by rounds-played bucket. Korean-language commentary throughout.

**EDA findings so far:**
- A/B split is balanced (~50/50)
- No missing values or duplicate userids
- 1-day retention: gate_30 ≈ 44.8% vs gate_40 ≈ 44.2%
- 7-day retention: gate_30 ≈ 19.0% vs gate_40 ≈ 18.2%
- gate_30 outperforms gate_40 on both metrics; statistical significance TBD

**Next step:** hypothesis testing (Bootstrap and/or Chi-square) to determine if the retention difference is statistically significant.

## Dependencies

`pyproject.toml` currently lists `numpy` and `pandas`. Notebooks also use `matplotlib` and `seaborn` — add them if running notebooks in a fresh environment:

```bash
uv add matplotlib seaborn
```

## Coding Conventions

- Prefer Python 3.12-compatible syntax and libraries.
- Use `uv` for package and environment management.
- Keep notebook text/comments in Korean where existing analysis is Korean-language.
- In notebooks, use relative data paths like `../data/...`.
- Keep analysis reproducible and avoid changing raw dataset files.

## macOS Font Note

Notebooks set `plt.rcParams['font.family'] = 'AppleGothic'` for Korean characters in chart labels. This is macOS-specific and will need adjustment on other platforms.
