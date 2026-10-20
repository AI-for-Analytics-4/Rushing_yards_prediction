# Part 9: Reflection

**Juan Zavala** · AI for Analytics · Handling Missing Values with Claude

Each answer below is backed by results from `rushing_yards_imputation.ipynb`. The 16 missing values were one whole week (week 12), filled three ways:

* `stats2`: the overall mean;
* `stats3`: the team mean;
* `stats4`: a ridge regression model on `name`, `tm_score`, `opp_score` and `pass_yds`.

---

### 1. Do more complex null replacement methods always, usually, sometimes or rarely perform better than less complex ones?

**Sometimes.** Complexity helped when it added real information and kept noise under control, and it hurt when it didn't.

* **The model usually won.**
  * On week 12 it had the lowest error on both metrics: MAE 38.38 vs 48.88 for the overall mean, and MSE 1,972 vs 3,052.
  * Scoring every week the same way, it had the lowest error in **15 of 18 weeks**. Seventeen of those weeks are the same hidden-week results used to choose the model, so week 12 is the only week not used to choose the model, and the model won it too.
* **But the middle option was not reliably better than the simplest.**
  * The team mean is more complex than the overall mean, yet the two split the weeks: the team mean was better in **7 of 18 weeks** by MAE and **10 of 18** by MSE.
  * It was *worse* in cross-validation (39.79 vs 37.62).
  * It won on week 12 by just 2.5 yards, a gap that resampling can't separate from noise.
* **The most complex model was the worst.**
  * Gradient boosting (40.56 average error per hidden week) did worse than simply using the mean (37.62).
  * A random forest (37.02) barely beat the mean.
  * Plain linear regression (36.43) lost to ridge (34.09), even though both used the same four columns.

So the winner was neither the simplest nor the most complex method. It was a moderately complex model with a penalty that kept it from chasing noise.

### 2. In the most complex method we chose some columns to include and some to exclude. Why did we do this?

We kept columns that plausibly carry information about rushing, and dropped columns that can't help or would mislead.

* **Kept:**
  * `tm_score`: the strongest link to rushing (r = +0.31).
  * `pass_yds`: a negative link (r = −0.21); teams that throw more tend to run less.
  * `name` (team): real but modest signal (ANOVA p = 0.015).
  * `opp_score`: little on its own, but cheap context.
* **Left out:**
  * The row ID: it's just a label.
  * `week`: no relationship to rushing (r = 0.001). Also, all the missing games are week 12, which never appears in the training rows.
  * `opp_name`: no clear effect (p = 0.25), and it would add 32 more columns to learn from only 256 rows.
  * Anything computed from rushing yards: that would leak the answer. The file had no such column, but this always needs checking.
* **The test agrees.** Adding `week` or 10 columns of random noise made the cross-validated error worse (34.34 and 34.67 vs 34.09). Adding `opp_name` changed nothing (34.06).

### 3. Would Claude be good at differentiating which columns to include?

**Good at proposing columns and explaining the risks. Not reliable enough to trust without a test.** Before running any feature test, Claude wrote down five predictions:

| Result | Predictions |
|---|---|
| Right (3) | `tm_score` matters most; `pass_yds` helps; team columns "will add little, or even hurt a plain linear regression" |
| Mostly right (1) | Extra columns make it worse. True, except that `opp_name` changed nothing |
| Wrong (1) | `opp_score` would help alongside `tm_score`. It didn't (36.36 vs 36.10 for `tm_score` alone) |

Claude also correctly flagged the leakage risk and the problem of thin team samples. The wrong guess sounded perfectly reasonable, though, and only the cross-validation caught it. The right workflow is to let Claude suggest the candidates and write the test, and to let the test decide.

### 4. Would understanding relationships among columns help in choosing which to include? Is there a downside to too many columns, or should more data mean more accuracy?

**Yes, understanding the relationships helps.** Two examples from this data:

* `tm_score` and `pass_yds` are correlated with each other (r = 0.38), yet both helped, because they measure different things: how much a team scored, and how much it relied on passing.
* `opp_score` adds nothing once `tm_score` is in. The score margin's link to rushing (r = +0.27) is weaker than `tm_score`'s own (r = +0.31), so the margin's signal mostly comes from `tm_score`, and the opponent's points barely relate to rushing on their own (r = +0.02).

**Yes, there is a downside to too many columns.**

* Count what the model actually receives: the four assignment columns become 35 inputs (32 team columns plus 3 numbers). With plain linear regression, error rose with every addition: **36.43** with those 35 inputs, then 36.81 (+ `week`, 36), 37.73 (+ 10 noise columns, 45), 38.79 (+ `opp_name`, 67) and **40.19** with all of them (78).
* Every input is one more thing the model has to estimate from the same 256 rows. Inputs without real signal let it fit noise in the training rows.
* Ridge resisted this (34.06 to 34.88) because its penalty shrinks useless inputs toward zero, but it didn't make the extra columns helpful.
* The four assignment columns weren't all needed: `name + tm_score + pass_yds` scored 34.06, essentially tied with all four (34.09).

More *rows* usually mean more accuracy. More *columns* only help when they carry real signal.

### 5. If we didn't have the actual values, how could we use Claude to test accuracy?

**Hide values we already know, predict them, and score those predictions.** That's exactly what this project did before any code read `actual.csv`. (The file was attached to the chat with the data, so its contents were visible. That's why the methods and the selection rule were written down before any modeling.)

* **Hide whole weeks.** Each of the 17 known weeks was hidden in turn and predicted from the other 16 (leave-one-week-out cross-validation). This mirrors how the real data was missing.
* **It worked.** The estimates, committed to git before any code read the answer key, picked the right winner: the model ranked first on both metrics, as predicted.
* **Its errors were optimistic for this particular week.** Week 12 was hard to guess: a heavy rushing week whose games were also more spread out than most weeks (standard deviation 55.0 yards, 4th-highest of 18 weeks). Even the best possible single number for all 16 games (any value from 153 to 155 yards) would have scored an MAE of 45.62. Still, every method's week-12 error landed inside the range the hidden-week test had produced. For the model, that range was 22.4 to 49.6 yards, and week 12 came in at 38.4.

Other checks Claude can design and run without an answer key:

* **A second way of splitting the data**, to see whether the winner holds up. Random 5-fold cross-validation agreed on the winner: ridge 34.61, against 37.77 for the mean and 37.92 for plain linear regression.
* **Sanity checks on the filled values:** whole numbers, inside the range of real games, a sensible spread.
* **Resampling (bootstrap)** to show how uncertain a small test is.
* **Asking Claude to list the assumptions** behind each method, then testing those assumptions. Part 1 did this by checking whether the missing week differed from the others.

Claude can't *know* the accuracy without data, but it can quickly design and write the tests that estimate it.
