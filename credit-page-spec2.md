# Credit Page — Revision Spec (v2)

**Purpose:** This document tells an agent exactly what to change in the current Credit page prototype so it matches the confirmed product model and fixes existing UX, IA, and data-integrity problems. Each issue is written as: what's wrong → why → the exact action to take.

**Read this first:** the current prototype was AI-generated and contains numbers that look plausible but are arithmetically impossible, and labels attached to the wrong content. Do not preserve existing values or labels on the assumption they were intentional. Where this spec gives a formula, compute from the formula.

---

## 0. Confirmed product model (do not deviate)

- Loop issues a **revolving Line of Credit (LOC)**. It has a monthly cycle, a statement balance, and standard revolving mechanics: pay in full / pay minimum / pay custom, with any unpaid remainder carrying to the next cycle. It has **no fixed end date and no predetermined multi-month payment schedule.**
- At any point the business may **convert some or all of the outstanding LOC balance into a Term Loan** instead of paying it down. A term loan is a separate object with fixed APR, fixed term (3/6/9/12 months), and a fixed amortizing monthly payment. It does **not** revolve.
- **Multiple term loans can be outstanding simultaneously.** Each needs its own detail page.
- **Converted principal reduces available LOC credit** until the loan is repaid.
  `Available credit = Credit limit − (revolving balance + Σ outstanding term loan principal)`
- **Bank connection is a hard gate on the entire Credit page.** No application of any kind can be started until a bank account is connected.
- Each term loan carries **its own APR**, locked at conversion. The LOC carries a separate APR. There is no single global APR for the page.
- **The LOC APR is a fixed product rate for this build** — use `16.99%` as a constant. (Whether this varies per customer in production is an open question, Section 10.)
- A term loan has **three possible states: Active, Overdue, Paid off.** Overdue means a scheduled payment was missed and requires error styling wherever it surfaces.
- **Conversion has no minimum amount.** A business may convert any portion of the outstanding balance, up to the full balance.

### Product structure: two clearly separated halves

The page must read as two distinct product areas, not one blended surface:

- **A — Revolving credit (the card / line of credit):** cycle-based, statement-driven, no end date. Lives on the Overview tab.
- **B — Term loans:** fixed-rate, fixed-term, amortizing, each with an end date. Lives on the Loans tab.

Do not mix vocabulary or mechanics across the two. Statement/minimum/cycle/utilization language belongs only to A. Term/APR-per-loan/amortization/payoff language belongs only to B. Where the two meet (conversion, available credit), state the relationship explicitly rather than blurring it.

---

## 1. Page-level information architecture

### ISSUE 1.1 — Nested tabs and mislabeled content
Current: the Overview page contains a card titled "Activity" whose sub-tabs read "Active Loans / Inactive Loans", while the table beneath displays card spend and repayments. Three different concepts are collapsed into one control, and the selected tab ("Inactive Loans") is displaying transactions, which is incorrect content for that label.

The underlying category error: **an activity is a past event; a loan is an object with ongoing state.** They cannot share a parent tab.

**ACTION:** Replace all current tab structures with exactly **four page-level tabs, no sub-tabs anywhere**:

| Tab | Contents |
|---|---|
| **Overview** | Line of credit summary card + Next statement card. No loan tables, no schedules. |
| **Loans** | Single table of all term loans, with status filter chips. Rows route to loan detail pages. |
| **Activity** | Credit-balance event ledger only. |
| **Applications** | Status tracking for the initial credit application and for credit-limit-increase requests. |

Remove the "Balances" tab entirely — its intended contents are now split between Overview (revolving balance) and Loans (term loans).

### ISSUE 1.2 — Active/Inactive as separate tabs
Splitting loan status into separate tabs creates an empty tab for every new customer, forces users to check two places to find a known loan, and does not scale now that a third status (Overdue) exists. Status is an attribute, not a location.

**ACTION:** Build **one** Loans table. Above it, place filter chips: **All · Active · Overdue · Paid off**, defaulting to **Active**. Status also appears as a badge in a Status column so the state is legible without filtering.

