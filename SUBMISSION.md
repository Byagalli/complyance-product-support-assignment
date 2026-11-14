# Submission

- **Name:**  Ganesh Ramchandra Byagalli
- **Date submitted:**  14 November 2026
- **Repository link:**  https://github.com/Byagalli/complyance-product-support-assignment
- **Total time spent (real time):** 7 hours

> Replies below are drafts written on the **exercise clock** (Monday 16 November 2026, Asia/Dubai). Keep your real time in the time log at the end. Put screenshots, log excerpts and query output in `evidence/` and link to them.

**Incident 2 – Evidence**
- [API investigation](evidence/incident-2-api/api-investigation.txt)
- [Retry code findings](evidence/incident-2-api/retry-code-findings.txt)

**Incident 3 – Evidence**
- [Import summary](evidence/incident-3-al-noor/import-summary.txt)
- [VAT calculations](evidence/incident-3-al-noor/vat-calculations.txt)

**Incident 4 – Evidence**
- [Import summary](evidence/incident-4-sahara/import-summary.txt)
- [Retry investigation](evidence/incident-4-sahara/retry-investigation.txt)


> **Required:** the Acknowledgement and Priority sections for **all five** items, and the full write-up (investigation log onwards) for **at least three** items of your choice. For an item you did not fully investigate, replace the remaining sections with "Not investigated — what I would check next: …".

- **Items I fully investigated, and why I chose them:**

Incident 2 – Internal API / Multiple customers: I selected this incident because it had the potential to affect multiple customers and production invoice processing. I investigated API container health, restart information, application logs, invoice statuses, and the Go submission and retry implementation. I also examined how the application handles ASP responses. The exact cause of the API restarts and the reported 502 errors could not be conclusively established from the available evidence.

Incident 3 – Al Noor Fuel: I selected this incident because invoice rejections could affect the customer's VAT reporting and auditor review. I traced the affected invoices through the application and ASP lifecycle, checked their database records and event histories, and compared the invoice totals against the expected VAT calculation. I identified three invoices with BR-CO-15 total calculation discrepancies.

Incident 4 – Sahara Medical: I selected this incident because the customer reported that only some invoices had been received despite successful HTTP 201 responses and Submitted statuses. I investigated the import, invoice records, event histories, application logs, ASP mock logs and Go submission/retry code. I identified 32 invoices remaining in Submitted without a recorded final outcome in the available lab evidence. I also identified retry-handling gaps, although I could not prove that they caused the specific missing confirmations.

---

## Incident 1 — `inbox/01-oasis-builders-upload-stuck.md`

**### Acknowledgement**

- **Sent at (exercise clock):** Draft prepared for 16 November 2026, 09:15 GST. Not actually sent.

- **Reply:**

Hi Rashid,

Thank you for reporting the issue with your November invoice upload. I understand the urgency, especially with your month-end closing activities and customers waiting for their invoices.

I am checking the previous upload attempts and invoice-level submission statuses to identify which invoices have already been accepted and which still require action.

Please avoid uploading the complete workbook again until we reconcile the existing submissions. This will help prevent duplicate invoices.

I will provide an update by 11:05 GST.

Regards,

Product Support Team

**### Priority and justification**

- **Priority:** P2

- **Why:** The customer reports that the monthly upload was retried three times and some invoices now return "document number already exists". This affects a subset of invoices and creates a risk of duplicate submissions. There is no confirmed evidence of a complete production outage.

- **SLA deadlines I am working to:** Received at 09:05 GST on Monday 16 November 2026.
  - First response: within 1 hour — 10:05 GST.
  - Next update: 11:05 GST (planned).
  - Resolution target: within 8 business hours — 17:05 GST.

**### Investigation log**

Not investigated to completion — what I checked and what I would check next:

- Found an earlier Oasis import dated 31 October 2026 containing 8 Accepted invoices.
- Replayed the November sample in the local environment. The replay produced 8 Accepted invoices with ASP message IDs.
- Confirmed that the November replay invoice numbers were not present in the earlier import.
- The original customer attempts have not been identified or reconciled.
- Next, obtain the original import IDs and invoice numbers, inspect their event histories, correlate ASP message IDs, and identify which invoices were accepted, rejected or never received.

**### Root cause**

Not investigated to completion — the original customer attempts have not been reconciled, so the root cause of the duplicate-number errors is not confirmed.

**### Customer resolution reply OR escalation note**

