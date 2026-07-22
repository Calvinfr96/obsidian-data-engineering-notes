## Active Applications
```dataview
TABLE role AS "Position", status AS "Interview Stage", applied_date AS "Date Applied"
FROM "01_Job_Hunting" OR #interview
WHERE status != "❌ Rejected" AND status != "🎉 Offer Accepted"
SORT applied_date DESC
```

## Rejected Applications
```dataview
LIST company
FROM "01_Job_Hunting"
WHERE status = "❌ Rejected"
SORT applied_date DESC
```