The **Overdue** chip is only rendered when at least one overdue loan exists — do not show a permanently empty filter. When overdue loans exist, the chip carries a count and error styling (e.g. `Overdue 1`).

### ISSUE 1.3 — Activity tab scope and duplication with sidebar Transactions
The sidebar already has a global Transactions page for day-to-day spend. The credit Activity tab must not become a filtered duplicate of it.

**ACTION:** Restrict Activity to events that change the credit balance:
- Draw / advance
- Repayment (LOC or term loan)
- Loan conversion
- Interest charge
- Fee

**Card spend must be rolled up, not itemized.** Show one aggregate row per statement cycle — e.g. `Card spend — August cycle · −$62,500.00` — with a link through to the sidebar Transactions page for the itemized view. Do not list individual merchant charges (Google Ads, Meta Platforms, etc.) in this tab.

Columns: Date · Description · Type · Amount. Keep the Type column — it is what disambiguates row kinds, which is why no sub-tab is needed.

### ISSUE 1.4 — Overview activity preview
Retaining a short activity preview on Overview with a "View all activity" link is a correct pattern (preview-then-full-list), not redundancy.

**ACTION:** Keep an Overview activity preview of the 5 most recent credit events, linking to the Activity tab. Acceptable to cut for time, but it is not a UX error.

---

## 2. Naming conventions

Apply these renames globally.

| Current label | Change to | Reason |
|---|---|---|
| "Inactive Loans" | **"Paid off"** | "Inactive" implies dormant or suspended, not completed |
| "Credit Overview" (card title) | **"Line of credit"** | "Overview" inside the Overview tab is redundant; the card describes the LOC specifically |
| "Repayment progress" | **"Credit utilization"** | Correct term for a revolving line; "repayment progress" implies a fixed payoff endpoint that a revolving line does not have |
| "Upcoming payments" (card title) | **"Next statement"** | Reflects the single upcoming statement, not a multi-month schedule |
| "Pay balance" (button, when autopay is on) | **"Pay now"** | Clarifies it is an optional early/manual payment, not a replacement for autopay |

**Do not** name any tab or card "Transactions" — it collides with the sidebar navigation item and would mean two different scopes under one name.

---

## 3. Data integrity — must fix before demo

### ISSUE 3.1 — Ending Balance column is arithmetically impossible
Current values: balance $86,600.00 with payments of $14,433.33 each, and ending balances shown as $42,000 → $79,000 → $76,000. The balance **increases** after a payment, which cannot happen on a repayment schedule.

**ACTION:** This table is being removed from Overview per ISSUE 4.1. Where a payment schedule *is* rendered (loan detail pages, Section 6), compute every row:
```
ending_balance[n] = ending_balance[n-1] − principal_portion[n]
```
Ending balance must decrease monotonically to exactly 0.00 on the final row. Split each payment into principal and interest portions using standard amortization; do not display a schedule where total payments equal principal exactly (that implies 0% interest and contradicts a displayed APR).

### ISSUE 3.2 — Statement due date precedes cycle close
Current: cycle Aug 1 – Aug 31 with payment due Aug 24 — the amount owed for a period that has not yet ended.

**ACTION:** Due date must fall **after** the cycle close date, using a grace period of ~21 days. For an Aug 1–31 cycle, due date is approximately **Sep 21**. Apply this rule wherever cycle and due dates are generated.

### ISSUE 3.3 — Available credit must account for outstanding loans
**ACTION:** Compute available credit using the formula in Section 0 everywhere it is displayed. When a conversion occurs, revolving balance decreases and outstanding loan principal increases by the same amount, so available credit stays flat at conversion and only recovers as the loan is repaid.

---

## 4. Overview tab

### ISSUE 4.1 — Revolving and amortizing mechanics are mixed on one surface
Current: "Repayment progress" and a fixed three-month payment schedule (amortizing concepts) sit alongside a monthly cycle and "Pay balance" (revolving concepts). A revolving line does not have a predetermined payment schedule; an amortizing loan does not have a statement cycle with carryover.

