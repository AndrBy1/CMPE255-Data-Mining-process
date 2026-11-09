# KDD: AI Jobs and Salaries

This document applies the Knowledge Discovery in Databases (KDD) process to `KDD/Data/ai_jobs_salaries_clean.csv`. The files under `KDD/Data` are treated as read-only source and supporting artifacts; this analysis does not modify them.

## 1. KDD objective

The objective is to discover useful and interpretable patterns in AI-related job salaries. The primary discovery variable is `salary_in_usd`, with job characteristics, experience, employment type, work mode, company size, location, and time used as explanatory dimensions.

The results are intended for exploratory career analysis, workforce planning, and compensation benchmarking. They should not be treated as a salary calculator for an individual offer because the data is self-reported and coverage is uneven.

### Questions

1. How are salaries distributed overall and across experience levels?
2. Which role families and employment types have the highest typical salaries?
3. How do work mode, company size, year, and company location relate to salary?
4. Which records should be treated as outliers or low-confidence segments?

## 2. Selection

The selected dataset is `KDD/Data/ai_jobs_salaries_clean.csv`. The accompanying schema, dictionary, and source information are in `KDD/Data/SCHEMA.md`, `KDD/Data/data_dictionary.md`, and `KDD/Data/README.md`.

The dataset contains 71,913 records and 17 columns covering work years 2020–2025. `salary_in_usd` is selected as the comparable target because the original records contain multiple currencies. The following fields are used as analysis dimensions:

- `work_year`
- `experience_level_label`
- `employment_type_label`
- `job_title` and `role_family`
- `work_mode` and `remote_ratio`
- `company_size`
- `company_location`
- `employee_residence`

The raw salary and currency fields remain useful for provenance checks, but raw salary should not be compared across currencies without conversion. The quantum-circuit and optimization artifacts also present in `KDD/Data` are outside the scope of this KDD study.

## 3. Preprocessing and data quality

### Observed quality checks

- All 71,913 rows have nonblank values in the main salary and categorical fields.
- The cleaned file contains 422 distinct job titles, 97 company locations, 104 employee-residence countries, 26 salary currencies, and 11 role families.
- There are no exact duplicate rows.
- There are 15,921 repeated job/year/salary/currency/residence/location combinations. These are not exact duplicates and may represent multiple observations, but they should not automatically be interpreted as independent employers or offers.
- `isco_group_hint` is missing for 40,540 rows, mainly the `Other / Unclassified` role-family group. It should be used only for analyses that explicitly handle missingness.
- `salary_outlier_flag` is populated for 1,754 rows. It is a derived field based on the salary distribution and should not be used as an input feature when predicting salary.

### Preprocessing decisions

1. Parse numeric fields such as `work_year`, `remote_ratio`, `salary`, and `salary_in_usd` using explicit numeric types.
2. Validate that decoded labels agree with their source codes, for example experience and employment type labels.
3. Use `salary_in_usd` for cross-currency comparisons.
4. Keep outliers flagged rather than deleting them; high compensation may be a legitimate executive or specialist observation.
5. Preserve missing `isco_group_hint` as an explicit missing category or exclude that field from a particular analysis.
6. Treat `role_family` and `isco_group_hint` as approximate derived mappings, not authoritative occupational classifications.
7. Report sample size with every grouped statistic and suppress or qualify very small groups.

## 4. Transformation

The following derived variables support KDD analysis:

- salary bands based on quartiles or business thresholds;
- log-transformed salary for models sensitive to skew;
- grouped location regions or country groups, with country sample size retained;
- experience-by-work-mode and role-by-experience interaction groups;
- year-over-year salary summaries;
- median, mean, interquartile range, and outlier rate for each segment.

The median is the preferred summary for salary because the distribution is right-skewed. Means may be included as a secondary measure, but they should be accompanied by counts and spread.

## 5. Data mining approach

The analysis is primarily descriptive and comparative:

- distribution analysis of `salary_in_usd`;
- grouped median and mean salaries;
- cross-tabulations such as experience level by role family and work mode by experience;
- time trends by work year;
- location comparisons only for countries with adequate sample sizes;
- outlier-rate and sensitivity analysis with and without flagged observations.

