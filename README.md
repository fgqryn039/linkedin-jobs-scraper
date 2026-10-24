# linkedin jobs scraper: scrape LinkedIn job listings without login or blocks — how it works, the 1,000-result cap, and what it actually costs

Most people searching for a linkedin jobs scraper want the same three things: extract job title, company, location and listing URL from a LinkedIn search; do it without logging into an account; and avoid getting their IP blocked after fifty requests. The good news is that LinkedIn exposes a public "guest" endpoint that returns exactly this data as plain HTML, no credentials needed. The catch is that LinkedIn rate-limits and IP-blocks that endpoint aggressively, so the real question isn't how to parse the page — it's how to keep requests flowing at scale. That's where a scraping API like ScraperAPI comes in: it routes your requests through a rotating proxy pool of 40 million+ IPs and handles CAPTCHAs and bans for you, at a documented cost of 30 API credits per LinkedIn request.

This guide covers how the method actually works, where LinkedIn hard-caps you at 1,000 results per search (a limit almost no comparison article mentions), what ScraperAPI's plans cost in real LinkedIn-scraping terms, and when a free open-source script is genuinely enough.

> **Quick pick:** if you scrape job listings occasionally, ScraperAPI's Hobby plan at **$49/month** covers it — 100,000 credits, which works out to roughly 3,300 LinkedIn requests, or about 83,000 job listings. The free plan gives you 1,000 credits every month forever, plus 5,000 bonus credits during your first 7 days, no credit card required. [👉 see ScraperAPI's current plans and pricing](https://www.scraperapi.com/pricing?fp_ref=coupons)

## How a LinkedIn jobs scraper actually works

Nearly every LinkedIn jobs scraper — the free GitHub scripts and the paid services alike — uses the same method: hit LinkedIn's public jobs search as a guest, then call the hidden AJAX endpoint that powers infinite scroll.

Open a LinkedIn jobs search in a private browser window and you'll notice there's no "next page" button and the URL never changes. Scroll down with Chrome DevTools' Network tab open, and you'll see the site fetching new results from this endpoint:

`https://www.linkedin.com/jobs-guest/jobs/api/seeMoreJobPostings/search?keywords=...&location=...&geoId=...&start=0`

That request returns raw HTML where each job sits inside an `<li>` element with predictable selectors:

- Job title: `h3.base-search-card__title`
- Company: `h4.base-search-card__subtitle`
- Location: `span.job-search-card__location`
- Listing URL: `a.base-card__full-link`

Because it's a guest endpoint, you don't log in and you stay clear of private data — which matters for the legality question below. A minimal scraper really is a few lines of Python (`requests` + `BeautifulSoup`) or Node.js (`axios` + `cheerio`).

One thing to know before you build: the guest search cards don't show salary ranges. Documentation from other API providers working with this same endpoint confirms pay information simply isn't on the search cards — it lives on the individual job pages when LinkedIn shows it at all. If salary data is essential to your project, you'll need a second request per listing, which doubles your cost. The cards do carry posting dates, "Actively Hiring" style badges and applicant signals in their markup, so date-based filtering (for example, LinkedIn's `f_TPR` parameter for past day/week/month) can be baked into the search URL itself.

The volume problem is where free scripts break down. LinkedIn's guest pages apply IP-based rate limiting, and once your home IP gets flagged, every subsequent request fails. ScraperAPI solves this by proxying the request: you prepend `http://api.scraperapi.com?api_key=YOUR_KEY&url=` to the target URL, and each call goes through a fresh IP with automatically matched headers. Failed requests are retried for up to 60 seconds before returning an error, and average latency runs 4–12 seconds with a reported success rate around 98%.

## Is scraping LinkedIn job listings legal?

For public, no-login data, ScraperAPI's own tutorial takes the position that scraping LinkedIn pages is legal, citing the 2019 LinkedIn vs. HiQ case, where the Ninth Circuit held that scraping publicly accessible data doesn't violate the CFAA. The tutorial notes the decision "remains good law."

