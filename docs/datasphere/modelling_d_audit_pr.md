# PR title

Audit layer: MODELLING_D inventory and batch framework migration

# PR description

Status counts:
- CONVERTED: 0
- OLD PATTERN: 0
- PARTIAL: 0
- NON-STANDARD: 0
- Tables inspected: 0
- Flow conversions performed: 0
- Flow-to-load-mode issues: 0

Flow inventory:
- No flow was converted because the required MODELLING_D object inventory could not be retrieved.

Manual review:
- No flows requiring manual review were identified because the CLI failed before any local-table inventory could be loaded.

Blocked by CLI error:
```text
Failed to output a list of the objects in JSON format
exit:1
```

The command attempted was:
```powershell
datasphere objects local-tables list --space MODELLING_D --select "technicalName,businessName,columnCount" --top 500
```

No Datasphere write, deploy, or run actions were taken.
