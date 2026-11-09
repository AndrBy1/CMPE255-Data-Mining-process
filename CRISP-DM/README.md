# CMPE255-Data-Mining-process

Kaggle dataset source: https://www.kaggle.com/datasets/muhammadahmerkhan/ai-agent-security-and-real-world-incidents-2024-26

## 1. Business understanding

Maintaining security is important for organizational resilience, operational reliability, and customer privacy. Studying documented AI-agent security events can help an organization prioritize controls such as least privilege, credential protection, sandboxing, prompt-injection defenses, supply-chain review, and human approval for high-impact actions.

The business problem is to understand how AI-agent security events have changed over time, which attack themes occur most often, and which themes are associated with confirmed harm or high severity. The intended audience is a security or risk team deciding where to focus monitoring and preventive controls.

### Business questions

1. How does the volume of recorded entries change by month and year?
2. Which attack taxonomy categories are most common when multi-label categories are expanded?
3. How do confirmed harm and severity differ across entry types and attack categories?
4. Which security controls should be prioritized based on the observed patterns?

### Success criteria

- Produce reproducible counts and trend summaries from the supplied CSV files.
- Separate observed incidents from vulnerabilities, research, policy entries, and reports.
- Identify recurring attack categories and high-risk combinations without overstating causality.
- Translate the findings into practical security-control recommendations.

## 2. Data understanding

The source bibliography is in `Security data/sources_bibliography.txt`. The source dataset contains documented AI-agent exploits, prompt injections, jailbreaks, vulnerabilities, policy events, research findings, and threat reports. The cleaned analysis file is `ai_agent_incidents_clean.csv`; the fuller source-oriented file is `ai_agent_incidents_full.csv`.

### Dataset profile

- 395 unique rows and 17 columns in the cleaned file.
- Date range: 2024-12-01 through 2026-10-09.
- Years: 1 entry in 2024, 121 in 2025, and 273 in 2026.
- Entry types: 162 incidents, 55 vulnerabilities, 91 research entries, 72 policy entries, and 15 reports.
- Confirmed harm is marked `True` for 141 rows and `False` for 254 rows.
- Severity: 48 critical, 157 high, 94 medium, 9 low, and 87 informational.
- Confidence: 339 A, 51 B, 2 C, and 3 D.
- AI involvement is confirmed in 386 rows; 5 are disputed, 2 are unverified, and 2 are not applicable.
- The data is heavily global: 310 rows use `GLOBAL`, with some rows containing multiple regions.

### Important interpretation rule

The row count is not the count of real-world breaches. In particular, policy, research, vulnerability, and report rows provide context but are not necessarily incidents with confirmed harm. For harm-rate calculations, the analysis should either use only `incident_kind = incident` or report results separately by entry type.

The attack taxonomy is multi-label. A row such as `MCP;IPI;EXFIL` represents several themes. Therefore, single-label row counts and expanded taxonomy counts answer different questions. Expanded counts should be used when measuring the prevalence of attack themes, and the number of labels per row should be retained for combination analysis.

### Data quality observations

- There are no blank values in the cleaned file and no duplicate incident IDs or titles.
- The cleaned file's `source_count` is `1` for all 395 rows, even though many `primary_source_url` values contain multiple semicolon-separated URLs. This field should be recalculated from the full source list before using it as a source-quality measure.
- Forty-seven rows have month-level date precision and six have a broader `part` precision in the full file. Daily ordering should therefore be treated as approximate for those records.
- The large increase in 2026 may reflect both a real increase and more active collection, broader coverage, or future-dated source material. It should not be interpreted as a population incidence rate.
- `confidence_level`, `real_harm_confirmed`, and `ai_involvement` are different concepts. Confidence describes evidence quality; real harm describes observed impact; AI involvement describes the relationship to AI. They should not be combined into one score without a documented rule.

## 3. Data preparation

The cleaned file is the primary input for the analysis. The preparation workflow is:

1. Parse `date` as a date and derive year and month checks.
2. Validate that `incident_id` is unique and that categorical values use the expected vocabulary.
3. Convert `real_harm_confirmed` and `is_landmark_event` to Boolean values.
4. Keep `incident_kind` as a separate stratification variable.
5. Split semicolon-delimited `attack_type_code` values into a long-format table so each row-category pair can be counted once.
6. Split semicolon-delimited regions and source URLs only when doing region- or source-level analysis.
7. Recalculate source counts from the full source field rather than relying on the current cleaned `source_count` column.
8. Record the dataset extraction date and exclude future or subsequently corrected entries when creating a time-series snapshot.

