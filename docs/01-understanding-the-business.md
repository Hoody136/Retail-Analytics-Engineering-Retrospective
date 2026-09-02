# 01 — Understanding the Business Before Touching Data

*Framework sources: Kimball, The Data Warehouse Toolkit Ralph Kimball & Margy Ross (Ch. 1–2, 17); Analytics Engineering with SQL and DBT Rui Machado & Helder Russa (Ch. 1–2)*

---

## What the books say

Kimball's lifecycle starts with business requirements, not data. Three rules:

1. **Anchor on business processes, not report requests.** A process is a verb the business does — selling, buying, forecasting. A report request is a snapshot of what someone wants today. Build for the process and the reports take care of themselves; build for the reports and you rebuild every quarter.
2. **Interview up and down the org**, then consolidate everything into a **findings document** — the written record of what the business needs, why, and what's feasible.
3. **Interleave business discovery with data-centric discovery.** Verify the source systems can actually support the requirements *before* scope is agreed.

The tools: a stakeholder interview question set, an **opportunity/stakeholder matrix**, a **source feasibility checklist**, and the findings document itself.

---

## The context

The client is a fast-scaling UK luxury retailer. Robena, the commercial director, is a fashion buyer by trade — a very good one, with intuition built over a long career. But she was wearing every commercial hat in the business, and the company had grown faster than its infrastructure. There were no formalised processes. Not because she was winging it, but because nobody had ever built them.

The visible symptom: at least three days every month spent manually pulling together reporting — internal decks for the owner, and sell-through summaries sent to her brand partners. Shopify exports with no product enrichment, copy-pasted into spreadsheets. Range plans scattered across seasons, each in a separate file with a slightly different format.

In her own words:

> "Shopify is notorious for having bad data attribution. I can't just export the data and run a pivot, slicing it how I want. I need to look up all the product data on my own sheets. It's a headache."

## How I ran discovery

Here's the honest version: I didn't run a formal discovery process. I'm a merchandiser by background — I've been responsible for €200m+ businesses — and Robena and I had worked on the same team earlier in our careers. We speak the same language. I knew roughly what she needed before she finished explaining it.

So discovery was a series of long, unstructured conversations, practitioner to practitioner. No interview scripts, no stakeholder matrix, no findings document. We then worked in an agile way from the start: I'd build something, Robena would give feedback, I'd go away and change it. The goal was never a perfect state — it was something useful, fast. An MVP.

What she needed, in her terms:

- **Trade the business.** See what's selling, across multiple timeframes — a week in isolation, but also that week against the full season. If discounting is heavy one week, what's the impact on full-season sell-through and profitability?
- **Drill from high level to SKU and size.** Selling patterns at the finest grain, not just brand-level summaries.
- **See the effectiveness of discounting.** Codes and markdowns, and what they do to profit.
- **Take action on the back of it.** Buy more of what's working with remaining OTB. Take markdown action on slow movers — while still seeing profitability, to understand the impact of the pricing decision.
- **Give brands visibility.** A sell-through summary she could send to brand partners without three days of manual work.
- **See the quality of her stock.** What she owns, what it's offered at live on the website, the offered discount, and the profit she'd make on that stock if it sold at that price.
- **Plan future buys.** She had no defined pre-season process and knew it needed to change.
- **Forecast sales and cash flow.** By month, by season.

One wrinkle worth naming: 95% of her data was Shopify. But periodically she uses a third party to run **sample sales** — quietly liquidating old, fragmented stock to a select customer group. That's customary in luxury fashion, and she wanted it tracked and reported on too. It arrived in no fit state to merge with the orders data, so I built a normalisation process for it.

## Where instinct covered for process — and where it didn't

Two things from this phase turned out to be worth more than a formal process would have caught:

**I profiled the data before committing to anything.** Before building, I consolidated the range plans into one clean file and played with the Shopify exports in Google Sheets — just to see how the data behaved. That caught the problems that shaped the whole architecture:

1. **Shopify applies discount codes at basket level, not line level.** Buy three products with a code and the discount sits on the subtotal — the line detail shows nothing. Any line-item margin reporting has to apportion that discount down, or every product-level number is wrong. Shipping costs had the same problem.
2. **Partial returns aren't attributed.** The returned *part* of a basket isn't identified in the export, so I had to make allocation assumptions in SQL. Acceptable, because partial returns were roughly 1% of transactions — but it's an assumption a reviewer should see stated, not discover.
3. **No enrichment in Shopify at all.** No category, no cost price, no RRP — which means no profit visibility from the raw source. The range sheets were the donor for all of it: category, sub-category, brand, gender, colour, size, cost, RRP, season, SKU, quantities ordered and received. The integrity of every downstream report depended on those sheets being maintained — current SKUs, no duplicates, received and on-order quantities kept up to date. I made that explicit to Robena from day one.
4. **Inventory and orders don't mix in one report.** With multiple order lines per style, adding inventory into an orders-based report means double-counted stock. So inventory got its own report. I didn't have the vocabulary for it then, but this is a *grain conflict* — the single most important concept in dimensional modeling, caught in a Google Sheet before a line of SQL was written.

**What I didn't do:** write any of it down. No findings document, no agreed success criteria, nothing in writing about scope. It worked because there were two of us and we trusted each other. If I'd been hit by a bus (god forbid), the rationale for every decision lived in my head.

---

## The questions — asked vs. not asked

Kimball's interview flow, mapped against what actually happened:

| Kimball's question | Did I ask it? | What happened |

| What are your responsibilities, where do you fit in the org? | Skipped — known | We'd worked on the same team. No icebreaker needed. |

| What are your key performance metrics? | Yes, implicitly | Sell-through, markdown, stock cover, profitability — shared vocabulary from day one |

| What do you *do* when a number moves? | Yes, implicitly | Markdown action, rebuys, OTB commitments — this shaped the deliverables |

| Walk me through the spreadsheets you rely on | **Yes — deliberately** | Found the 3-day monthly reporting cycle, the fragmented range plans, the manual exports |

| What's your 2–3 year vision for information? | **No** | Never asked the owner. The commercial director was the audience for everything; the owner consumed her deck second-hand |

| What's the measurable impact of better information? | Yes | ~3 days/month recovered. Later, a concrete cost call: moved the platform from dbt Cloud to Dataform because €100/month per seat wasn't justifiable for handover |

| What are your specific success criteria? | **No** | Deliverables agreed in conversation, never written down |

The two gaps are honest ones. On a single-stakeholder project the missing executive interview is survivable — but I was one relationship away from building for the wrong audience, and I got lucky that I wasn't.

---

## The frameworks, filled in retrospectively

### A — Opportunity/Stakeholder Matrix

*Book version: business processes as rows, business functions as columns, X where a function depends on that process. This project's reality: one primary audience, plus brand partners for a single report.*

| Business process | Robena (Commercial Director) | Brand Partners |

| Retail sales (online) | X | |

| Sample sales (3rd-party liquidation) | X | |

| Sell-through reporting | X | X |

| Inventory quality & profit opportunity | X | |

| Pre-season planning / OTB | X | |

| Sales forecasting & cash flow | X | |

| ~~Stock snapshot history~~ | *out of scope — phase 2* | |

| ~~Intake / inbound deliveries~~ | *out of scope — phase 2* | |

<img width="1848" height="2613" alt="1 Data warehouse toolkit - Raph Kimball_260902_143117" src="https://github.com/user-attachments/assets/eabefde7-7269-4790-8641-5b27a6295b1a" />

The struck-through rows are Kimball's prioritisation grid in action. Stock history had no capture process (Shopify stock is a live feed only) and intake lived in a messy multi-source spreadsheet — both were projects in themselves, and the goal was an analytics solution fast that could help drive data driven decisions. Scoping them out *was* the framework working; I just did it in conversation instead of on paper.

### B — Source Feasibility Checklist

*Book version: verify availability, grain, keys, and attribute quality before committing scope.*

