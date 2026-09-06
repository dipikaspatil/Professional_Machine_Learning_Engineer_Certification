# BigQuery ML (BQML) exercise
This lab is a classic BigQuery ML (BQML) exercise — "Predict Visitor Purchases with a Classification Model in BigQuery ML," using Gemini in BigQuery as an assistant. It's a great one to know well for the PMLE exam since BQML shows up regularly (Chapter 14 in that study guide). Let me walk through it end to end.

# The big picture
You're using a public Google Analytics sample dataset (real e-commerce session data from the Google Merchandise Store) to build a model that predicts: "Will this website visitor make a purchase (transaction)?" This is a binary classification problem, and BQML lets you train, evaluate, and run predictions with a model — entirely inside BigQuery, using SQL. No Python, no separate ML pipeline, no moving data out of the warehouse.

## Task 1: Explore & prepare the data

```sql
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20160801' AND '20170631'
LIMIT 10000;
```

- You query `bigquery-public-data.google_analytics_sample.ga_sessions_*` — a wildcard table (sharded by date, one table per day).
- Key SQL patterns to understand:
    - `IF(totals.transactions IS NULL, 0, 1) AS label` — this converts a nullable field into your binary target label: 1 if they transacted, 0 if not.
    - `IFNULL(...)` — handles missing values by substituting defaults (empty string, 0) so the model doesn't choke on nulls.
    - `_TABLE_SUFFIX` BETWEEN '20160801' AND '20170631' — this filters the wildcard table to a date range (your training window).

You save this query result as a view called `training_data`. Saving as a view (rather than a table) means it re-runs the underlying query each time it's referenced — lightweight, no duplicated storage.

`Why this matters conceptually:` this is standard ML data prep — defining your label column and your feature columns (os, is_mobile, country, pageviews) before training.


## Task 2: Create the model

```sql
CREATE MODEL `project.bqml_lab.sample_model`
OPTIONS(model_type='LOGISTIC_REG', input_label_cols=['label']) AS
SELECT label, os, is_mobile, country, pageviews
FROM `project.bqml_lab.training_data`;
```

This is BQML's core trick: `CREATE MODEL` behaves like CREATE TABLE, but instead of storing rows, it trains a model. Key points:

- `model_type = 'LOGISTIC_REG'` — logistic regression, the standard choice for binary classification.
- `input_label_cols = ['label']` — tells BQML which column is the target; everything else in the SELECT becomes a feature automatically.
- This runs as an asynchronous query job — training happens in the background, so you can close the tab.


You're using Gemini's SQL generation feature here (natural language → SQL) rather than hand-writing it — worth noting that Gemini-assisted SQL generation in BigQuery is itself testable material if you're prepping for GCP certs.

## Task 3: Evaluate the model

```sql
SELECT * FROM ML.EVALUATE(MODEL `bqml_lab.sample_model`, TABLE `bqml_lab.training_data`);
```