- **Which one and why:** Further investigation and reconciliation are required before providing a definitive resolution. The original customer upload attempts have not been identified in the local database.

**### Workaround offered**

Advise the customer to hold off on uploading the complete workbook again. Identify invoices already accepted and resubmit only invoices confirmed not received, following a safe recovery procedure.

**### Prevention ideas**

1. Provide invoice-level import and submission status.
2. Add duplicate invoice checks before resubmission.
3. Maintain ASP message IDs and submission history for reconciliation.
4. Provide a safe retry mechanism for partially completed batches.

---

















## Incident 2 — API restarts and Marina Bay intermittent 502 errors

### Acknowledgement

- **Sent at (exercise clock):** Draft prepared for 16 November 2026, 09:50 GST. Not actually sent.

- **Reply:**

Hi Priya,

Thank you for raising the API availability alert. I understand the urgency, especially with multiple customers experiencing intermittent upload failures and Marina Bay Hospitality preparing for its upcoming go-live.

I am investigating the API restarts, upload errors, and invoices that remain in Validated status. I am also checking whether affected invoices have reached the ASP before recommending any retry.

I will provide you with an update by 10:30 GST.

Regards,

Product Support Team

### Priority and justification

- **Priority:** P1 pending confirmation of invoice delivery impact. P2 if invoices are still reaching the ASP despite intermittent failures.

- **Why:** The alert reports three API restarts within 15 minutes, intermittent HTTP 502 errors affecting multiple customers, and invoices stuck in Validated status since 09:30. This indicates a potentially broad production issue. Marina Bay Hospitality also has a go-live scheduled for 18 November, increasing the business impact.

  The SLA matrix defines P1 as production down or no invoices reaching the ASP for one or more customers. Since complete delivery failure has not been established, P1 remains provisional.

- **SLA deadlines I am working to:** Internal alert received at 09:42 GST on Monday 16 November 2026.
  - First response: within 15–30 minutes if P1 — 09:57–10:12 GST.
  - If classified P2: first response within 1 hour — 10:42 GST.
  - Next update: 10:30 GST (planned).
  - Resolution target: within 4 hours — 13:42 GST if P1, or within 8 hours — 17:42 GST if P2, subject to the documented SLA and confirmed priority.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 16 Nov, 09:42 | Reviewed the internal API availability alert. | Alert reported three API restarts within the previous 15 minutes. |
| 16 Nov, 09:48 | Reviewed Priya Nair's incident report. | Multiple customers reported intermittent HTTP 502 upload errors, and some invoices were reportedly stuck in Validated status since 09:30. |
| Lab investigation — 14 Nov, approximately 16:58 GST | Inspected API container state using Docker. | API container had `RestartCount=2`, `OOMKilled=false`, and `ExitCode=0`. |
| Lab investigation — 14 Nov | Reviewed API logs around the first startup. | API was processing imports and ASP responses immediately before a new startup message at 12:58:05 UTC. No panic, fatal error, or shutdown message was visible in the inspected excerpt. |
| Lab investigation — 14 Nov | Reviewed API logs around the second startup. | API was processing imports and ASP responses immediately before another startup message at 12:58:37 UTC. No corresponding shutdown or fatal error was visible in the inspected excerpt. |
| Lab investigation — 14 Nov | Inspected the current API container health. | Container was running and healthy. Health endpoint returned `{"status":"ok"}` and `FailingStreak=0`. |
| Lab investigation — 14 Nov | Reviewed API Docker Compose configuration. | API has a 128 MB memory limit, health checks every 5 seconds, and `restart: unless-stopped`. |
| Lab investigation — 14 Nov | Reviewed API logs during synthetic load. | Multiple ASP HTTP 429 responses were recorded for several senders, including MBH, PCE, ZPL and DRT. Some submissions received HTTP 200. |
| Lab investigation — 14 Nov | Reviewed the API submission worker source code. | HTTP 429 is not explicitly handled by the response switch. The worker does not process the `Retry-After` header. |
| Lab investigation — 14 Nov | Reviewed the retry implementation. | The retry query selects only Validated and Failed invoices. Submitted invoices are excluded. |

*Note: Lab investigation timestamps are from the local test environment on 14 November, not the Monday 16 November exercise clock. API log timestamps are UTC; 12:58 UTC corresponds to approximately 16:58 GST. Synthetic load results are supporting evidence, not proof of the exact production incident.*

### Root cause

- **What happened:** The API container restarted twice during the local synthetic load test. During the same test, the ASP mock returned HTTP 429 for multiple invoice submissions. The current API implementation does not explicitly handle HTTP 429 or schedule a retry using `Retry-After`.

