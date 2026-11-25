# Rotating proxy API: what a ScraperAPI request really costs, how long a sticky session lasts, and where Bright Data or Proxora wins (full plan table inside)

ScraperAPI sells exactly that: one HTTP endpoint that takes a URL, pushes the request through a rotating datacenter or residential pool, and returns the page HTML. No proxy list to download, no rotation logic to maintain. New accounts get 1,000 free credits without a card, and you're only charged for requests that succeed.

The part most comparison pages skip is that a request and a credit aren't the same thing. A plain HTML page costs 1 credit, a JavaScript-rendered page costs 10, and a rendered page pushed through residential proxies costs 25. That multiplier decides both which plan you need and whether the free tier can finish your test.

> **Quick pick.** For a scraper pulling plain HTML at a few hundred thousand pages a month, Startup (1,000,000 credits, 50 threads, $149/month) is the first tier that isn't cramped. If more than half your targets need JS rendering, read the credit math below before you buy, because 1,000,000 credits buys 100,000 rendered pages, not a million. 👉 [See the current ScraperAPI plan table](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Every ScraperAPI plan side by side

ScraperAPI runs seven paid tiers plus a free plan, and the differences that matter are credits, concurrent threads, and how far geotargeting reaches. Credits reset monthly; threads cap how many requests can be open at once.

| Plan | Credits / month | Concurrent threads | Geotargeting | Price |
| --- | --- | --- | --- | --- |
| Hobby | 100,000 | 20 | US & EU | — |
| Startup | 1,000,000 | 50 | US & EU | $149/mo, or $134.10/mo billed annually |
| Business | 3,000,000 | 100 | Country-level | — |
| Scaling | 5,000,000 | 200 | Country-level | $475/mo, or $427.50/mo billed annually |
| Professional | 10,500,000 | 300 | Country-level | — |
| Advanced | 21,500,000 | 500 | Country-level | — |
| Enterprise | 22,000,000+ | 500+ | Country-level | — |
| Free | 1,000 | 5 | US & EU | $0 |

Three details the grid doesn't show. Annual billing is offered on every tier, and the two annual prices confirmed here are Startup at $134.10/month ($1,609.20/year, $178.80 below the monthly rate) and Scaling at $427.50/month ($5,130/year, $570 below). Tiers without a figure in the price column are quoted on the live page, and those figures move. Analytics history also changes with the plan: 30 days on Hobby and Startup, unlimited from Business upward.