**ACTION:** Overview describes **only** the revolving LOC. Specifically:
- **Remove** the multi-row "Upcoming payments" schedule table entirely.
- **Replace** it with a single **Next statement** card (contents in ISSUE 4.3).
- **Rename** "Repayment progress" to "Credit utilization".
- **Move** all fixed payment schedules to individual loan detail pages, where they are correct.
- **Move** "Download loan agreement" off this card — it is singular while the model supports multiple loans, so it belongs on each loan's detail page.

### ISSUE 4.2 — Line of credit card contents
**ACTION:** The Line of credit card contains:
- Outstanding revolving balance (primary figure)
- Credit utilization bar
- `$X available of $Y` supporting line
- Metadata line, low prominence: `Limit $250,000 · APR 16.99%` (LOC APR, always visible — see Section 5)
- Cycle date range in the card header

Remove "Next payment" and "Last payment" from this card; next payment now lives on the Next statement card, and last payment is visible in Activity.

### ISSUE 4.3 — Next statement card contents and hierarchy
Minimum payment due is currently only visible inside the Pay modal. It determines whether the account goes delinquent and must be visible without opening anything — but it must not be visually dominant, since emphasizing it nudges users toward carrying interest.

**ACTION:** Build the card with this exact hierarchy:

```
Next statement                          Due Sep 21, 2026
$86,600.00                              ← largest type, primary figure
Statement balance · pay in full to avoid interest

Minimum due   $4,330.00                 ← smaller, secondary
Interest applies to any remaining balance

Autopay on · pays statement in full on Sep 21
[Pay now]                               ← secondary button styling
```

Requirements:
- Statement balance is the primary figure; minimum due is secondary in size and weight.
- The consequence of paying the minimum is stated factually, not moralized.
- **Autopay state must be explicit** — on/off, the date it runs, and **what amount it pays** (full statement vs. minimum). The current "Edit autopay" button alone does not communicate any of this. Keep an "Edit autopay" affordance alongside the state line.
- When autopay is on, the manual pay button is **secondary** styling, not primary, so the two do not compete as instructions.

### ISSUE 4.4 — Pay modal needs a pay-vs-convert fork
**ACTION:** Add a two-option fork at the top of the modal, resolved before any other options render:
- **"Pay it off"** → existing options: statement balance / minimum payment / custom amount
- **"Convert to a loan"** → term picker (3/6/9/12 months) with monthly payment, total to repay, total interest, APR, and the "No hidden fees / pre-pay anytime, no penalty" messaging from the reference frame

Modal title changes with the selected path ("Pay balance" / "Convert to a loan"). Never render both option sets simultaneously.

Add a conversion amount input to the convert path — the model permits converting **some or all** of the balance, so a fixed full-balance-only conversion is incorrect. Default to the full outstanding balance, editable downward. **No minimum conversion amount** for this build; the only bound is the outstanding balance as the maximum. All figures in the modal (monthly payment, total to repay, total interest) recalculate live as the amount or term changes.

---

## 5. APR and rate disclosure

There is no single APR for this page. Three distinct placements are required.

**ACTION:**
1. **LOC APR** — low-prominence metadata line on the Line of credit card (`Limit $250,000 · APR 16.99%`). Always visible, never the hero.
2. **Per-loan APR** — a column in the Loans table, and prominently in each loan detail summary card. Values differ per loan (each locked at conversion).
3. **At conversion** — keep APR, monthly payment, total to repay, and total interest displayed together in the conversion modal. This is the highest-stakes disclosure moment and should remain prominent.

Note for the agent: APR disclosure carries regulatory weight in lending. Do not defer, collapse, or hide rate display behind an interaction anywhere it currently appears.

**Nice-to-have, not MVP:** an interest preview on the minimum-payment option ("paying the minimum would cost about $X in interest this cycle").

---

## 6. Loans tab and loan detail pages (new build)

### ISSUE 6.1 — Term loans have no home and no detail view
The current prototype has no loans list and no way to open an individual loan, despite multiple concurrent loans each requiring a detail page.

**ACTION — Loans tab:** one table, filter chips (All · Active · Paid off, default Active), columns:

`Loan ID · Principal · Term · APR · Next payment · Status · ›`

Each row navigates to that loan's detail page. Empty state when no loans exist: explain that a business can convert an outstanding LOC balance into a fixed-term loan, with a CTA into the conversion flow.

**ACTION — Loan detail page:**
- Header: Loan ID + status badge (Active / Overdue / Paid off)
- Summary card: term length, APR, current balance, origination and maturity dates, "Make a payment" button, "Download loan agreement" link (moved here from Overview)
- Tab group: **Summary · Payment Schedule · Early Payoff**
  - **Summary** — principal, remaining balance, next payment amount and date, total interest remaining
  - **Payment Schedule** — full month-by-month table: payment date, payment amount, principal portion, interest portion, remaining balance. The reference frame had a gray placeholder here; it needs real computed content per ISSUE 3.1.
  - **Early Payoff** — payoff amount if paid today, interest saved, explicit "no prepayment penalty" confirmation, and a CTA to pay off

**Make a payment on a term loan:** a single amount input, defaulting to the scheduled payment, editable up to the full early-payoff amount. Do **not** reuse the statement/minimum/custom framing — that is revolving-only.

### ISSUE 6.2 — Overdue state has no treatment anywhere
A term loan can go overdue (missed scheduled payment). Nothing in the current prototype accounts for this.

**ACTION:** Implement overdue at three levels, escalating in prominence:

**1. Loans table row**
- Status badge reads **Overdue** with error styling (red/destructive token, not the neutral Active token)
- Next payment cell shows the missed date and amount, also in error styling
- Row is sorted to the top of the table regardless of active filter

**2. Loan detail page**
- Header badge: **Overdue**
- An error banner above the summary card stating what is owed and by when: amount past due, original due date, days overdue, and any late fee applied
- Primary CTA in the banner is **Make a payment**, pre-filled with the past-due amount (not the regular scheduled amount)
- In the Payment Schedule table, missed rows are marked **Missed** with error styling; future rows remain Scheduled

**3. Overview tab**
- A persistent alert banner at the top of the Overview tab when any loan is overdue: `1 loan payment is past due` with a link to the affected loan. This is the only case where loan information is permitted on the Overview tab, because it is an account-health alert rather than loan detail.

Copy guidance: state facts and the required action. Do not use alarming or shaming language, and do not imply consequences (credit reporting, default) that have not been confirmed by Credit.

---

## 7. Applications tab

### ISSUE 7.1 — Application status has no surface
**ACTION:** The Applications tab covers **both** the initial credit application and subsequent **credit-limit-increase requests**. It shows a list of applications, each with:
- Type (New credit line / Limit increase)
- Date submitted
- Requested amount
- Status: Draft · In review · More info needed · Approved · Declined
- Action affordance appropriate to status — resume a draft, upload requested documents, view decision

Include a primary action to **request a credit limit increase** for businesses with an existing LOC.

### ISSUE 7.2 — Credit limit increase uses a shortened flow, not the full application
A business that already has an approved LOC has already passed pre-qualification, ownership verification, and document collection. Re-running the full 7-step application for a limit increase would be redundant and is the single most likely place for an existing customer to abandon.

**ACTION:** Build the limit increase as a short flow that **re-verifies connected data rather than re-asking questions**:

1. **Requested limit** — new limit amount, with the current limit shown for reference
2. **Confirm your connected data** — a review list of the business's existing connections, each with a freshness indicator and a reconnect affordance:
   - Bank accounts
   - Online storefronts / sales channels
   - Bookkeeping / accounting accounts
   Show connection status per source (Connected · Needs reconnection · Not connected). Stale or disconnected sources are the only thing the applicant must act on. Offer any not-yet-connected source as an optional way to strengthen the request, using the same "more sources connected → higher potential limit" framing already established on the zero-state accelerator card.
3. **Confirm and submit** — read-only summary of the requested limit and the data sources backing it

Do not re-ask the Step 3 questionnaire, owners, or documents unless a data source cannot be verified. If verification fails, the resulting state is **More info needed** in the Applications list, which is where any additional document request surfaces.