- **Where (component, `file:line`):**
  - API submission response handling: `services/api/internal/submit/submit.go` — ASP response status switch.
  - Retry selection: `services/api/internal/store/store.go` — `Retryable()` function.
  - Retry endpoint: `services/api/internal/httpapi/httpapi.go` — `retryImport()` function.
  - API container configuration: `docker-compose.yml` — API memory limit, health check and restart policy.

  Exact source-code line numbers have not yet been recorded.

- **Why it happens:** The submission worker handles HTTP 200, 422 and selected 5xx responses, but has no explicit HTTP 429 branch. An invoice can therefore remain in Submitted status without a final outcome when a 429 response is returned. The retry query selects only Validated and Failed invoices, excluding Submitted invoices.

  Separately, the reason for the API container restarts has not been established. Docker reported `OOMKilled=false`, and the inspected application log excerpts did not show a panic, fatal error, or shutdown reason.

- **Evidence:**
  - Docker inspection: API `RestartCount=2`.
  - Docker inspection: `OOMKilled=false`, `ExitCode=0`.
  - API startup log messages at `12:58:05 UTC` and `12:58:37 UTC`.
  - API health status: `healthy`, `FailingStreak=0`.
  - API logs contain ASP HTTP 429 responses for multiple senders during synthetic load.
  - `services/api/internal/submit/submit.go`: HTTP 429 is not explicitly handled.
  - `services/api/internal/store/store.go`: retry query excludes Submitted invoices.
  - `services/api/internal/httpapi/httpapi.go`: retry endpoint calls `Retryable()`.
  - `docker-compose.yml`: API memory limit is 128 MB and restart policy is `unless-stopped`.

- **Confidence and open questions:** High confidence that the API restarted twice during the lab test and that the application has an HTTP 429 handling gap. The reason for the API restarts is unknown. It has not been established that ASP 429 responses caused the API restarts or Marina Bay's reported 502 errors. The affected Marina Bay invoice IDs and their actual ASP delivery outcomes still need reconciliation.

### Customer resolution reply OR escalation note

- **Which one and why:** Engineering escalation note. The API restart cause requires investigation beyond L1. The invoice delivery status must also be reconciled with ASP records before recommending resubmission.

**Escalation — API availability and invoice delivery investigation**

**Priority:** P1 pending confirmation of complete delivery failure; otherwise P2.

**Customer/business impact:**
- Multiple customers report intermittent 502 errors during uploads.
- Some invoices are reportedly stuck in Validated status since 09:30 GST.
- Marina Bay Hospitality has a go-live scheduled for 18 November.

**Evidence supplied:**
- API container restart count and Docker state.
- API startup log excerpts.
- ASP HTTP 429 response samples from synthetic load testing.
- Submission worker response-handling code.
- Retry query and retry endpoint implementation.

**Requested Engineering actions:**

1. Identify the reason for the API restarts using Docker events, host logs, and resource metrics.
2. Check API health-check history and resource utilisation during the reported incident.
3. Reconcile Marina Bay's affected invoice IDs with ASP records.
4. Investigate intermittent 502 errors and determine whether they originated at the web, API, or upstream layer.
5. Implement explicit HTTP 429 handling, including `Retry-After` and durable retry scheduling.
6. Ensure invoices left in Submitted status can be safely reconciled and recovered without creating duplicates.

L1 Product Support will continue customer communication, monitor affected invoice statuses, and provide updates according to the applicable SLA.

### Workaround offered

- Do not blindly retry invoices with uncertain delivery status.
- Reconcile invoices stuck in Validated or Submitted status against ASP records.
- Confirm which invoices have been accepted, rejected, or not received before resubmitting.
- For invoices confirmed not received by the ASP, coordinate a safe retry after Engineering confirms the appropriate recovery process.
- Monitor API health and upload results while the investigation continues.

### Prevention ideas

1. Implement explicit HTTP 429 response handling and respect the ASP `Retry-After` header.
2. Introduce durable retry scheduling with exponential backoff and appropriate retry limits.
3. Add reconciliation for invoices left in Submitted status without a final ASP outcome.
4. Ensure retry workflows can safely recover eligible invoices without duplicate submissions.
5. Add monitoring and alerting for API restarts, HTTP 502/503 responses, and invoices stuck in Validated or Submitted status.
6. Review API resource utilisation and container memory limits under expected production load.
7. Add automated integration tests covering ASP rate limiting, API restarts, and interrupted submission workflows.


