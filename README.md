# Invoice Processing Bot (UiPath RPA)

An RPA bot built in **UiPath Studio** that automates accounts-payable invoice validation. It reads vendor invoice PDFs, extracts the key fields, checks each invoice against business rules, and writes an Excel report that separates clean invoices from ones that need a human to review them.

> **Portfolio prototype.** The vendors and invoices are fictional sample data created for this project.

---

## The business problem

Finance teams often check supplier invoices by hand before paying them. It's slow and repetitive, and mistakes are expensive:

- **Duplicate invoices** can mean paying the same bill twice
- **Invoices without a purchase order (PO)** are spending that nobody approved
- **Unknown vendors** are a common sign of invoice fraud
- **Large amounts** need a manager's sign-off

This bot handles the routine checks automatically, so people only spend time on the exceptions. This approach is known as *exception-based processing*.

---

## What the bot does

```
Input folder of invoice PDFs
        │
        ▼
1. READ       Read PDF Text pulls all the text out of each invoice
        │
        ▼
2. EXTRACT    Regex patterns pick out 5 fields:
              Invoice Number · Vendor · Invoice Date · PO Number · Total Amount
        │
        ▼
3. VALIDATE   Each invoice is checked against 4 business rules
        │
        ▼
4. DECIDE     Status = Approved, or Exception with a reason
        │
        ▼
5. REPORT     Results are saved to Invoice_Report.xlsx
```

### Business rules

| # | Rule | Exception reason |
|---|------|------------------|
| 1 | Invoice number was already processed | Duplicate invoice number |
| 2 | PO number is blank | Missing PO number |
| 3 | Vendor is not on the approved vendor list | Vendor not approved |
| 4 | Total amount is above ₹50,000 | Over approval limit - needs manager |

The bot records the **first** rule an invoice breaks.

---

## Results (sample run)

| Metric | Result |
|---|---|
| Invoices processed | 15 |
| Approved automatically | 11 |
| Flagged as exceptions | 4 |
| Straight-through processing rate | 73% (11 of 15) |
| Run time | About 1 second |
| Accuracy vs answer key | 15 of 15 correct |

| File | Status | Reason |
|---|---|---|
| invoice_01 to invoice_11 | Approved | |
| invoice_12 | Exception | Duplicate invoice number |
| invoice_13 | Exception | Missing PO number |
| invoice_14 | Exception | Over approval limit - needs manager |
| invoice_15 | Exception | Vendor not approved |

<!-- Add a screenshot of your Excel report here, for example:
![Invoice report](screenshots/invoice_report.png)
-->

---

## Repository contents

| Folder / file | What it is |
|---|---|
| `InvoiceProcessingBot/` | The UiPath project (open `project.json` in UiPath Studio) |
| `sample-data/Input_Invoices/` | 15 sample invoice PDFs, including 4 with planted problems |
| `sample-data/Answer_Key.xlsx` | The expected result for every invoice, used to test the bot |
| `output/Invoice_Report.xlsx` | The report the bot produced |

---

## How to run it

1. Install **UiPath Studio** (the free Community edition works).
2. Download this repository and open `InvoiceProcessingBot/project.json` in UiPath Studio.
3. In `Main.xaml`, update the folder path in the first **Assign** (`files`) and in **Write Range Workbook** so they point to where you saved the files.
4. Click **Run**. Open `Invoice_Report.xlsx` to see the results.

**Packages used:** `UiPath.PDF.Activities`, `UiPath.Excel.Activities`, `UiPath.System.Activities`

---

## Key UiPath concepts used

- **For Each** loop to process every PDF in the folder
- **Read PDF Text** to extract text from digital PDFs
- **Regex** (regular expressions) to pull out individual fields
- **Nested If expressions** to apply the business rules in order
- **Variables and scope**, including a running list of invoice numbers already seen, to catch duplicates
- **DataTable** (Build Data Table → Add Data Row → Write Range Workbook) to build the Excel report
- **Log Message** for an audit trail of every decision

---

## Limitations and next steps

What I'd add to take this toward production:

- **Configuration file:** read the vendor list and approval limit from `Config.xlsx` instead of hard-coding them
- **Error handling:** wrap each invoice in **Try Catch** so one corrupt PDF is logged and skipped instead of stopping the run
- **3-way match:** compare each invoice with its purchase order and goods receipt
- **Many invoice layouts / scanned invoices:** use OCR or UiPath Document Understanding instead of fixed Regex patterns
- **ERP integration:** post approved invoices into an ERP (SAP, Oracle, ERPNext) through its API
- **Scheduling:** publish to UiPath Orchestrator and run daily, with **REFramework** for retries and logging
