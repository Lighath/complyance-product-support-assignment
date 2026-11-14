# Submission

- **Name:** K. Raghavv Lighath
- **Date submitted:** 16 November 2026
- **Repository link:** [Add your GitHub repository URL]
- **Total time spent (real time):** Approximately 3–4 hours

> All reply times below are proposed exercise-clock times (Asia/Dubai). They are drafts, not messages actually sent.

## Items fully investigated, and why I chose them

I prioritised the production API alert, Oasis Builders' duplicate-risk upload, and Sahara Medical's missing invoices because they involve potential invoice delivery failures and business impact. I also investigated Al Noor's validation errors because of the auditor's request.

---

## Incident 1 — Oasis Builders upload stuck

### Acknowledgement

- **Sent at (exercise clock):** 09:15
- **Reply:**

> Hello Rashid,
>
> I understand that your November upload has remained in Processing after multiple attempts and that you are now seeing document-number-already-exists warnings. I appreciate the month-end urgency.
>
> Please do not retry the upload for now. I am checking the invoice records and submission statuses to identify what has already been processed and what remains outstanding. I will provide an update within the next 2–4 hours.
>
> Regards,
> Support Team

### Priority and justification

- **Priority:** P2
- **Why:** One customer's upload is affected, with duplicate-number warnings after repeated attempts. Month-end close and customers waiting for invoices increase the business impact. No evidence currently proves that all customers are unable to submit invoices.
- **SLA:** First response within 1 hour (10:05); resolution target within 8 business hours (17:05); updates every 2–4 hours.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 09:15 | Read customer report | Multiple upload attempts; some invoices show document number already exists |
| 09:25 | Checked invoice status counts | Accepted: 230; Submitted: 228; Validated: 24; Parsed: 18; Rejected: 7 |
| 09:35 | Ran duplicate invoice-number check | Query grouped by seller and invoice number returned zero duplicate groups |
| 09:45 | Reviewed API and ASP logs | ASP logs contain rate-limit errors and Retry-After: 1 second |
| 09:55 | Compared status evidence | Need to identify the exact November batch and correlate each invoice with its submission result before retrying |

### Root cause

- **What happened:** Repeated upload attempts left the customer uncertain about which invoices had been processed. The portal's duplicate-number warnings indicate that at least some invoice numbers may already exist.
- **Where:** Upload processing and invoice submission path. Exact source file and line have not yet been established.
- **Why it happens:** ASP logs show rate limiting. This may have contributed to delayed processing, but the available evidence does not prove it caused the customer's Processing screen or duplicate warnings.
- **Evidence:**
  - `evidence/invoice-status.txt`
  - `evidence/duplicate-check.txt`
  - `evidence/api-asp-logs.txt`
- **Confidence and open questions:** The duplicate check returned no duplicate groups in the queried dataset. This does not prove the batch is safe to resend. Need to correlate the customer's exact November file rows with invoice IDs, ASP message IDs and final statuses.

### Customer resolution reply OR escalation note

- **Which one and why:** Escalation to engineering/ASP operations is appropriate before any retry because invoice delivery state must be confirmed.

> Please keep the original upload file and do not submit it again yet. We are reconciling the invoice numbers against portal and ASP records. We will provide a per-invoice status list and confirm which invoices, if any, are safe to retry. We will not recommend a blind re-upload because it could create duplicate submissions.

### Workaround offered

Do not re-upload the complete file. Reconcile each invoice against its existing portal and ASP status first. Retry only invoices confirmed not to have been submitted, after support approval.

### Prevention ideas

- Add an upload idempotency key to prevent duplicate processing on retries.
- Show per-invoice processing results instead of leaving the whole batch on Processing.
- Provide a downloadable reconciliation report.
- Improve timeout and retry handling for ASP rate limits.

---

## Incident 2 — API restarted / Marina Bay

### Acknowledgement

- **Sent at (exercise clock):** 09:55
- **Sent to:** Support on-call; internal update to Priya Nair
- **Reply:**

> Hi Priya,
>
> I am taking ownership of the API restart alert. The alert reports three production API restarts within 15 minutes, and your report of intermittent 502 errors and invoices stuck in Validated may be related.
>
> I am checking API health, application logs and ASP submission behaviour. I will provide another update within 30–60 minutes. I will also include the current findings and outstanding risks for Marina Bay's Wednesday go-live.
>
> Regards,
> Support On-call

### Priority and justification

