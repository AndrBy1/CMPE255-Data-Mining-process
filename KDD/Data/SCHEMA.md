# Data dictionary

## fusion/all_inputs.npz and boundary_cases.npz

All arrays have one row per x. The full oracle is sorted by x and has x=1..3133452 exactly. Loading with `np.load(..., allow_pickle=False)` is sufficient.

| Key | dtype | Meaning |
|---|---|---|
| x | uint32 | Original positive input |
| normalization_shift | uint8 | 21 - floor(log2(x)) |
| table_index | uint8 | Parity bit followed by 7 normalized mantissa bits |
| residual | uint16 | Low 14 bits of normalized x |
| reference_phase_units | uint64 | Official reference, modulo 2^64 |
| tolerance_units | uint64 | Allowed circular phase distance |
| selected_phase_units | uint64 | Selected circuit's exact integer-model phase |

One turn is 2^64 units. Compute circular signed differences with `(selected_phase_units - reference_phase_units).view(np.int64)`, then compare its absolute value to tolerance. Do not take the absolute value of an unsigned subtraction. In this domain the differences never equal the signed minimum.

`selected_coefficients.npz` contains the 256 × 3 `(K2,K1,K0)` table, precision tuple, dropped row, low width LB and high split S. `segment_certificates.csv` separates original-input counts from unique reachable normalized residual counts. A certificate row is an arithmetic statement, not an extra circuit verification.

## dqi/weighted_examples.npz, hard_case.npz and hard_batch.npz

| Key | dtype/shape | Meaning |
|---|---|---|
| syndromes | uint16 [N,70] | S0..S69, field basis little endian; S_i uses alpha^((i+1)*position) |
| record | uint16 [N,4095] | Classical E0..E4094 outcome records, each in GF(4096) |
| expected_phase | bool [N] | Required sign flip |
| error_weight | uint8 [N] | Number of nonzero error positions |
| error_indptr | uint32 [N+1] | CSR offsets |
| error_positions | uint16 [sum weights] | Zero-based error positions |
| error_values | uint16 [sum weights] | Nonzero field values |

Required phase parity is the sum of popcounts of `error_value & record[position]`. Sparse labels independently reconstruct every syndrome and phase. `field_tables.npz` stores the official field log/antilog representation and weight probabilities. The hard-case metadata records both sampling and simulator seeds, original batch offset and observed failures on both decoder versions. It deliberately records an allowed failure, not a passing test.

## chemistry/lookup_targets.npz

| Key | Shape | Meaning |
|---|---|---|
| selection | [3976] | Valid selection addresses |
| angle_words | [3976,75] | Ten-bit angle words |
| tree_edges | same as submission | Givens tree edge ordering |
| batch_deltas | [15,3976,5] | XOR deltas between five-word batches, first batch XOR zero |
| packed_deltas | [15,3976] | Five words packed in uint64, least-significant word first |
| even_words | [15,1988] | Packed d0 for the banked loader |
| xor_difference | [15,1988] | Packed d0 XOR d1 |

The canonical factorization and tree are in `selected/dfthcsos/submission.npz`. `factor_singular_values.npz` gives per-group singular values of W for rank/compression analysis, not an assertion that lower rank is accurate. Lookup-design rows distinguish native construction rejection, width rejection, numerical identity and complete circuit verification.

## Experiments and reports

`fusion_search_trials.csv` contains architecture, precisions, dropped rows, layout, worst modeled error, available resource estimates, validation scope, decision and raw-log provenance. Null values mean the corresponding stage was not measured. Measured costs are populated only for the selected fully checked variant. Estimates are not leaderboard labels.

`catalog/verified_profiles.csv` lists selected complete-check resources. `verification/` holds the exact reports. `SCORE_CONTEXT.json` separates the user-reported baseline, local projection and absent new confirmed score. `atlas_manifest.json` inventories immutable files; publishing metadata is mutable and excluded.
