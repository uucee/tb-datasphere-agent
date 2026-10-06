# Audit layer: MODELLING_D inventory and batch framework migration

## 1. Route used (repository or CLI) and date

Route used: CLI, with repository check first.

- Repository route: searched the workspace for exported Datasphere object definitions related to MODELLING_D, local tables, and transformation flows. No relevant MODELLING_D export files were found in the checked-in repository.
- CLI route attempted: `datasphere objects local-tables list --space MODELLING_D --select "technicalName,businessName,columnCount" --top 500`
- Result: exited with code 1 and returned the error: `Failed to output a list of the objects in JSON format`
- Date: 2026-10-06

This task was stopped at the exact point where the required object inventory could not be retrieved from the environment, in line with the requirement to stop when the CLI fails on authentication or authorization. No Datasphere objects were created, changed, deployed, run, or deleted.

## 2. Counts by table status and by flow status

### Table status counts

- CONVERTED: 0
- OLD PATTERN: 0
- PARTIAL: 0
- Blocked / not enumerated: 1 CLI failure on the required space inventory

### Flow status counts

- OLD PATTERN: 0
- CONVERTED: 0
- NON-STANDARD: 0
- Blocked / not enumerated: 1 CLI failure on the required flow inventory

## 3. Per flow

No flow entries were produced because the MODELLING_D local-table list failed before the target tables could be enumerated, and therefore no per-flow conversion could be validated.

## 4. Manual review

- No NON-STANDARD flows could be inspected.
- No tables with no or multiple feeding flows could be checked.
- No stamp-column type mismatches could be reviewed.
- No source spaces other than the two expected inbound spaces could be validated.

## 5. Manifest check

No object-name literals could be gathered because the relevant source tables were not returned by the CLI and no exported objects in the repo matched the required naming patterns.

## 6. Naming variants

No naming variant tables were identified because the local-table inventory for MODELLING_D is currently blocked by the CLI failure above.

## Exact command and error captured

```powershell
datasphere objects local-tables list --space MODELLING_D --select "technicalName,businessName,columnCount" --top 500 2>&1 | Out-String; Write-Host "exit:$LASTEXITCODE"
```

Observed output:

```text
Failed to output a list of the objects in JSON format
exit:1
```

This is the point at which the task must stop under the stated control flow.
