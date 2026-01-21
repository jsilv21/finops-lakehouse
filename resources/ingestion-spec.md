# **Ingestion Spec (FOCUS 1.2, Parquet, AWS S3 default)**

## **Goals**

- Default to FOCUS 1.2 Parquet exports from AWS S3.
- Keep ingestion local-first and idempotent with immutable Bronze + manifest.
- Handle empty exports gracefully until data appears.
- Support drop-in local files as an override or offline mode.

## **Inputs and Assumptions**

- Users configure AWS FOCUS export in S3 (FOCUS 1.2, Parquet, Daily cadence by default).
- Exports can be delayed; data might be unavailable on the first run.
- CUR inputs are supported but normalized to FOCUS before Bronze.

## **Configuration (project.yml)**

Minimum fields:

```yaml
source:
  type: focus            # focus|cur
  focus_version: "1.2"   # default
  format: parquet        # default
  cadence: daily         # daily|hourly|monthly
  s3:
    bucket: my-bucket
    prefix: cost-exports/focus/
    region: us-east-1    # optional if inferred
    profile: default     # optional
    partition_regex: "date=(\\d{4}-\\d{2}-\\d{2})"  # optional
  local_input_path: input/  # optional fallback
  availability_lag_hours: 24 # default for daily
  backfill_window_days: 7     # default lookback for late files
  strict_schema: true         # warn/fail on missing columns
```

## **CLI Interface**

- `costlake run`: standard ingestion.
- `--force`: re-ingest a snapshot even if manifest exists.
- `--from YYYY-MM-DD --to YYYY-MM-DD`: backfill range.
- `--input PATH`: override local input folder.

## **Retrieval Flow (AWS S3 default)**

1) Determine target window based on `cadence`, `availability_lag_hours`, and `backfill_window_days`.
2) List S3 keys under `s3.prefix` for the target window.
3) Derive `snapshot_date` from key path using `partition_regex`; fall back to S3 LastModified date if no match.
4) Download or copy each file into local Bronze location (immutable).
5) If no files found, write a manifest with status `no_data` and exit 0.

## **Bronze Layout**

- Path: `bronze/focus/snapshot_date=YYYY-MM-DD/*.parquet`
- Files are immutable. Re-ingest only with `--force`.

## **Manifest**

Store at `bronze/_manifest/manifest.parquet` (or JSON for v1). Fields:

- `snapshot_date`
- `cadence`
- `source_type` (focus|cur)
- `focus_version`
- `format`
- `file_keys` (S3 keys or local paths)
- `file_hashes`
- `row_count`
- `schema_version`
- `status` (success|partial|failed|no_data)
- `error_count`
- `created_at`

## **Validation and Schema Drift**

- Validate required FOCUS 1.2 columns on ingest.
- If `strict_schema` is true, fail on missing required columns; otherwise warn and continue.
- Record schema version and drift warnings in manifest.

## **Quarantine**

- Invalid rows written to `bronze/_quarantine/` with reason codes.
- Manifest records quarantine counts.

## **Idempotency and Recovery**

- Use file hashes + manifest to skip already ingested snapshots.
- Partial failures leave a manifest with `partial` and allow resume.

## **Credentials and Security**

- Use AWS default credential chain (env vars, profile, instance role).
- No secrets stored in repo.
