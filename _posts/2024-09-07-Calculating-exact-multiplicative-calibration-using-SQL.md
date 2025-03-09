---
layout: post
title: Calculating exact multiplicative calibration using SQL
tags: SQL
---

This post will talk about what exact multiplicative calibration (EMC) is, and how to calculate this metric using SQL.

### What is exact multiplicative calibration?

In the context of predictive modeling, we often think about prediction error from two different perspectives: bias and variance. Roughly speaking, bias is about if our predictions are systematically larger or smaller than the ground truth.

One way to think about bias is this: Given a set of predictions, is it possible to make it more accurate (defined using some loss function such as MSE or quantile loss) by simply applying _the same adjustments to all predictions_? (Well, you can always multipley each forecast by a _different_ constant to make it perfect! But it's trivial.) For example, if we add a constant $c$ or multiply a non-zero constant $k$ to all predictions, does that give us more accurate predictions?

![g_vs_pred]({{site.baseurl}}/assets/g_vs_pred.png)

![g_vs_adj_pred]({{site.baseurl}}/assets/g_vs_adj_pred.png)

In the example above, we see that the original forecasts are over-biased. They have a higher MSE of $13.8$, too. However, if we multiply every forecast by a constant $0.375$, then the calibration looks much better. The MSE also decreases to $2$.

So this ia _a_ better forecast. The natural follow-up question is, then, can we find an even better forecast? That is a fair question, and $0.375$ is no magic number. Obviously we should look for a constant that's smaller than 1 to adjust the over-biasness. We can do a grid search over the interval $(0, 1)$ with step size $0.025$.

Below is a plot showing different MSEs when we multiply predictions with different multiplier. It is quadratic (why?), and we see that multiply all predictions by $0.25$ will give us the smallest MSE, which is around $1.69$.

![mse_vs_m]({{site.baseurl}}/assets/mse_vs_m.png)

Formalizing the idea of grid search, exact multiplicative calibration (EMC) for this problem is a constant $c$ such that $\text{MSE} (\mathbf{y}, c\cdot \hat{\mathbf{y}})$ is minimized.

### How to calculate EMC using SQL?

Finding EMC analytically for a given metric can be difficult. Even for MSE, the analytical solution can be somewhat complicated. In practice, grid search is a powerful idea and can be used for other metrics such as MAE or quantile loss.

Data scientists often work with SQL tables. How do we calculate EMC using SQL? The idea is to first create a table of multiplier candidates:

```SQL
DECLARE
  smallest_m  FLOAT := 0.1;
  largest_m   FLOAT := 2.0;
  search_step FLOAT := 0.01;
-- We do a grid search over (0.1, 2.0) with step size of 0.01

BEGIN
  -- Create a 1-column table with all possible multipliers for grid search.
  DROP TABLE IF EXISTS grid_search_table;
  CREATE TEMP TABLE grid_search_table (
    m FLOAT
  );
  WHILE smallest_m <= largest_m
    LOOP
      INSERT INTO grid_search_table VALUES (smallest_m);
      smallest_m = smallest_m + search_step;
    END LOOP;
```

Once we have this table, we can _cross join_ it with another table `prod_tbl` containing both predictions as well as ground truth. In practice, we usually want to look at metric and EMC of certain subgroups. For example, if we are predicting housing prices, we might want to know if predictions for single family house are better-calibrated than the ones for townhomes. So we apply `GROUP BY` on those fields. The resulting table `main_calc_tbl` will be a much larger table than `prod_tbl` due to the CROSS JOIN.

```SQL
DROP TABLE IF EXISTS main_calc_tbl;
CREATE TEMP TABLE main_calc_tbl AS (
  SELECT
    prod_tbl.house_type,
    prod_tbl.prediction,
    prod_tbl.target,
    grid_search_table.m,
    -- We should also calculate the loss metric in this step.
    -- We calculate both the actual MSE as well as hypothetical MSE when predictions
    --   are multiplied by the multiplier m.
    AVG(POWER(prod_tbl.target - prod_tbl.prediction, 2)) AS actual_mse,
    AVG(POWER(prod_tbl.target - prod_tbl.prediction * grid_search_table.m, 2)) AS mse,
  FROM prod_tbl CROSS JOIN grid_search_table
  GROUP BY prod_tbl.house_type
);
```

Finally, we need to find which multiplier `m` gives us the smallest loss. To do this, we create a CTE with minimized loss, and then use this CTE to get the final result.

```SQL
DROP TABLE IF EXISTS emc_result;
CREATE TEMP TABLE emc_result AS (
  WITH h AS (
    SELECT house_type,
           MIN(actual_mse) AS actual_mse,  -- This doesn't really do anything, just keeping the actual MSE.
           MIN(mse)        AS mse  -- Minimum of all MSEs, each of which is based on a different multiplier.
    FROM main_calc_tbl
    GROUP BY house_type)

  SELECT house_type,
         h.actual_mse,
         h.mse           AS minimized_mse,
         main_calc_tbl.m AS emc
  FROM h
    JOIN main_calc_tbl USING (house_type, mse)
  );
```
