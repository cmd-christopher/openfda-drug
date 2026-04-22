---
name: openfda-drug
description: |
  Query the openFDA API for information about prescription and over-the-counter drugs.
  Use this skill whenever the user asks about FDA-approved drugs, drug approvals, drug labels,
  adverse events, drug recalls, NDC codes, or anything related to FDA drug data.
  This includes queries about specific drug names, approval dates, drug ingredients,
  manufacturer information, drug shortages, or comparing drugs.
  Even if the user doesn't explicitly mention "openFDA" or "FDA API", if they ask about
  prescription medications, OTC drugs, drug approvals, or any drug regulatory information,
  use this skill to fetch accurate data from the FDA's public API.
---

# openFDA Drug API Skill

This skill provides structured guidance for querying the openFDA API to retrieve information about drugs approved in the United States. Use this to inform physicians about new drug approvals, look up drug details, or find regulatory information.

## API Overview

**Base URL:** `https://api.fda.gov/drug/`

The openFDA drug API has several endpoints, each serving different types of drug information:

| Endpoint | Purpose | Update Frequency |
|----------|---------|------------------|
| `drugsfda` | Drugs@FDA - approved drug products since 1939 | Daily |
| `label` | Structured product labeling (prescribing info) | Weekly |
| `event` | Adverse event reports (FAERS) | Quarterly |
| `ndc` | National Drug Code directory | Daily |
| `enforcement` | Drug recall enforcement reports | Weekly |
| `drugshortages` | Drug shortage information | Daily |

## Query Syntax

All queries use the pattern: `search=field:term`

**Common patterns:**
- `search=field:term` - Search for exact term in field
- `search=field:term+AND+field:term` - Match both terms
- `search=field:term+field:term` - Match either term
- `sort=field:desc` - Sort descending
- `count=field` - Count occurrences
- `limit=N` - Return N results (max 1000 per request)

## Key Endpoints and Fields

### Drugs@FDA (`/drugsfda.json`)

Use this endpoint to find approved drugs and their details. This is the primary endpoint for finding recently approved prescription drugs.

**Important fields:**
- `openfda.brand_name` - Brand/trade name (array)
- `openfda.generic_name` - Generic name(s) (array)
- `openfda.manufacturer_name` - Manufacturer (array)
- `openfda.route` - Route of administration (array)
- `openfda.rxcui` - RxNorm Concept Unique Identifier (array)
- `openfda.unii` - Unique Ingredient Identifier (array)
- `openfda.pharm_class_epc` - Established pharmacologic class (array)
- `products.active_ingredients` - Active ingredients with strength (array of objects with `name` and `strength`)
- `products.dosage_form` - Dosage form (tablet, injection, etc.)
- `products.marketing_status` - Marketing status code: `"1"`=Prescription, `"4"`=Over-the-counter
- `sponsor_name` - Company that submitted the application
- `application_number` - NDA/ANDA/BLA number
- `submissions.submission_status_date` - Approval date as YYYYMMDD string (e.g., `"20170328"`)
- `submissions.submission_type` - Submission type: `"ORIG"` for original approvals, `"SUPPL"` for supplements
- `submissions.submission_status` - Status: `"AP"` = Approved

**Example queries:**
```
# Find recent original prescription drug approvals (NEW feature)
https://api.fda.gov/drug/drugsfda.json?search=submissions.submission_type:ORIG+AND+submissions.submission_status:AP&sort=submissions.submission_status_date:desc&limit=100

# Find drugs approved in a specific date range
https://api.fda.gov/drug/drugsfda.json?search=submissions.submission_status_date:[20240101+TO+20241231]&sort=submissions.submission_status_date:desc&limit=100

# Find drugs approved since a date
https://api.fda.gov/drug/drugsfda.json?search=submissions.submission_status_date:[20250101+TO+20251231]&sort=submissions.submission_status_date:desc&limit=100
```

## Common Workflows

### Finding Recently Approved Prescription Drugs

This is the primary use case for keeping physicians informed about new drug approvals.

