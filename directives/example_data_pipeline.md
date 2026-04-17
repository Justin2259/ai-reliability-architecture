# Directive: Example Data Pipeline

## Purpose

Fetch records from a source API, apply transformations, and write the result to a destination (Google Sheet, database, or file). This is the canonical pattern for any ETL-style automation.

## Inputs

| Input | Source | Notes |
|-------|--------|-------|
| `SOURCE_API_URL` | `.env` | Base URL for the source API |
| `SOURCE_API_KEY` | `.env` | API key or OAuth token |
| `DESTINATION_SHEET_ID` | `.env` | Google Sheet ID for output (if applicable) |
| `--since` | CLI arg | ISO 8601 date - only pull records after this date |
| `--dry-run` | CLI flag | Print what would be written without writing |

## Execution

```bash
# Pull records from the last 7 days
python execution/run_pipeline.py --since 2024-01-01

# Dry run to verify output before writing
python execution/run_pipeline.py --since 2024-01-01 --dry-run
```

## Steps

1. Run `python execution/fetch_records.py --since <date>` - outputs `.tmp/records.json`
2. Verify `.tmp/records.json` is non-empty. If empty, check source API health before continuing.
3. Run `python execution/transform_records.py --input .tmp/records.json --output .tmp/transformed.json`
4. Run `python execution/write_destination.py --input .tmp/transformed.json`
5. Confirm row count matches expected volume. Flag if more than 20% off from prior run.

## Output

- Rows written to the configured destination
- `.tmp/records.json` and `.tmp/transformed.json` (intermediates, safe to delete)
- Print: `Done. N rows written.` on success

## Error Handling

**Source API returns 0 records:** Stop at step 2. Do not write an empty result to the destination - this usually means the source is down or the date filter is wrong.

**Transform fails on a specific record:** Log the record ID and skip it. Do not abort the full run for one bad record. Report the skip count at the end.

**Destination write fails (auth error):** Check `DESTINATION_SHEET_ID` is correct and credentials have write access. The transformed file is still in `.tmp/` so the write can be retried independently.

**Rate limit (429):** The fetch script retries with exponential backoff up to 3 times. If still failing, narrow the `--since` window and run in smaller batches.

## Learnings

- The source API paginates at 500 records per page. The fetch script handles pagination automatically.
- Records deleted at source still appear in the API response with `status: deleted` for 30 days. Filter these in the transform step.
- Sheet writes time out after 60 seconds for batches over 5,000 rows. Chunk into batches of 1,000 if needed.
