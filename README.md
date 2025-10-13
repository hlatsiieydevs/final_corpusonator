
# finalcorpusnator

finalcorpusnator is a lightweight pre-filtering pipeline for OCR'd text. It normalizes and
cleans noisy OCR output, removes obvious OCR artifacts (gibberish lines), preserves valid
Latin and isiXhosa text, and supports batch processing with robust logging.

## Key goals
- Remove page numbers, form-feed artifacts and noisy OCR lines.
- Keep extended-Latin characters (accents) and preserve valid isiXhosa text.
- Batch-process many files in parallel with per-file logs and a processed-files registry.

## Features
- Flexible cleaning with a toggleable gibberish-line remover.
- Heuristic gibberish detection (non‑Latin ratio, short-token ratio, vowel density, repeats).
- isiXhosa-preservation via a seed whitelist plus an automatic frequency-based whitelist builder.
- Parallel batch processing with per-file logs, alerts for low-preserve files, and a registry to avoid re-processing.

## Quick start
1. Open the notebook `finalcorpusnator_v0-2.ipynb` in Jupyter / VS Code.
2. Configure top-level variables in the first code cell:
	 - `GIBBERISH_REMOVER` (True/False)
	 - `GIB_NONLATIN_THRESH` and `GIB_SHORT_TOKEN_THRESH` (tunable detection thresholds)
	 - `AUTO_BUILD_XH_WHITELIST`, `XH_WHITELIST_TOP_N`, `XH_WHITELIST_MIN_FREQ` (whitelist builder)
	 - `BATCH_CATEGORY` and `RAW_INPUTS_DIR` (for batch runs)
3. Run the notebook cells in order. The batch-processing cell (labelled "Batch Processing (parallel)") will scan the `raw_outputs/` directory for `.txt` files and process them.

### Command-line example (run notebook headless with jupyter nbconvert — optional)
```bash
# convert the notebook to a script and run it (example — adjust as needed)
jupyter nbconvert --to script finalcorpusnator_v0-2.ipynb
python3 finalcorpusnator_v0-2.py
```

## Output locations & logs
- Processed output files: `final_corpus/{category}_{YYYYMMDD_HHMMSS}_{origname}_cleaned.txt`
- Registry (prevents duplicates): `logs/processed_files.csv` (columns: filepath, processed_at, out_path, preserved_ratio, dropped_lines)
- Per-file logs: `logs/process_logs/{category}_{timestamp}_{origstem}.log` (detailed stats + snippet)
- Drop audit: `logs/removed_gibberish_lines.txt` (appended list of removed lines with file headers)
- Alerts (low-preserve): `logs/alerts/{category}_{timestamp}_{origstem}_low_preserve.log`

## How the whitelist works
- `XH_WHITELIST` is built from a small seed set and optionally auto-expanded by scanning `final_corpus/` (preferred) or `raw_outputs/`.
- The builder extracts tokens, filters short tokens and digits, and selects the top-N frequent tokens (configurable). The result is merged with the conservative seed list.
- During cleaning, any line containing at least one whitelist token or matching common isiXhosa morphological prefixes (e.g. `ndi-`, `si-`, `u-`, `ba-`) is preserved automatically.

## Tuning recommendations
- If valid isiXhosa lines are removed unexpectedly:
	- Raise `GIB_NONLATIN_THRESH` (tolerate more non-Latin characters) or lower `GIB_SHORT_TOKEN_THRESH`.
	- Expand `XH_WHITELIST_TOP_N` and/or reduce `XH_WHITELIST_MIN_FREQ` to include more local vocabulary.
- For very noisy input, try increasing the strictness (lower nonlatin_thresh) and inspect `logs/removed_gibberish_lines.txt`.

## Troubleshooting
- If files are not being processed, check `logs/processed_files.csv` to confirm they're not already recorded.
- Per-file process logs in `logs/process_logs/` contain the exact commands and a 30-line sample of output. Use them to diagnose what was removed and why.
- Use the low-preserve alerts in `logs/alerts/` to find files where less than 25% of characters were preserved.

## Next steps / improvements you may want
- Add a dry-run mode that lists lines that would be removed without writing outputs.
- Produce a token-frequency CSV from the whitelist builder for manual review before committing.
- Optionally switch to multiprocessing for CPU-heavy workloads.

## Contact / notes
- The notebook is intentionally self-contained and configured via top-level variables. Edit these variables to fit your corpus and run environment.

## License
- MIT-style permissive usage (adjust if you need a different license).

Enjoy — run the batch cell and check `logs/` for per-file diagnostics.
