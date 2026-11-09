# VQC Verified Optimization Atlas — v1.0

A reusable optimization corpus for **Google Quantum AI – Verifiable Quantum Circuit Compilation**.

This release packages the exact circuits, numerical targets, failed experiments, generators and complete-check records behind the V5 portfolio. It replaces dependencies on older notebook outputs with one versioned dataset. It does not claim a new leaderboard result or a guaranteed score increase.

## Start here

1. Upload `vqc_verified_optimization_atlas_v1.zip` as a new Kaggle dataset. Suggested title: **VQC Verified Optimization Atlas**. Preserve its folders.
2. Import `vqc_atlas_consumer_v6.ipynb` as a Kaggle notebook.
3. Attach this dataset and the **Google Quantum AI – Verifiable Quantum Circuit Compilation** competition dataset.
4. Set Accelerator to **None**. Internet is unnecessary. Run all cells.
5. Submit `/kaggle/working/submission.zip`. The adjacent validation report states whether complete checks were repeated or matched recorded files and dependencies.

The original Signed-Digit Memory notebook output, deleted decoder notebook, separate starter kit and GPU are unnecessary. The official competition dataset still supplies its verifier, native kickmix library, fixed shifts and reference Hamiltonian; those files are not redistributed here.

If using the Kaggle API instead of the upload page, replace `YOUR_USERNAME` in `dataset-metadata.json`. The metadata uses **Other (specified in description)** because the bundle has mixed provenance; it does not apply a new blanket license to third-party binaries. Source credit is in `licenses/`, `catalog/license_map.csv` and `code/metadata.json`.

## What the dataset contains

| Directory | Contents and purpose |
|---|---|
| `selected/` | The complete selected portfolio: inverse square root, decoder, factorization and 15 consecutive ANGLE circuits. Every member matches a complete local task-check record. |
| `fusion/` | Exact reference phase/tolerance and selected phase for **all 3,133,452 inputs**; normalized features; 7,686 boundary/stress cases; 256 segment certificates; calibrated coefficient bank. |
| `experiments/` | **33** recorded fusion precision/layout trials, including accuracy rejections. Estimates and complete circuit checks are explicitly separated. |
| `chemistry/` | The complete 3976 × 75 angle-word table, 15 × 3976 × 5 delta tables, packed banked-QROM targets, factor spectra, and **20** lookup-design attempts. Native-construction failures and width failures are retained. |
| `dqi/` | **1,024** weighted examples with syndromes, classical records, exact phase labels and sparse error patterns; field tables; one independently reproduced rare failure and its entire 256-shot batch. |
| `source/` | The exact pre-implication-pass decoder, enabling regeneration and paired comparisons. |
| `verification/` | The original complete-check reports, source comparison, compilation ledger and previous integration report. |
| `code/` | Readable coefficient fitting, arithmetic, memory scheduling, exact compiler, banked lookup, oracle generation, verification and export utilities. |
| `notebooks/` | A clean consumer notebook template. The separately supplied consumer notebook has executed outputs. |
| `atlas_manifest.json` | File sizes, SHA-256 hashes, selected member hashes and official dependency/reference fingerprints. |

`dataset-metadata.json` is deliberately excluded from the integrity inventory so changing your publishing username does not invalidate the numerical release. Python bytecode caches are also excluded.

## Measured portfolio and score context

| Task | Selected result | Complete local verification |
|---|---|---|
| Fusion | 91 qubits; 1584 static Toffolis; loss 173002.7486137991 | 200,540,928 shots; every domain input tested 64 times |
| DQI | 1818 qubits; 583505 static Toffolis; loss 2307647381280.9233 | 102,400 shots; 1 value failure and 0 counted phase failures |
| Chemistry | 681 whole-block qubits; 72514 whole-block Toffolis; loss 2348537554.5852766 | 254,464 shots; 15 ANGLE circuits, each at most 122 qubits |

The user-reported baseline leaderboard score is **5.51910**. The selected portfolio's local projection is **5.663396**, including the observed decoder failure penalty. Holding that penalty fixed gives a hardware-based estimate of **5.672054**. Neither is a new confirmed leaderboard score. This dataset reproduces those improvements; it does not add another numerical gain by repackaging them. Fresh private sampling can change the decoder confidence penalty.

## How the improvements work