Note: the underlying 7-step **new** application flow (pre-qualification → connect data → questionnaire → owners → documents → guarantors → review) is specified in the source questionnaire document and is **out of scope for this revision.** Only the status-tracking surface, its entry points, and the shortened limit-increase flow above are in scope here.

---

## 8. Application entry and gating

### ISSUE 8.1 — Two contradictory zero-states
Current: one screen offers "Apply for credit" immediately with bank connection framed as an optional accelerator; another blocks applying until a bank account is connected.

**ACTION:** Implement a single ordered sequence:
1. **No bank connected** → "Connect a bank account to apply for credit" (blocking; single CTA; no path past it)
2. **Bank connected, no LOC** → "You don't have a line of credit yet" → "Apply for credit" primary CTA, plus the optional "Connect a sales channel to get approved faster" accelerator card
3. **LOC approved** → full Overview

Delete the variant that allows reaching pre-qualification without a connected bank account.

### ISSUE 8.2 — Compliance copy is likely inaccurate
Current copy: "As per compliance regulations, you must connect a bank account before applying for a credit line."

The source questionnaire document treats banking data as load-bearing for **underwriting** (credit risk), not as a regulatory requirement. Asserting a regulatory basis in user-facing copy may be inaccurate.

**ACTION:** Change to: "To apply for a credit line, connect a bank account so we can verify your business activity." Flag for compliance review before any external release.

---

## 9. Decisions confirmed — implement as stated

1. Converted principal **reduces** available LOC credit until repaid.
2. Term loan payments use a **simple amount input**, not statement/minimum/custom.
3. Transaction-level conversion is **out of scope** for MVP.
4. Bank connection **gates the entire Credit page**.
5. **Four page-level tabs**, no nested sub-tabs. Applications is its own tab.
6. Term loans **can go overdue** — third status with error styling per ISSUE 6.2.
7. **No minimum conversion amount.** Maximum is the outstanding balance.
8. **LOC APR is a fixed 16.99%** for this build.
9. **Autopay covers both** the LOC statement and term loan payments, configured once at account level.
10. **Mid-cycle conversion does not disturb autopay** — autopay pays the remaining revolving statement balance; the loan's schedule runs independently.
11. **Credit limit increase uses a shortened re-verification flow** per ISSUE 7.2, not the full 7-step application.
12. **Activity includes term loan payments** as individual rows, typed "Loan repayment" with the loan ID in the description.

**Flagged for future implementation (not MVP):** transaction-level installment conversion — letting a business split an individual purchase (e.g. one card spend line) into its own term loan, similar to Amex Plan It. Good direction, revisit post-MVP.

---

## 10. Open questions — build the default, flag in the handoff

Not blocking. Implement the default and note it as an assumption so it can be revisited.

1. **Does the LOC APR vary by customer, or is it a single product rate?** Currently hardcoded at 16.99%. If it is risk-priced per customer, it must become an account-level value and the prototype's single constant will need replacing. **Needs confirmation from Credit.**
2. **Is there a cap on concurrent outstanding term loans?** Default for this build: unlimited, bounded only by available credit. If Credit imposes a cap, the conversion entry point needs a blocked state explaining the limit.
3. **Is the revolving product called a "credit card" or a "line of credit" in customer-facing language?** This spec uses "line of credit" throughout, but the sidebar already has a separate **Cards** item, and the source underwriting document describes a card product (monthly spend, credit limit, card cycle). Using both names for the same object across Credit and Cards will confuse users. Pick one term and apply it globally. **This one is worth resolving early** — it touches every label on the page, so it is cheaper to decide now than to rename later.
4. **What are the consequences of an overdue loan?** Copy in the overdue banner currently states facts only (amount, date, days overdue, late fee if any) and deliberately avoids mentioning credit reporting or default, since neither has been confirmed. If Credit has defined an escalation path, the banner copy should reflect it.
5. **Late fees** — does an overdue term loan payment incur a fee, and how is it calculated? The overdue banner has a slot for it; currently displays only if a fee value exists.
