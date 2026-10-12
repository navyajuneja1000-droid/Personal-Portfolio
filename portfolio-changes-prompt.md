# Portfolio changes — instructions for Claude Code

Save this file into the portfolio repo and say: "read
portfolio-changes-prompt.md and work through it."

---

## Context

Navya Juneja's portfolio: a hash-routed React SPA on Vercel. Existing
structure, which must be preserved:

- Hero: "The expensive failures are the quiet ones", three headline stats
- A provenance key labelling every figure **Result**, **Target**, **Modelled**
  or **Sourced**. This is non-negotiable and applies to every number below.
- "Selected work": case cards with a case number, tagline, description, topic
  tags, one headline stat, and an "Open the case study" link to `#/work/<slug>`
- A Capabilities evidence matrix: capabilities as rows, projects as columns
- Journey, Experience, Off the clock, Contact

**Read the existing ReconOS case page first.** Match its structure, depth and
the way it presents "Open the live build". Everything added below should feel
native to the site, not bolted on.

### Rules for all tasks

1. Never label a modelled or target figure as a Result. Where a number is a
   projection or measured against synthetic data, say so beside it, not in a
   footnote.
2. Do not invent numbers, quotes, user research or experiments. If a section
   needs something not supplied here, leave a clearly marked `TODO`.
3. Team projects state what Navya personally owned, separately from team
   output.
4. Update the Capabilities matrix after each case.
5. Renumber cases so they stay sequential after the removal in Task 1.
6. Every case with a live build gets the same "Open the live build" treatment
   as ReconOS, **plus two or three screenshots embedded on the page**. The
   proof must not depend on the reader clicking an external link.

---

## TASK 1 — Remove the café case

Delete "The Café, the Cup & Where the Money Goes" entirely: its card, its
`#/work/...` route and detail component, its column in the Capabilities
matrix, any nav or related-work references, and its unused assets. Then
renumber the remaining cases with no gap.

---

## TASK 2 — Add: MID Procurement Automation

**Live build:** https://workflowmidprocurement.vercel.app/ (verified live, no
sign-in)

### Card

```
Title: MID Procurement Automation
Tagline: Four teams, six banks, and nobody could say where the weeks went.
Description: An event-sourced system that measures turnaround per party,
reads bank replies out of the inbox, and catches missing data before it
costs an acquirer cycle. Built end to end.
Tags: system design, event sourcing, metric design, process automation
Headline stat: 98.7% of bank replies applied without a human · Modelled
Badge: Live build
```

### Detail page

**01 — The problem.** A payment aggregator cannot put a merchant live until an
acquiring bank issues a MID. Every merchant needs its own dedicated MID from
each acquirer and goes to 3 or 4 of 6 or 7 acquirers in parallel. Roughly
1,000 requests a month, three people, a Google Sheet updated by hand. 30 to
40% of requests bounced back for missing information. Turnaround per team was
worked out manually each week for a leadership review, and the question nobody
could answer was *why* a given request took 22 days.

Frame this as how MID procurement works across Indian acquiring, generalised
from operator experience. Not any one employer's internal process. Invented
acquirer names throughout.

**02 — What the tracker could not do.** A table:

| The tracker | What was needed |
| --- | --- |
| One row per merchant | One record per merchant, use case and acquirer |
| A status column per bank, dates typed by hand | Turnaround derived, never entered |
| "Partially procured", with no way to say which part | Per-acquirer state at a glance |
| Counting began when a request reached a bank | The queue before that, which was invisible |

**03 — The modelling decisions.** This section carries the product thinking.

The unit of work is **merchant + MCC + acquirer**, not merchant + acquirer.
One merchant can run several use cases, each needing its own MCC and its own
MID request, which is also why bank replies are matched on legal name plus
MCC. A one-row-per-merchant sheet structurally cannot hold this.

Then the turnaround rules:

- Working days only, per-acquirer holiday calendars
- A 2pm cut-off: a batch sent after it starts the bank's clock next day
- **Clocks overlap in two places.** Ops runs alongside the acquirer, because
  getting the MID is ops' job. Onboarding runs alongside BD, because
  onboarding owns the ticket until it comes back clean.