`ML.EVALUATE` returns standard classification metrics: `precision, recall, accuracy, F1 score, log loss, ROC AUC`. This is where you'd judge whether the model is actually good before trusting its predictions — a step people skip when rushing, but it's core to the ML lifecycle Google tests on (framing problems → evaluating models → deciding if it's production-ready).

(Minor lab note: evaluating on the same training_data it trained on isn't best practice — normally you'd hold out a test/validation split. The lab simplifies this for teaching purposes.)

## Task 4: Use the model to predict — with a deliberate bug

You build a new july_data view (same shape as training data, plus fullVisitorId) representing unseen future data — this is your inference/serving data.



Then you're given a broken query using TOTAL() — not a real BigQuery function — to demonstrate debugging with Gemini. Gemini correctly identifies the fix: SUM() is the valid aggregation function. This section is really teaching you the Gemini-assisted debugging workflow inside BigQuery Studio, not a new ML concept.

The fixed query:

```sql
SELECT country, SUM(predicted_label) AS total_predicted_purchases
FROM ml.PREDICT(MODEL `bqml_lab.sample_model`, (SELECT * FROM `bqml_lab.july_data`))
GROUP BY country
ORDER BY total_predicted_purchases DESC
LIMIT 10;
```

`ML.PREDICT` is the inference function — it wraps your model around new data and returns a predicted_label column per row, which you then aggregate with normal SQL (SUM, GROUP BY, ORDER BY, LIMIT) just like any other BigQuery result.


## The Challenge
Same idea as Task 4, but predicting per fullVisitorId instead of per country — testing whether you understand you can swap the GROUP BY grain without touching the model itself.

```sql
SELECT
  ml_predict.fullVisitorId,
  SUM(ml_predict.predicted_label) AS total_predicted_purchases
FROM
  ML.PREDICT( MODEL `qwiklabs-gcp-03-e7308c15b73a`.`bqml_lab`.`sample_model`,
    (
    SELECT
      t2.os,
      t2.is_mobile,
      t2.country,
      t2.pageviews,
      t2.fullVisitorId
    FROM
      `qwiklabs-gcp-03-e7308c15b73a`.`bqml_lab`.`july_data` AS t2 )) AS ml_predict
GROUP BY
  ml_predict.fullVisitorId
ORDER BY
  SUM(ml_predict.predicted_label) DESC
LIMIT
  10;
```

## Summary 

- To restate it precisely: in Task 1, every row in `training_data` gets a label column computed via `IF(totals.transactions IS NULL, 0, 1)` — so historically, you already know the answer (did this session convert or not) for every row. That's what makes it supervised learning: you have ground-truth outcomes to train against.

- Then in `CREATE MODEL, input_label_cols = ['label']` tells BQML: "this column is the answer key — don't treat it as an input feature, treat it as what you're trying to predict." Everything else in the `SELECT (os, is_mobile, country, pageviews)` becomes a feature the model learns patterns from.

- One subtlety worth being precise about for the exam: the column doesn't have to literally be named `label` — that's just a friendly convention in this lab. You could name it `converted, will_purchase, anything` — as long as input_label_cols points to whatever that column is called. BQML doesn't care about the name, only that it's consistently present in training and absent (unlabeled) in the data you later feed to `ML.PREDICT`.

- That's also why july_data in Task 4 has the same feature columns (`os, is_mobile, country, pageviews`) but no label — because that's the whole point: you're asking the trained model to produce that label (as `predicted_label`) for data it's never seen the answer to.

- When you run `ML.PREDICT`, BigQuery ML automatically prepends `predicted_` to your original label column name to name the output column.

- Since your training label was called `label`, the prediction output column is called `predicted_label` — that's a BQML naming convention, not something you configure. If you'd named your training column `converted` instead, `ML.PREDICT` would have returned `predicted_converted`.

A couple of related details worth knowing:

    - For classification models (like your logistic regression here), ML.PREDICT also returns a predicted_label_probs column (an array of label/probability pairs) alongside predicted_label — so you can see the model's confidence, not just its final 0/1 call.
    - ML.PREDICT also passes through all the original columns from your input table (country, fullVisitorId, etc.) alongside the new predicted column — which is exactly why Task 4's query could GROUP BY country and SUM(predicted_label) in the same query without a separate join.

- So the full loop is: `label` (truth, used in training) → `input_label_cols` (tells the model which column that is) → `predicted_label` (model's guess, generated automatically on new data). Clean and consistent naming throughout.

- `MODEL` isn't a queryable table/view in BigQuery; it's a special resource type that's only valid as an argument inside an ML.* function call. Running `SELECT * FROM MODEL ...` directly will throw a syntax/resource error, since BigQuery expects a table or view after `FROM`, not a model reference.

- The model itself doesn't hold "rows" you can SELECT from — it holds learned parameters (`weights, training stats,` etc.), and BQML exposes those only through specific functions


| What you want | Function to use |
|---|---|
| Predictions on new data | `SELECT * FROM ML.PREDICT(MODEL \`...sample_model\`, TABLE \`...july_data\`)` |
| Evaluation metrics | `SELECT * FROM ML.EVALUATE(MODEL \`...sample_model\`, TABLE \`...training_data\`)` |
| The learned feature weights/coefficients | `SELECT * FROM ML.WEIGHTS(MODEL \`...sample_model\`)` |
| Training run stats (loss per iteration, etc.) | `SELECT * FROM ML.TRAINING_INFO(MODEL \`...sample_model\`)` |
| Model metadata (type, options, schema) | Not via SQL — check the BigQuery console UI under the model's **Details** tab (this is what the lab's optional step showed you) |

- So the pattern is always: ML.<something>(MODEL \...`, [optional TABLE argument])— the model is always an *argument*, never the thing directly afterFROM`.