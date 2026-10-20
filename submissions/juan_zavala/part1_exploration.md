# Part 1: Getting Started with the New Data (Discussion Before Code)

**Data:** `data/season_stats.csv` holds 272 games. Each row is one team's game, with columns `week`, `name` (team), `opp_name`, `tm_score`, `opp_score`, `rush_yds` and `pass_yds`, plus a leading unnamed ID column (12 to 283). `actual.csv` is the answer key and is kept for Part 7.

**How I set up the conversation with Claude** (as the assignment asks):

> "I'm working in Python with pandas. Before any code is written, I want to discuss the process. Keep your answers short; a paragraph is usually enough. Here is `season_stats.csv`: tell me about any nulls in the data."

> "Do the rows that contain nulls have anything in common?"

Every number below was produced by code that reads only `season_stats.csv`. The notebook re-runs the same checks in its Part 1 section.

---

## Are there nulls?

Yes, but only in one column.

| Column | Nulls |
|---|---:|
| `rush_yds` | **16 of 272 (5.88%)** |
| every other column | 0 |

`rush_yds` is stored as a decimal (`float64`) only because of the nulls. All 256 known values are whole numbers, from 29 to 350 yards. That's why the assignment asks us to keep the column an integer after filling it: a fraction of a yard isn't a real stat.

## Do the rows with nulls have anything in common?

**Yes: all 16 are week 12, and every week-12 game is missing.** No other week has a single null. This isn't scattered missing data. One whole week of rushing totals was never recorded.

Two more facts about the file matter for every method:

- **Every row is the winning team.** `tm_score` is higher than `opp_score` in all 272 rows, with no ties. The file is one row per game, from the winner's side. The 16 missing games are wins too, so a team's average from its other wins is a like-for-like comparison.
- **Each team has a different number of rows**, from 2 games (Panthers) to 13 (Ravens). The 16 teams missing a week-12 value have between 5 and 12 known games each, so every one of them has a team average available.

## Are the week-12 games unusual in other ways?

No. On every other column, week 12 looks like the rest of the season:

| Column | Week 12 average | Other weeks | p-value (Welch t-test) |
|---|---:|---:|---:|
| `tm_score` | 26.44 | 27.46 | 0.686 |
| `opp_score` | 14.88 | 16.21 | 0.455 |
| `pass_yds` | 233.94 | 249.42 | 0.350 |
| score margin | 11.56 | 11.25 | 0.895 |

Rushing yards also don't trend across the season. The Spearman correlation between week and `rush_yds` is +0.025 (p = 0.69), and a Kruskal-Wallis test across weeks gives p = 0.12.

**What kind of missingness is this?** It depends only on the week, and the week is unrelated to rushing yards and to the other columns. So filling week 12 from the other 17 weeks shouldn't push the estimates up or down. One limit to be honest about: without the true values, we can't fully rule out that something about week 12's rushing *itself* caused the gap. Part 8 checks this against the answer key.

## What could help predict the missing values?

These are correlations with `rush_yds` across the 256 known rows. They are a preview of what the Part 6 model can work with.

| Column | Pearson r | p-value |
|---|---:|---:|
| `tm_score` | **+0.312** | < 0.0001 |
| score margin | +0.273 | < 0.0001 |
| `pass_yds` | **−0.209** | 0.0008 |
| `opp_score` | +0.020 | 0.75 |
| `week` | +0.001 | 0.99 |

What the table says:

- **`tm_score`** has the strongest link: teams that score more tend to rush more.
- **`pass_yds`** has a negative link, which fits the idea that teams lean on either the run or the pass.
- **Team (`name`)** matters too. Team averages differ more than chance would explain (ANOVA p = 0.015). In-sample they account for about 19% of the variation, an optimistic figure because some teams have very few games.
- **Opponent (`opp_name`)** shows no clear effect (ANOVA p = 0.25).

These are modest relationships, so no method will predict rushing yards closely. That shapes the expectations below.

**Leakage check:** no column is computed from rushing yards. There's no "total yards" column, for example, so nothing hands the answer to the model.

---

## The plan (agreed before writing code)

| Part | Method | How it fills the 16 nulls | Expected strength | Expected weakness |
|---|---|---|---|---|
| 4 | **Overall mean** (`stats2`) | The average of the 256 known values, rounded to a whole number | Simple; doesn't skew the average when the gap is unrelated to the values, which matches what we found above | All 16 games get the same number; ignores team and game |
| 5 | **Team mean** (`stats3`) | Each team's average over its own known games, rounded | Uses team identity, which does carry signal | Based on only 5–12 games per team; ignores the game itself |
| 6 | **Model** (`stats4`) | A scikit-learn model using `name`, `tm_score`, `opp_score` and `pass_yds` | Uses game context, like a high-scoring win | More moving parts; can overfit with only 256 rows and 32 teams |

**How the model will be chosen, before any answer key is used:** with leave-one-week-out cross-validation on the 256 known rows. Each of the 17 known weeks is hidden in turn and predicted from the other 16. That copies the real situation exactly, where one whole week is missing. The candidates and the rule were fixed in advance:

- **Candidates:** linear regression, ridge regression, random forest and gradient boosting, all using the four assignment columns.
- **Rule:** pick the lowest cross-validated mean absolute error. If another candidate is within one standard error of the best, take the simpler one.

**A note on the answer key:** `actual.csv` was attached to the chat with `season_stats.csv`, so its contents were visible. To keep the test fair:

- The methods and the selection rule above were written down before any modeling.
- The model is chosen only from known rows.
- The expected errors are committed to git *before* any code reads `actual.csv`, which first happens in Part 7. The git history shows the order.