The full picture: LinkedIn appealed to the US Supreme Court, which has not ruled, so this area remains unsettled. The practical safe zone is what the guest endpoint gives you — publicly visible job listings, no account, no private profiles, nothing behind a login. If your project drifts toward authenticated data or personal profiles, the risk profile changes completely, and that's outside what this guide covers.

## What it actually costs: the credit math most comparisons skip

Here's the detail most tool roundups leave out: **a LinkedIn request on ScraperAPI costs 30 API credits**, not 1. A standard page on an easy site costs 1 credit; Amazon costs 5; Google and Bing cost 25; LinkedIn costs 30 because of its anti-bot protection. Sites behind Cloudflare, DataDome or PerimeterX add another 10 credits per request when the bypass kicks in — automatically, with no way to opt out.

So the "100,000 credits" on the Hobby plan doesn't mean 100,000 LinkedIn requests. It means roughly 3,333 of them. Since each page of guest results returns up to 25 listings, that's around 83,000 job postings per month — plenty for tracking a handful of searches, clearly not enough for continuously monitoring every posting in a large metro area.

Three ways to keep the burn rate down:

1. **Skip JavaScript rendering.** The guest jobs endpoint returns plain HTML, so `render=true` is pure waste: a rendered request costs 10 credits on its own, render + premium proxy is 25, and render + ultra premium is 75. Note that these surcharges don't stack additively — premium + render is +25 total, not +20 — and `ultra_premium=true` alone adds +30. For LinkedIn's guest endpoint you need none of it; stay at the base 30 credits.
2. **Check costs before you run.** The dashboard's Domain Cost Estimator shows the exact credit cost of any URL, and a `max_cost` parameter caps what any single request can spend, so a misconfigured scrape can't drain your balance.
3. **Know what happens at zero.** Credits don't roll over — they reset at each renewal. If your usage is bursty, set the 95% usage auto-renewal in the dashboard instead of losing the tail of your balance. On Hobby through Professional, an opt-in auto-upgrade moves you to the next tier mid-month (usually at better per-credit pricing); Scaling, Professional, Advanced and Enterprise support pay-as-you-go continuation at a fixed per-credit rate, capped by a monthly spending limit you set.

