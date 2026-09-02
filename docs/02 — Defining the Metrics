# 02 — Defining the Metrics

*Framework sources: Retail Math-Made Simple (5th ed.); Lean Analytics (Part 2–3)*

---

## What the books say

Retail Math organises a commercial director's numbers into three families:

**Sales and margin.** Gross sales (top-line demand, before deductions), net sales (gross minus returns and allowances — "the amount you take to the bank"), and gross margin (net sales minus COGS). The classic trap: confusing **margin %** (margin ÷ selling price) with **markup %** (margin ÷ cost). Suppliers pitch in markup because it sounds bigger; retailers must plan in margin. Agreeing a "50%" in a negotiation without specifying which one is a 16-point error on a £40-cost, £60-retail item.

**Inventory productivity.** Sell-through % (units sold ÷ units received, within a defined window), weeks of cover (stock ÷ average weekly sales), inventory turnover (net sales ÷ average inventory), and GMROII (margin % × turnover — the single best measure of whether inventory is earning its keep). The traps: sell-through without a defined time window, and weeks of cover calculated off an anomalous sales week.

**Buying and planning.** Open-to-buy (planned EOM stock + planned sales + planned markdowns − BOM stock − on order) and intake margin (planned retail − cost, ÷ planned retail). The trap: omitting planned markdowns from OTB, which guarantees under-buying.

Lean Analytics adds the filter: a metric is only worth tracking if it changes what you *do*. Everything else is vanity.

---

## What I actually did

I presented Robena a first-pass KPI list of 41 metrics. She came back with wording changes. That was it.

I want to be precise about why, because it's the thesis of this whole repo: the list was right first time not because I followed a process, but because I've traded businesses with these numbers for years. I knew which metrics a commercial director acts on, because I'd acted on them. The books now confirm the list — but the list didn't come from the books.

The full list, grouped by the book's families:

**Buying and intake**
- Ordered @ Retail (RRP) £, Received @ Retail (RRP) £, To Be Received @ Retail (RRP) £, Received %
- Ordered @ Cost £, QTY Ordered #Units, #SKUs Ordered
- Ordered Average RRP £, Ordered Average Cost £, Ordered Average SKU Depth
- Margin %

**Trading (sales)**
- Previous Week Sales #Units, Previous Weeks Cover #Units
- Total Sales if @ RRP £, Total Gross Sales £, Total Gross Sales Mix %
- Full Price Gross Sales £, Full Price to Discount Sales Mix %
- Total Net Sales £, Total Shipping £, Total VAT £, Total VAT %
- Total Profit £, Total Profit %
- Total Sales Discounts £, Total Discount Rate %
- Total Sales #Units, Total Sell-Through % #Units
- Total Sales @ Cost £, Total Sales Average Cost £
- Average Selling Price Gross Sales £

**Returns**
- Total Returned Net Sales £, Total Returned #Units, Return Rate Unit Based %

**Stock**
- Stock On Hand Total #Units, Stock On Hand Value @ Cost £
- Stock On Hand Value @ Live Price £, Stock On Hand Value @ RRP £
- Gross Profit Potential of Stock (Before Taxes) %
- Live Discount Off Of RRP Offered %

---

## Where the books confirm the list — and where the list goes beyond them

**Confirmed, with formulas checked against Retail Math:**

- **Sell-through %** — book: units sold ÷ units received. My SQL computes lifetime units sold ÷ (units sold + stock on hand), which is the same thing expressed through what's measurable without stock history. The book's caveat — always bound it to a time window — is exactly why the model also carries previous-week, 4-week, and season-to-date versions.
- **Weeks of cover** — book: stock ÷ average weekly sales. Mine: stock ÷ previous week's units. The book warns against computing the run-rate off an anomalous week; a single previous week *is* a noisy denominator. Defensible for a fast trading view, but a 4-week average would be the book-correct version. Noted for next time.
- **Margin %** — correctly margin-on-selling-price throughout, never markup. The intake margin logic in the pre-season planner (average selling price before discount vs average cost) is the book's IMU.

**Beyond the books — these came from the business, not a text:**

- **Total Sales if @ RRP £** — the baseline of what the stock would have made at full price. Retail Math assumes you know your markdowns; this metric makes the *entire discount burden* visible in pounds, which is how a luxury retailer actually feels it.
- **Full Price to Discount Sales Mix %** — the books track discount rate; they don't track the *mix* of full-price vs discounted selling as a health indicator. In luxury, that mix is brand equity in numeric form.
- **Live Discount Off Of RRP Offered %** and **Gross Profit Potential of Stock %** — forward-looking metrics on *unsold* stock: what's currently offered on the website, and what the remaining stock would return if it sold at live price. The books measure what happened; these measure what's still on the table. These exist because Robena trades stock, not just sales.

**What the books have that the list doesn't — and why:**

- **GMROII and inventory turnover** — the book's "most powerful metric in buying" — are absent, and the reason is structural: both need *average inventory over time*, and Shopify stock is a live feed with no history (see Chapter 1 — stock snapshots were explicitly out of scope). You can't compute turns without a time series of stock. This is the clearest example in the whole project of a data limitation constraining the metric set — and why the stock snapshot project matters.

---

## The definition decisions a reviewer should see

Three judgment calls buried in the KPI list, stated plainly:

1. **Net sales is before returns in the model; returns are netted off in the reporting layer.** The fact table stores `net_sales_gbp` (after discounts, before returns) alongside `refund_amount_gbp` as separate columns. Looker does the subtraction. Reason: returns land later than sales, and keeping them separate preserves both the trading view (what sold this week) and the financial view (what we kept). The book defines net sales as after-returns — and the Looker layer delivers exactly that. The model just doesn't throw the information away on the way.
2. **Two gross sales baselines, deliberately.** `gross_sales_before_discount` is actual selling price × quantity; `gross_sales_if_at_rrp` is RRP × quantity. The gap between them is physical markdown; the gap between gross and net is checkout discount. Splitting these lets Robena see *where* value is leaking — ticket price vs basket code.
3. **Sample sale execution costs are inside profit.** The "true gross profit" figure deducts the third-party liquidation costs, not just COGS. The books don't have a line for this; a business that liquidates through sample sales needs one.

---

## What I'd do next time

1. **Write the KPI dictionary first, not last.** I built a KPI dictionary at handover — definitions, formulas, grain. It should have existed before the first model, as the contract between me and Robena. The definitions were stable (only wording changed), so writing them down early would have cost an hour and made every later conversation sharper.
2. **State the formula next to every metric, in the document, including the ones that seem obvious.** "Net sales" means three different things in three businesses. The one wording-level review Robena did would have caught a definition-level problem if there had been one.
3. **Fix weeks of cover to a 4-week average denominator**, per the book's warning about anomalous weeks.
4. **Name the missing metrics and why they're missing.** GMROII and turnover weren't oversights — they were blocked by the lack of stock history. Saying so in the dictionary turns a gap into a roadmap item.