The derived files `monthly_incident_trends.csv` and `taxonomy_attack_types.csv` are useful validation outputs. The train/test files in this folder are not used for this security-incident analysis because they do not describe the same schema or business problem.

## 4. Modeling / analysis approach

Because the dataset is a curated event archive with no representative denominator and a small number of confirmed-harm outcomes, the primary model is descriptive rather than predictive:

- monthly and yearly volume trends;
- severity and confirmed-harm cross-tabulations;
- entry-type comparisons;
- expanded attack-taxonomy frequencies;
- multi-label co-occurrence counts;
- high-confidence versus lower-confidence sensitivity checks.

A predictive model should not be presented as a general breach-risk predictor. If an educational classification model is required, a documented target such as `severity in {high, critical}` could be used with a time-based holdout, but fields that directly encode the outcome or were assigned after review must be excluded to avoid leakage. The sample is also too small and selection-biased for strong generalization claims.

## 5. Evaluation and findings

### Observed findings

- The archive grows from 121 entries in 2025 to 273 in 2026. This is a change in recorded volume, not proof that the real-world incident rate increased by the same amount.
- Across all rows, the most common expanded attack themes are governance/policy (`GOV`, 67), credential theft (`CRED`, 64), weaponization (`WEAPON`, 61), indirect prompt injection (`IPI`, 52), and infrastructure compromise (`INFRA`, 50). These counts include rows with multiple labels.
- Among the 162 rows classified as incidents, 129 have confirmed harm and 128 are high or critical severity. This is a property of the curated archive and should not be treated as the probability that an arbitrary AI deployment will be harmed.
- Confirmed harm is concentrated in critical and high rows: 46 of 48 critical rows and 79 of 157 high rows are marked as harmful. Medium rows have 12 harmful entries, while informational rows have none.
- The 72 policy rows have no confirmed harm by definition of how this archive is labeled. Vulnerability, research, and report rows should therefore not be pooled with incidents when evaluating harm.
- The monthly derived file shows the highest recorded volume in September 2026 with 80 entries. This peak should be checked for collection or publication-batch effects before being described as a sudden real-world surge.

### Evaluation checks

- Reconcile monthly totals to 395 rows.
- Reconcile taxonomy totals using both row-level labels and expanded labels; they should not be expected to match.
- Report denominators for every percentage and stratify by `incident_kind`.
- Repeat key summaries using only confidence levels A and B, and compare against the full archive.
- Inspect multi-label combinations separately from single-category frequencies.

## 6. Deployment / action plan

The results can support a recurring security-review dashboard with monthly volume, entry type, severity, harm status, confidence, and expanded attack themes. Recommended control priorities are:

1. Protect credentials, tokens, and secrets with short-lived credentials, scoped permissions, rotation, and secret scanning.
2. Treat external content and tool responses as untrusted to reduce indirect prompt injection and tool-poisoning risk.
3. Enforce sandbox boundaries, network egress controls, and human approval for destructive or financial actions.
4. Review model, package, plugin, and MCP dependencies before deployment and continuously monitor them for compromise.
5. Maintain an incident-response process that records evidence quality, confirmed harm, affected assets, and remediation status separately.

Refresh the archive on a defined schedule, preserve historical snapshots, deduplicate repeated reports of the same event, and review taxonomy definitions as new attack patterns emerge.

## 7. Limitations

This is a curated, source-dependent archive rather than a random sample of all AI-agent activity. It has no exposure denominator, so it cannot estimate the probability of attack or compare organizations fairly. Records may represent the same underlying event from different perspectives, dates may have different precision, and taxonomy and severity labels depend on the dataset's curation rules. Findings should be used for prioritization and hypothesis generation, not causal claims or precise forecasting.

## Suggested revisions to the original sections

- Replace “personal growth” with “organizational resilience, reliability, and customer privacy”; security is being framed as a business-risk and trust issue.
- State that the archive covers 2024–2026 and contains more than incidents, including research, policy, vulnerability, and report entries.
- Define what “security incidents” means in the project, since the file's `incident_kind` field distinguishes several types of evidence.
- Add the collection-date and source-quality caveats above before interpreting the 2026 increase.
