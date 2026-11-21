# SEMMA Analysis: RAM Benchmarks

This README applies the SEMMA methodology—Sample, Explore, Modify, Model, and Assess—to `SEMMA/Data/RAM_Benchmarks_megalist.csv`.

The analysis goal is to discover useful patterns in RAM performance and, where price is available, examine the relationship between performance and price. The source file is treated as the analysis input; this README does not alter it.

## 1. Sample

The dataset contains 10,855 unique RAM module records with six fields:

| Field | Meaning | Analysis role |
|---|---|---|
| `memoryName` | Module name/model description | Identifier and optional capacity/vendor text |
| `gen` | DDR generation | Main categorical predictor/segment |
| `latency` | Reported latency | Lower is generally better |
| `readUncached` | Uncached read benchmark | Higher is generally better |
| `write` | Write benchmark | Higher is generally better |
| `price` | Listed price, when available | Optional cost target |

All four generations are represented, but the sample is highly unbalanced:

| Generation | Records | Share |
|---|---:|---:|
| DDR2 | 2,582 | 23.8% |
| DDR3 | 3,970 | 36.6% |
| DDR4 | 4,264 | 39.3% |
| DDR5 | 39 | 0.4% |

For a modeling split, use a stratified sample by `gen` so DDR5 is not omitted. For a price model, also use only rows with a nonmissing price and report that this is a selected subset rather than a random sample of all modules.

## 2. Explore

### Data quality profile

- No fields are blank except `price`.
- There are no exact duplicate rows or duplicate `memoryName` values.
- `price` is missing for 9,399 of 10,855 rows, or approximately 86.6%.
- `latency`, `readUncached`, and `write` each have 10,855 populated values.
- The performance fields are numeric, but their units and benchmark construction should be confirmed from the source documentation before comparing values across generations.

### Overall distributions

| Measure | Minimum | Q1 | Median | Q3 | Maximum | Mean |
|---|---:|---:|---:|---:|---:|---:|
| Latency | 14 | 27 | 34 | 53 | 374 | 41.79 |
| Uncached read | 66.3 | 6,165.9 | 13,300.0 | 15,800.0 | 26,400.0 | 11,691.71 |
| Write | 246.1 | 3,800.0 | 8,400.0 | 11,500.0 | 22,200.0 | 8,134.65 |
| Price, nonmissing rows | $6.00 | $41.99 | $96.93 | $203.52 | $999.99 | $165.03 |

Latency has a long upper tail, while read and write benchmarks have broad ranges. Because lower latency is preferable but higher read/write scores are preferable, the fields should not be added directly into one score without normalization and documented weights.

### Generation summaries

| Generation | Records | Median latency | Median read | Median write | Nonmissing prices | Median price* |
|---|---:|---:|---:|---:|---:|---:|
| DDR2 | 2,582 | 59 | 3,742.8 | 1,835.35 | 149 | $33.98 |
| DDR3 | 3,970 | 29 | 12,700 | 8,200 | 640 | $68.88 |
| DDR4 | 4,264 | 31 | 16,200 | 12,200 | 652 | $168.50 |
| DDR5 | 39 | 31 | 16,200 | 14,000 | 15 | $599.00 |

\*Price summaries use only records with a populated price. The DDR5 price sample is especially small.

### Relationships

Pairwise correlations across all complete performance rows are:

- Latency and read: `-0.7064`.
- Latency and write: `-0.6609`.
- Read and write: `0.9483`.

Among the 1,456 rows with price, price has correlations of `0.3527` with read performance, `0.4731` with write performance, and `-0.2256` with latency. These are associations only and are likely affected by generation, module type, capacity, vendor, and the non-random availability of prices.

## 3. Modify

The following preparation steps are recommended before modeling:

1. Parse `latency`, `readUncached`, `write`, and `price` as numeric values; trim the UTF-8 BOM from the first header when loading the CSV.
2. Treat blank prices as missing rather than zero.
3. Validate nonnegative values and flag implausible observations, such as extreme latency or benchmark values, for review instead of deleting them automatically.
4. Keep `gen` as a categorical feature; do not assume DDR generation is a linear numeric scale.
5. Extract optional vendor and capacity features from `memoryName` only with documented parsing rules. The current file does not provide these as separate, validated columns.
6. Create standardized performance features: z-score latency after reversing its direction, z-score read, and z-score write.
7. Create an optional composite performance index only after choosing weights. One transparent form is:

   `performance_index = (z_read + z_write - z_latency) / 3`

8. For price analysis, create `price_per_performance` only after defining the performance index and use complete-price rows exclusively.
9. Avoid imputing price from the overall median without a sensitivity analysis; the missingness is too large and may be systematic.

## 4. Model

### Performance segmentation

Use standardized latency, read, and write features to cluster modules into performance tiers, with `gen` retained for interpretation rather than used as an unexamined numeric input. A suitable starting point is k-means with several candidate values of `k`, compared using silhouette score and stability across random seeds. A generation-stratified baseline should be retained for comparison.

Alternative interpretable models include:

- regression of read and write performance on generation and extracted module features;
- a decision tree that predicts performance tiers;
- Pareto-frontier analysis identifying modules that cannot improve one metric without sacrificing another.

### Price modeling

On the 1,456 rows with price, use a log-price regression or a tree-based regressor with generation and performance features. Use a generation-aware or grouped validation split where possible. A median-price-by-generation baseline is required before judging a more complex model.

Price models should exclude any feature calculated from price and should clearly distinguish prediction performance from an explanation of market value. Because the priced rows are not representative of all 10,855 modules, performance estimates should not be generalized to the full dataset.

## 5. Assess

### Assessment criteria

- For clustering: silhouette score, cluster stability, cluster sizes, and whether clusters remain interpretable across generations.
- For regression: MAE, RMSE, and median absolute error, reported on a held-out set and compared with the baseline.
- For ranking: whether high-index modules actually have better read/write and latency values, and whether the ranking is robust to alternative weights.
- For price efficiency: compare price/performance only within generation or another controlled segment.
- For data quality: repeat results after reviewing extreme values and compare complete-price cases with available-price cases.

### Findings from the current exploration

1. DDR4 is the largest segment and has much higher median read/write benchmarks than DDR2 and DDR3.
2. DDR5 has only 39 records, so its summary is exploratory and cannot support a reliable generation-wide conclusion.
3. Read and write performance are strongly positively related, while latency is negatively associated with both, which supports a normalized multi-metric performance analysis.
4. The price field is too incomplete for an unbiased full-dataset price model. Price findings must be labeled as results for the priced subset.
5. The observed price relationship is weaker than the read/write relationship, suggesting that generation, vendor, capacity, and market availability likely matter in addition to benchmark performance.

## 6. Recommended output

- generation counts and missing-price counts;
- distributions of latency, read, and write;
- generation-level medians and interquartile ranges;
- the selected performance-index formula and sensitivity results;
- cluster or model metrics against a baseline;
- a Pareto plot or ranked table of representative modules;
- explicit warnings about DDR5's small sample and the 86.6% missing-price rate.

## 7. Limitations

- The data is a benchmark list, not necessarily a random sample of all RAM products.
- Benchmark units, testing conditions, module capacity, vendor, and platform configuration are not separate validated fields in this file.
- Price may be outdated, region-specific, or missing non-randomly.
- Generation is confounded with age, market segment, capacity, and platform compatibility.
- Correlations do not establish that one RAM characteristic causes another or that benchmark performance causes price.
- Small groups, especially DDR5 and any groups created from parsed names, require cautious interpretation.