- Therefore per-party days sum to more than elapsed time, so a share-of-delay
  pie chart would be arithmetically wrong. The honest visual is a timeline per
  request.
- Ops time is measured per pickup, not merged, so the pickup count is itself
  the rework measure
- Ops time splits into hands-on and waiting-on-a-bank, so ops efficiency can
  be read without the bank's queue inside it
- Acquirer SLA is judged on the slowest single response round, not the total.
  A bank that answers in 4 days, asks for a document, then answers in 4 days
  has met a 5-day SLA twice; the 8-day sum would bill the bank for our own
  turnaround in between.

**04 — What was built.** Four parts. Every figure is **Modelled**: measured
against a seeded synthetic dataset of 360 merchants, 1,380 requests and 653
bank emails, not live traffic. Say that plainly, not in a footnote.

- **Event-sourced turnaround engine.** Nothing stores "TAT = 5 days". Every
  figure derives from an immutable event log, so numbers cannot go stale, a
  rule change recalculates history instead of losing it, and any number opens
  into its own timeline.
- **Bank reply parser.** Replies arrive as tables in the email body, numbered
  lists, or attached sheets, each acquirer with its own column names and
  wording. 1,179 rows from 590 emails, 98.7% applied automatically, 100%
  precision on what it applied, 1.3% held for a person.
- **Intake readiness check.** Per-acquirer field requirements plus format and
  cross-field validation. 82.3% of records clean on arrival, 9.0% ready for
  some acquirers and blocked for others, 8.7% blocked everywhere.
- **The dashboard.** Blended ops turnaround with the unraised queue reported
  separately, an acquirer scorecard, bounce and rejection reasons, and a
  per-request timeline.

**05 — What the data surfaced.** Three findings, written as findings:

- **380 requests assigned to ops and never raised**, median wait 24 working
  days. Invisible in any report that starts counting once a request reaches a
  bank, and it counts against ops.
- **The approval-to-MID gap.** Approval and credentials arrive on different
  email threads, so the days between them sit inside procurement turnaround
  and nobody measures them.
- **One merchant, eight live applications, four different states.** The
  "partially procured" case the old sheet had a word for but could not
  represent.

**06 — Two mistakes and how they were caught.** Include this. It matters more
than the results.

- **The name matcher merged two different companies.** Normalising legal names
  stripped "India" as noise. It isn't: "Oakhurst Ventures India Private
  Limited" and "Oakhurst Ventures Pvt Ltd" are different entities. Five bank
  replies were matched to the wrong request **at 1.00 confidence**, nothing
  flagged. A row held for review costs a minute; a row applied to the wrong
  request writes a false event into the clock and every figure downstream
  inherits it. The fix generalises: entity suffixes are safe to canonicalise,
  including misspelled ones, because they are not the distinctive part of a
  name. Anything before them is not.
- **The test data was made harder on purpose.** After that fix the parser hit
  100% with an empty review queue. A perfect score against synthetic mess of
  one's own invention proves little, and it left the human-review path
  untested. Status wordings outside the vocabulary and names mangled past
  recognition were added. The parser declines all of them rather than guessing.

**07 — What was deliberately not built.** Bank file generation per acquirer,
a website policy scanner with MCC recommendation, post-live monitoring, an
SLA chaser. Say why the sequencing put them last.

**08 — Open the live build.** ReconOS pattern, plus embedded screenshots of
the acquirer scorecard and the drill-down timeline.

---

## TASK 3 — Add: DisputeDesk

**Live build:** `TODO_DISPUTEDESK_PRODUCTION_URL`

The URL supplied (`dispute-desk-bkesor4vh-...vercel.app`) is a protected
preview deployment that redirects to `vercel.com/login`. Leave the placeholder
until Navya supplies the production URL with deployment protection off.

### Accuracy guardrail — read before writing

The project's own context document is explicit about this, and it must be
followed exactly:

**Can claim:** identified the problem and chose it after evaluating six other
ideas; defined the target user, positioning and success-fee pricing model;
designed and shipped a production-quality pre-launch site with an interactive
demo covering four reason codes across Visa and Mastercard; did platform and
competitor research that exposed a flawed core assumption before any build
money was spent; wrote a phased build and launch plan with gates; applied
payments domain knowledge.

**Must not claim:** that DisputeDesk is launched, live with customers, or
taken "from idea to live product". Only the marketing site and demo exist.
No users, merchants, signups, revenue, win rates or recovered amounts. Not
that the AI drafting works on real merchant data; the demo responses are
scripted samples. No Shopify App Store listing or approval.

Where the page describes the product working, it describes the **concept** and
the **demo**, never live operation.

### Card

```
Title: DisputeDesk
Tagline: Shopify fills in the basics. DisputeDesk builds the case.
Description: An AI chargeback assistant for small US online stores, scoped
around one insight: the evidence that wins a dispute depends entirely on the
reason code. Pre-launch site and interactive demo built; research killed the
original integration plan before a line of product was written.
Tags: product discovery, AI product, pricing model, platform research, live demo
Headline stat: 4 reason codes across Visa and Mastercard, in a working demo · Result
Badge: Live demo
```

### Detail page

**01 — The problem.** A chargeback happens when a customer goes to their bank
instead of the seller. The bank pulls the money back and asks the seller to
prove the sale was legitimate. Small sellers usually lose, and mostly not
because they are wrong:

- **The clock is short.** 7 to 21 days depending on network and processor.
  Many notice late. Miss it and the bank rules for the customer by default.
- **Generic responses lose winnable cases.** Each reason code has its own
  evidence standard. A tracking number wins "item not received" and does
  nothing for "unauthorized" or "not as described". Processor forms do not
  tell sellers which evidence matters.
- **They lose more than the sale.** The product, the revenue, and a dispute
  fee. Stripe added a separate counter fee in 2025.
- **They handle it alone.** The target seller is the founder reading the
  dispute email at 11pm, digging through Shopify for a tracking number.

**02 — Choosing this problem.** The idea was picked after evaluating and
killing six others. Show the table, because the kills are the evidence of
judgement:

| Idea | Why it was dropped |
| --- | --- |
| Reconciliation checker for kirana and small D2C | Not convinced by the direction |
| Stock and portfolio tracker | Crowded, weak differentiation |
| E-commerce price scraper | Dropped |
| Voice note to vendor order converter | Live mic reliability, weak repeat use |
| WhatsApp order-to-invoice generator | Led for a while, then set aside |
| AI competitor watchdog for US web agencies | Narrowed, then set aside |
| **DisputeDesk** | **Money at stake, success-fee model, direct fit with payments background** |

**03 — The insight.** The evidence that wins depends entirely on the reason
code. Everything in the product follows from that one line.

Show the before and after from the site:

- What most sellers submit, and lose with: "Hi, the customer got their order.
  Tracking attached. Please reverse the chargeback." Plus one screenshot.
- What a reason-code-specific draft contains for Visa 13.1 (not received):
  carrier delivery to the address-verified billing address, customer signed in
  and viewed the order two days after delivery, shipping and refund terms
  accepted at checkout, no contact before the dispute. Five exhibits, each
  mapped to a Visa 13.1 requirement.

**04 — The target user.** A US Shopify store doing roughly $5k to $100k a
month, with a handful of disputes a month: enough to hurt, not enough to hire
someone for. Later narrowed to clothing and accessories, which see many "not
as described" disputes.

**05 — How it works, and what the demo shows.** Four steps: connect the store
read-only, a dispute comes in and matching order data is pulled, a response is
drafted for that reason code, the seller reviews and submits. Every statement
in a draft must be backed by real store data.

The demo on the site runs four dispute types for a fictional store, Northfield
Goods LLC. Show this table:

| Dispute | Network and code | Amount | How the draft wins |
| --- | --- | --- | --- |
| Product not received | Visa 13.1 | $186.40 | USPS delivery to the verified address, post-delivery sign-in from the same IP, timestamped policy acceptance, no prior contact |
| Not as described | Mastercard 4853 | $128.00 | Archived listing against packing photo with SKU and care label, colour disclosure, full refund offered within 3 hours, prepaid return label never used |
| Unauthorized | Visa 10.4 | $242.75 | Qualifies for Visa Compelling Evidence 3.0: two earlier undisputed orders on the same card with matching IP, device, account and address |
| Duplicate charge | Mastercard 4834 | $64.00 | Two orders 38 minutes apart with different items, separate charge IDs and auth codes, the customer's own checkout note |

State clearly: all demo data is fictional and the responses are scripted
samples, not live AI output on real merchant data.

**06 — The business model.** $0 setup, $0 subscription, a percentage of what
is recovered, nothing on disputes that lose. "We only make money when you get
your money back." Pricing hypothesis after competitor research: **15% of
recovered**, against Chargeflow at 25% and Stripe Smart Disputes at 30%.
Label as a **hypothesis, not validated with sellers**.

**07 — The research that changed the plan.** This is the strongest section on
the page. Lead the reader through it properly.

Three findings, before any product was built:

1. **US Shopify stores cannot use Stripe as their gateway.** Where Shopify
   Payments is available, Stripe is not offered separately and disputes are
   handled in the Shopify admin. "Shopify sellers on Stripe" was not a real
   segment. The product had to integrate with Shopify's own dispute system.
2. **The free baseline is stronger than assumed.** Shopify already auto-fills
   basic evidence and auto-submits on the due date, including AI insights for
   "not received". Stripe's Smart Disputes auto-counters some disputes at a
   30% success fee.
3. **Competitors serve small sellers too.** Chargeflow serves 20k+ Shopify
   merchants at 25% of recovered with no minimum. The site's claim that such
   tools are only for large merchants was inaccurate and was flagged for
   removal.

What changed as a result:

- Shopify-first, integrating with Shopify Payments disputes
- The wedge narrowed to disputes needing evidence from **outside** the payment
  record: customer emails, the listing as it looked at purchase, packing
  photos, return label status, policy acceptance. Mainly "not as described",
  weak "not received", and "credit not processed".
- Undercut on price
- A platform constraint found: Shopify's API lets an app **save** dispute
  evidence but not **submit** it. Design response: the app saves the full
  package, the seller clicks submit, and Shopify auto-submits saved evidence
  on the due date if they do not.
- Build as a Shopify app with an in-admin extension, not a Chrome extension.
  A browser extension cannot detect disputes in the background, has no
  reliable data access and cannot bill.

Draw the conclusion explicitly: the research invalidated the core integration
assumption **before** any product was built, and the response was to re-scope
rather than proceed.

**08 — What exists and what does not.** A plain two-column table. Built:
pre-launch marketing site in Next.js and Tailwind, interactive four-tab demo,
waitlist capture, a standalone single-file version, tested on desktop and
mobile in light and dark mode. Not built: the real product. No Shopify
integration, no live dispute processing, no AI drafting on real merchant data,
no users, no revenue, no win-rate data.

One detail worth keeping: the waitlist never falsely claims an email was
saved. Without a backend configured it says signups are not open. Mention it,
because it is a small honesty decision in the product itself.

**09 — Where it stands and what I would do next.** As of October 2026 the
product is being reconceptualised and development restarted from zero. The
planned next steps: narrow the customer to a group reachable in practice,
collect ten real lost disputes, and test a simple paste-in reply writer on
them before building the app. An open question is whether to target Indian
merchants instead, where the payments background and network are a stronger
edge.

Ending on a restart rather than a launch is the honest version and it is
stronger than it sounds. Write it as a decision, not a failure.

**10 — Open the live demo.** ReconOS pattern, plus embedded screenshots of the
before/after hero and one of the four demo tabs mid-run.

---

## TASK 4 — Add: Found It

Position as venture building and commercial execution, not a college
entrepreneurship write-up. It is the only case on the site with real revenue
and real customers.