Going past 3,000,000 credits a month puts you in sales territory rather than self-serve checkout, which is why Business sits where it does. 👉 [Compare the tiers that unlock country-level targeting](https://www.scraperapi.com/pricing?fp_ref=coupons)

## The real cost of a request: credits per page type

A credit is not a page. ScraperAPI prices each request by what the API has to do to get you a clean response, and the multiplier compounds fast.

| Request type | Credits | Notes |
| --- | --- | --- |
| Standard page | 1 | Plain HTML |
| Amazon, Walmart, eBay | 5 | Per request |
| Google, Bing (incl. subdomains) | 25 | Per request |
| LinkedIn | 30 | Per request |
| Bot-protection bypass | 10 | Cloudflare, Turnstile, DataDome, PerimeterX/HUMAN, Akamai |
| `render=true` | 10 | JavaScript rendering |
| `premium=true` | 10 | Residential and mobile IPs |
| `screenshot=true` | 10 | Turns rendering on automatically |
| Premium + rendering | 25 | `premium=true` with `render=true` |
| `ultra_premium=true` | 30 | Paid plans only |
| Ultra premium + rendering | 75 | Paid plans only |
| `country_code`, `session_number`, `autoparse`, `keep_headers`, `output_format` | 0 | No extra cost |

Two habits follow from this table. First, escalation should be stepwise: send the request at 1 credit, add `render=true` only when the HTML comes back without the content you need, add `premium=true` when you're seeing blocks, and treat `ultra_premium` as the last resort at 30 credits. Sending everything with rendering on by default can turn a 1-credit scrape into a 10-credit one for pages that never needed it.

Second, if you only need readable text rather than raw HTML, `output_format=markdown` or `text` costs nothing extra and saves downstream parsing. Every response carries its cost in the `sa-credit-cost` header, so you can log actual spend per URL instead of estimating it. 👉 [Start on the free 1,000 credits and measure your own cost per page](https://www.scraperapi.com/?fp_ref=coupons)

## One endpoint, four parameters, and what each one does to your bill

Everything is controlled by query parameters on the request URL, which is why a rotating proxy API is a drop-in for an existing scraper: you keep your HTTP client and change the host.

**`render=true`** runs the page in a headless browser and returns the DOM after JavaScript executes. Costs 10 credits, and another 15 on top if `premium=true` is also set.

**`country_code=us`** picks the proxy location. This one is free, but it isn't available everywhere: Hobby and Startup are limited to US and EU proxies, and country-level choice requires Business or above. If your target only serves meaningful content to, say, Brazilian or Japanese IPs, the cheap tiers can't reach it.

**`session_number=123`** pins several requests to the same proxy. Any integer works, and sending a different integer starts a new session. No extra credits.

**`premium=true`** routes through the residential and mobile pool and rotates the IP after each request. Costs 10 credits and can't be combined with `ultra_premium`.

## How long a sticky session actually lasts

ScraperAPI sessions expire 15 minutes after the last usage, and the clock restarts with each request that carries the same `session_number`. That window is long enough for pagination, form flows, and most multi-step scraping, and it's the one number worth checking against other providers before you commit.

Bright Data is stricter in three documented ways. Its session idle time is 5 minutes, so a pause longer than that hands you a different proxy even with an identical session value. Two consecutive requests originating from different global regions (AMER, EMEA, APAC) also break the session. So does changing country, city, or any other targeting parameter between requests. If you'd rather fail than be silently rotated, Bright Data's `-const` flag returns HTTP 502 when the peer is unavailable.

Proxora exposes the lifetime as a parameter: residential sticky sessions run from 1 second up to 7 days (1–59 seconds, 1–59 minutes, 1–24 hours, 1–7 days), while mobile and datacenter cap at 240 minutes.

The practical read: if your flow has gaps longer than five minutes between steps, such as queue waits, human approval, or scheduled batches, Bright Data will move you to a new IP mid-flow and ScraperAPI's 15-minute window covers it. If you need an IP to hold for hours, neither default gets you there on its own, and Proxora's residential TTL is the one you can dial up. 👉 [Check which ScraperAPI tier fits your session-heavy workflow](https://www.scraperapi.com/pricing?fp_ref=coupons)

## Country, city or ISP: where geotargeting stops

ScraperAPI documents geotargeting at country level only, through `country_code`. There is no published city, state, or ISP selector, so any project that needs to look like a specific metro user will hit a wall here.

| Provider | Country | State / region | City | ISP |
| --- | --- | --- | --- | --- |
| ScraperAPI | Yes, Business and up (US/EU only on Hobby, Startup) | No | No | No |
| Proxora (residential, mobile) | Yes | Yes | Yes | Yes |
| Proxora (datacenter) | Yes | No | No | No |
| ProxyRotator | Yes | No | City on residential IPs | No |

Bright Data targets countries through a `-country` parameter and references city-level parameters in its session documentation; if city targeting matters to you, verify the exact syntax with the provider before building around it.

For SERP tracking, ad verification, and price checks that need a city-level view, Proxora's residential and mobile products are the only ones in this group with documented state, city, and ISP filters. One gotcha if you go that route: US residential states are passed as full lowercase names such as `california`, not `CA`, and an unrecognised code returns an empty list rather than an error.

## Credits, gigabytes and threads are not the same currency

Three vendors here sell three different units, and converting between them isn't possible from published data. ScraperAPI bills in API credits, Proxora bills per gigabyte of traffic (with a quote endpoint that returns the same per-GB rate you'd pay), and ProxyRotator sells concurrent threads starting at $24.95/month.

The only fair comparison is within one vendor's own unit. On Startup at $149/month for 1,000,000 credits:

- plain HTML at 1 credit: 1,000,000 pages, about $0.15 per 1,000
- JS rendering at 10 credits: 100,000 pages, about $1.49 per 1,000
- premium plus rendering at 25 credits: 40,000 pages, about $3.73 per 1,000

Same plan, same price, a 25x spread in cost per page depending on what the API has to do. Anyone quoting you a flat "cost per 1,000 requests" for ScraperAPI without knowing your parameters is guessing. Proxora's per-GB model flips the dependency to page weight and asset loading, and a thread-based plan caps parallelism rather than volume, so a slow target can leave paid threads idle.

## Free credits, concurrency limits and what happens at the ceiling

The free plan gives 1,000 credits and 5 concurrent connections with no credit card, which is 1,000 plain pages, 100 rendered pages, or 40 rendered pages through residential proxies. Run your twenty hardest URLs through it first, because those numbers decide whether the free tier answers your question or just delays the purchase.

Exceeding your concurrency limit returns HTTP 429 with no charge; the request is turned away rather than queued, so a retry loop with backoff is the fix, not an upgrade. Running out of credits pauses scraping until your renewal date unless you've opted into auto-upgrade, which is available from Hobby through Professional and moves you to the next tier automatically. Scaling, Professional, Advanced, and Enterprise also offer pay-as-you-go at a fixed per-credit rate up to a monthly cap you set, which is the safer option for a one-off large crawl.

## FAQ

**Is 1,000 free credits enough to test properly?**
It's enough for 1,000 plain HTML pages, or around 100 pages if your targets need JavaScript rendering. That's sufficient to validate success rates on a sample of your real targets, not enough to benchmark a large crawl. Budget for one paid month at Hobby if you need more.

**Rotating or sticky: which do I actually need?**
Rotating is the default and the right choice for high-volume list pages and search results. Sticky sessions make sense for paginated flows, logins, and any sequence where step two only works if step one came from the same IP. In ScraperAPI you get sticky behaviour for free by reusing the same `session_number`, up to 15 minutes after your last request.

**Will high concurrency reuse the same IP?**
Bright Data documents this explicitly: executing requests at a high rate or from multiple calling agents may result in reuse of the same proxy IPs. ScraperAPI doesn't publish a per-request uniqueness guarantee, so if your target rate-limits by IP, test at your real concurrency before scaling rather than assuming every request gets a fresh address.

**Do I need `premium=true` from day one?**
No. It costs 10 credits per request, ten times a standard page. Start without it and add it only for the domains that return blocks or empty bodies. Keeping that flag off for the 80% of sites that don't need it is the single biggest lever on your monthly bill.

## Which plan to buy, by scenario

Under 100,000 pages a month of plain HTML, and mostly US or EU targets: Hobby. A few hundred thousand pages: Startup. Above 3,000,000 credits a month, you're talking to sales, so work out your credit burn per 1,000 URLs before that call rather than after.

If a meaningful share of your targets render content with JavaScript, double the tier you think you need. The 10-credit multiplier is the most common reason people outgrow a plan in the first week. If you need city or ISP targeting, ScraperAPI isn't the tool, and Proxora's residential product is. If you want flat-rate concurrency at low volume, ProxyRotator starts at $24.95/month. If you're prototyping for free, Proxifly publishes a REST proxy API on GitHub with no stated limits or SLA, so treat it as a toy.

Before checkout, confirm three things: which parameters your target sites actually require, whether your geographies fall inside US and EU if you're buying a cheap tier, and how many concurrent threads your scraper will really open. Those three answers map to credits, geotargeting tier, and plan size in that order.

👉 [Pick a ScraperAPI tier with the free 1,000-credit trial first](https://www.scraperapi.com/?fp_ref=coupons)