**Exact-domain coefficient fitting.** Normalize `x` with shift `s = 21 - floor(log2(x))`, so `xn = x << s`. The table index is `128*(s mod 2) + ((xn >> 14) & 127)` and the residual is `xn & 16383`. For odd parity, reachable residuals are even. For even parity above the largest original x, they are multiples of four. Weighted integer minimax fitting uses these exact reachable values and the emitted partial-product rounding, rather than only fitting a floating-point quadratic.

The selected configuration is `(F2,F1,FYR,GY) = (8,19,27,1)`, with R-row 5 dropped. Its coefficient table is fixed data. Optional regeneration can refit it; changed circuits need fresh complete verification. The worst selected phase error consumes 96.281989% of the contract tolerance, with minimum remaining margin 40,189,295 phase units.

**Memory scheduling.** Phase the table's high anchor word first, clear it by measured unlookup and release it before Horner arithmetic. Phase the low accumulator including its overflow bit separately. A folded top carry reduces temporary storage. The normalized public bit `x[21]` is known to be one; it is flipped, borrowed as a clean carry and restored before unnormalizing. These transformations preserve the fixed-point result while changing peak memory.

**Exact decoder implications.** The postpass tracks value versions, constants, implications and classical condition joins. If one control implies the other, a Toffoli becomes a CNOT. It removes another 231 static Toffolis from the already optimized external decoder. No reset, measurement correction or phase is discarded. The known rare failure occurs on both input and output decoders.

**Banked ANGLE lookups.** For address `a = 2h+l`, apply `target ^= d0[h] ^ (l * (d0[h]^d1[h]))`. The difference bank is cleaned with measurement-based unlookup. Five ten-bit words use 50 persistent bits; the temporary bank fits within workspace already dominated by state preparation. Both angle passes and the analytical state-preparation/rotation costs enter the loss. A 128-qubit limit applies to individual ANGLE circuits, not the full block encoding.

## Correct interpretation

This is an optimization corpus, not a conventional supervised-learning dataset. Its value is exact arithmetic targets, sparse labelled decoder examples, reusable circuits and measured constraints. Thirty-three historical trials can guide a search; they do not establish that a fitted score predictor will generalize.

The full fusion oracle certifies the arithmetic approximation. Quantum input restoration, scratch cleanup and measurement feed-forward are covered separately by circuit checks. Likewise, resource estimates in search tables are not complete verifier passes. Rejected trials remain rejected even when their resource count looks attractive.

The decoder examples are public diagnostic inputs drawn from the official distribution. The hard batch comes from our fixed-salt development run, not a hidden leaderboard set. Its failing shot has weight 30; its exact cause has not been established. A phase mismatch on a dirty shot is documented separately, because the official failure count treats that shot as a value failure. These cases help debug general decoder behavior; fresh official sampling remains the evaluation method.

## Rebuild and explore

In the consumer notebook, set `REBUILD_ARITHMETIC_AND_LOOKUPS=True` to regenerate the selected fusion and ANGLE binaries. Set `REBUILD_DECODER=True` too to rerun the exact decoder pass; it takes several minutes and uses temporary circuit text. Regenerated files must match the selected hashes. The default path uses the prepared circuits and repeats fresh independent checks.

To rebuild numeric targets independently:

```bash
python code/rebuild_targets.py --competition /path/to/competition --atlas /path/to/atlas --output /path/to/new-targets
```

For a genuinely new experiment, use the generators and coefficient bank, evaluate all original x values or the unchanged Hamiltonian, then run the complete task verifier before promoting it into a new release. Keep different validation scopes visible and preserve versioned manifests. The consumer exports only the recorded selected profile.

## Provenance and licensing

- Sumit Chongder's public Apache-2.0 signed-digit arithmetic and banked QROM: https://www.kaggle.com/code/sumitchongder/notebook2f196e646b?scriptVersionId=361357149
- The supplied `quantum-circuits-signed-digit-memory.ipynb`: high-word schedule, folded carry and normalized-input borrowing, with its attribution retained.
- Artem1410's pinned Open QROM v4 output: https://www.kaggle.com/code/anazemcev/quantum-circuits-open-qrom-measured-gains?scriptVersionId=361406534
- Kozykappa's physical ensemble compression and sparse rotations, with the subsequent effective-norm optimization credited to Artem1410.
- The original compiled Reed–Solomon decoder is credited upstream to k256.dev. Its private generator is not supplied or claimed. The upstream notebook's full ownership/license chain was not independently established; this release retains its provenance and does not relicense that binary.
- New atlas tooling and derived numerical generation code: Apache-2.0. Existing materials retain their authors' terms. See the component license map.
