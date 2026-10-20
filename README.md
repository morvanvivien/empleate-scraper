# How to Scrape Empléate.gob.es: Spain's Official Public Job Board (Coordinates, Category, Full Description)

> **Run it now, no code: [empleate-jobs-scraper on the Apify Store](https://apify.com/bibian/empleate-jobs-scraper)** &nbsp;|&nbsp; $2.00 per 1,000 postings &nbsp;|&nbsp; no browser, no proxy

Empléate is Spain's official public employment service (SEPE/SNE) job board, aggregating postings from SEPE and regional partners (SOC, LABORA, and others). It has no public export for third parties. This guide shows how to get its postings as clean structured data: title, category, province, city, coordinates, contract type and full description, with a free Apify Actor, from the console or from code.

Searches this page answers: empleate scraper, SEPE job postings scraper, scrape Spain public job board, Spanish public employment data.

## What you get from the Empléate scraper

- Every active public posting, with title, category, subcategory, province, city and country.
- Latitude and longitude for every posting, ready for mapping.
- Contract type, work schedule, education required and a disability-friendly flag.
- Full description as plain text, read from the board's own public search API.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, personal data removed) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `sourceUrl` | Empléate posting id and reference link |
| `title` | Job title |
| `category`, `subcategory` | Empléate job category and subcategory |
| `region`, `province`, `city`, `country` | Where the job is based |
| `latitude`, `longitude` | Coordinates of the job location |
| `contractType`, `workSchedule` | For example `Indefinido`, `COMPLETA` |
| `positionsOffered` | Number of openings on the posting |
| `educationRequired` | Minimum education level, as published |
| `disabilityFriendly` | `true` when the posting is reserved for workers with a disability certificate |
| `sourceSystem` | Which partner published it (for example `SNE`, `TECNO_EMPLEO`) |
| `applyUrl` | Outbound application link, when the posting has one |
| `publishedAt`, `expiresAt` | ISO timestamps |
| `description` | Full text (only when "Include full descriptions" is on) |

## Use cases

- **Monitor the Spanish public job market.** Track new postings by category or province, updated daily.
- **Feed a job alert or an aggregator.** Run it on a schedule and dedupe by `id` to catch only the new postings.
- **Map job demand by region.** Every posting carries province, city and coordinates.
- **Track disability-friendly postings.** Filter on the `disabilityFriendly` flag.

## Input

| Field | What it does |
|---|---|
| `keywords` | Job title or skill, in Spanish, for example `programador` or `enfermera`. Leave empty to get every active posting. |
| `province` | Spanish province name as Empléate labels it, for example `MADRID` or `BARCELONA`. Leave empty for all of Spain. |
| `category` | Empléate job category, for example `INFORMÁTICA/TELECOMUNICACIONES` or `SALUD/DEPORTE`. Leave empty for all categories. |
| `publishedWithinDays` | Keep only postings published in the last N days. Use 1 for a daily monitoring run. |
| `maxItems` | The run stops once this many postings are saved (default 50). |
| `includeDescriptions` | Add the full job description as plain text. |

Example input:

```json
{
  "province": "MADRID",
  "maxItems": 50,
  "includeDescriptions": true
}
```

## Three ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/empleate-jobs-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the input form (keywords, province, category and the maximum number of results) and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/empleate-jobs-scraper').call({
    "province": "MADRID",
    "maxItems": 50,
    "includeDescriptions": true
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.category} | ${item.city}, ${item.province}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/empleate-jobs-scraper").call(run_input={
    "province": "MADRID",
    "maxItems": 50,
    "includeDescriptions": True,
})

for item in client.dataset(run.default_dataset_id).iterate_items():
    print(f"{item['title']} | {item.get('category')} | {item.get('city')}, {item.get('province')}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

## Pricing

Pay per result: **$2.00 per 1,000 postings**, plus a tiny start fee of $0.00005 per run. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/empleate-jobs-scraper) when this guide was written; the Store page is the reference.

## Limits and honest notes

- Empléate does not publish an employer name or a structured salary field for most postings: this is how the board itself works, not a gap in the actor. A salary figure sometimes appears inside the free-text description instead.
- A line that carries an office or application-portal e-mail address is dropped from the description: no contact data is collected.
- Most postings have no outbound application link (`applyUrl` is empty): applicants apply through Empléate itself or through the regional portal named in the description.
- This is an unofficial tool, not affiliated with SEPE or Empléate. Check the site terms and the data protection rules that apply to your use.

## FAQ

**Does it need a browser or a proxy?**
No. The Actor reads the board's own public search API directly, so runs are fast and cheap.

**Why don't I see a company name or a salary field?**
Empléate itself does not publish them for most postings; the full description sometimes mentions a salary figure in free text.

**How do I get only the new postings every day?**
Set `publishedWithinDays` to `1` and schedule the Actor once a day.

**Can I get the data in Excel or Google Sheets?**
Yes. Export the dataset as CSV, Excel or JSON from the Apify console, or connect the Apify Google Sheets integration.

## Try it

**[Open empleate-jobs-scraper on the Apify Store](https://apify.com/bibian/empleate-jobs-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