**Step 1:** Filter for original approvals only using `submissions.submission_type:ORIG` (not supplements)

**Step 2:** Filter for approved status using `submissions.submission_status:AP`

**Step 3:** Optionally filter by marketing status for prescription-only drugs

**Step 4:** Sort by `submissions.submission_status_date` descending to get most recent first

**Full example:**
```
/drugsfda.json?search=submissions.submission_type:ORIG+AND+submissions.submission_status:AP&sort=submissions.submission_status_date:desc&limit=100
```

**Response structure:**
```json
{
  "results": [
    {
      "submissions": [
        {
          "submission_status_date": "20260417",
          "submission_type": "ORIG",
          "submission_status": "AP",
          "submission_class_code_description": "Type 1 - New Molecular Entity"
        }
      ],
      "openfda": {
        "brand_name": ["DUPIXENT"],
        "generic_name": ["dupilumab"],
        "manufacturer_name": ["sanofi-aventis U.S. LLC"],
        "route": ["SUBCUTANEOUS"],
        "pharm_class_epc": ["Interleukin-4 Receptor alpha Antagonist [EPC]"]
      },
      "products": [
        {
          "brand_name": "DUPIXENT",
          "active_ingredients": [{"name": "dupilumab", "strength": "300 MG/2ML"}],
          "dosage_form": "INJECTION",
          "marketing_status": "1"
        }
      ],
      "sponsor_name": "sanofi-aventis U.S. LLC",
      "application_number": "BLA761055"
    }
  ]
}
```

### Filtering Prescription vs OTC Drugs

In `/drugsfda`, `products.marketing_status` indicates how a drug is sold:
- `"1"` = Prescription
- `"4"` = Over-the-counter (OTC)
- `"2"` = Discontinued
- `"3"` = None (Tentative Approval)

To get only prescription drugs, check the `products` array in each result.

### Checking Drug Status (Prescription vs OTC)

In `/drugsfda`, check `products.marketing_status`:
- `"1"` = Prescription
- `"4"` = Over-the-counter
- `"2"` = Discontinued
- `"3"` = None (Tentative Approval)

### Getting Detailed Drug Information

1. Search Drugs@FDA for the drug by name or application number
2. Use the returned `openfda.rxcui` or `openfda.unii` to cross-reference other endpoints
3. Query the label endpoint for full prescribing information

### Finding a Drug by Application Number

NDA/ANDA/BLA numbers follow the pattern: `NDA123456`, `ANDA123456`, `BLA123456`

```
/drugsfda.json?search=openfda.application_number:"NDA123456"&limit=1
```

### Cross-Referencing Drugs

The `openfda` object is harmonized across endpoints. Use `openfda.rxcui` or `openfda.unii` to link data:

```
# Get label using RxCUI
/label.json?search=openfda.rxcui:"123456"&limit=1
```

## Important Caveats

1. **Not for clinical decisions** - openFDA data is not validated for production use. Always consult a healthcare provider.

2. **Adverse events don't prove causation** - Reports list drugs and reactions together, but no causal relationship is established.

3. **Data lags** - Adverse event data may lag by 3+ months. Drug shortages are updated daily.

4. **OTC drugs** - Many OTC drugs are not individually approved but marketed under monographs.

5. **Respect rate limits** - The public API has usage limits. Use `limit` parameter to fetch only what's needed.

6. **Date format** - The `submission_status_date` field uses YYYYMMDD format (e.g., `"20170328"`), not YYYY-MM-DD.

7. **Multiple submissions** - A drug may have multiple submissions with different dates. The `ORIG` submission type indicates the original approval, while `SUPPL` indicates supplemental changes.

## Output Format

When presenting drug information:
- Include brand name, generic name, and manufacturer
- Report the approval date (from `submissions.submission_status_date`)
- Note the application number (NDA/ANDA/BLA)
- Describe the drug class or mechanism of action if available
- Include all relevant URLs or references

## Error Handling

Common issues:
- Empty results: Try broader search terms or different field
- 403 errors: May need to check API terms of service
- Large result sets: Use pagination with `skip` parameter
