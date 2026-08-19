# Questionnaire spec vs. prototype — conformance check

**Date** 2026-08-19
**Spec** `Loop - Credit Application Questionnaire Flow (August 2026).pdf` (Cato Pastoll)
**Prototype** `~/Downloads/next-app-main/src/components/pages/_prototype-scratch/credit/application/` →
`index.html` (live at <https://rebeccasunloop.github.io/loop-credit-prototype/>)

The PDF re-downloaded on 2026-08-19 is textually identical to the copy already in this folder — the only
differences are bullet/whitespace artifacts of the Google Docs PDF export. Nothing in the spec changed;
everything below is prototype divergence, not spec drift.

> **Update, later on 2026-08-19:** the "Application strength" meter has since been removed from the UI, and
> Continue no longer blocks in the single-file demo (`enforceGating` / *Skip required fields*). Reasons and
> restore path are in `README.md` → *Strength meter removed, and the flow walks freely*. Findings 1.1–1.9
> below describe the gating logic as authored, which is still what Storybook renders and still what would
> ship — the demo bypass sits on top of it rather than replacing it.

Structurally the prototype is faithful: 7 steps with the right names and exits, Step 6 conditional so the
rail handles 6 or 7, seven questionnaire sub-sections in the spec's order, section-based progress (never
question-based), all 8 revenue bands, all 6 revenue models, 4 data sources with the exact completeness
weights (30/20 · n-a/25 · 20/10 · 25/15), the governing-spend formula verbatim, all 15 Step-5 documents
with the spec's own reason copy, the 6-card cap, DOC-02 correctly living on the DS-3 card, DOC-09 derived
after Step 3, and the PQ-10 consent text word for word. What follows is what doesn't match.

Already documented as deliberately out of scope in `README.md` → *Application spec states not built*
(save-and-resume across devices, 30-day expiry, analyst request for more info, second round on a long-text
answer, DS-2 connect failure/expiry states, Quebec French) — excluded from this list.

---

## 1. Blockers — the flow gives a wrong or dead-end outcome