Want to verify the math yourself before paying anything? [👉 claim ScraperAPI's 7-day free trial](https://www.scraperapi.com/?fp_ref=coupons) — 5,000 API credits, no credit card, which at 30 credits per request is about 166 LinkedIn pages, or 4,000+ job listings, before you spend a cent. If the service disappoints after you do pay, there's a documented 7-day, no-questions-asked refund policy.

## ScraperAPI plans and pricing

All plans share the same feature set (JS rendering, premium proxies, CAPTCHA handling, JSON auto-parsing); they differ by credits and concurrency. Annual billing takes 10% off across the board. These are the plans shown on the official pricing page:

| Plan | Monthly | Annual (per mo.) | Credits / month | Concurrent threads | Subscribe |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | $0 | 1,000 | 5 | [View plan](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [View plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [View plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [View plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [View plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [View plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [View plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [View plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |

Two limits don't show up in the headline numbers. Hobby and Startup only support US and EU geotargeting; country-level targeting for the rest of the world starts at Business via the `country_code` parameter. And the billing docs list a separate Student tier whose pricing isn't published on the main pricing page — if you're a student, ask support before assuming Hobby is your floor.

Concurrency has its own ceiling per plan. Requests beyond your plan's concurrent-thread limit return a 429 and are simply turned away — not queued, not charged. ScraperAPI also doesn't accept batch requests, so you can't bundle multiple target URLs into one call to save credits.

## The pagination cap almost nobody mentions: 1,000 results per search

Before you design a scraper around the guest endpoint, know its hard ceiling. ScraperAPI's tutorials, tested against live pages, document these behaviors:

- The `start` parameter increases by 25 for each new page of results.
- The last working page is currently `start=975`; hitting 1,000 returns a 404 page.
- Nudging `start` by 1 or 2 doesn't give finer control — it just hides earlier listings.
- The `pageNum` parameter, despite sitting in every URL, does nothing.

The 1,000-result cap isn't just one company's observation — technical documentation from at least one other scraping API provider confirms the same behavior, with LinkedIn rejecting any `start` offset at or above 1,000. So a single search query yields roughly 1,000 job listings maximum. For a niche role in one city, that's plenty. For "all software engineering jobs in the United States," it isn't: you'll need to split queries by narrower keywords, locations or `geoId` values to get complete coverage. This cap shapes how many credits a "complete" dataset costs regardless of which tool you use, so plan your crawl around it.

## When a free open-source scraper is enough

GitHub has several working linkedin-jobs-scraper projects that hit the same guest endpoint with nothing but Python's standard library and `curl`, no paid API. If you're pulling a few dozen listings once a week for your own job search, one of these scripts costs nothing and works fine — until LinkedIn throttles your IP, at which point you'll be back reading the proxy section.

The honest trade-off: free scripts have no proxy rotation, no CAPTCHA handling, and no maintenance guarantee. If you want richer fields with zero code, hosted no-code scrapers like Apify's LinkedIn Jobs Scraper extract salary ranges, applicant counts, seniority and employment type into Google Sheets or S3 exports; Zenrows offers a comparable scraping API with its Developer plan reported at $69/month. Those fit teams that want scheduler-driven structured exports. If your goal is specifically reliable volume on the jobs endpoint, a proxy API gives you the most control per dollar, because you keep your own parsing logic and only outsource the blocking problem.

## Frequently asked questions

**Do I need a LinkedIn account to scrape job listings?**
No. The jobs-guest endpoint serves public listings to logged-out visitors, and every approach covered in this guide is built on that. Staying logged out also keeps you clear of private data.

**How many LinkedIn listings can I get on the free plan?**
The free plan gives 1,000 credits per month, and LinkedIn requests cost 30 credits each — about 33 requests, or roughly 800 job listings, at up to 5 concurrent connections. Your first 7 days add 5,000 bonus credits for larger-scale testing.

**What happens if my requests get blocked anyway?**
ScraperAPI rotates IPs automatically and retries for up to 60 seconds before failing. Expect 1–3% of requests to fail; those aren't charged. If failures exceed 10%, support will tune your setup for your target site.

**Can I cancel or get a refund?**
Yes on both. Cancel anytime from the dashboard at no charge, and the 7-day no-questions-asked refund policy covers you if the service disappoints. There's also a permanent free tier, so you never have to subscribe blind.

## Which setup should you pick

- **A few hundred listings, once in a while, for your own job search:** a free GitHub script on the guest endpoint is genuinely fine. Don't pay for anything yet.
- **Recurring monitoring of one or a few searches, a few thousand listings a month:** ScraperAPI Hobby at $49/month, or $44.10 on annual billing, covering roughly 3,300 LinkedIn requests. [👉 check the Hobby plan's current price](https://www.scraperapi.com/pricing?fp_ref=coupons)
- **Structured, code-free exports with salary and applicant data:** a hosted scraper like Apify's is the better fit for that field set — just compare its per-result pricing against your volume before committing.
- **Continuous large-scale monitoring across many countries:** Startup ($149) or Business ($299) for country-level geotargeting, with queries split under the 1,000-results cap.

One last check before you subscribe: run your actual target query through the free tier and multiply requests × 25 to confirm the listing volume matches your expectations — that single number tells you whether Hobby suffices or you should budget a tier up. Then [👉 start with the 5,000-credit free trial](https://www.scraperapi.com/?fp_ref=coupons) and upgrade only if the math holds.