---












## Incident 3 — `inbox/03-al-noor-fuel-br-co-15.md`

### Acknowledgement

- **Sent at (exercise clock):** Draft prepared for 16 November 2026, 10:25 GST. Not actually sent.
- **Reply:**

Hi Fatima,

Thank you for reporting the invoice rejections. I understand that this is important because your auditor is reviewing the VAT records.

I am investigating the rejected invoices and checking the ASP validation responses. I will share the affected invoice references and the required corrective action once the checks are complete.

I will provide you with an update by 12:15 GST.

Regards,  

Product Support Team

### Priority and justification

- **Priority:** P2

- **Why:** Three of the six invoices in the reproduced batch were rejected by the ASP, while the other three were accepted. This affects a subset of invoices and has additional business impact because the customer has an ongoing VAT audit. The issue has a customer-data correction path rather than evidence of a complete production outage.

- **SLA deadlines I am working to:** Received at 10:15 GST on Monday 16 November 2026.
  - First response: within 1 hour — 11:15 GST.
  - Next update: 12:15 GST (planned).
  - Resolution target: within 8 hours — 18:15 GST, subject to the documented business-hours SLA and customer dependency for corrected invoices.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 16 Nov, 10:15 | Read the customer report and identified the BR-CO-15 rejection complaint. | Customer reported that some invoices were rejected while others from the same file were accepted. |
| Lab replay — 14 Nov, 16:55 GST | Replayed the Al Noor sample using the provided load test. | Batch ID `f556c4e1-5334-41bc-811b-d8f195813253`; 6 invoices, 3 Accepted and 3 Rejected. |
| Lab replay — 14 Nov, 16:55 GST | Opened invoice `ANF-2611-0042-HT88`. | ASP rejected it under BR-CO-15. Actual total AED 15,022.42; expected AED 15,022.43. |
| Lab replay — 14 Nov, 16:55 GST | Opened invoice `ANF-2611-0044-HT88`. | ASP rejected it under BR-CO-15. Actual total AED 11,817.57; expected AED 11,817.59. |
| Lab replay — 14 Nov, 16:55 GST | Opened invoice `ANF-2611-0046-HT88`. | ASP rejected it under BR-CO-15. Actual total AED 5,610.99; expected AED 5,611.00. |
| Lab replay — 14 Nov, 16:55 GST | Compared invoice status histories and ASP responses. | All three invoices passed portal validation, were submitted to the ASP, and were then rejected. None had an ASP reference. |

*Note: The lab replay timestamps above are from the local test environment, not the Monday 16 November exercise clock.*

### Root cause

- **What happened:** Three invoices contained inconsistent VAT-inclusive totals. The ASP rejected them because the declared total including VAT did not equal the total excluding VAT plus the VAT amount.

- **Where (component, `file:line`):** The observed rejection occurred in the ASP validation component. The portal validation accepted these invoices before submission. Exact source-code file and line responsible for the validation gap have not yet been identified.

- **Why it happens:** The three invoice totals differ from the expected calculated totals by AED 0.01, AED 0.02 and AED 0.01 respectively. The portal validation did not catch these discrepancies, but the ASP enforced BR-CO-15.

- **Evidence:**
  - Batch ID: `f556c4e1-5334-41bc-811b-d8f195813253`.
  - Batch result: 6 invoices; 3 Accepted and 3 Rejected.
  - Invoice `ANF-2611-0042-HT88`: BR-CO-15, actual 15022.42, expected 15022.43.
  - Invoice `ANF-2611-0044-HT88`: BR-CO-15, actual 11817.57, expected 11817.59.
  - Invoice `ANF-2611-0046-HT88`: BR-CO-15, actual 5610.99, expected 5611.00.
  - Evidence screenshots: `evidence/al-noor-batch-summary.png`, `evidence/al-noor-invoice-0042.png`, `evidence/al-noor-invoice-0044.png`, `evidence/al-noor-invoice-0046.png`.

- **Confidence and open questions:** High confidence in the immediate cause because all three ASP responses explicitly identify BR-CO-15. The underlying reason for the incorrect totals in the customer's source system is not known. The portal-to-ASP validation gap should also be reviewed by Engineering.

### Customer resolution reply OR escalation note

- **Which one and why:** Customer resolution reply. The immediate issue is incorrect invoice totals in the source data. Customer correction is required before resubmission. Engineering review of the portal validation gap can be raised separately.

