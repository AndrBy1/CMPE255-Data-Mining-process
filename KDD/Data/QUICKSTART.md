Upload the dataset ZIP to Kaggle, then attach it and the competition dataset to the companion notebook. Select CPU (Accelerator None), turn Internet off, and Run All. The notebook auto-finds atlas_manifest.json, verifies hashes, checks all 3,133,452 arithmetic targets and repeats independent circuit checks. It writes /kaggle/working/submission.zip. No previous notebook outputs are needed.

The full verifier can be repeated with FORCE_FULL_VERIFICATION=True. The default reuses complete-check records only for matching files, native library, verifier code and scientific reference fingerprints.
