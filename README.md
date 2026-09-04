
From Delivery to Best Practice: A Retail Analytics Engineering Retrospective

I spent 15 years driving businesses with data before I ever built a data pipeline, as a Merchandiser. My job, was to ensure that I grow a part of the business profitably by ensuring we purchased the right stock, at the right time, at the right quantity, at the right price. I'd build the plan for a Season (Spring/Summer or Autumn/Winter) months before it would start using data. Then I would 'trade' it once the season started - responding to customer signals, making adjustments. It was intense, but I enjoyed it. I've worked in companies (eBay, Ralph Lauren, TK Maxx, Zalando, Puma) where the data foundations were excellent — where a meeting meant everyone looking at the same numbers, agreeing on the opportunity, and executing with confidence. And I've worked in companies where none of that existed, where leadership demanded answers from data that was never engineered to provide them. 

It's the difference between a business that runs on evidence and one that runs on argument.

That's why I do this work now. I'm passionate about empowering people to make data driven decisions, understand their business and ultimately make happy customers. I think it can be (not always) the difference between loving and hating your work. 

The project

In the middle of this year, I started working Fractionally (or, Freelance). I designed and delivered a complete analytics platform for a UK luxury department store — a fast-scaling business where the commercial director was losing three days a month to manual Shopify exports and spreadsheet reporting. The platform:

<img width="2613" height="1848" alt="Notes_260901_135840" src="https://github.com/user-attachments/assets/eaf97e7a-93b9-4f39-96de-80bbcd76e909" />

1. Raw Shopify and buying data/range sheets (enrichment) → BigQuery
2. Transformation and modeling in dbt/Dataform — staging, intermediate, and mart layers *dbt was initially used, but then I moved the project to Dataform to save the client the enterprise cost of dbt and to give them more flexibility.
3. Reporting in Looker dashboards — monthly trade reporting, season snapshots
4. Forecasting tools in Google Sheets, plugged directly into the data models, so the commercial team could plan pre-season buys and forecast sales without touching SQL

It's live, it works, it's used, and the client is happy.

Why this repo exists

That project was delivered largely on domain instinct. I'm a merchandiser by background — I've been responsible for €200m+ businesses — so I knew what the client needed before she finished explaining it. That worked. But it also meant I skipped steps, made undocumented decisions, and built some things in ways I wouldn't defend in a code review.

So this repo is me going back through the project properly, against the established literature — Kimball's Data Warehouse Toolkit, the analytics engineering canon, retail mathematics, and data storytelling — and asking: what did I get right by instinct, where did I get lucky, and what would I do differently?

It's a retrospective, not a rebuild. The code examples are real (anonymised), the mistakes are real, and the reasoning is the point.

Architecture


The framework
I structured the retrospective around the six phases of an analytics engineering project:

1. Understanding the business before touching data
2. Defining the metrics
3. Profiling and landing the raw data
4. Data modeling: grain, facts, and dimensions
5. Building the pipeline: dbt layering and conventions
6. Testing, documentation, and delivery


And the summary: what I'd do differently

If you read nothing else

**[one honest, specific lesson — the best one, once we know what it is]**