- **Priority:** P1
- **Why:** Production API instability is affecting multiple customers. Intermittent 502 errors and invoices not progressing indicate a potential service-wide submission issue. The P1 definition covers production being down or invoices not reaching the ASP for one or more customers.
- **SLA:** First response within 15–30 minutes of the 09:42 alert (by 10:12); resolution target 4 hours (13:42); updates every 30–60 minutes.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 09:55 | Reviewed Grafana alert | API restarted 3 times in 15 minutes; production severity critical |
| 10:05 | Reviewed customer-success escalation | Multiple customers report intermittent 502 errors; some invoices stuck in Validated since around 09:30 |
| 10:15 | Checked container status | Saved container health and status evidence |
| 10:25 | Reviewed API logs | Participant lookup failures include context deadline exceeded; seller lookup requests include context canceled |
| 10:35 | Reviewed ASP logs | Rate-limit errors and Retry-After: 1 second are present |
| 10:45 | Compared evidence | API dependency timeouts and ASP rate limiting are both observed; exact trigger for API restarts is not yet proven |

### Root cause

- **What happened:** The production API restarted three times in 15 minutes. Customers reported intermittent 502 errors and invoices stuck in Validated.
- **Where:** API service and its downstream participant/seller lookup dependencies.
- **Why it happens:** Logs show dependency lookup timeouts and cancelled requests. These are credible contributors to the instability, but the exact restart trigger requires container and application-level root-cause analysis.
- **Evidence:**
  - `evidence/container-status.txt`
  - `evidence/api-asp-logs.txt`
  - `evidence/asp-status.txt`
- **Confidence and open questions:** Production instability and dependency timeouts are confirmed observations. The precise cause of the three restarts, and whether ASP rate limiting caused or worsened them, remains unconfirmed.

### Customer resolution reply OR escalation note

- **Which one and why:** Immediate escalation to engineering/on-call infrastructure because this is a P1 production incident.

> P1 escalation: Production API restarted three times within 15 minutes. Multiple customers report intermittent 502 responses and invoices stuck in Validated. API logs show participant lookup timeouts and cancelled seller lookup requests. ASP logs also show rate limiting. Please investigate API restart events, dependency health, request timeouts and ASP submission backlog. Support retains ownership of customer updates. Marina Bay's 18 November go-live requires a confirmed stability assessment before we provide assurance.

### Workaround offered

Do not repeatedly retry uploads while the API is unstable. Preserve affected invoice references and reconcile their statuses after service recovery before resubmitting anything.

### Prevention ideas

- Add alerts for restart loops and dependency latency.
- Review API timeout and retry behaviour.
- Add circuit breakers and bounded retries for downstream dependencies.
- Monitor invoices stuck in Validated and ASP submission backlog.
- Run a stability and end-to-end invoice submission test before Marina Bay's go-live.

---

## Incident 3 — Al Noor Fuel BR-CO-15

### Acknowledgement

- **Sent at (exercise clock):** 10:30
- **Reply:**

> Dear Fatima,
>
> I understand that several November invoices were rejected with BR-CO-15 while other invoices from the same upload were accepted. I also understand that you need a clear written explanation for your auditor.
>
> We are reviewing the validation results and the VAT amount calculations for the rejected invoices. Please avoid changing the source figures until we identify the exact discrepancy. I will update you within 2–4 hours.
>
> Regards,
> Support Team

### Priority and justification

- **Priority:** P2
- **Why:** A subset of invoices has been rejected by a tax validation rule. The auditor's review makes a clear explanation important, but the evidence does not show that all invoice submissions are unavailable.
- **SLA:** First response within 1 hour (11:15); resolution target within 8 business hours (18:15); updates every 2–4 hours.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 10:30 | Read customer report | Several invoices rejected with BR-CO-15; other invoices accepted |
| 10:45 | Reviewed validation log messages | Rejected totals differ from expected VAT arithmetic by small amounts |
| 11:00 | Compared reported amounts | Differences observed: 0.01, 0.01 and 0.02 |
| 11:10 | Considered rounding as a hypothesis | Rounding or precision handling may explain the differences; calculation path still needs code-level verification |

### Root cause

- **What happened:** BR-CO-15 rejected invoices where the reported total including VAT did not equal the total excluding VAT plus VAT.
- **Where:** Invoice tax-total calculation or validation path. Exact source file and line have not been confirmed.
- **Why it happens:** The observed discrepancies are small decimal differences. Rounding or precision handling is a likely explanation, but must be verified against the source invoice values and calculation logic.
- **Evidence:**
  - `evidence/api-asp-logs.txt`
  - Validation messages recorded during the investigation:
    - 15022.42 expected 15022.43
    - 5610.99 expected 5611.00
    - 11817.57 expected 11817.59
