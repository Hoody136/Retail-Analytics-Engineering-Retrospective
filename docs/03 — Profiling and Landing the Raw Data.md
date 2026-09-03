03 — Profiling and Landing the Raw Data

Framework sources: Kimball, The Data Warehouse Toolkit (Ch. 19–20, data profiling subsystem)
What the books say

Kimball treats data profiling as a formal subsystem of the architecture — not an optional tidy-up, but a diagnostic audit that runs before you commit to a design. And it runs at two distinct moments:
Strategic profiling — a light assessment during requirements gathering. Is this source suitable at all? An early go/no-go before resources are committed.
Tactical profiling — a deep investigation during data modeling, before the source-to-target mapping is written. Types, distributions, relationships, rules.

The checks themselves follow Jack Olson's taxonomy, in three progressive tiers:

Column screens — single-field integrity: null incidence, distinct value counts, min/max boundaries, type verification, format adherence
Structure screens — relationships: foreign key to primary key integrity, hierarchy rollups, consistency across grouped fields
Business rule screens — contextual logic: multi-field dependencies, time-sequence rules (order date ≤ ship date), aggregate thresholds

The book's warning on skipping this is blunt: unprofiled data means "dirty" data discovered mid-build, impromptu cleaning systems, missed milestones, and over-promised schedules.

On the raw layer itself, the principle is immutability with a chain of custody: land the data exactly as it arrived, never overwrite it, and be able to link any transformed record back to its time-stamped raw archive — like evidence. The book also recommends staging to disk after each major pipeline activity: after extraction, after cleaning, after delivery.

What I actually did

The sequence, honestly:

1. Consolidated the range sheets. Each season lived in a separate file with a slightly different format. I merged them into one clean, consistently formatted master sheet — painstakingly, column by column — because everything downstream depended on it. Cost price, RRP, category, season, SKU: none of it exists in Shopify.
2. Played with the Shopify exports in Google Sheets. Before touching BigQuery, I loaded the orders export into a sheet and explored. This is where the real profiling happened.
3. Uploaded raw to BigQuery via CSV. Untouched — no cleaning before landing. The raw tables are exactly what came out of the exports.
4. Built a sample-sale normalisation tool. The third-party sample sale data arrived in no fit state to merge with Shopify orders. I built a simple Google Sheet with lookups: the team pastes raw sample sale data in one tab, and an upload-ready version appears in another — standardised to the Shopify order structure, with "sample sale" flagged in the discount code field so it stays drillable downstream.
5. Staged in dbt/Dataform, with cleaning and typing happening in the staging layer — not before landing, not after marts.

So the order was right (profile before build, land raw untouched), even though I didn't know the book's names for any of it at the time.

Strategic vs tactical: I did one and skipped the other

The book's two-moment model describes my project uncomfortably well.

My Google Sheets exploration was strategic profiling, textbook style: a light, early assessment during requirements that shaped scope and caught the big structural issues before any build. It worked.

The tactical phase — the deep, column-level audit the book schedules during modeling — I never ran. And the bill arrived on time: the failed BigQuery loads, the days of debugging, the SAFE_CAST scar tissue in every staging model. The book predicts exactly this failure mode: skip tactical profiling and you discover dirty data mid-build, reactively, at the most expensive possible moment.

The four discoveries, mapped to Olson's taxonomy

This is the part I'd point a reviewer at. The Sheets exploration caught four problems, and each maps onto the audit's tiers:

Table
Discovery	| Olson tier | Consequence
Discount codes apply at basket level, not line level	| Business rule |	Line-item margins are wrong unless discounts are apportioned down — solved in int_orders_filled
Shipping costs also only exist at basket level	| Business rule	| Same apportioning problem, same family of fix
Partial returns aren't attributed to specific lines	| Column screen (absence/null pattern)	| Allocation assumptions in SQL; acceptable at ~1% of transactions, but stated as an assumption
No category, cost, or RRP in Shopify	| Structure (missing relationship)	| No profit visibility from raw data at all — the range sheet is the enrichment donor, and its maintenance became a stated dependency

These four were the architecture. The layered design of the pipeline — the header/line split, the apportioning logic, the enrichment joins — is a direct consequence of this list.

The honest part: how the SAFE_CASTs got there

Look at my staging models and every single cast is a SAFE_CAST. That's not a style choice I read about — it's scar tissue.

The first loads into BigQuery failed. A lot. Currency fields with text in them, dates in three formats, numbers stored as strings, the occasional column where the range sheet had been edited by hand and a formula had leaked into a value. Each failure meant going back through the error, finding the offending values, understanding why they existed, and fixing the approach — with AI doing a lot of the debugging legwork with me.

Two things I'd take from that experience:

The failures were the profiling. I'd done the business-level profiling in Sheets (the four discoveries above), but I hadn't done the column-level profiling the book describes — type distributions, format checks, null audits. I paid for that in failed builds. A systematic column audit upfront would have found in an afternoon what I found over days of errors.

SAFE_CAST is the right response, but it should be paired with testing. A safe cast turns a bad value into a NULL silently. That's correct for keeping pipelines alive, but it means bad data now flows through invisibly. The book-correct version — which I'd implement next time — is a safe cast plus a test that counts how many NULLs each cast produced, so silent degradation becomes visible.

What landed well

Raw immutability. The raw tables are the exports, untouched — the book's "chain of custody" holds: any number in any report can be traced back to the exact file that landed. I rebuilt everything downstream from scratch several times, and could.

What I didn't do: stage to disk after cleaning and delivery, as the book recommends — my staging layer is views over raw, not persisted snapshots. And no lineage metadata on the raw uploads (which CSV, when, loaded by whom). For a two-person project this was fine; under any compliance regime it wouldn't be.

The sample sale sheet as an interface. The book talks about "data quality screens" at the boundary. My version was a Google Sheet with lookups that the commercial team could actually operate. Not sophisticated, but it put the fix in the hands of the people who owned the data, and it standardised the input at the point of entry rather than downstream.

The range sheet conversation. Telling Robena explicitly that the integrity of every report depended on those sheets being maintained — SKUs current, no duplicates, quantities up to date — was a data governance conversation before I knew the term.

What I'd do next time

Run the tactical audit the book schedules — a scripted pass over every field before the first load: type distribution, null count, distinct values, format samples, min/max. One afternoon, in place of days of failed builds.

Pair every SAFE_CAST with a NULL-count test so silent failures surface.

Write the four discoveries into a data profiling report at the time, not retrospectively. Two of them (basket-level discounts, partial returns) became architectural decisions — they deserved a document, not a memory.

Version the range sheet. It was a single point of failure maintained by hand. Even a simple change log would have made it auditable.
