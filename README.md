# airtable-patient-intake-engagement-project-data
Patient intake and engagement dataset exported from Airtable as CSVs, including intake requests, contact attempts, scheduling status, outcomes, and helper dictionaries. Organized for quick analysis and reporting for an Intake Coordinator workflow.
# Patient Intake and Engagement Project Data

A clean export of an Airtable base used to track the intake funnel, outreach, scheduling, and outcomes. This repo lets anyone review the process, reproduce simple metrics, and build lightweight reports for an Intake Coordinator workflow. All data is synthetic or anonymized and for demo only.

## What's inside

- `/data/`
  - `intake_requests.csv`  // each inquiry, submission date, source, chief concern, insurance, priority
  - `contacts.csv`         // call, text, email attempts, timestamps, outcome per attempt
  - `patients.csv`         // deduped people, demographics, preferred contact, timezone
  - `scheduling.csv`       // scheduled, no show, completed, cancellations, provider, appointment time
  - `dispositions.csv`     // final status per lead, reasons lost, follow-up needed
  - `reference/*.csv`      // code lists like sources, reasons, insurance payers
- `/docs/`
  - `data_dictionary.md`   // field names, types, and meaning
  - `intake_metrics.sql`   // sample queries for KPIs

## How to use

1) Download or clone the repo  
2) Open CSVs in Excel or Google Sheets, keep UTF-8 encoding  
3) Join keys  
   - `patients.patient_id` to `intake_requests.patient_id`  
   - `intake_requests.request_id` to `contacts.request_id`  
   - `intake_requests.request_id` to `scheduling.request_id`  
   - `dispositions.request_id` final status  
4) Calculate core KPIs  
   - Inquiry to contact rate = contacted inquiries ÷ total inquiries  
   - Contact to scheduled rate = scheduled ÷ contacted  
   - Show rate = completed ÷ scheduled  
   - Median time to first contact and to first appointment  
5) Slice by source, insurance, weekday, time of day, coordinator

## Example insights

- Identify best contact windows by comparing first-attempt success by hour.  
- Find drop-off reasons from `dispositions.reason` to improve scripts or routing.

## Data notes

- Deidentified dataset, no PHI.  
- Timestamps in ISO 8601, timezone stored per record, convert before comparing.  
- Status values normalized in `reference/`.

## Reproduce quick metrics in Python

```python
# Reads CSVs and prints basic funnel metrics
import pandas as pd

# Load data
req = pd.read_csv('data/intake_requests.csv', parse_dates=['submitted_at'])
ct  = pd.read_csv('data/contacts.csv', parse_dates=['attempt_at'])
sch = pd.read_csv('data/scheduling.csv', parse_dates=['appt_at'])

# Contacted requests: any successful attempt
contacted_ids = ct.loc[ct['outcome'].eq('Reached'), 'request_id'].unique()
scheduled_ids = sch.loc[sch['status'].eq('Scheduled'), 'request_id'].unique()
completed_ids = sch.loc[sch['status'].eq('Completed'), 'request_id'].unique()

total = len(req)
contacted = len(set(contacted_ids))
scheduled = len(set(scheduled_ids))
completed = len(set(completed_ids))

print({
    'total_inquiries': total,
    'contact_rate': round(contacted / total, 3) if total else 0,
    'contact_to_scheduled': round(scheduled / contacted, 3) if contacted else 0,
    'show_rate': round(completed / scheduled, 3) if scheduled else 0
})
MM