| Check | What I found |
|---|---|
| Source availability | 95% Shopify. Sample sales via a third party — captured, but not ingestion-ready; needed a normalisation tool before it could merge with orders |
| Atomic grain | Line-item level, yes. But discount codes and shipping land at **basket level** — apportioning logic required |
| Partial returns | Returned part of a basket not attributed — allocation assumptions in SQL. Acceptable at ~1% of transactions, but stated as an assumption |
| Key integrity | SKU is the bridge between Shopify and the range sheets — and the range sheets arrived in different formats per season, in separate files. Consolidated first, before anything else |
| Enrichment | No category, cost, or RRP in Shopify — **no profit visibility at all** from the raw source. Range sheets were the donor for everything |
| History | Stock is a live feed only — no snapshots. All reporting designed around this: sell-through derived from season buy minus sales, not from stock history |
| Go/no-go | Trade reporting: go. Stock history: no-go for this sprint — no capture process existed |

That last row is the one I'd point a reviewer to. I didn't just find problems — I designed around a known data limitation and flagged it as future work. That's the book's go/no-go rule operating in real life.

### C — The Findings Document (what I'd produce next time)

The Kimball skeleton, filled in for this project — one page, not ten:

```markdown
# Findings Document — Luxury Retailer Analytics Platform

## 1. Executive summary
Fast-scaling luxury retailer. Commercial director losing ~3 days/month
to manual reporting for internal stakeholders and brand partners.
Initial scope: trade reporting, sell-through, inventory quality,
pre-season planning, sales & cash-flow forecasting.

## 2. Methodology
Series of extended working sessions with the commercial director;
hands-on profiling of Shopify exports and range plans in Google Sheets
before any build. Agile delivery: build, feedback, iterate (MVP-first).

## 3. Business process: Retail sales
- Justification: replace manual trade reporting; see what's driving
  sales and margin across any timeframe
- Metrics: net sales, gross sales, COGS, discount rate, margin %,
  sell-through
- Pain points: manual exports, no enrichment, basket-level discounts
  and shipping, inconsistent range plans
- Impact: ~3 days/month returned to commercial work
- Feasibility: high — line-item data available; apportioning logic
  required for discounts/shipping

## 4. Business process: Inventory quality
- Justification: assess stock quality and profit opportunity at live
  website prices
- Metrics: stock on hand, weeks of cover, offered discount, profit if
  sold at live price
- Feasibility: medium — live feed only, no history; kept in a separate
  report to avoid grain conflict with orders

## 5. Business process: Pre-season planning & forecasting
- Justification: no defined buying process; cash-flow visibility needed
- Metrics: OTB, intake margin %, forecast sell-through, monthly sales
  forecast, cash in/out (30% deposit, 70% two months after season start)
- Feasibility: high for planning templates; stock snapshots deferred

## 6. Prioritisation
Trade reporting first (highest impact, highest feasibility).
Stock history and intake tracking explicitly deferred — no capture
process exists; each is a project in itself.

## 7. Success criteria
- Monthly reporting cycle reduced from ~3 days to under an hour
- Sell-through summary sendable to brand partners without manual work
- Robena able to answer "what's selling, at what margin, at what
  discount" across any timeframe without asking anyone
- Pre-season plans built on actuals, not intuition alone
- Platform handover to a contractor completed, with documentation,
  at near-zero running cost
```

---

## What was deliberately left out

Two processes were named and excluded, with reasons on the record:

- **Stock snapshots.** Shopify stock is a live feed; there was no process capturing an end-of-week position. All reporting was designed around this — the forecast derives sell-through from the season buy minus sales, which is crude but good enough for the decisions it supports. Capturing weekly snapshots is a project in itself.
- **Intake tracking.** Inbound deliveries lived in a messy spreadsheet fed from multiple places. Most stock arrived on time; the bigger prize was trading decisions. Deferred.

Scope decisions with stated trade-offs. That's what "prioritisation" means in the book — I just did it verbally.

## What I'd do next time

Keep the conversations and the agile loop — they were the right method for a client this size, and the MVP framing is why the project shipped at all. But add the lightweight written layer:

- A one-page findings document (the one above) agreed before the build
- Written scope boundaries — including the explicit no-gos
- Success criteria in writing, so "is this working?" has an answer that isn't a feeling
- The data profiling notes written up, not left in a scratch spreadsheet

An afternoon's work. It changes nothing about the relationship, and everything about what happens when a second stakeholder joins — or when the client asks "why did we build it this way?" six months later.