### Card

```
Title: Found It
Tagline: Sourcing from Jaipur workshops, selling at Gurugram stalls.
Description: A Jaipur-sourced D2C venture built with a Masters' Union team.
Sourcing, pricing, stall operations and corporate gifting, run on a weekly
P&L.
Tags: venture building, sourcing, unit economics, channel strategy, B2B sales
Headline stat: ₹6.94L revenue in three months at 10.1% EBT · Result
```

### Detail page

**01 — The venture.** Found It, a Jaipur-sourced D2C brand, July to September
2026, Gurugram. Co-launched with a Masters' Union team. Positioning: "Timeless
finds, rooted in Jaipur", Rajasthani craftsmanship reimagined for modern
living. Products: block-print makeup pouches and totes, suits and co-ords,
copper bottles, jewellery, snacks.

**Navya's role:** led sourcing, sales and people management. Built the P&L
template, the stall setup guidelines, and the design and quality-check
checklist. State this explicitly and separately from team outcomes.

**02 — Sourcing and pricing.** Landed-cost comparison across Jaipur workshops,
which set a 13.1% gross margin. Minimum order quantities, lead times, and the
inventory risk in buying depth versus variety. A design and quality-check
checklist, because craft sourcing has variable output.

Include a product-selection matrix only if it reflects criteria actually used
at the time. Do not construct one after the fact.

**03 — Channels, and the one that surprised us.** Stalls at Cyber Hub and IFC,
WhatsApp orders, and corporate gifting won through cold emails and catalogues.
**B2B reached 30% of orders at roughly ₹800 AOV.** Write that as the finding
it is: a venture set up to sell at stalls found a third of its orders came
from a channel reached by cold email.

**04 — Operations.** Stock tracking, order packing, Shiprocket fulfilment,
weekly P&L and cash tracking. The P&L workbook was the single source of truth
for every number on this page.

**05 — Results.**

| Figure | Value | Provenance |
| --- | --- | --- |
| Revenue, three months | ₹6,93,559 | Result |
| B2C | ₹2,59,659 | Result |
| B2B (corporate gifting) | ₹4,33,900 | Result |
| Gross margin | 13.1% | Result |
| EBT | 10.1% | Result |
| Average order value | ~₹800 | Result |
| Order split | ~70% B2C / 30% B2B | Result |

**06 — What I'd do differently.** Two or three real trade-offs: variety versus
depth, bulk rates versus unsold stock, time sourcing versus selling, footfall
versus cost of operating a location. Honest reflection, not every decision
framed as a win.

Then one thing worth building: the P&L was weekly and manual, so product-level
profitability and restocking by sales velocity were invisible. Propose that,
grounded in what was observed.

---

## TASK 5 — Add: Prolectro

A team GTM project. The product was **proposed, never commercially launched**,
and that must be clear on the page rather than buried. Lead with product
decisions; the campaign work is evidence of execution, not the headline.

### Card

```
Title: Prolectro
Tagline: Protein people will actually finish.
Description: A go-to-market strategy for a clear protein and electrolyte
sachet, built for the 23% who bought protein once and quit. Consumer
research through to pricing, launch gates and campaign assets.
Tags: consumer research, segmentation, product definition, pricing, GTM
Headline stat: 41 of 74 who tried protein powder had stopped · Sourced
```

### Detail page

**01 — The insight.** Open with the product mockup from the deck. India buys
the protein argument, not the habit: 93% say they watch their nutrition, yet
only 33% use protein powder today.

Three facts, labelled as project inputs rather than traction: 101 urban
respondents in a 12-question survey, September 2026 (**Sourced**); 23% are
Lapsed Intenders, the selected ICP (**Sourced**); ₹129 proposed single-sachet
price (**Target**).

**02 — Why people quit.** 55% of everyone who tried protein powder has walked
away: 41 of 74. 47% found shakes too thick or too filling; only 15% blamed
taste. 40% will pay ₹120 or more for a 36g serving, but only 6% reach ₹150,
where the closest rival sits.