Hi Fatima,

We investigated the reproduced batch and identified three invoices rejected by the ASP under validation rule BR-CO-15.

The invoice totals including VAT do not match the sum of the totals excluding VAT and VAT amounts. The discrepancies are AED 0.01, AED 0.02 and AED 0.01 across the three affected invoices.

The other three invoices in the reproduced batch were accepted.

Please correct the totals in your source system and regenerate the three affected invoices. Before resubmission, we will verify the invoice references and check for existing submissions to avoid duplicate invoices.

We will confirm the outcome after the corrected invoices are submitted.

Regards,  
Product Support Team

### Workaround offered

Ask the customer to correct the three affected invoice totals in the source system and regenerate the invoices. Verify invoice identifiers and existing submission status before resubmitting. Do not blindly retry or create duplicate submissions.

### Prevention ideas

1. Add BR-CO-15 validation in the portal before invoices are submitted to the ASP.
2. Use decimal-safe monetary calculations and consistent rounding rules.
3. Display the expected and actual totals in validation error messages.
4. Add automated test cases for VAT calculation and rounding edge cases.
5. Monitor ASP rejection rates by validation rule so repeated data-quality issues can be identified early.
---


















## Incident 4 — `inbox/04-bluepeak-sahara-not-received.md`

### Acknowledgement

- **Sent at (exercise clock):** Draft prepared for 16 November 2026, 11:40 GST. Not actually sent.
- **Reply:**

Hi Daniel,

Thank you for reporting the missing ASP references. I understand your concern, especially because your connector received HTTP 201 while the portal shows the invoices as Submitted.

We are checking the invoice-level processing history and ASP delivery records to identify which invoices were accepted and which still need reconciliation.

Please hold off on resending the batch for now. Resending before checking the ASP records could create duplicate submissions.

I will provide an update by 15:00 GST.

Regards,

Product Support Team

### Priority and justification

- **Priority:** P2

- **Why:** The customer reports that only some of the 40 invoices appear to have reached the hospital, while the portal shows Submitted. This is a partial-delivery issue affecting a healthcare customer. There is no confirmed evidence that all invoice submissions have failed.

- **SLA deadlines I am working to:** Received at 11:30 GST on Monday 16 November 2026.
  - First response: within 1 hour — 12:30 GST.
  - Next update: 15:00 GST (planned).
  - Resolution target: within 8 business hours. Assuming the clock pauses outside the documented 08:00–18:00 GST business window, the target is Tuesday 17 November at 09:30 GST.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 16 Nov, 11:30 | Read the customer report. | Bluepeak ERP reported that 40 invoices were posted with HTTP 201, but the hospital received only some of them. |
| Lab investigation — 14 Nov | Replayed the supplied Sahara sample using the local test environment. | Import ID `59c3e017-87a7-49a2-8476-2d54d03dabbf`; 40 invoices, 8 Accepted and 32 Submitted. |
| Lab investigation — 14 Nov | Inspected invoice-level details in the portal. | Example invoice `SMS-1116-009-HT88` remained Submitted, with no ASP reference or recorded error. |
| Lab investigation — 14 Nov | Queried the `invoices` and `invoice_events` tables. | The first 8 invoices had Accepted events. The remaining 32 had Parsed, Validated and Submitted events only. |
| Lab investigation — 14 Nov | Reviewed API logs for Sahara invoice numbers. | No matching Sahara invoice-specific response records were found in the inspected API logs. |
| Lab investigation — 14 Nov | Reviewed ASP mock logs for Sahara invoice numbers. | No matching Sahara-specific acceptance or rejection evidence was found in the inspected logs. |
| Lab investigation — 14 Nov | Inspected the submission worker and retry implementation. | HTTP 429 is not explicitly handled by the submission worker. The retry query selects only Validated and Failed invoices, excluding Submitted invoices. |

*Note: Lab investigation timestamps above refer to the local test environment on 14 November, not the Monday 16 November exercise clock.*

### Root cause

- **What happened:** The reproduced batch contains 40 invoices. Eight reached Accepted status, while 32 remain Submitted without a final recorded ASP outcome.

- **Where (component, `file:line`):**
  - Invoice state and event history: PostgreSQL tables `invoices` and `invoice_events`.
  - Submission response handling: `services/api/internal/submit/submit.go`.
  - Retry selection: `services/api/internal/store/store.go`, `Retryable()` function.
  - Retry endpoint: `services/api/internal/httpapi/httpapi.go`, `retryImport()` function.

  Exact source-code line numbers have not yet been recorded.