- **Confidence and open questions:** The arithmetic mismatches are confirmed. The exact calculation stage responsible, and whether the source spreadsheet or platform rounding introduced the differences, remain open.

### Customer resolution reply OR escalation note

- **Which one and why:** Escalate to the tax-validation/product engineering team for confirmation before advising changes to financial figures.

> Dear Fatima,
>
> Our initial review confirms that the rejected invoices contain small differences between the submitted VAT-inclusive total and the amount expected by the BR-CO-15 calculation. We have observed differences of 0.01 and 0.02 in the reviewed examples.
>
> We are checking whether these differences originate from rounding or precision handling in the source data or validation process. Please do not manually alter the invoice totals until this review is complete. We will provide a written explanation suitable for your audit records.

### Workaround offered

Review the affected invoice's net amount, VAT amount and gross total using the applicable invoice calculation rules. Correct only values confirmed to be inaccurate, then revalidate. Do not apply an arbitrary one-cent adjustment.

### Prevention ideas

- Use consistent decimal precision and rounding rules.
- Add pre-submission validation that explains the expected total.
- Include the calculated net, VAT and gross values in rejection details.
- Add automated tests for decimal rounding edge cases.

---

## Incident 4 — BluePeak / Sahara Medical invoices not received

### Acknowledgement

- **Sent at (exercise clock):** 11:45
- **Reply:**

> Hi Daniel,
>
> Thanks for providing the batch details. I understand that your connector received HTTP 201, while Sahara's invoices remain Submitted and the buyer has received only some of them.
>
> Please do not resend the 40-invoice batch yet. We are checking the import, participant lookup and ASP acknowledgement records to establish which invoices were accepted downstream. I will provide an update within 2–4 hours.
>
> Regards,
> Support Team

### Priority and justification

- **Priority:** P2
- **Why:** A subset of a 40-invoice batch is reportedly missing at the buyer, and many portal records have empty ASP references. The payment run this week increases urgency. P1 would require evidence that invoices cannot reach the ASP at all; that has not been established across the service.
- **SLA:** First response within 1 hour (12:30). The 8-business-hour resolution target pauses outside 08:00–18:00 Dubai business hours; target is 09:30 on 17 November if no earlier resolution. Updates every 2–4 business hours.

### Investigation log

| Time (exercise clock) | What I checked / did | What I found |
|---|---|---|
| 11:45 | Read partner report | 40 invoices; connector received HTTP 201; portal shows Submitted |
| 11:55 | Checked import logs | Sahara import parsed 60 rows and 40 invoices; completed with 40 validated and 0 rejected |
| 12:05 | Checked seller record | Sahara Medical Supplies FZE identified as seller_id 5 |
| 12:15 | Reviewed API logs | Participant lookup failures show context deadline exceeded; seller lookup requests show context canceled |
| 12:25 | Queried seller invoice records | 52 records for seller_id 5: 20 Accepted and 32 Submitted |
| 12:35 | Checked Submitted records | 32 Submitted records had missing ASP message IDs and empty ASP responses in the inspected data |
| 12:45 | Considered resend request | A blind resend could create duplicate invoices; exact mapping of the 40 imported invoices to the 32 Submitted records still needs confirmation |

### Root cause

- **What happened:** The import completed successfully at the parsing and validation stage, but downstream submission/acknowledgement is incomplete for a group of Sahara records.
- **Where:** Participant lookup and downstream ASP submission/acknowledgement path.
- **Why it happens:** Participant lookup timeouts are recorded in API logs. The seller-level query found 32 Submitted records with no ASP message ID or response. This suggests the hand-off or acknowledgement stage did not complete for those records.
- **Evidence:**
  - `evidence/sahara-investigation.txt`
  - `evidence/api-asp-logs.txt`
  - `evidence/asp-status.txt`
- **Confidence and open questions:** Import success and the seller-level Submitted records are supported by the collected logs and query output. The exact mapping between the 40 invoices in this import and the 32 Submitted records has not yet been proven. Buyer-side delivery must also be confirmed with ASP message references.

### Customer resolution reply OR escalation note

- **Which one and why:** Escalate to engineering and ASP operations. The partner should not resend until invoice-level submission status is reconciled.