### 1.1 Knockouts never reach the applicant in the clickable prototype
`credit-application-flow.tsx:135` gates the decline screen on the `showKnockout` prop, which
`standalone/demo-app.tsx:228` never passes. `KO-01/03/07/08` therefore exist **only as Storybook stories**.
In the real click-through, hitting a knockout sets `canContinue = false` (`:322`) and pushes **no blocker
string**, so the footer renders an empty `<div/>`: Continue greys out with no explanation and no decline
screen. Picking *Money services business* or the lowest revenue band is a silent dead end.
Spec §3.4 requires one decline screen with a variable body.
(README's *Worth trying* claims this path works — it works in Storybook, not in the single file.)

### 1.2 PQ-8 = No traps the applicant
Answering "No" to *Are you authorized to apply…* correctly reveals the referral capture and hides PQ-9/PQ-10
(`step-prequalification.tsx:329`), but the Continue blocker still requires `consentCreditCheck`
(`credit-application-flow.tsx:283`). The applicant is told to "Tick the soft credit check consent" for a
checkbox that is no longer on screen, and cannot proceed. Spec §3: *"If No: capture the correct contact's
name and email rather than declining"* → referral flow. The referral name/email are also not required.

### 1.3 Unanswered questions are read as favourable answers
Step 3 gates Continue only in section **3C** (`credit-application-flow.tsx:293`). Sections 3A, 3B, 3D, 3E,
3F, 3G have no gating at all, and the rules engine treats blank as benign rather than unknown:

| Left blank | Rules engine reads | Suppressed |
| --- | --- | --- |
| Q11 profitability | `undefined` ≠ `false` | full PG, DOC-12, FQ-03, FQ-04, FQ-09, and the whole Guarantors step |
| Q14 total debt | `0` | DOC-10, and the Q14÷Q9 arm of DOC-04 |
| Q7 concentration | `0` | FQ-08, DOC-19 |
| Q12 / operating expenses | `0` (guarded by `> 0`) | the cash-runway arm of DOC-04 |
| Q16 affiliated entities | `undefined` | FQ-10, DOC-08 |
| Q17 litigation | `undefined` | DOC-13 |

Answering nothing produces the lightest possible document set and no personal guarantee. Spec §2 gives
Step 3's exit as *"All required answered."* The narrow definition of mandatory in README →
*Application-flow conventions* is a deliberate call, but it lets a materially different underwriting
outcome fall out of non-answers.

### 1.4 Q18's specified behaviour is not implemented
The *Edit* button on Q18 (`step-questionnaire.tsx:519`) has no `onClick` — it is inert. Spec §5.7 requires
editing to return to the 3.2 derived-limit behaviour and, on a threshold crossing, re-derive the
requirements and update the document counter. Three §9 states depend on this and are unreachable:
*derived value overridden* at Q18, *requirements re-derived mid-application* at Q18, and *DS-2 and DS-4
escalated after a change at Q18*.

### 1.5 Ownership rules are displayed but not enforced
`step-owners.tsx:137` shows "Combined ownership can't exceed 100%" but `overAllocated` never enters the
blocker list, so 140% continues. Removing every owner also continues — the blockers only iterate over
owners that exist. Spec §6: *"Minimum one entry. Combined ownership must not exceed 100%."*

### 1.6 Prefill indicator lies when there is no prefill
Q1, Q2, Q3 and Q16 pass `prefilled` unconditionally (`step-questionnaire.tsx:88, 94, 100, 388`), so an
applicant Loop holds no data on sees "Autofilled from your Loop account" above four empty fields. Step 1
does this correctly via `hasPrefill`. Spec §9 requires a blank variant for *every* pre-filled field.

### 1.7 Q9 doesn't check the band it claims to check
The error copy reads *"…and doesn't match the band you chose earlier"* but the condition is only
`totalRevenue < 100_000` (`step-questionnaire.tsx:239`). Revenue of $200,000 against a chosen band of
$5–10 million passes silently. Spec §5.3: *"consistent with the band chosen at PQ-6 — prompt a
confirmation if not."*

### 1.8 Nested corporate ownership can never fire
`nestedCorporateOwnership` is read by `rules.ts:111` but set by nothing outside tests and stories — there
is no corporate-owner concept in Step 4 and no look-through prompt. So the *Q16a nested ownership* arm of
DOC-18 is dead; only the PQ-5 `< 24 months` arm can trigger it. Spec §6 requires corporate owners with a
look-through prompt, and §5.5 makes nested ownership a DOC-18/EDD-05 trigger.

### 1.9 Retracted follow-ups keep their answers
`Reveal` deliberately keeps content mounted, and only three places clear on retraction (`operatesAbroad` →
`abroadCountries`, `hasAffiliatedEntities` → `affiliatedEntities`, `totalDebt` → `debtPastDue`). Answer
Q11 = No, fill FQ-03/04/09, switch to Yes: the three answers survive in state and go to Review and submit.
Spec §9: *"Conditional reveal and retraction (answer changed; the follow-up's answer is discarded)"* — for
all FQ-nn.

---

## 2. Specified behaviour missing or weaker than specified

| # | Spec | Prototype |
| --- | --- | --- |
| 2.1 | §4.1 *"Connection happens in a modal and must return the applicant to this screen with state preserved."* | `onConnect` flips straight to `connected`. `select-institution-modal.tsx` exists in the parent folder but isn't wired into Step 2; the `connecting` state is defined and never set. |
| 2.2 | §3 PQ-7 *"Min and max enforced by the control — an ineligible amount cannot be entered."* | Only the max clamps (`step-prequalification.tsx:218`). $500/month can be typed; the minimum is enforced later as a Continue blocker. Same for the PQ-7a override: max clamps at $1M, the $25,000 minimum in the hint is not enforced. |
| 2.3 | §3.2 *"Bounded by the product minimum and maximum."* | The two bounds don't reconcile: spend range 5k–700k × 1.5 gives limits of **7.5k–1.05M**, against a stated limit range of 25k–1M. A $5,000/month answer derives a $7,500 limit, below the product minimum the same screen advertises. |
| 2.4 | §3.4 KO-08 *"Soft decline, with the date they become eligible."* | Copy is "Loop credit needs at least 12 months of trading history" — no date. (KO-01's *Join the waitlist* also just hrefs to `/credit`, with no capture.) |
| 2.5 | §5 design note *"Every financial field takes a currency selector, defaulted from Q8."* | `CurrencyField` renders the currency as a static prefix addon. The selector exists once, at Q8. |
| 2.6 | §5.7 Q20 *"Hidden entirely for customers with 6+ months of Loop card history."* | Always shown; there is no card-history input to the rules engine. |
| 2.7 | §3 PQ-1 address = street, city, province/state, country, **postal code**, autocomplete on edit, not a PO box | Three fields (line 1, province, country). No city, no postal code, no autocomplete; the PO-box rule is helper text only. |
| 2.8 | §3 PQ-3 *"Searchable single-select, NAICS-mapped… Type-ahead must resolve to a listed value."* | Plain `Select` over 8 hardcoded industries, no search, no NAICS mapping. |
| 2.9 | §3 PQ-3 *"Retail / Wholesale / E-commerce → FQ-01"* | FQ-01 fires on `industryRisk === 'inventory'`, which includes **Manufacturing**; there is no Retail option in the list at all. |
| 2.10 | §5.5 Q16a *"Add / remove rows"* | Add only — no remove control on affiliated-entity rows. |
| 2.11 | *"Other (+ text)"* at PQ-2, PQ-4a, Q19 | Write-in wired only at FQ-01 (`MultiSelectChips` needs `onOtherChange`, which only FQ-01 passes). PQ-2 has an *Other* radio with nowhere to type. |
| 2.12 | §3 PQ-5 *"Month + Year… Cannot be in the future"* | Two free numeric text boxes, no month picker, no future-date validation. `monthsInBusiness` hardcodes today as 2026-08. |
| 2.13 | §4.1 *"Where documents are uploaded, use AI to detect whether the document uploaded is the correct type."* | Upload is a state toggle; no detection, correct or otherwise. |
| 2.14 | §5.1 Q1 *"Format-validated by jurisdiction"* | Free text. |
| 2.15 | §6 *"No individual owner resident in an eligible jurisdiction → refer to Credit."* | Country of residence is free text with no rule. (Note the spec sentence itself reads inverted — presumably *no owner resident in an eligible jurisdiction*.) |
| 2.16 | §7 review-and-submit summary | No guarantor-consent card on the Review step, so a conditional step with a blocking exit isn't represented in the read-only summary. |

---

## 3. Prototype asks things the spec doesn't

Both are defensible, but both are applicant-facing questions with no ID, no specced copy, and no place in
the 11–12 / 18–32 question counts:

1. **"What will the higher limit be used for?"** — required free text revealed whenever PQ-7a is overridden
   (`step-prequalification.tsx:292`), and it gates Continue. Spec §3.2 treats the override as a bounded
   number, nothing more. If this stays it needs an ID and a place in §3.
2. **"Roughly what are your monthly operating expenses?"** — nested under Q12
   (`step-questionnaire.tsx:325`). It's the only way to evaluate DOC-04's *"< 1 month of operating
   expenses"* trigger, so the spec has a hole here, but §5.3's Verification note says cash position is
   checked against DS-1 bank data — which suggests this should be derived, not asked.

### And one that contradicts a design note
Spec §3 design notes: *"Nothing about documentation requirements is shown until the applicant has
qualified."* The PQ-7a override panel says "…so we'll ask for accountant-prepared financial statements"
inline, before qualification (`step-prequalification.tsx:290`). §3.2 asks for requirements to re-derive on
a threshold crossing, but names *the panel in 3.4* as where that surfaces. Pick one.

---

## 4. Where the spec, not the prototype, needs a decision

- **KO-08's minimum is invented.** The spec says only "time in business below minimum". The prototype
  chose 12 months, which is consistent with §3's `<24 months → full PG` band implying 12–24-month
  businesses are eligible — but the number is a product decision that isn't written down anywhere.
- **Q11 is a self-assessment carrying the heaviest consequence in the flow** (full PG, DOC-12, three
  follow-ups, the entire Guarantors step). §5.3 says profitability is verified against DS-1/DS-2 and the
  verified figure governs — so a PG requirement can flip after submission, from data the applicant already
  connected. The spec has no state for "requirements changed after you submitted", and §9 covers only the
  analyst asking for more information. Worth deciding before build.
- **§9's "Requirements re-derived mid-application" has no re-derivation *downward* story.** If an applicant
  lowers their limit at Q18 after uploading documents in Step 5, the spec doesn't say whether already-
  uploaded documents are dropped, kept, or shown as no-longer-needed.
- **DOC-04 double-asks above the DS-2 threshold.** DS-2 collects accountant-prepared financial statements;
  DOC-04 asks for "monthly P&L and balance sheet". §8's suppression rule names financial statements
  explicitly, so these two probably overlap for anyone above $50,000 governing spend.