- **Why it happens:** The database records Parsed, Validated and Submitted events for the 32 affected invoices, but no Accepted or Rejected event. The inspected logs do not establish whether the ASP received these specific invoices.

  A separate code-level issue was identified: HTTP 429 is not explicitly handled by the submission worker, and the retry query selects only Validated and Failed invoices. Therefore, an invoice left in Submitted status after an unhandled response may not be recovered through the existing retry endpoint.

  However, I did not find evidence proving that HTTP 429 caused the 32 Sahara invoices to remain Submitted.

- **Evidence:**
  - Import ID: `59c3e017-87a7-49a2-8476-2d54d03dabbf`.
  - Batch result: 40 invoices; 8 Accepted and 32 Submitted.
  - Example invoice: `SMS-1116-009-HT88`, status Submitted, with no ASP reference.
  - SQL query of `invoices` and `invoice_events` showed only Parsed, Validated and Submitted events for the 32 invoices.
  - API and ASP log searches did not find matching Sahara invoice-specific outcomes.
  - `services/api/internal/submit/submit.go`: response switch does not explicitly handle HTTP 429.
  - `services/api/internal/store/store.go`: retry query excludes Submitted invoices.
  - `services/api/internal/httpapi/httpapi.go`: retry endpoint calls `Retryable()`.

- **Confidence and open questions:** High confidence in the invoice status counts and the absence of final outcomes in the local database. High confidence in the identified code-level retry gap. The actual ASP delivery status of the 32 invoices remains unknown. Engineering must reconcile the invoice references with ASP records before any resubmission.

### Customer resolution reply OR escalation note

- **Which one and why:** Engineering escalation. The 32 invoices have no final recorded ASP outcome, and L1 cannot safely determine whether they were received without ASP-side reconciliation.

**Escalation — Sahara Medical partial invoice delivery**

**Priority:** P2.

**Customer impact:**
- 40 invoices in the reproduced batch.
- 8 Accepted.
- 32 remain Submitted without a final recorded outcome.
- Customer reports that the hospital received only some invoices.
- Customer has paused resubmission due to duplicate risk.

**Evidence supplied:**
- Import ID `59c3e017-87a7-49a2-8476-2d54d03dabbf`.
- Invoice-level database event history.
- Portal status for example invoice `SMS-1116-009-HT88`.
- API and ASP log searches.
- Submission worker and retry-query source references.

**Requested Engineering actions:**

1. Reconcile all 32 invoice numbers with ASP-side document records.
2. Confirm which were received, accepted, rejected or not received.
3. Investigate why the local system has no final response events for these invoices.
4. Review HTTP response handling and durable retry behaviour.
5. Provide a safe recovery procedure that prevents duplicate submissions.

L1 Product Support will own customer communication and provide updates according to the P2 SLA.

### Workaround offered

Ask the customer not to resend the batch until the 32 invoice references have been reconciled with ASP records. Resubmit only invoices confirmed not received, using an Engineering-approved recovery procedure.

### Prevention ideas

1. Persist ASP request and response correlation details for every invoice.
2. Add monitoring for invoices remaining Submitted without a final outcome.
3. Implement explicit HTTP 429 handling and respect the `Retry-After` header.
4. Add a reconciliation process for uncertain submissions.
5. Ensure retries are idempotent and cannot create duplicate documents.
















## Incident 5 — `inbox/05-ghaf-cafe-date-error.md`

**### Acknowledgement**

- **Sent at (exercise clock):** Draft prepared for 16 November 2026, 12:20 GST. Not actually sent.

- **Reply:**

Hi Hessa,

Thank you for reporting the invoice date validation issue.

I understand that the date is valid in your Excel workbook and that the upload is being rejected because of its format.

I will check the date-format validation requirements and confirm the correct format to use. Please do not repeatedly upload the same file until we verify the correction.

I will provide an update by 16:00 GST.

Regards,

Product Support Team

**### Priority and justification**

- **Priority:** P3

- **Why:** The issue affects one customer whose invoices are rejected because of an issue-date format validation error. The customer can potentially correct the date format and retry after checking for duplicate submissions. There is no evidence of a wider production outage.

- **SLA deadlines I am working to:** Received at 12:10 GST on Monday 16 November 2026.
  - First response: within 4 business hours — 16:10 GST.
  - Next update: 16:00 GST (planned).
  - Resolution target: within 3 business days — Thursday 19 November 2026, 12:10 GST.