> Hi Daniel,
>
> The import endpoint accepted and processed the batch: 40 invoices were validated and none were rejected during import. However, our investigation found participant lookup timeouts and 32 Sahara seller records in Submitted status without ASP message IDs or responses.
>
> We are escalating the downstream submission and acknowledgement issue. Please do not resend the batch yet. We will reconcile the exact 40 invoice references and confirm which invoices reached the ASP before advising any retry. Please share connector request IDs and timestamps if available.

### Workaround offered

Hold the batch. Reconcile each invoice against portal and ASP records. Retry only invoices confirmed not to have reached the ASP, using a controlled process that prevents duplicates.

### Prevention ideas

- Track import, validation, ASP submission and buyer delivery as separate states.
- Alert on Submitted records without ASP message IDs after a defined interval.
- Persist participant lookup failures and provide safe retry handling.
- Provide partner-facing per-invoice acknowledgement reports.

---

## Incident 5 — Ghaf Tree Café date error

### Acknowledgement

- **Sent at (exercise clock):** 12:25
- **Reply:**

> Hello Hessa,
>
> I understand that your invoices were rejected even though 13/11/2026 is a valid date in the date format you normally use. The portal currently requires the issue date in YYYY-MM-DD format.
>
> Please change 13/11/2026 to 2026-11-13 in the spreadsheet and check the other date fields before uploading again. I will remain available if the corrected file is still rejected.
>
> Regards,
> Support Team

### Priority and justification

- **Priority:** P3
- **Why:** The issue affects one customer and is caused by an input-format requirement with a clear workaround. It is not evidence of a wider service issue.
- **SLA:** First response within 4 hours (16:10); resolution target within 3 business days; daily updates.

### Investigation log

See `evidence/ghaf-date-investigation.txt`.

### Root cause

- **What happened:** The customer entered 13/11/2026, while the portal expects YYYY-MM-DD.
- **Where:** Invoice issue-date input validation.
- **Why it happens:** The entered date uses DD/MM/YYYY, which does not match the portal's required format.
- **Evidence:** `evidence/ghaf-date-investigation.txt`
- **Confidence and open questions:** The validation message explicitly identifies the expected format. The date can be represented as 2026-11-13. No evidence currently shows a platform defect.

### Customer resolution reply OR escalation note

- **Which one and why:** Customer resolution reply with a clear workaround is appropriate.

> Hello Hessa,
>
> Your date is valid; the issue is the format expected by the upload template. Please use 2026-11-13 instead of 13/11/2026 and check the other date cells for the same format. If the corrected upload still fails, please share the updated error and we will investigate further.

### Workaround offered

Convert dates to YYYY-MM-DD before uploading.

### Prevention ideas

Provide a clear spreadsheet example and date-format instructions. Consider accepting common local date formats or returning a validation message with an example.

---

## Overall

### Prioritisation

I prioritised the P1 production API alert first because multiple customers were experiencing 502 errors and invoices were stuck. Oasis Builders was next because repeated upload attempts created duplicate risk during month-end. I then investigated Sahara's missing invoices because the payment run was approaching, followed by Al Noor's tax validation issue because of the audit impact. Ghaf was classified P3 because the date-format workaround was clear.

I would raise any P2 incident to P1 if evidence showed that invoices could not reach the ASP for one or more customers.

### What I would do differently

- Capture exact timestamps and correlation IDs at the start of each investigation.
- Query by import ID and invoice ID rather than relying only on seller-level totals.
- Confirm ASP acknowledgements before recommending retries.
- Record the exact SLA deadline and next customer update time in the ticket.
- Verify code-level root causes before describing a suspected cause as confirmed.

### What I learned / what I would need to learn

I practised reading application and ASP logs, checking container health, querying invoice statuses, identifying duplicate invoice numbers and distinguishing import success from downstream submission. I also learned the importance of idempotency, retry handling, tax validation and customer update ownership.

I would next study distributed tracing, API timeout handling, SQL joins for invoice-event reconciliation, ASP submission workflows and production incident management.

### Time log (real time)

| Date | From–to | What I worked on |
|---|---|---|
| 14 Nov 2026 | [Enter actual start–end] | Environment setup, starting the lab and loading data |
| 14 Nov 2026 | [Enter actual start–end] | Reviewing logs, database queries and evidence |
| 14 Nov 2026 | [Enter actual start–end] | Writing incident investigations and submission |

### AI usage disclosure

I used ChatGPT to help structure the investigation, interpret log and query outputs, draft customer acknowledgements and escalation notes, and organise the submission. I checked the drafts against the provided inbox messages, SLA matrix, database query outputs and saved evidence files. I treated unverified explanations as hypotheses and did not use AI-generated assumptions as confirmed system findings.
