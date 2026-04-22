# Quick Reference: openFDA Drug API

## Base URL
```
https://api.fda.gov/drug/
```

## Key Queries

### Find Recently Approved Prescription Drugs
```
/drugsfda.json?search=submissions.submission_type:ORIG+AND+submissions.submission_status:AP&sort=submissions.submission_status_date:desc&limit=100
```

### Filter by Date Range
```
/drugsfda.json?search=submissions.submission_status_date:[20250101+TO+20251231]&sort=submissions.submission_status_date:desc&limit=100
```

### Find Specific Drug by Name
```
/drugsfda.json?search=openfda.brand_name:"DRUGNAME"&limit=10
```

### Get Drug Label/Prescribing Info
```
/label.json?search=openfda.brand_name:"DRUGNAME"&limit=1
```

### Find Adverse Events
```
/event.json?search=patient.drug.drugindication:"INDICATION"&limit=10
```

## Important Field Names

| Field | Description |
|-------|-------------|
| `submissions.submission_status_date` | Approval date (YYYYMMDD format) |
| `submissions.submission_type` | "ORIG" for original, "SUPPL" for supplements |
| `submissions.submission_status` | "AP" = Approved |
| `openfda.brand_name` | Brand/trade name |
| `openfda.generic_name` | Generic name |
| `openfda.manufacturer_name` | Manufacturer |
| `products.marketing_status` | "1"=Prescription, "4"=OTC |
| `openfda.rxcui` | RxNorm Concept Unique Identifier |

## Marketing Status Codes
- `"1"` = Prescription
- `"4"` = Over-the-counter
- `"2"` = Discontinued
- `"3"` = None (Tentative Approval)

## Date Format Note
The `submission_status_date` field uses **YYYYMMDD** format (e.g., "20170328"), not YYYY-MM-DD.