**### Investigation log**

Not investigated to completion — what I would check next:

- Reproduce the issue using the supplied Ghaf sample.
- Inspect the rejected invoice's date value and portal validation response.
- Check the date parsing and validation logic in the importer.
- Test whether the ISO date format `2026-11-13` is accepted.
- Verify invoice identifiers and duplicate status before resubmission.

**### Root cause**

Not investigated to completion — the reported issue indicates a date-format validation problem, but the exact validation behaviour and source-code location have not been independently established.

**### Customer resolution reply OR escalation note**

- **Which one and why:** Customer resolution reply after confirming the required date format through a controlled test.

The intended customer guidance is to use the required `YYYY-MM-DD` date format, for example `2026-11-13`, after confirming the importer accepts the corrected format.

**### Workaround offered**

Ask the customer to use the required `YYYY-MM-DD` date format after confirming the corrected format is accepted. Verify invoice status before resubmission.

**### Prevention ideas**

1. Provide clear date-format instructions in the upload template.
2. Support valid Excel date cells consistently.
3. Show the exact invalid date and expected format in validation errors.
4. Add automated tests for date parsing and validation.

---







## Overall

### Prioritisation

The order in which you worked the five items, and why. Did you change your mind as you learned more?

**## Overall**

**### Prioritisation**

I prioritised the incidents based on customer impact, production availability, invoice delivery risk and SLA requirements.

My priority assessment was:

1. **Incident 2 — API restarts and Marina Bay 502 errors (provisional P1):** Potentially broad production impact affecting multiple customers, so it required immediate attention. P1 remained provisional until complete invoice delivery failure could be confirmed.

2. **Incident 1 — Oasis Builders (P2):** Month-end upload issue with repeated attempts and duplicate invoice risks.

3. **Incident 3 — Al Noor Fuel (P2):** Three of six invoices were rejected, affecting VAT compliance and an ongoing audit.

4. **Incident 4 — Sahara Medical (P2):** Partial invoice delivery with uncertain ASP outcomes and duplicate-submission risk.

5. **Incident 5 — Ghaf Tree Café (P3):** A single-customer date-format issue with a potential data correction path.

I selected Incidents 2, 3 and 4 for full investigation because they provided opportunities to investigate production availability, invoice validation and external ASP delivery behaviour. I documented the remaining two incidents with their priorities, acknowledgements and next investigation steps.

As I investigated the API incident, I kept P1 provisional because the available evidence did not establish complete delivery failure. The local test showed some invoices reaching the ASP, so the actual production impact would need confirmation before finalising the priority.

**### What I would do differently**

- Establish a clean baseline before replaying sample files.
- Capture import IDs, invoice IDs and ASP message IDs immediately.
- Save relevant logs and SQL query outputs in the evidence directory during the investigation.
- Avoid repeated uploads when duplicate-submission risk exists.
- Verify exact source-code line numbers before documenting code findings.
- Spend more time understanding the ASP submission lifecycle and retry mechanism before recommending recovery actions.
- Separate local synthetic test findings from the fictional production incident timeline more clearly.

**### What I learned / what I would need to learn**

During this assignment, I learned how to investigate invoice-processing issues across the frontend, API, database and ASP integration.

I used Docker Compose to manage services and inspect container health, restart counts and logs. I used PostgreSQL queries to examine imports, invoice statuses and event histories.

I learned how to trace invoice states such as Parsed, Validated, Submitted, Accepted and Rejected. This helped me distinguish between an invoice being submitted to an ASP and an invoice actually being accepted by it.

I also inspected Go source code to understand HTTP response handling and retry selection. I used Python's built-in `zipfile` and XML libraries to inspect an Excel workbook without uploading it again.

Areas I would study further:

1. Go HTTP clients, error handling and context timeouts.
2. Distributed systems, retries, idempotency and duplicate prevention.
3. PostgreSQL queries and transaction handling.
4. Docker monitoring, container events and resource utilisation.
5. Observability using Prometheus, Grafana and Loki.
6. Peppol and PINT AE invoice validation rules.

**### Time log (real time)**

| Date | From–to | What I worked on |
|---|---|---|
| 14 November 2026 | Fill actual start–end time | Environment setup, Docker services and understanding application architecture |
| 14 November 2026 | Fill actual start–end time | API incident investigation, logs, container health and source-code review |
| 14 November 2026 | Fill actual start–end time | Al Noor and Sahara invoice investigation, SQL queries and ASP response analysis |
| 14 November 2026 | Fill actual start–end time | Oasis and Ghaf initial investigation, report preparation and evidence review |