An optional predictive extension could use a time-based train/test split to estimate salary bands. Suitable baseline models would include a median-by-role-and-experience model, regularized regression on log salary, and a tree-based model. `salary_outlier_flag`, any field computed from the target, and post-hoc salary bands must be excluded to prevent target leakage. Random row splitting should be avoided when repeated observations may represent the same underlying job pattern.

## 6. Results and discovered patterns

### Overall salary distribution

- Minimum salary: $15,000.
- First quartile: $96,000.
- Median: $138,750.
- Third quartile: $190,315.
- Maximum: $800,000.
- Mean: approximately $151,161.

The difference between the mean and median indicates a right-skewed distribution. The median and interquartile range are therefore more reliable for describing a typical record.

### Experience level

| Experience level | Records | Median salary | Mean salary |
|---|---:|---:|---:|
| Entry-level | 7,975 | $85,469 | $98,017 |
| Mid-level | 23,395 | $122,000 | $136,267 |
| Senior-level | 37,702 | $156,160 | $167,808 |
| Executive-level | 2,841 | $190,000 | $202,080 |

Salary generally increases with experience level in this dataset. This is an observed association, not proof that experience alone causes the difference; role, location, employer, and year are also changing factors.

### Role family

The largest role groups are `Other / Unclassified` (40,540), Data Analyst (8,335), Data Engineer (7,246), Data Scientist (7,216), ML Engineer (3,642), and Research Scientist (2,852). Among the more specifically classified groups, median salaries are highest for AI Architect ($195,000), Research Scientist ($179,635), ML Engineer ($175,950), and Analytics Manager ($170,000). Small groups, especially NLP with 19 records, should not be ranked confidently against large groups.

### Work mode and company size

- On-site records: 54,081; median salary $138,300.
- Remote records: 17,505; median salary $140,000.
- Hybrid records: 327; median salary $69,999.
- Medium companies account for 70,110 records and have a median salary of $139,200.
- Large companies have 1,588 records and a median of $129,300.
- Small companies have 215 records and a median of $75,000.

The hybrid and small-company results have substantially smaller samples than the dominant groups, so they should be interpreted cautiously. Work mode and company size are also confounded with location, experience, and role mix.

### Time and geography

The dataset is concentrated in 2024 and 2025: 27,690 records are from 2024 and 38,282 from 2025. Earlier years have much smaller samples. The United States accounts for 60,147 company-location records, followed by Canada (4,571) and the United Kingdom (2,835). Company-location medians for these three groups are approximately $147,200, $119,875, and $76,202, respectively.

These comparisons reflect the composition of the source data as much as labor-market differences. They should not be generalized to countries with small sample sizes or to the global AI labor market.

## 7. Interpretation and evaluation

The strongest consistent pattern is the association between experience level and salary, with additional differences across role families. The dataset also suggests that location and role specialization are important segmentation variables. However, the analysis cannot establish causal effects because it is observational and does not contain a controlled sample, employer-level identifiers, benefits, equity, or complete job-scope information.

Evaluation should include:

- comparing mean and median results;
- reporting sample sizes and interquartile ranges;
- repeating results after excluding flagged outliers;
- checking whether findings remain for the dominant 2024–2025 years;
- restricting geography comparisons to adequately represented countries;
- comparing a simple baseline with any predictive extension using a time-based holdout.

## 8. Deployment and use

The discovered summaries can be used in a compensation dashboard with filters for year, experience, role family, work mode, company size, and location. Each dashboard view should display its denominator, median, mean, quartiles, and outlier policy. Small groups should be labeled as low-confidence rather than overinterpreted.

For future refreshes, preserve the same field definitions, record the source and extraction date, recheck currency conversion, document changes to role-family mappings, and compare new distributions with the current baseline before combining releases.

## 9. Limitations

- The data is self-reported and may contain selection and reporting bias.
- The sample is heavily concentrated in the United States, medium-sized companies, full-time employment, and 2024–2025 records.
- Repeated non-identical records may not be statistically independent.
- Role-family and ISCO mappings are manual and approximate.
- Salary excludes or incompletely represents equity, bonuses, benefits, taxes, and cost of living.
- `salary_in_usd` is comparable currency-wise but does not make salaries economically equivalent across countries.
- Observed relationships are descriptive associations, not causal estimates.
