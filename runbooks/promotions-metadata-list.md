# Runbook: list Promotions headers only

```bash
gws gmail +triage --max 100 --query 'category:promotions' --format json
# or tab-scoped:
gws gmail +triage --max 100 --query 'in:inbox category:promotions' --format json
```

Returns id, from, subject, date — not bodies. Good first step before any cleanup.