The reading: they rejected drinking it, not the protein. The category acquires
people and cannot keep them.

State the limitation on the page: 101 urban respondents is directional
evidence, not proof of national demand.

**03 — The customer.** The Lapsed Intender, 23% of sample, n=23. Highest
willingness to pay of four segments: ₹125 mean against a ₹106 sample mean, 65%
will pay ₹120+, concept appeal 4.1 out of 5, 52% call shakes too thick.

The decision to explain: targeting people who already quit, rather than
competing for habitual users. Say which segments were rejected and why.

**04 — The product and the price.** Prolectro Refresh: 11g whey isolate plus
electrolytes in a 36g sachet, mixing clear in cold water. One hero SKU until
repeat is proven; Pro (20g) and Hydrate (5g) proposed later.

| Decision | Proposed approach |
| --- | --- |
| Trial | ₹129 single sachet on quick commerce |
| Repeat | 5 to 30-packs, ₹110 down to ₹90 a sachet |
| Distribution | Quick commerce and sampling first |
| Expansion | Broader channels only after repeat behaviour holds |

Unit economics: ₹129 nets ₹109 after GST; landed cost ₹42 to ₹55 leaves a 50
to 61% gross margin before channel and marketing. Label **Modelled**.

**05 — The launch, and the gate.** Delhi NCR first. Bengaluru only when repeat
holds above 35%. Six-week pre-launch with a live waitlist page, teaser film
and 25 to 30 creators; soft launch on Blinkit and Zepto at week 0; growth
levers from month 3.

The strongest decision in the deck is the gate: prove repeat before expanding
geography. Call it out.

Six-month figures are **Modelled targets, not results**, labelled wherever
they appear: 18,750 buyers in Delhi NCR, 38% repeat, ₹189 CAC. Market sizing
likewise: TAM ₹15,960 Cr, SAM ₹3,670 Cr, SOM ₹26 Cr by year 3.

**06 — From strategy to artefacts.** Embed the real work rather than linking a
PDF: the waitlist landing page at `partner-project-glow.lovable.app` (embed a
screenshot, link it live), the teaser film (`TODO_TEASER_VIDEO`), product and
pack design from the deck, and a small selection of campaign creatives.

**07 — My contribution.** `TODO_NAVYA_PROLECTRO_CONTRIBUTION`

A five-person team: Anushka Mittal, Ishani Handa, Yash Dhariwal, Navya Juneja,
Himanshu Mataray. Leave the placeholder until Navya specifies what she owned.
Do not guess.

**08 — What remains unvalidated.** The concept was a GTM project, not a live
launch. The credible next step validates taste, willingness to repurchase at
₹129, and actual contribution economics with a small pilot.

---

## TASK 6 — Fix the ₹2,000 worked example in the hero

The "Follow the money" illustration shows a 2.35% fee producing a ₹91.80
deduction on ₹2,000. 2.35% of ₹2,000 is ₹47, and ₹55.46 with 18% GST on the
fee. ₹91.80 is 4.59%, which the page does not explain.

Find the calculation, work out what it is doing, and either fix the arithmetic
or label the extra components so the figures reconcile on screen. Also check
the ₹0 shown at ₹100 Cr GMV, which looks like an unfilled placeholder.

This sits in the hero of a payments portfolio and is the single most damaging
error on the site. Report what you find before changing it.

---

## TASK 7 — Update the Capabilities matrix

After four additions and one removal, revisit the matrix. These should fill
cells the existing cases do not:

- MID Procurement: built a working system, measured it against ground truth,
  documented its own failure modes
- DisputeDesk: product discovery and idea selection, research that invalidated
  a core assumption pre-build, business model design
- Found It: real revenue and customers, sourcing and channel decisions, P&L
  ownership
- Prolectro: primary consumer research, segmentation, pricing architecture

Leave cells genuinely empty where there is no evidence. An honest gap is worth
more than a filled row.

---

## Finally

Run the build and lint. Report what changed, and list every `TODO` placeholder
left in the code so none ship to production.
