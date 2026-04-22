# openfda-drug

A Claude skill for querying the openFDA API to retrieve information about FDA-approved prescription and over-the-counter drugs.

## Use Case

This skill helps physicians and healthcare professionals stay informed about new FDA-approved drugs. It provides structured guidance for querying the openFDA API to find recently approved prescription drugs, look up drug details, and retrieve regulatory information.

## What This Skill Does

- **Find recently approved prescription drugs** with date range filtering
- **Look up specific drugs** by brand name, generic name, or application number
- **Retrieve drug labeling information** including indications, dosage, and warnings
- **Check adverse events** reported for specific drugs
- **Find drug recalls** and enforcement reports
- **Query drug shortages** and NDC directory information

## Installation

Place the `openfda-drug` directory in your Claude skills directory (typically `~/.claude/skills/` or the equivalent directory for your Claude setup).

## Usage

The skill activates automatically when you ask about:
- FDA-approved drugs or drug approvals
- Specific prescription medications
- Drug labeling or prescribing information
- Adverse events or drug recalls
- NDC codes or drug regulatory information

Example queries:
- "What prescription drugs did the FDA approve this month?"
- "Find information about Dupixent including its approved uses"
- "Show me any new drugs approved since January 2025"
- "What is the NDA number for eliquis?"

## API Endpoints

The skill uses the openFDA drug API (`https://api.fda.gov/drug/`):

| Endpoint | Purpose |
|----------|---------|
| `drugsfda` | Approved drug products since 1939 |
| `label` | Structured product labeling |
| `event` | Adverse event reports (FAERS) |
| `ndc` | National Drug Code directory |
| `enforcement` | Drug recall reports |
| `drugshortages` | Drug shortage information |

## Important Notes

- Data is from the FDA's public openFDA API and is not validated for clinical decisions
- Adverse events do not prove causation — reports list drugs and reactions but no causal relationship is established
- Data may lag by several months depending on the endpoint
- The API has rate limits; use the `limit` parameter to fetch only what's needed

## Skill Structure

```
openfda-drug/
├── SKILL.md           # Main skill file with API guidance
├── README.md          # This file
└── references/
    └── quick-ref.md   # Quick reference for common queries
```

## License

This skill is provided as-is for educational and informational purposes.
