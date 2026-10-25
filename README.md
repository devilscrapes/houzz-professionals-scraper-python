<p align="center"><a href="https://apify.com/DevilScrapes/houzz-professionals-scraper"><img src="assets/banner.svg" alt="Houzz Professionals Scraper — Python example" width="100%"></a></p>

# Houzz Professionals Scraper in Python: contractor and designer leads for pandas

<p align="center">
<a href="https://apify.com/DevilScrapes/houzz-professionals-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="Python" src="https://img.shields.io/badge/Python-3.11%2B-3776AB?logo=python&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/houzz-professionals-scraper-python/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/houzz-professionals-scraper-python/actions/workflows/ci.yml/badge.svg"></a>
</p>

This repository is a compact, tested Python wrapper around the **[Houzz Professionals Scraper](https://apify.com/DevilScrapes/houzz-professionals-scraper)** on Apify. Choose one trade, for instance `general-contractor` or `interior-designer`, add a `City, ST` filter if you only care about one market, and the script hands back the matching Houzz pros as a table: business name, trade, city and state, phone, star rating, review count, and, from each pro's own page, the license number, website and number of portfolio photos. The crawl happens on Apify's servers, so your laptop never requests a Houzz page. What you keep locally is `results.json` and `results.csv`, shaped for a DataFrame, a notebook or the CRM import you run on Mondays.

> **Actor:** [Houzz Professionals Scraper](https://apify.com/DevilScrapes/houzz-professionals-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in JavaScript:** [houzz-professionals-scraper-javascript](https://github.com/devilscrapes/houzz-professionals-scraper-javascript)

## 🔥 What you get

- **Twenty trades selectable by slug**: general contractors, architects, interior designers, roofers, HVAC, plumbing, solar installers and more, each mapped to the directory page Houzz itself uses for that trade.
- **A city filter you can reason about**: `location` keeps only rows whose city, and optionally state, equal what you typed, ignoring case. When the page cap arrives before enough matches, the run's status message reports the shortfall in plain numbers.
- **Optional second pass per pro**: with `fetchProfileDetails` on, every kept professional's own Houzz page is opened to add `license_number`, `website` and `project_or_photo_count`; switch it off and those three stay `null` at a lower price.
- **One row per Houzz ID**: the numeric `houzz_professional_id` deduplicates pros that show up on more than one listing page, so a contractor is never billed twice in one run.
- **Blank means blank**: a badge, rating or license number that Houzz does not show stays `null`. `average_rating` is empty for pros with zero reviews instead of a misleading 0.
- **Loud failure over quiet emptiness**: if every listing page fails, or pages load but contain no professionals at all, the run ends as failed instead of reporting success with nothing in it. Throttled or flaky pages are retried a few times with growing pauses first.

## 🎯 Use cases

### Score a trade before a sales team calls it

Pull one category, then let pandas compute what share of pros list a license, how many reviews the median firm has and who carries a phone number. Reps get a call sheet sorted by evidence, not by the order Houzz happened to display.

### Compare markets for a franchise or acquisition thesis

Run the same trade with different `location` values and stack the CSVs. Counts of pros, rating spread and portfolio depth per city give an analyst a first read on where a roll-up or a new territory has room.

### Watch a category change month to month

A cron entry or a scheduled notebook can repeat the run each month and append the rows with their `scraped_at` stamp. Review counts that climb, new IDs that appear and licenses that get added all become a short `merge` away.

### Build a sample for home-services research

Survey designers, insurers and materials suppliers often need a list of real firms in one trade to sample from. Houzz IDs make the draw reproducible, and the profile link lets a researcher check any row by hand.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/houzz-professionals-scraper-python.git
cd houzz-professionals-scraper-python
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export APIFY_TOKEN=your_token_here
python main.py                      # uses input.json
python main.py my-input.json --out data --max-charge 2
```

The script starts the [Houzz Professionals Scraper](https://apify.com/DevilScrapes/houzz-professionals-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Stripped to the minimum, the integration is the snippet below. `apify-client` launches the Actor, waits for the finished status, and yields the dataset rows one at a time.

```python
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("DevilScrapes/houzz-professionals-scraper").call(run_input={'category': 'general-contractor', 'maxResults': 5, 'maxPages': 1, 'fetchProfileDetails': True})
for item in client.dataset(run.default_dataset_id).iterate_items():
    print(item)
```

## 📥 Input

`main.py` forwards `input.json` exactly as written. The bundled example asks for general contractors nationwide, reads a single listing page and stops at five profiles, so a first test costs a few cents. Houzz paginates its trade directory nationwide rather than by city, which is why a city filter can need more pages than a bare category run; raise `maxPages` when a smaller town comes back short.

```json
{
  "category": "general-contractor",
  "maxResults": 5,
  "maxPages": 1,
  "fetchProfileDetails": true
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `category` | string | yes | `"general-contractor"` | `general-contractor`, `interior-designer`, `architect`, `landscape-architect`, `landscape-contractors`, `kitchen-and-bath-remodelers`, `kitchen-remodelers`, `home-builders`, `electrical-contractors`, `plumbing-contractors`, `hvac-contractors`, `roofing-and-gutter`, `painters`, `handyman`, `carpenter`, `design-build`, `solar-energy-contractors`, `pools-and-spas`, `custom-closet-designers`, `tile-and-stone-contractors` |
| `location` | string | no | — | — |
| `maxResults` | integer | no | `200` | — |
| `maxPages` | integer | no | `20` | — |
| `fetchProfileDetails` | boolean | no | `true` | — |
| `proxyConfiguration` | object | no | `{"useApifyProxy": false}` | — |

## 📤 Output

Each item is a single professional. Fields that depend on the profile pass are `null` when `fetchProfileDetails` is off or that pro's page could not be read; the listing fields are still there. Phone numbers keep Houzz's own formatting, and ratings run from 0 to 5.

| Field | Type | Always present |
|---|---|---|
| `houzz_professional_id` | integer | yes |
| `name` | string | yes |
| `category` | string | yes |
| `city` | string or null | no |
| `region` | string or null | no |
| `phone` | string or null | no |
| `website` | string or null | no |
| `review_count` | integer | yes |
| `average_rating` | number or null | no |
| `project_or_photo_count` | integer or null | no |
| `years_in_business` | integer or null | no |
| `license_number` | string or null | no |
| `houzz_profile_url` | string | yes |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.02 |
| Result emitted | $0.002 |
| Profile page scraped | $0.0005 |

Prices as configured on 2026-08-06. The [Actor page](https://apify.com/DevilScrapes/houzz-professionals-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in Python

The pandas snippet below (`pip install pandas`) does two jobs: it summarises license coverage and review depth per state, then writes `call_list.csv` with only the pros that list a phone number, licensed firms first.

```python
import json

import pandas as pd

leads = pd.DataFrame(json.load(open("results.json", encoding="utf-8")))
leads["licensed"] = leads["license_number"].notna()
leads["rating"] = leads["average_rating"].fillna(0)

coverage = leads.groupby("region").agg(
    pros=("houzz_professional_id", "nunique"),
    licensed_share=("licensed", "mean"),
    median_reviews=("review_count", "median"),
)
print(coverage.sort_values("pros", ascending=False).round(2).to_string())

callable_leads = leads[leads["phone"].notna()].sort_values(
    ["licensed", "review_count", "rating"], ascending=False
)
keep = ["name", "city", "region", "phone", "website", "license_number", "review_count", "rating", "houzz_profile_url"]
callable_leads[keep].to_csv("call_list.csv", index=False)
print(f"{len(callable_leads)} leads with a phone number written to call_list.csv")
```

## ❓ FAQ

### Do I need a Houzz account or a Houzz Pro subscription?

No. The Actor reads the directory pages anyone can open in a browser. All you supply is an Apify API token; nothing about your Houzz account, if you have one, is involved.

### Why are there no email addresses?

Houzz does not publish them on directory or profile pages, so the dataset has no email column rather than a guessed one. Pair the `website` field with your own enrichment tool when you need inboxes.

### Can I collect a trade that is not in the category list?

Not yet. Houzz has roughly 78 trades and the input currently accepts the 20 listed in the schema. Any other value fails input validation before a single page is requested.

### Does it cover Houzz sites outside the United States?

No. The Actor reads the US directory on houzz.com only. Regional sites such as the UK or Australian Houzz are out of scope for this version.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/houzz-professionals-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Houzz Professionals Scraper on Apify Store](https://apify.com/DevilScrapes/houzz-professionals-scraper)
- [JavaScript version of this example](https://github.com/devilscrapes/houzz-professionals-scraper-javascript)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Professional License Lookup Scraper — DE + CO Roster Export](https://apify.com/DevilScrapes/professional-license-lookup-scraper) — cross-check a contractor's license against a state roster.
- [US Building Permit Leads Scraper (Socrata)](https://apify.com/DevilScrapes/us-building-permit-leads-scraper) — find who is pulling permits for new jobs in a city right now.
- [New Business Filings — US State Registry Leads Feed](https://apify.com/DevilScrapes/new-business-filings-leads-scraper) — catch trade businesses the week they register with a state.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/houzz-professionals-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Houzz.
