# Credit Page

A **Credit** tab in the Loop dashboard sidebar: one page listing every credit account — lines of credit and
term loans — each opening its own detail page, plus application tracking.

Built 2026-08-10 from Mercury's Financing page as a reference, filtered through Loop's ICP. Restructured
2026-08-13 to `credit-page-spec2.md` (product model, data integrity), then again 2026-08-17 to the
accounts-list structure below, which on the same day gained a summary band and a full naming pass.
**Cato's priority is the application flow — it ships next — so the accounts view is deliberately functional
rather than polished.**

## Live demo

**<https://rebeccasunloop.github.io/loop-credit-prototype/>** — clickable, no install. Deep-link with the
hash: [`#/credit`](https://rebeccasunloop.github.io/loop-credit-prototype/#/credit) ·
[`#/credit/apply`](https://rebeccasunloop.github.io/loop-credit-prototype/#/credit/apply) ·
[`#/credit/line`](https://rebeccasunloop.github.io/loop-credit-prototype/#/credit/line) ·
[`#/credit/applications`](https://rebeccasunloop.github.io/loop-credit-prototype/#/credit/applications).

GitHub Pages serves `main` at root, so **pushing `index.html` to `main` is the deploy** — there is no build
step and no action to wait on, only Pages' own ~1 minute cache. `.nojekyll` is committed so the bundle is
served verbatim rather than run through Jekyll.

## Files

| Path | What it is |
| --- | --- |
| **`index.html`** | **The prototype.** One self-contained file, ~2.1 MB. Open it and click, or use the live link above. No server, no network, no build. Real React, real Loop components, real state. Named `index.html` so GitHub Pages serves it at the root URL; the build script still emits it as `dist/loop-credit-prototype.html`, so copy it over that name. |
| `credit-page-spec2.md` | **The current spec.** Revision spec v2 — confirmed product model, the IA restructure, data-integrity fixes, and the surfaces built in this pass. Where it disagrees with anything below, it wins. |
| `credit-page-spec.html` | The original build spec — segment-fit verdict, Mercury to Loop feature analysis, screen-by-screen, states, copy, design-system decisions. Superseded on IA and data by v2; still the reference for the segment reasoning. |
| `Loop - Credit Application Questionnaire Flow (August 2026).pdf` | Cato Pastoll's design specification for the application. The questionnaire was built from it directly rather than re-specced. |

## Product model

Loop issues a **revolving line of credit** — a cycle, a statement balance, and normal revolving mechanics
(pay in full / minimum / custom, unpaid amount carries). At any point the business can **convert some or
all of the outstanding balance into a term loan**: fixed APR, fixed term (3/6/9/12 months), fixed
amortising payment, no carryover. Multiple loans can run at once, each with its own detail page.

**Available credit = limit − revolving balance − outstanding term-loan principal.** Converting doesn't free
up the line; the principal stays against the limit until the loan is repaid. Computed from that formula
everywhere it appears, and said in plain words on the Line of credit card and its detail page. Conversion
takes *any portion* of the drawn balance — $80k drawn on a $100k line, convert $40k, and the line still shows
$80k of the limit in use until the loan pays down.

**The page is two separated halves, and the vocabulary doesn't cross over.** Statement, minimum, cycle and
utilization belong to the revolving line and its detail page. Term, per-loan APR, amortisation and payoff
belong to term loans and theirs. The LOC carries its own APR (**16.99%**, a fixed product rate for this build);
each loan carries its own, locked at conversion. There is no single APR for the page.

**Bank connection is a hard gate on the whole Credit page.** No bank, no application — `#/credit/apply`
redirects to the connect screen rather than rendering the flow. This supersedes the questionnaire doc's
Step 2 placement.

## Using it

Open the [live demo](https://rebeccasunloop.github.io/loop-credit-prototype/), or `index.html` in any
browser. A **Pick a user** panel sits bottom-left.

| User | What you get |
| --- | --- |
| **Has a credit line** | Everyday dashboard — balance, repayment schedule, activity, pay-down |
| **Several term loans** | Four term loans on one limit — 70.6% used, so the utilization bar has something to say |
| **Payment returned** | Autopay bounced, balance past due, fee charged — the combined late + failed case |
| **Ready to apply** | Bank connected, no application yet — the apply pitch |
| **No bank account** | Applying is gated. *Connect a bank account* really connects one and moves you on. |

Everything is live: the sidebar, both loan tables, the line-of-credit and term-loan detail pages with their
per-account transactions, the pay/convert modal, the limit-increase flow, and all seven application steps
through to submission. Deep-link with the hash — `#/credit`, `#/credit/line`, `#/credit/loans/1010`,
`#/credit/applications`, `#/credit/apply`. Leaving the application and coming back keeps your progress;
**Reset everything** clears it.

## One credit page: a summary band and two tables

**There is no Overview.** The list of what you owe *is* the landing page — that's the 2026-08-17 structure
call, and it replaces spec v2's separate Overview tab. `#/credit` stacks, in this order:

1. Page title **Credit** + the **[Apply ▾]** menu (top right)
2. The **overdue banner** — only when a term loan is actually past due
3. The **summary band**
4. **Term loans**
5. **Lines of credit**

Two tables, because a line of credit and a term loan are different objects with different mechanics, and
blending them is what made the earlier page confusing:

| Table | Rows | Columns |
| --- | --- | --- |
| **Term loans** | Fixed-rate, fixed-term, amortising | Loan · Principal · Outstanding · Term · APR · Next payment · Status |
| **Lines of credit** | Draw, repay, draw again, no end date | Account · Limit · Drawn · Available · APR · Statement due · Status |

**The Term loans status filter is a dropdown, not chips** (2026-08-18) — a `secondary` Button with a trailing
chevron, labelled with the current selection (*All loans · Active · Overdue · Paid off*). The Overdue option
is still conditional on a loan actually being overdue and carries its count via `Dropdown.Item`'s `addon`.

⚠️ `Dropdown.Menu` hardcodes `selectionMode="single"`, which suppresses React Aria's close-on-action — the
filter applied but the menu stayed open. Passing `selectionMode="none"` restores dismissal. Any future
single-select dropdown in this prototype needs the same override.

⚠️ Trade-off accepted: the overdue count used to be visible at a glance as a chip and is now one click away.
The overdue banner above the tables still carries the alert, so the signal is not lost — but the table header
no longer shows it unprompted.

Each table title carries a **definition tooltip** (`definition-tooltip.tsx`, an `InfoCircle` in the header's
`badge` slot). It replaced the inline *"Draw, repay and draw again, up to your limit"* strapline on Lines of
credit — the definition is available on demand rather than occupying the header, and Term loans now gets the
same treatment where it previously had none.

**Term loans sit first — ordered by urgency, not product hierarchy.** They have deadlines and can go overdue;
a line of credit rarely needs attention. There is deliberately **no category heading above the two tables**:
the table titles are the labels, and any umbrella word ("Loans", "Facilities", "Products") would either be
wrong or would file a line of credit under "loans", which is not how customers think of it.

Every row opens its own detail page, and **transactions are scoped per account** — there is no global ledger.

### The summary band (added 2026-08-17)

Removing the Overview page left the landing view with no account-level figures. The band is a thin strip of
**exactly three** figures above the tables — no card chrome, no borders, one hairline rule underneath, so it
reads lighter than the tables it introduces. It is **not** a card grid. Three figures remains the cap; charts
and any fourth metric belong on the facility detail pages.

| Figure | Derivation |
| --- | --- |
| **Available credit** (primary, largest type) | `limit − (line balance + Σ outstanding term-loan principal)`, with `of $X limit` beneath it |
| **Total owed** | `line balance + Σ outstanding term-loan principal` |
| **Next payment due** | soonest obligation still ahead across *all* facilities — statement minimum or a loan's scheduled payment — with its date |

Below the three figures sits a **utilization bar** (`ValueProgressBar`, the same component as the line-of-credit
card) — filled = total owed, remainder = available, each half naming its own breakdown on hover.
⚠️ This reverses the band's original "no utilization bars, charts or additional metrics here" rule, on
Rebecca's direction 2026-08-18. Note it renders the **same computation** as the bar on the line-of-credit
detail page, so the two must not drift.

All three come from `totalOwedFor` / `availableCreditFor` / `nextPaymentDueFor` in `mock-data.ts`. Nothing is
typed. **Available credit + total owed = the credit limit exactly**, verified across all 11 fixtures that
render a band.

⚠️ **`availableCredit` is no longer a field on `CreditMock`** (removed 2026-08-18). It used to be a stored
value that a hand-maintained loop at the bottom of `mock-data.ts` recomputed for a *list* of fixtures — so any
new fixture silently inherited a stale zero and rendered "$0.00 available / Limit reached". That is exactly
what the three payment fixtures did on first build. Every consumer now calls `availableCreditFor(data)`, so
there is one source of truth and a new fixture cannot get it wrong.

⚠️ `nextPaymentDueFor` deliberately **skips anything already past due** and takes each unpaid loan's first
scheduled row on or after `ACTIVITY_TODAY`. Past-due amounts are the banner's job; showing one here would put
the same debt on screen twice under a label that says "next". This is why the `loanOverdue` fixture reads
*$3,122.79 · Sep 12* (Loan #1021's next live payment) rather than its missed Jul 12 one.

**Every money value carries its ISO code** (`$122,421.30 CAD`) — set 2026-08-18 because Loop is
multi-currency and a bare `$` is ambiguous across CAD/USD. It is rendered by `Figure` (one change, global
effect) with the code in a smaller, muted span; the five local `Intl.NumberFormat` helpers in the modals and
application steps append it too, so nothing is left bare. The **available-credit figure also carries a
circular CAD flag** (`CurrencyFlag` in a `rounded-full` ring). ⚠️ The cost is repetition: `CAD` now appears
about twenty times down a table column. If that reads as noise, the usual fix is the code in the column
header instead of every cell.

⚠️ **The Outstanding column on Term loans is new, and it exists so the band reconciles.** The table used to
show only `Principal` — the *original* amount — so a reader summing the visible rows got $146,600 against a
band saying $127,578.70. Principal stays for context; Outstanding is what rolls up.

### Payment states on the line of credit (added 2026-08-18)

Term loans could go overdue; the line of credit could not — it had no late state and no failure state at all,
and the only terminal state in the pay modal was *Payment scheduled*. Three states now exist, all data-driven
off `paymentAttempt` / `statementOverdue` on `CreditMock`:

| State | Where it shows | Fixture |
| --- | --- | --- |
| **Processing** | `Payment processing` badge, the pay button becomes a disabled *Payment sent*, and an inline note gives the amount, source, and clearing date | `paymentProcessing` |
| **Returned** | Error banner naming the source, the return date, the reason and the fee; autopay shown as paused | `paymentReturned` |
| **Past due** | Card retitles to *Payment past due*, `Past due` badge, `Was due …`, minimum row becomes *Amount past due* in error colour with a day count | `statementOverdue` |

**Late and unsuccessful are two different flows.** A returned payment may or may not make you late; you can be
late with no failed payment at all. They are separate fields and render independently — but when both are true
(autopay bounces two days before the due date, which is the realistic bad case) the banner tells **one** story
rather than stacking two: *"Your payment was returned and the balance is now past due."* That combined case is
the `paymentReturned` fixture and the *Payment returned* persona.

⚠️ **Processing deliberately does not reduce the balance.** The note says so out loud — *"Your balance and
available credit update once it clears."* A payment that looks applied and then bounces three days later is
the worst version of this flow.

⚠️ **These fixtures use the July cycle** (closed Jul 31, due Aug 21) because `ACTIVITY_TODAY` is fixed at
2026-08-31 and the August statement isn't due until Sep 21 — nothing can be past due on it. Ten days overdue
is derived from those two dates, not typed.

Copy follows the compliance rules from the 2026-08-17 review: it states the fact, the fee and the required
action, and says nothing about credit reporting or default. The old hardcoded `payment-failed` page variant —
which had no story and a fabricated `CA$34,000.00 on Jul 21` — was deleted and replaced by this.

⚠️ **Open product questions this exposed, all currently assumed:** the returned-payment fee is $25 and is
Loop's (a bank NSF fee may stack), autopay pauses after one failure and does not retry, and there is no grace
period before a late fee. None of these are confirmed.

### Tabs appear only when there is something behind them (2026-08-18)

`visibleTabsFor()` in `credit-layout.tsx` is the single rule:

| State | Tabs | What `/credit` shows |
| --- | --- | --- |
| No credit, **no applications** | **none** | The two product cards |
| No credit, **application in flight** | **Applications only** | The Applications page — `CreditPage` delegates straight to `CreditApplicationsTab` |
| Active credit | Summary · Applications | The summary view |

A first-time visitor sees no tab bar at all, because a single unusable tab is worse than none. Once they
apply, Applications is the only tab — there is no summary to show yet — and `/credit` renders that page
rather than leaving a selected tab pointing somewhere else.

⚠️ **The phase panels on `/credit` are gone as a consequence** — the *Under review · Track your application*,
*Waiting on guarantor* and *Not approved* cards. Their content is on the Applications table, which was always
the better home for it. The one thing worth keeping, the **declined adverse-action reviewer callout**, was
moved to the top of the Applications page rather than lost.

**Both pages carry the same two actions.** `credit-actions.tsx` is shared, rendered by the summary band and
by the Applications page, and gated on active credit — a business with no limit is not offered a limit
increase. This replaced the Applications table's own header button, so there is now one definition instead
of two.

**The actions sit on the tab row**, right-aligned and vertically centred against Summary · Applications, via
the `actions` slot on `CreditLayout`. The page title row is just "Credit". They briefly lived inside the
summary band, level with the *Available credit* figure — that placement is in git history if it is ever
wanted back; it put the actions beside the numbers but only worked on the Summary page, since Applications
has no band. The tab row is the one place both pages share, so the buttons land identically on each.

⚠️ `CreditActions` carries `shrink-0`, and the `Tabs` wrapper `min-w-0`. Without both, React Aria's Tabs
takes the full row width and the two buttons wrap onto separate lines.

### Applying is no longer gated on a bank account (2026-08-18)

The apply surface used to be hidden behind a connect-a-bank panel: no bank, no marketing, no CTA. That is
reversed. **Everyone sees both products and both CTAs**; the bank connection happens inside the application,
where DS-1 (Business bank accounts) is already a required source at Step 2.

- `credit-product-cards.tsx` — two cards, each with an icon, a tagline, four facts
  (**Use it for · Amount · Rate · Repayment**) and an **Apply now** CTA. Referenced against TD's personal
  loan / personal line of credit cards for structure only — the copy is Loop's and business-framed
  (working capital, suppliers, payroll) rather than consumer ("life's sudden expenses").
- `ConnectBankPanel`, `INSTANT_CONNECTION_BENEFITS` and the page's `hasBankAccount` prop are **deleted**.
  So is the compliance-policy bullet that §8.2 disputed — it went with the panel.
- `prefilledAnswers` now starts DS-1 **idle**, so the connect step is real rather than pre-satisfied.
- Header actions are two buttons instead of the Apply ▾ dropdown: **Request a limit increase**
  (secondary, `TrendUp02`) and **Apply for a term loan** (primary, `CoinsStacked01`).

⚠️ **Why not "New term loan" / "Increase credit limit"** as suggested: both imply the action completes. Neither
does — a limit increase is re-verified and can come back as *More info needed*, and a term loan is
underwritten. `Request` and `Apply for` are the honest verbs, and they match the language already used on the
Applications tab and in the empty states.

⚠️ **Bank connection sits at Step 2, not literally first.** Step 1 is a 90-second pre-qualification that
collects nothing sensitive and can knock the applicant out (wrong country, prohibited industry, revenue too
low). Asking for a bank connection *before* that means collecting financial data from people who were always
going to be declined — a data-minimisation problem. The gate is gone, which was the point; the ordering
inside the flow is pre-qualify, then connect.

⚠️ **Known bug, deliberately not shipped.** Wiring `SelectInstitutionModal` into the DS-1 Connect button
worked for the happy path — the picker opened, choosing an institution set DS-1 to Connected and moved
Application strength 0% → 30% — but **the modal would not close afterwards, and its X button did not close it
either**. `setInstitutionOpen(false)` had no effect while `setSource` in the same handler did; it is not a
stale bundle (the dev server was serving the new code) and not an orphaned portal (same live node, one
overlay). Root cause not found, so the wiring was backed out rather than ship a dialog with no exit. DS-1's
Connect marks the source connected directly, as it did before. `SelectInstitutionModal` is now only reachable
from its own story — worth either fixing or retiring.

### Alert banners are a local fork (2026-08-18)

`_prototype-scratch/credit/alert-banner.tsx` replaces `ui/alert-banner` throughout the credit prototype,
rebuilt from **Loop Design System → node `15329:475` (Alert banner states)**. The shipped shared component is
out of date against that node in five ways: it has only `success`/`error` (no warning, no info), no border,
`rounded-md`, no title-optional mode, and it **stacks the action below the text, centred**, where the design
puts it inline on the right.

| Slot | Behaviour |
| --- | --- |
| `variant` | `success` · `error` · `warning` · `info` — sets surface, border, icon and title colour |
| `title` | semibold in the accent colour; **medium** weight when it is the only line (matches both action rows in the design) |
| `message` | always `text-secondary`, regular |
| `action` | optional node, inline right, container switches to `items-center` with a 24px gap |

Geometry from the node: `rounded-[12px]`, `px-4 py-3.5`, 20px icon, 10px icon gap, 2px title/message gap,
14/20 type. Icons are the project's `@untitledui/icons` — `CheckCircle` / `AlertCircle` / `AlertTriangle` /
`InfoCircle` — verified glyph-identical to the exported SVGs (same path data, different viewBox scale), so no
assets were committed. Type stays **DM Sans**, not the node's Inter, per the standing prototype rule.

Actions use the real `Button`: `primary` on success, `primary-destructive` on error. The design's pill
geometry and fills (`#004c31`, `#e31b54`) already match those variants exactly.

⚠️ **This is why "override the design system" was the right instruction.** Every colour token this component
needs **drifts between Figma and code**: `theme.css` aliases Loop's semantic names onto *Tailwind's stock
palette* (`--color-utility-green-200: var(--color-green-200)`), while the Figma design system is on the
**Untitled UI palette**. Same token names, different values — measured, not assumed:

| Token | Code (computed) | Figma |
| --- | --- | --- |
| `--color-utility-red-700` | `oklch(0.505 0.213 27.518)` — a plain red | `#c01048` — crimson |
| `--color-bg-error-solid` | `oklch(0.577 0.245 27.325)` | `#e31b54` |
| `--color-utility-green-200` | `oklch(0.925 0.084 155.995)` | `#aaf0c4` |
| `--color-utility-neutral-200` | `oklch(0.922 0 0)` — pure grey | `#e3e8ef` — cool grey |

All twelve are pinned to the Figma values in `prototype-overrides.css`, the same token-layer mechanism the
Button already uses. **The radius scale drifts too**: `rounded-md` is 18px in this theme and `rounded-xl` is
24px, so nothing maps to the node's 12px — hence the literal `rounded-[12px]`.

⚠️ **Two tokens deliberately left alone: `--color-text-primary` (code `#333` vs Figma `#121926`) and
`--color-text-secondary`.** They drift the same way, but they colour every heading and paragraph in the
prototype, so overriding them as a side effect of an alert-banner task would restyle everything. The only
visible cost today is the `info` title, and no info banner is in use yet. Worth a separate decision.

Verified by reading computed styles in the browser: 19 of 21 colour/geometry assertions exact, the two
misses being those text tokens. Gallery: `Prototype/Credit Alert Banner → AllStates`.

⚠️ The reviewer callouts (declined-application note, the Step 6 Legal placeholder) still use their own
hand-rolled `border-warning bg-warning-secondary/40` treatment. They are annotations, not product UI, and were
left as-is on purpose — but a real `warning` variant now exists if they should become one.

### The overdue banner

Error-token `AlertBanner`, rendered only when at least one term loan is overdue.

- **One loan:** names it, the amount past due and the original due date, with **Make a payment** linking to
  that loan's detail page.
- **Several:** a count and the combined amount, and **View overdue loans** filters the Term loans table to
  Overdue in place (`twoLoansOverdue` fixture, `TwoLoansOverdue` story).

Copy states the fact and the required action and stops there — no shaming, and **no mention of credit
reporting or default**, neither of which is confirmed (see the assumptions list below).

⚠️ The button says **Make a payment**, not *Pay now* as the 2026-08-17 request's sketch showed, because the
same request scopes "Pay now" to revolving actions only and keeps "Make a payment" as the term-loan verb.
The two mechanics differ, so the verbs stay distinct; the sketch is the odd one out. It is a one-word change
if the sketch should win.

### The Apply menu

The header's two buttons (*Request a limit increase*, *Apply for a term loan*) are now one **[Apply ▾]**
dropdown, and it only ever offers what the account can actually do:

| State | Menu |
| --- | --- |
| Has a line of credit | Term loan · Credit limit increase |
| No credit yet | Line of credit · Term loan · Credit limit increase *(disabled — there is no limit to raise)* |

*Line of credit* drops out once one exists, because Loop issues one revolving line. In the no-credit state the
menu **replaces the panel's old "Apply for credit" button** rather than adding a second apply affordance to
the header. A draft application still gets a plain **Continue application** button — resuming isn't a choice
between products.

| Detail page | Tabs |
| --- | --- |
| **Line of credit** (`#/credit/line`) | Summary (limit, drawn, available, APR, cycle close, due date) · Transactions (draws, card spend, payments, interest, fees) — with the balance and statement cards above them |
| **Term loan** (`#/credit/loans/1010`) | Summary · Payment schedule · Transactions (that loan's payments, origination and fees) · Early payoff |

`Applications` remains a tab: it tracks the initial application and limit-increase requests. Post-approval,
both ways to ask for more live in the header's **[Apply ▾]** menu — a **Term loan** (the full application,
pre-filled from what Loop already knows) and a **Credit limit increase** (the lighter re-verification flow).
The Applications tab keeps its own *Request a limit increase* button, since that surface has no Apply menu.

⚠️ **What this replaced.** Spec v2 §1.1 specified four tabs (Overview · Loans · Activity · Applications).
Overview and Activity are both gone: the accounts list took Overview's job, and the global Credit activity
ledger — built on 2026-08-13, merged into Overview on the same day — is now two per-account Transactions
tabs. `#/credit/activity` and `#/credit/loans` still resolve to the list so older links don't break.

⚠️ **The remaining tab pair is `Summary · Applications`.** It was **Loans** (the umbrella word the naming
pass rules out), then **Accounts**, now **Summary** at Rebecca's direction on 2026-08-18. Two known costs of
*Summary*, both accepted: it is the vocabulary of the Overview page the CEO removed, and **the loan detail
page already has a tab called Summary**, so the path reads *Credit → Summary → Loan #1010 → Summary*. If
either bites in review, *Accounts* is the fallback — the page's own copy already says "your credit accounts".

**Connecting a bank is its own screen with its own modal.** With no bank connected, the page is a single
panel, stripped to two ticks and one button: *Makes you eligible for a Loop credit limit application, as per
compliance policies* · *Instant bank account verification – no statements, no micro-deposits*, then
**Connect a bank account**. The explanatory paragraph, the *Instant bank connection* heading and the muted
*Applying for credit will be available once connected* were all cut on 2026-08-17 — the ticks say it, so the
prose was repetition. The button opens a **Select your institution** modal — search across 100+ institutions, the Flinks
trust line, and rows for Flinks Capital / TD / RBC / BMO / Scotiabank / CIBC with their brand-coloured initials.
Picking one connects the account and the page unlocks. Two notes: the institution avatars use **bank brand
hexes**, which sit outside Loop's palette on purpose because they belong to the banks; and *Can't find your
institution?* is Loop's brand-green link, not the blue of the Flinks screenshot, since the shell is ours.

⚠️ **A compliance claim survives in the bullet, against spec v2 §8.2.** The standalone sentence is gone, but
the first tick now reads *"…as per compliance policies"*. §8.2 flagged exactly this framing as likely
inaccurate — the source questionnaire treats banking data as load-bearing for **underwriting**, not as a
regulatory requirement — and recommended saying we connect a bank *to verify your business activity*. It's a
one-string change in `credit-page.tsx` (`INSTANT_CONNECTION_BENEFITS`) if compliance disagrees, and it should
be checked before anything external.

**With a bank connected but no credit, the page is one card plus a quiet prompt.** The card says
*You don't have a line of credit yet* / *Apply to get a credit limit you can distribute across your cards.* /
**Apply for credit**. Below it, outside the card, sits *Get approved faster* — one line about connecting a
sales channel to verify revenue, with a secondary **Connect sales channel** button. ⚠️ This replaces the
product's `ConnectSalesChannelInlineCta`, which is a large image/video card and overpowered the apply action
it was meant to support; the copy here is also shorter than that component's `next-intl` string. If this
lands, either the component or the string needs to follow.

**Zero states carry the Apply CTA.** An empty Lines of credit table offers *Apply for a line of credit*; an empty
term-loan table offers *Apply for a term loan* alongside *Convert a balance*, since converting is the other
way a term loan comes into being. With no credit at all, the page is a single apply panel and the tabs are
hidden until a bank account is connected.

**Per-account activity** filters by timeframe (last 30 days · 3 months · 12 months · all time, defaulting to
3 months) and paginates at 10 / 25 / 50 with a `1–10 of 27` counter. Rows are attributed with a `loanId`, so
a term loan's Transactions tab shows only its own payments, origination and late fee, while the line's tab
shows what actually moved the revolving balance. Filtering is relative to a fixed `ACTIVITY_TODAY`
(2026-08-31), not the real clock, so snapshots stay deterministic.

**The application is its own page, not a dashboard tab.** Starting an application leaves the app
shell entirely — no sidebar, no Credit tabs. Sticky 64px bar (Loop logo, "Credit application",
*Save and exit*, account chip) from
`Project #4 - Onboarding/Prototype/Iterations/Onboarding_Redesign_V3.7.html` — same translucent bar
with an `#e3e8ef` rule, same 600px column, same 38px/-0.9px title. Decline and submitted screens
share the chrome.

**Progress lives in a left rail, deliberately unlike onboarding.** Onboarding uses a horizontal
numbered indicator in the top bar; the credit application uses a vertical rail — a 3px bar per step in
brand green once reached, grey ahead, a filled check on completed steps, current step in semibold.
Following the Car IQ Pay and Mercury *Create a SAFE* references. **There is no progress indication in
the top bar**, so the two flows read as different jobs at a glance.

The questionnaire is **centred on the page** (720px) and the 220px rail sits in the left margin, 64px
clear of it, sticky as you scroll. Built as `grid-cols-[1fr_720px_1fr]` so the middle column is
page-centred regardless of the rail. Below `xl` the rail drops out and a "Step N of M" line sits above
the content. The column was 600px until the two-up choice cards forced *Virtual office or
mail-forwarding service* onto a second line; 720px is the width at which every option in that question
fits on one row, so the cards stay a uniform height.

⚠️ **Two routing/wiring bugs fixed 2026-08-18, both introduced earlier the same day:**

1. **The application flow became unreachable from the UI.** The product-card and header CTAs point at
   `/credit/apply?product=…`, but the standalone router matched routes with `knownRoutes.includes(pathname)`
   and `usePathname()` returned the raw hash *including the query* — so every Apply CTA fell through to the
   *Not part of this prototype* screen. The shim now splits query from path: `usePathname()` returns the
   pathname alone, and `useRouter()` exposes a real `query` object and full `asPath`. The `?product=` param is
   kept deliberately — nothing consumes it yet, but it is what a forked application flow would read.
2. **Every pay action opened the conversion path.** `CreditPage` hardcoded `defaultPath="convert"` on the
   shared `PayBalanceModal`, from when the only trigger was *Convert a balance*. Once the overdue banner and
   the statement card started using the same modal, **Pay now** landed on *Convert to a loan*. Both pages now
   track a `payPath` and open the modal where the action means: `pay` from the overdue banner and the
   statement card, `convert` from *Convert a balance* / *Convert to a term loan*, `choose` from the legacy
   `#/credit/pay` deep link. `CreditPage` takes `defaultPayPath` so stories can pin either path — see
   `ConvertModal` and the new `PayModal`.

**Paying is a modal, not a page.** *Pay now* on the line-of-credit page and *Convert a balance* / *Convert to
a term loan* on the credit page open the same dialog over whatever you were looking at, so you never lose the balance you're paying down. It forks
before anything else renders — **Pay it off** (statement / minimum / custom) or **Convert to a loan** —
and the title follows the path — *Pay now* or *Convert to a loan*. The two option sets never appear together.
There is no separate pay tab; the old `#/credit/pay` link still works and opens the credit page with the
modal already up.

**Choosing a loan term** lives in that same modal, on the *Convert to a loan* path: pick how much to
convert, then the duration on a 3 / 6 / 9 / 12-month segmented control, and the monthly payment, total
to repay and APR re-derive as you change it (`monthlyPaymentFor` amortises properly — these are not
hardcoded). The control is the same rounded-pill segmented control the accounts chart uses, so it reads
as one system with the Savings prototype. Conversion is the highest-stakes disclosure moment, so APR,
monthly payment, total to repay and total interest stay on screen together, and there is **no minimum
conversion amount** — the only bound is the outstanding balance. ⚠️ The APR table (`APR_BY_TERM`,
12.9–14.9%) is placeholder. The *No hidden fees* callout is rendered on a neutral surface, not the lilac
of the mock, because accent purple is graphic-design-only and never appears in product UI.

**The line's second card is a single statement, not a schedule.** ⚠️ This reverses the
*Upcoming statements* schedule built on 2026-08-12 at Rebecca's request, because spec v2 §4.1 rules it
out: a revolving line has no predetermined multi-month payment schedule, so rendering one imported
amortising mechanics into a cycle-based surface. Multi-month schedules now live only on loan detail
pages, where they're correct. If the schedule is still wanted on the line, that's a product-model
question to settle before rebuilding it.

The **Next statement** card is ordered so the cheapest choice is the loudest: statement balance as the
primary figure with *pay in full to avoid interest*, minimum due secondary in size and weight with the
consequence stated factually (`Interest applies to any remaining balance at 16.99% APR`), then an
explicit autopay line — on/off, the date it runs, **and which amount it pays**.

**Pay now sits on the statement-balance row as a green primary button; Edit autopay sits in the footer as a
white tertiary button with a pencil icon.** ⚠️ This is Rebecca's call from 2026-08-13 and it overrides v2
§4.3, which asked for the manual pay button to be *secondary* when autopay is on so the two wouldn't
compete. The reasoning for the change is hierarchy: paying is the action, editing autopay is a setting, so
the action gets the weight and the setting gets the quiet treatment — and *Pay now* now sits beside the
number it acts on instead of below a sentence about autopay.

**Every figure is computed, not typed.** Payment schedules amortise properly — principal + interest =
payment on each row, ending balance falls monotonically to exactly `$0.00` on the final row, and total
payments exceed principal (a schedule where they matched would imply 0% against a displayed APR). The
due date is the cycle close plus a ~21-day grace period, so an Aug 1–31 cycle is due **Sep 21** rather
than mid-cycle. Available credit comes from the formula above, which is why converting to a loan leaves
it flat and only repayment recovers it.

*Download loan agreement* is on each loan's detail page, not the list — the model supports several loans,
so a single link on a shared surface was wrong. It's still a stub: there's no loan-agreement entity in
the schema (see Known gaps).

## The vocabulary (naming pass, 2026-08-17)

Applied everywhere the words render — tables, empty states, modals, tooltips, tabs, alert copy and the
seeded activity descriptions:

| Was | Is |
| --- | --- |
| Revolving loans / Revolving lines of credit | **Lines of credit** |
| Installment loans | **Term loans** |
| Pay balance *(revolving only)* | **Pay now** |
| Amortization schedule *(tab)* | **Payment schedule** |
| Revolving balance *(inside line-of-credit detail)* | **Balance** |
| Loans *(first tab)* | **Accounts** |

Unchanged because they were already right: the page title **Credit**, the loan detail title (the object name,
*Loan #1010*), its tabs (Summary · Payment schedule · Transactions · Early payoff), **Make a payment**, and
**Convert a balance**.

**Verb discipline.** A line of credit is paid with **Pay now**; a term loan with **Make a payment**. The
mechanics differ — one clears a revolving statement, the other services an amortising schedule — so the verbs
stay distinct wherever they appear, including inside the overdue banner.

**Never in UI copy:** "LOC" or any abbreviation of line of credit · "Facilities" / "Products" / "Loans" as an
umbrella heading over both tables · "Revolving loan", because a line of credit is not a loan in customer copy.

Verified by scanning the **rendered** text of all 25 credit story snapshots in `demo/` plus the built
single file: **zero** occurrences of any banned term. Two survivors are deliberate and invisible to users —
the `LOC_APR` constant and one pre-existing code comment in `mock-data.ts`. `next-app`'s house rule forbids
rewriting existing comments, so that line was left alone; rename it whenever the file is next edited for
other reasons.

## The fixture has one coherent story

A finance-literate reviewer reads the ledger against the cards, so the seed data has to hold together.
It was rebuilt on 2026-08-13 after four contradictions were caught:

- **Loan #1010 originates Apr 24, 2026**, and the conversion row in Activity is dated Apr 24 — it used to say
  Aug 12, which is impossible for a loan with four payments already made (May/Jun/Jul/Aug 24, next Sep 24).
- **Its monthly payment is $5,412.67 everywhere.** Activity had $5,401.62 — transposed digits against the real
  amortised figure. Activity also gained the Jul 24 payment it was missing, and lost a Jul 12 payment on
  Loan #1021 that contradicted that loan being *overdue on exactly that payment*.
- **Autopay was switched on Aug 5, 2026**, and the card says so. That's what makes the history legal: the
  Jul 1 and Aug 1 interest charges and the partial $34,000 payment on Jul 21 happened while the account was
  paying manually. If autopay had always paid in full, no interest could have accrued at all. The old
  $14,433.33 "line of credit payment" is gone — it was a leftover from the fabricated amortising schedule that
  §3.1 of the spec killed.
- **The $45 late fee is dated Jul 27 and names Loan #1021**, so the fee in the overdue banner has a matching
  ledger row instead of appearing from nowhere.
- **Days overdue is derived**, not typed: `ACTIVITY_TODAY − missed due date`, which is why the banner says 50
  days rather than the hardcoded 12 it used to claim while the cycle sat in August.

⚠️ **A fifth contradiction, caught 2026-08-17 by the summary band.** `limitReached` set the line's balance to
the full $250,000 limit *and* kept Loan #1010's $40,978.70 outstanding, so the account owed $290,978.70
against a $250,000 limit. Invisible while nothing added the two together; the band made it a headline. The
line's balance is now derived — `limit − outstanding term-loan principal` = $209,021.30 — with the statement
balance following it and the minimum at the same 5% the other fixtures use. Owed is now exactly the limit,
which is what "limit reached" should mean.

**New fixture: `moreTermLoans`** — four term loans (`#1055` $20k/3mo, `#1042` $45k/9mo, plus the existing
`#1010` and paid-off `#1004`) against the same $250,000 limit. Owed $176,382.35, available $73,617.65, **70.6%
utilization**, which is what makes the new bar worth looking at. Each new loan has matching conversion and
payment rows in the ledger, merged and re-sorted by date. Reachable as the *Several term loans* persona in
the single file and the `MoreTermLoans` story.

**New fixture: `twoLoansOverdue`** (Loan #1021 + Loan #1033, $30,000 over 9 months at 13.4%, 4 paid, $45 late
fee) exists to exercise the multi-loan branch of the overdue banner. Its numbers amortise from the same
`buildLoan` helper as the rest, and it still totals to the limit.

## Fixed 2026-08-13 — the three things that blocked testing

**1. Numeric and text inputs were inert, and owner data corrupted to `[object Object]`.** My own regression
from swapping the hand-rolled fields onto the design system: `InputBase` spreads `value`/`onChange` straight
onto the native input, so `onChange` handed back a **DOM event**, not a string. Every field stored the event
— text fields rendered `[object Object]`, and numeric fields parsed to `NaN → 0`, which is why revenue,
margin, spend and phone looked typeable but changed nothing. The fix is to put `value`/`onChange` on the
`TextField` / `InputGroup` wrapper (which is the React Aria field) and leave `<InputBase/>` bare inside it.
Applied to `TextBox` and to all three money inputs (pay/convert, loan payment, limit increase — same bug).
Proof it's really fixed: typing `52000` into monthly spend now re-derives the indicative limit to `$78,000`.

⚠️ **How this got missed:** the earlier verification clicked radios, chips, selects and the term control, but
never typed into a text field. Type into inputs when verifying a form, and check a derived value changes.

**2. Post-submit was a dead end.** Submitting left the Applications tab empty and said nothing about the
documents still owed. Now the submit hands a summary (requested limit + outstanding documents) back to the
app, so the Applications tab shows a real row — *More info needed*, the requested amount, the documents it's
waiting on by name, and an **Upload documents** action. The submitted screen names the amount under review,
lists the outstanding documents, and offers **Track your application** plus **Upload documents now** instead
of a lone *Back to Credit*.

**3. Validation was strictest about the things that mattered least.** The gate was a silent boolean: hard
stops on consent checkboxes with no explanation, and nothing at all on the financials. It's now a list of
**named blockers** per step, rendered beside the disabled *Continue* so the button explains itself
("Tick the personal information consent", "Upload 5 remaining documents"). Newly gated: total revenue
≥ $100,000, gross margin between 1–100%, every triggered document uploaded, per-owner email and phone
(named individually), each guarantor's consent, monthly spend ≥ $5,000, and — a new required field — what a
requested over-limit would be used for.

## Application-flow conventions (set 2026-08-17)

- **Prefilled fields say "Autofilled from your Loop account"** in plain grey text. The sparkle icon is gone —
  the words carry it, and a decorative icon on every prefilled question was noise.
- **Mandatory questions carry a red asterisk**, and "mandatory" means exactly *what gates Continue*: monthly
  spend, the over-limit reason, the credit-check consent, the three data consents, total revenue, gross
  margin, per-owner email and phone, the triggered documents, and each guarantor's consent. Asterisk and
  blocker list are driven by the same rule, so the page can't mark something required that doesn't stop you.
- **Canadian English, -ize:** authorize / authorized / itemized / summarized / amortizing. `-our` spellings
  (colour, favour) stay. The answer field was renamed `isAuthorizedSigner` so code and copy agree.
- **Save and finish later is a tertiary button beside Continue**, on every step — it was a grey text link
  floating after the primary, which read as a caption rather than a choice.
- **Icons use brand green, never pale green.** Success featured-icons are now `color="brand" theme="gradient"`
  (solid brand disc, white glyph) and inline confirmation icons use `text-fg-brand-primary`.
- **Conditional follow-ups ease in and out** — `Reveal` in `application/fields.tsx`. It keeps the content
  mounted and transitions height + opacity over 300ms, so the collapse animates as well as the open;
  `motion-reduce` turns it off. A measured pixel height is used **only while the transition runs**; once open it
  hands back to `height: auto` with `overflow: visible`.
  ⚠️ That hand-back is the whole point, and it took three tries. A frozen measured height **clips** anything
  that grows after opening — chips reflow when the webfont loads, picking *Other* adds a write-in — which is
  exactly the bug Rebecca caught on 2026-08-17 (the inventory chips cut off mid-row). The two rejected
  approaches: framer-motion's `height: 'auto'` never measured (stuck at 0), and the CSS `grid-rows-[0fr→1fr]`
  trick collapses because the inner element needs `overflow: hidden`, which gives an `fr` track an automatic
  minimum of 0. **If you touch this, check both states in the DOM:** closed must be `height: 0px` +
  `overflow-hidden`, open must be `height: auto` + `overflow-visible`. A screenshot of the open state alone
  won't catch the clip — the content is there, just cut off.
- **Two review annotations came out of the UI**: the yellow "pending Legal" placeholder under the credit-check
  consent, and the line explaining the soft-credit-check consent wasn't repeated. The open question about
  `[legal entity name]` still lives in *What needs a decision*, which is where reviewer notes belong.
- **An approved application links out with "Open page →"** (a real `ArrowRight` icon, not an emoji) rather
  than "View decision" — there's a page to go to, so the label says so. Declined still says View decision.

### Strength meter removed, and the flow walks freely (2026-08-19)

**The "Application strength" meter is gone from the UI.** Spec §4.2 asks for it, but its own weight table
makes it unusable as written: `completeness()` sums all four sources regardless of which ones the applicant
was asked for, so a business below the $50,000 threshold that connects its bank — satisfying every
requirement in the application — is told it is at **30%**, with no route to 100% that doesn't involve
volunteering data the product never requested. 100% is reachable only by an applicant who is both above
$150,000 governing spend *and* B2C, i.e. the heaviest-documentation case in the product. It also scores
uploading 10 points below connecting, in plain view, which contradicts §4.1's instruction that the upload
fallback "must be treated as a genuine equivalent rather than an afterthought."

`completeness()` and `SOURCE_WEIGHTS` **stay in `rules.ts` with their unit tests**, so restoring the meter is
re-adding one block to `application-shell.tsx` and one prop pass. Nothing was deleted from the rules engine.
The **document counter is untouched** — spec §2's running counter still shows on Steps 2 and 3, now in its own
panel rather than tucked under the bar.

**`enforceGating` prop on `CreditApplicationFlow`, default `true`.** When false, Continue is never disabled and
the "Before you continue" blocker list is hidden. The single-file demo passes `enforceGating={false}` via a
**Skip required fields** checkbox in the prototype panel, **on by default** — so the whole application can be
walked Step 1 → submitted without filling anything, which is what iterating on it needs. Turning the checkbox
off restores real validation, blocker copy included. Storybook is unaffected: every story keeps
`enforceGating` at its default, so the gated states stay reviewable.

⚠️ **With gating off, the knockouts don't stop you either** — picking *Money services business* now continues
rather than silently disabling Continue. That path was already unreachable in the single file (the decline
screen is gated on `showKnockout`, which only the stories pass — see `spec-conformance-2026-08-19.md` §1.1);
wiring it properly is still outstanding.

## What ships next: the application flow

Cato's call is that **the application is the next feature to ship**, so that's where the design effort goes;
the accounts view above is correct but unpolished. What exists today:

- **The 7-step new application** (pre-qualification → connect data → questionnaire → owners → documents →
  guarantors → review), driven by a real rules engine — see *What actually derives* below.
- **Apply ▾ → Term loan** reuses that same application, **pre-filled** from what Loop already knows, rather
  than a second flow. ⚠️ It does not yet ask the term-loan-specific questions (amount, term, purpose) — that's
  the next thing to design.
- **Apply ▾ → Credit limit increase** is the deliberately lighter path: requested limit → re-verify connected
  data (reconnect anything stale) → confirm. It never re-asks the questionnaire, owners or documents.
- **Post-submit** names the amount under review, lists outstanding documents, and routes to Applications.

### Worth trying

- Change the monthly spend in Step 1 — the limit re-derives at 1.5x. Then *Request a different
  limit* above $75,000 and watch the requirements change before you leave the step.
- Answer **“Is the business profitable over the last 12 months?” = No** in Step 3. The progress indicator goes from 6 steps to 7 as
  Guarantors appears, three follow-ups open inline, and the document list grows.
- Pick **Money services business** under *What industry are you in?*, or the lowest revenue band,
  to hit a knockout.
- Tick **Other** under *Where do you store your inventory?* — a write-in field opens for the answer.
- Hover either half of the line of credit's utilization bar. The green half names what's used, the grey
  half what's left.
- *Pay now* → **Convert to a loan** → change the term. 3 / 6 / 9 / 12 months re-derives the monthly
  payment, total to repay and APR live.
- **The Overdue option on Term loans.** The status filter is a secondary-button dropdown in the table header;
  Overdue only appears in the menu when a loan is actually overdue, carries its count as an addon pill, and the
  overdue rows sort to the top whichever filter is on. Open **Loan #1021** for the escalation: header
  badge, an error banner with amount past due / original due date / days overdue / late fee, a
  *Make a payment* CTA pre-filled with the past-due amount rather than the scheduled one, and the missed
  row marked **Missed** in the schedule.
- **Read the band against the tables.** Total owed is the line's *Drawn* plus every term loan's *Outstanding*,
  and Available credit is the limit minus that. They tie out to the cent in every state.
- **Story `TwoLoansOverdue`.** The banner switches to a count and a combined amount, and **View overdue loans**
  filters the table to the two overdue rows without leaving the page.
- **Open [Apply ▾] in `Default`, then in `NoCreditYet`.** The first offers Term loan and Credit limit increase;
  the second swaps in Line of credit and greys out the limit increase, because there's no limit to raise.
- **Story `LimitReached`.** Available $0.00, owed exactly $250,000.00 — the fixture no longer overshoots
  its own limit.
- **Applications → Request a limit increase.** Three steps, not seven: the new limit, a re-verification
  pass over connected data (one source shows *Needs reconnection* — reconnect it and watch the review step
  change), then confirm. Submitting without reconnecting warns that it may come back as *More info needed*.
- **Open the line of credit, then its Transactions tab**, and switch the timeframe to *All time* — three
  pages of events. No merchant rows: card spend is one *Card spend — August cycle* line linking out to
  Transactions. Then open **Loan #1010 → Transactions** and see only that loan's payments and origination.
- Step 4 has one owner with a missing phone. Continue stays disabled until you fix it.
- In the pay modal, enter a custom amount above the outstanding balance, or pick the RBC account for
  a payment it can't cover — both block Continue with the reason stated.
- Submit, then go back to Credit — the dashboard now shows the application under review.

## Prototype source

Lives in the next-app sandbox, **not** in this folder:

```
~/Downloads/next-app-main/src/components/pages/_prototype-scratch/credit/
                                       |-- application/   the 7-step flow + rules engine
                                       `-- standalone/    the single-file build
```

Landing-view files worth knowing: `credit-page.tsx` (order, Apply menu, loan-filter state),
`credit-summary-band.tsx` (new), `overdue-loan-banner.tsx`, `term-loans-table.tsx` (filter is now
controlled by the page, so the banner can drive it), `lines-of-credit-table.tsx` (**renamed** from
`revolving-loans-table.tsx`; the export is `LinesOfCreditTable`), and `mock-data.ts` for the derivations.

The 25 `demo/prototype-credit--*.html` snapshots are regenerated from the running Storybook with
`chrome --headless --dump-dom`, one per story id. Re-dump them after any copy change or they'll assert
vocabulary the app no longer uses.

Storybook has the same work as isolated states — **`Prototype/Credit`** (38) and
**`Prototype/Credit Application`** (23):

```sh
cd ~/Downloads/next-app-main
./node_modules/.bin/storybook dev -p 6006      # `corepack yarn storybook` fails: stale lockfile
```

### Rebuilding the single file

```sh
cd ~/Downloads/next-app-main
node src/components/pages/_prototype-scratch/credit/standalone/build-css.mjs   # after CSS/token changes
node src/components/pages/_prototype-scratch/credit/standalone/build.mjs
cp src/components/pages/_prototype-scratch/credit/standalone/dist/loop-credit-prototype.html \
   "$HOME/loop/projects/Other Projects/Credit Page/index.html"                # then commit + push to deploy
```

⚠️ **Run `build-css.mjs` whenever a component is added or swapped**, not just on token edits. `app.css`
only contains the Tailwind classes present when it was generated, so a new component's classes are
silently missing from the single file — Storybook looks right while the built page renders flattened
controls. If something looks subtly wrong only in the built `index.html`, this is why.

esbuild bundles the real React tree; `next/image`, `next/link`, `next/router` and `next-themes` are
swapped for small shims in `standalone/shims/`; SVGs, the logo and the DM Sans webfont are all inlined as
data URIs. The file dropped ~280 kB on 2026-08-17 when the sales-channel prompt stopped using the product's
`ConnectSalesChannelInlineCta` — that component's artwork was the largest inlined PNG in the bundle.

## Figma

**[Credit Page](https://www.figma.com/design/MfhBC6j94TsNMCemEAIUDf/Credit-Page)** — page *Credit — screens*.
Five editable frames at 1500×1050 — **Credit — Overview**, **Credit — Balances**, **Credit — Activity**,
**Credit — Ready to apply**, **Credit — No bank account**.

⚠️ **The Figma file predates the v2 restructure** and is now out of step with the prototype: it still has a
Balances frame and a three-tab component, no Loans or Applications frames, and the old Overview card layout
with next/last payment. Re-sync it from the current prototype before using it for anything.

Built with auto-layout throughout and wired to local tokens, so it edits like a design file rather than a
screenshot:

- **Loop colours** — 22 semantic variables (`bg/*`, `text/*`, `brand/*`, `button/secondary-*`, `status/*`)
  bound to every fill and stroke. Change `brand/solid` and all three screens follow.
- **12 text styles** on the DM Sans ramp + **Shadow/lg**, **Shadow/xs** effect styles.
- **Components** — `App / Sidebar`, `App / Top nav`, `Nav item` (Default/Active, swappable icon +
  text prop), `Button` (Primary/Secondary), `Badge` (Success/Warning/Gray), `Credit tabs`
  (Active=Overview/Balances/Activity), and 10 Untitled UI icon components imported from the real
  `@untitledui/icons` path data. `Button` also carries an optional leading icon (boolean + instance swap).

⚠️ The connect-a-sales-channel artwork on *Ready to apply* is an **image placeholder** in Figma, and the
prototype no longer uses that artwork at all — the prompt is now a plain text-and-button row (see *Zero
states* above), so the Figma frame and the prototype differ here.

## Sidebar consolidation

Credit made a 13-item flat list worse. It is now **9 destinations in 3 labelled groups**, no hidden items:

| | |
| --- | --- |
| — | Home |
| **Banking** | Accounts · Cards · **Credit** |
| **Payments** | Payments · Transactions · Approvals |
| **Perks** | Rewards |
| pinned bottom | Settings |

**No chevrons — deliberately.** Collapsible groups trade recognition for recall: you have to remember
which group holds what, and every visit costs a click to re-open. For a list this size that is a bad
trade. Static labels keep every destination visible and scannable.

The reduction comes from merging destinations that are already tab-shaped, not from hiding them:

- **Send Payments + Request Payments + Automations → Payments** (tabs on one page — the pattern Cards,
  Credit and Transactions already use)
- **Travel → a tab under Rewards**
- **Subscription → Settings** (it is billing admin)

Credit sits under Banking with Accounts and Cards because the limit funds the cards — the same
relationship the build spec found in `§3`.

## Design system source

Buttons and brand colour come from **[Loop Design System](https://www.figma.com/design/wT39SoimuAWJsoZZxEE5Sn/Loop-Design-System?node-id=15202-6386)**
(Primary Button, node `15202:6386`), pulled via the Figma MCP on 2026-08-12. Adopted at the **token
layer** in `prototype-overrides.css` rather than by forking the shared `Button` component, so the
prototype keeps using the real component and only the values change.

| Role | Token | Figma | Value |
| --- | --- | --- | --- |
| Primary fill | `--color-bg-brand-primary` / `-solid` | Primary/600 | `#004c31` |
| Primary hover | `--color-bg-brand-solid_hover` | Primary/700 | `#003f25` |
| Primary edge | `--color-border-brand_alt` | Primary/600 | `#004c31` (inset ring, keeps 36/40px) |
| Secondary fill | `--color-button-secondary-bg` | Gray/50 | `#f8fafc` |
| Secondary hover | `--color-button-secondary-bg_hover` | Gray/100 | `#eef2f6` |
| Secondary border | `--color-button-secondary-border` | Gray/300 | `#cdd5df` |
| Secondary text | `--color-button-secondary-text` | Gray/800 | `#202939` |
| Filter | `--color-button-filter-*` | same greys | lilac retired |

Geometry: pill, 14px horizontal padding, 8px (sm, 36px tall) / 10px (md, 40px tall) vertical, 14/20
semibold, `0 1px 2px #0000000D`, 50% opacity disabled.

**The pale Loop light-green secondary is retired.** It was `--loop-light-green`; the current system
makes secondary an off-white `#f8fafc` with a `#cdd5df` border and `#202939` text. Brand-tinted
*surfaces* (`--color-bg-brand-secondary`, selection and success washes) are a different role and stay
green — only the button changed.

This moves brand green off the old `#045b3f`/`#004932`/`#003926` ramp. Two things to know:

- Figma binds the primary fill to a token named **`--color-button-purple-bg`**, which now holds a
  green. Confusing name, correct value — mapped onto the brand tokens here rather than carried over.
- Secondary values came from the **semantic-token page** (`15120:3413`), read off the rendered
  swatches rather than the row labels. The Primary Button's component description says Neutral is
  `#F8FAFC`, which agrees.
- ⚠️ The shipped `Button` keeps its skeuomorphic inset shadow on secondary. I had no Figma node for
  the secondary button itself, so I changed only the four colour tokens and left the shadow alone —
  worth confirming against the real component.
- The Figma variable sets **`--font-body: Inter`**. The prototype stays on DM Sans per the 2026-08-12
  instruction below; worth reconciling in the Figma file.

**Typeface: DM Sans, everywhere.** The legacy theme sets `--font-body` / `--font-display` to Inter.
Both the build and the sandbox `FontLoader` override those *and* `--font-mono` to **DM Sans**, so every
piece of text — including money — renders in it. There is no DM Mono in the prototype.

Money keeps `font-variant-numeric: tabular-nums`, which is a DM Sans OpenType feature rather than a
second typeface, so columns of amounts still line up. Note this is a deliberate divergence from
`loop-design-system`, which specifies DM Mono for `number-tabular` figures — Rebecca's call, 2026-08-12.

⚠️ **Run `build-css.mjs` before `build.mjs` whenever components or classes change.** The prebuilt
`standalone/app.css` only contains the utilities present when it was generated; skipping it silently
flattens new UI in the single file while Storybook still looks correct. The router shim is what makes `href="/credit/pay"` work as hash
navigation without touching a single component.

Nothing outside `_prototype-scratch/` was touched. No product code was edited.

### Spec identifiers stay behind the scenes

The questionnaire's spec ids — `PQ-1`, `Q11`, `FQ-06`, `KO-01`, `CN-2`, the `spec §n` citations —
are **not rendered anywhere in the UI**. They're internal reference, so an applicant never sees
them. They survive in the code as the `id` prop on each `QuestionBlock` / `FollowUp`, which keeps
every question traceable to Cato's spec without putting a form number in front of a customer. The
reviewer callouts (legal-entity placeholder, document overflow, declined exit) are still on screen —
only the numbering was stripped, not the open question.

### Fields use the real design system

`application/fields.tsx` used to hand-roll its inputs, radios, pills and searchable select in raw
Tailwind, which drifted from the rest of the app — border-vs-ring focus, `rounded-xl` cards, 1px
fake radio dots, no focus rings, no radio/combobox semantics. Every control is now the real Loop
component, the same ones the Savings Account prototype was transcribed from:

| Field | Component |
| --- | --- |
| `TextBox` | `base/input` — `InputBase`, wrapped in `InputGroup` with `InputPrefix` addons for `$` / `%` / currency |
| `ChoiceGroup`, `YesNo` | `base/radio-buttons` — `RadioGroup variant="bordered"` + `RadioButton` |
| `ChipToggle`, `MultiSelectChips` | `base/buttons` — `Button`, `tertiary` unselected → `primary` selected |
| `SearchSelect` | `base/select` — `Select.ComboBox` + `Select.Item` |
| `Field` labels/hints | `base/input` — `Label`, `HintText` |
| `ConfirmCard` “Change” | `base/buttons` — `Button color="link-color"` |

`LongText` is the one exception — the design system has no textarea, so it stays hand-rolled, now
matching the app's own precedent (`pages/transactions/components/note-editor.tsx`) with the DS ring
treatment. Selected chips are solid brand green and unselected are neutral, which is the same
one-solid-action rule the Savings prototype follows; no lilac `color="filter"` variant is used.

Side effects worth knowing: the raw `<button>` ESLint suppressions are gone, keyboard arrow-key
navigation and screen-reader roles now come free with the radio groups and the combobox, and
`build-css.mjs` **must** be re-run before `build.mjs` — the new components pull in Tailwind classes
the prebuilt CSS didn't have, and skipping it silently flattens the radio cards.

**Applying is gated on a connected bank account**, matching the live rule in `useHomeState`
(`showApplyForCredit = isAccountOwnerOrAdmin && hasVerifiedBankAccounts && notStartedCreditApplication`).
It also means DS-1 arrives at Step 2 already connected, which the copy says out loud.


## What actually derives, rather than being hardcoded

`application/rules.ts` is a real engine with 33 unit tests (`rules.test.ts`). Answers drive:

- **Credit limit** = 1.5 × monthly spend; an override recalculates **governing spend** as
  `max(monthly spend, override ÷ 1.5)`, so you can't reach a higher limit on a lower documentation
  burden.
- **Required data sources** — DS-2 above $50k governing spend, DS-4 above $150k, DS-3 for any B2C.
- **Personal guarantee** — full / limited / none, from time in business, profitability, and leverage.
- **Whether Step 6 exists at all** — the progress indicator is 6 or 7 steps depending on the above.
- **The document list** — 15 triggers, capped at 6 with the overflow surfaced rather than hidden.

## Scope

**Card line of credit only.** Everything maps to real GraphQL types — `LineOfCredit`, `Fund`,
`CreditApplication`, `PaymentSchedule`. Nothing invented.

Deliberately excluded, with reasons in the spec:

- **Capital for Invoices** (invoice/PO financing) — real product, real ICP fit, but zero data model.
- **Venture Debt / SAFEs** — Mercury's other two tabs. Loop's ICP is cross-border revenue
  businesses, not VC-backed startups; SAFEs also contradicts Loop's published equity-free
  positioning.

## What needs a decision before this goes further

1. **Placement** — do `/cards/balances` and `/cards/pay` move under `/credit`, or does Credit link
   out? The prototype builds the move. Changes shipped routes.
2. **Declined exit** — `CreditApplicationStatusEnum.DEAD` exits only via "contact support", which
   fails Self Serve by construction. Built with placeholder copy so it is visible in review.
3. **Nav gate** — the existing `capital` ability requires `hasLineOfCredit`, so an applicant never
   sees the tab that exists to let them apply.
4. **Backend** — is `Fund.paymentSchedule` one payment or a list? Is cross-currency repayment
   supported?
5. **Compliance** — the bank-connection sentence (see *Connecting a bank* above — "as per compliance
   regulations" is back at Rebecca's direction but §8.2 disputes it), cost-of-credit disclosure and
   adverse-action wording. ⚠️ **The rate-free stance is
   over**: v2 §5 requires APR in three places (LOC metadata line, the Loans table and each loan's summary,
   and the conversion modal) and explicitly forbids hiding rate display behind an interaction. Every rate
   in here is a placeholder — LOC 16.99%, per-loan 12.9–14.9% — so compliance needs to confirm both the
   numbers and the disclosure wording before this is shown externally.
6. **Revolving product name** — "line of credit" throughout, but the sidebar already has **Cards** and the
   underwriting doc describes a card product. Two names for one object will confuse users. v2 §10 flags
   this as the one worth resolving early because it touches every label on the page.

### Assumptions built from v2 §10 — implemented as the default, flag if wrong

- **LOC APR is a single product rate** (`LOC_APR = 16.99`). If it's risk-priced per customer it becomes an
  account-level value and the constant has to go.
- **No cap on concurrent term loans** — bounded only by available credit. A cap would need a blocked state
  on the conversion entry point.
- **Overdue consequences are unstated.** The banner gives amount, date, days overdue and late fee, and
  deliberately says nothing about credit reporting or default, neither of which is confirmed.
- **Late fee is data-driven** — the banner shows one only when a fee value exists (`$45` on Loan #1021).
  How it's calculated is unconfirmed.

### Raised by the questionnaire spec itself

7. **PQ-10 legal entity name** — the consent text still reads `[legal entity name]`, and spec §3.3
   flags that it authorises a credit check on whoever fills the form, who may be neither an owner
   nor a guarantor. Legal needs to confirm that is intended. Rendered as a visible placeholder.
8. **DS-3 sequencing** — spec §4.1 leaves this open: DS-3 becomes required from Q5, which isn't
   answered until Step 3. The prototype takes the first option — offer it to everyone in Step 2 and
   escalate the card later — over accepting DOC-02 in Step 5.
9. **Document overflow selection** — spec §8 caps Step 5 at 6 cards and says Credit picks what to
   drop. The prototype shows the count that would be dropped rather than inventing a priority order.
10. **Guarantee wording** — Step 6 consent copy is a placeholder, same reason as (5).

## Known gaps

- No loan-agreement entity in the schema, so Mercury's *Download loan agreement* is not built.
- `MoneyValue` renders in the sans stack; `--font-mono` is Roboto Mono, not DM Mono. The prototype
  applies DM Mono locally to every figure. Correcting the shared component is a system-wide change,
  not this feature's to make.

### Application spec states not built

From §9 of the questionnaire spec, deliberately out of this pass: save-and-resume across devices,
application expiry after 30 days, the analyst's post-submission request for more information, the
second round on a specific long-text answer, connection *failure* and *expiry* states on Step 2
cards, and the Quebec French-language flow.

## To graduate

`next-intl` keys + fr-ca counterparts for all new copy, real Apollo hooks, a real route under
`src/pages/credit/`, and the placement decision above. Then PR review.