Total time spent: 7 hours.

**### AI usage disclosure**

I used ChatGPT as an AI assistance tool to understand the assignment, plan investigation steps, explain Docker and PostgreSQL commands, understand Go source code, inspect Excel workbook structure using Python, and improve the clarity of my written report.

I did not rely on AI-generated findings without verification. I checked investigation results against the local application, PostgreSQL records, Docker container state, application logs and source code. Where evidence was incomplete, I documented the limitation rather than claiming a confirmed root cause.

All customer communications included in this submission are drafts for the exercise. No actual customer emails were sent.










### What I would do differently

If I were handling this assignment again, I would follow a more structured investigation process from the beginning.

1. **Prioritise by business impact:** I would first investigate the API incident because multiple customers were affected and invoices could be stuck in production. I would then investigate Oasis because of its month-end closing requirement, followed by the other customer issues.
2. **Capture evidence before making changes:** I would record the initial container status, restart count, logs, import IDs, invoice statuses and database events before replaying any workload. This would make it easier to distinguish original customer activity from test activity.
3. **Investigate invoice-level history:** Instead of relying only on an import's overall status, I would trace individual invoices through Parsed, Validated, Submitted, Accepted and Rejected states. I would also correlate ASP message IDs with invoice numbers.
4. **Avoid unsafe retries:** I would never recommend uploading an entire workbook again when some invoices may already have reached the ASP. I would first reconcile which invoices were accepted, rejected or still awaiting confirmation.
5. **Use a consistent investigation checklist:** For every ticket, I would follow the same sequence: understand the customer impact, check the SLA, inspect logs, identify the import ID, query the database, review relevant code, test a safe reproduction and document the findings.
6. **Keep confirmed facts separate from hypotheses:** For example, I found code paths that do not properly handle HTTP 429 responses, but I could not prove that this caused the specific Sahara invoices to remain Submitted or caused the API restarts. I would make this distinction even clearer in the initial investigation notes.
7. **Verify the monitoring environment earlier:** I would check Grafana and the available logs at the start, so that any dashboard limitation is identified early and alternative investigation methods can be used.

### What I learned / what I would need to learn

This assignment helped me understand how a production support engineer investigates problems across an application, database, external integrations and infrastructure.

**Concepts and tools I learned or explored:**

- **Docker and Docker Compose:** I learned how to start and stop services, inspect container health, check restart counts, view logs and understand service dependencies.
- **Linux/terminal troubleshooting:** I practised using terminal commands to inspect running services, search logs and investigate the application environment.
- **Loki and log queries:** I explored filtering structured logs by service and log level to narrow down errors and investigate API and ASP activity.
- **PostgreSQL:** I used SQL queries to inspect imports, invoices and invoice event histories. This helped me understand why an import-level Completed status does not necessarily mean every invoice was accepted by the ASP.
- **API integration and HTTP status codes:** I explored the difference between HTTP 200, 201, 422, 429 and 5xx responses, and how external ASP responses affect invoice processing.
- **Retry and duplicate handling:** I learned why retrying an invoice is not always safe, particularly when the external system may already have received it.
- **Application code investigation:** I reviewed the Go API code, especially the submission and retry logic, to understand how invoice statuses are updated and which statuses are eligible for retry.
- **SLA management:** I learned to distinguish priority, first-response deadlines, update frequency and resolution targets, including how business-hour SLAs pause outside working hours.

**How I got up to speed:**

I started by reading the repository documentation, SLA matrix, service configuration and ticket descriptions. Then I started the environment, reproduced the provided sample workloads and compared application logs with database records. When I found unclear behaviour, I inspected the relevant Go code to understand the implementation rather than assuming the cause from the error message alone.

**What I would study next:**

1. Production-grade observability using Prometheus, Grafana, Loki and distributed tracing.
2. Go application debugging, goroutines, concurrency, context cancellation and graceful shutdown.
3. Reliable message processing, retry policies, exponential backoff, dead-letter queues and idempotency.
4. External API integration patterns, especially handling HTTP 429, Retry-After and uncertain delivery outcomes.
5. PostgreSQL query optimisation, transactions, indexing and data consistency.
6. Incident management, severity assessment, escalation procedures and customer communication during production incidents.

### Time log (real time)

| Date | From–to | What I worked on |
|---|---|---|
| 14 Nov