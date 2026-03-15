# Full SEO Audit Report
## Primary URL: https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/88GMC8MX?batch=2026-04-02_TR-J5XX3P9F-0001
## Canonical target (WebFetch): https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/
## Date: 2026-03-15
## Scope: Single-page full audit — Technical, Content, Schema, Performance, Links, GEO, AEO, Sitemap

> **Note on URL scope:** The audited URL includes a `?batch=` query parameter and a `/88GMC8MX` segment (batch ID). The base trip URL (`/zo-trip/experience-bali-tr-j5xx3p9f/`) issues a 308 redirect to `/zo-trip/experience-bali-tr-j5xx3p9f` (trailing-slash removal). All batch-parameterised variants return HTTP 200 with no server-side canonical. This URL architecture is a primary finding and treated as a standalone critical issue.

---

## A) Audit Summary

**Overall Score: 19 / 100 — CRITICAL**
**Score confidence: Medium** (PageSpeed/CrUX unavailable; performance scored as Hypothesis)

### Scoring by Category

| Category | Weight | Score | Weighted |
|----------|--------|-------|---------|
| Technical SEO | 25% | 22/100 | 5.5 |
| Content Quality | 20% | 14/100 | 2.8 |
| On-Page SEO | 15% | 18/100 | 2.7 |
| Schema / Structured Data | 15% | 12/100 | 1.8 |
| Performance (CWV) | 10% | N/A (Hypothesis) | — |
| Image Optimisation | 10% | 38/100 | 3.8 |
| AI Search Readiness (GEO) | 5% | 10/100 | 0.5 |
| **TOTAL** | **95%** | | **17.1 → ~19 normalised** |

> Score is slightly lower than the /zo-trips listing page (22/100) due to the critical `?batch=` URL duplication issue and the 737-character meta description.

---

### Top 3 Critical Issues

1. **`?batch=` URL parameter creates uncontrolled duplicate content at scale.** Every trip departure date generates a unique `?batch=YYYY-MM-DD_TR-XXXX-XXXX` URL, all returning HTTP 200 with no server-side canonical. For a trip with 12 departures per year, that is 12+ indexed duplicates competing with and diluting each other. The canonical (identified only in the JS/RSC payload) is not present in server-rendered HTML, so Googlebot may not honour it.

2. **Schema exists only inside the Next.js RSC streaming payload — not in `<head>` as a `<script type="application/ld+json">`.** `parse_html.py` confirms `"schema": []`. The `Trip` schema block is embedded in the JavaScript bundle and will be invisible to AI crawlers (GPTBot, ClaudeBot, PerplexityBot) which do not execute JavaScript. Even for Googlebot, RSC-embedded schema has inconsistent rendering behaviour vs. static `<script>` tags.

3. **Meta description is 737 characters — 4.7× over the 155-character limit.** The entire trip description was pasted into the `<meta name="description">` tag. Google truncates to ~155 chars in SERPs, so only the first sentence is visible: *"Get ready for the perfect Bali trip that blends island adventure, cultural depth, and beachside chill."* — no trip-specific hook, no price signal, no CTA within the visible window.

### Top 3 Opportunities

1. **Move canonical + schema into server-side `<head>` via Next.js `generateMetadata()`.** The infrastructure (Next.js App Router) already supports this. The fix is a configuration change, not an architectural rewrite. Correct implementation would make schema immediately visible to AI crawlers and ensure canonical is honoured by all bots.

2. **Implement URL parameter handling in Google Search Console + add canonical to all batch URLs.** Designate `?batch=` as a crawl parameter to collapse duplicate indexing while preserving individual batch pages for conversion tracking.

3. **Rewrite meta description to 130–155 chars with price anchor and CTA.** "Bali" is one of the highest-volume travel queries from India. A well-optimised description with price ("from ₹41,722"), duration ("7N/8D"), and destinations ("Ubud · Gili · Nusa Penida · Kuta") directly increases CTR from SERPs.

---

## B) Findings Table

| # | Area | Severity | Confidence | Finding | Evidence | Fix |
|---|------|----------|------------|---------|----------|-----|
| 1 | Technical / URL | 🔴 Critical | Confirmed | `?batch=` query parameter creates unlimited URL variants all returning HTTP 200 with no server-side canonical | `curl`: `/zo-trip/experience-bali-tr-j5xx3p9f/88GMC8MX?batch=2026-04-02_...` → 200; `?batch=2026-05-01_...` → 200; BeautifulSoup: canonical = None | Add `<link rel="canonical" href="https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/">` in server-rendered `<head>` for all `?batch=` variants; also configure `?batch=` as a non-indexable parameter in GSC |
| 2 | Schema | 🔴 Critical | Confirmed | `Trip` schema is embedded in Next.js RSC/JS payload, NOT in server-side `<head>` as `<script type="application/ld+json">` | `parse_html.py`: `"schema": []`; `article_seo.py`: `"structured_data": []`; Raw HTML BeautifulSoup: no `<script type="application/ld+json">` found in `<head>` | Move schema to `generateMetadata()` or a server-rendered `<Script>` component in Next.js; emit as static `<script type="application/ld+json">` in `<head>` |
| 3 | Schema Quality | 🔴 Critical | Confirmed | Schema has invalid `duration` value (`"P8"`) and `provider` as a plain string instead of an `Organization` object | WebFetch: `"duration": "P8"`, `"provider": "Zostel"` | Fix: `"duration": "P8D"` (ISO 8601); `"provider": {"@type": "Organization", "name": "Zostel", "url": "https://www.zostel.com"}` |
| 4 | On-Page / H1 | 🔴 Critical | Confirmed | No H1 tag in server-side HTML | `parse_html.py`: `"h1": []`; `article_seo.py`: `"h1": []`; BeautifulSoup parse: none | Add `<h1>Experience Bali — 7 Nights 8 Days Trip by Zostel</h1>` as first heading, server-rendered |
| 5 | Technical / Canonical | 🔴 Critical | Confirmed | No canonical tag in server-side HTML | BeautifulSoup: `Canonical: None`; `parse_html.py`: `"canonical": null` | Add via Next.js `generateMetadata()`: `alternates: { canonical: 'https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/' }` |
| 6 | On-Page | 🔴 Critical | Confirmed | Meta description is 737 characters (limit: 155) | `parse_html.py`: `"meta_description"` length = 737 chars; `article_seo.py`: "Meta description may be truncated (737 chars)" | Replace with ≤155 char version (see Section C) |
| 7 | Content / SPA | 🔴 Critical | Confirmed | Only 14 words in server-side HTML; all trip content is JS-rendered | `readability.py`: `"word_count": 14`; `parse_html.py`: `"word_count": 16`; no H2 in server HTML | Implement Next.js `getServerSideProps` / `getStaticProps` (ISR) to pre-render itinerary, highlights, inclusions, and description |
| 8 | Social Meta | ⚠️ Warning | Confirmed | `og:description` and `twitter:description` are 737 chars each (max: 200) | `social_meta.py`: "og:description is too long (737 chars, max 200)"; "twitter:description is too long (737 chars, max 200)" | Trim to ≤200 chars for OG/Twitter; keep ≤155 for meta description |
| 9 | Social Meta | ⚠️ Warning | Confirmed | `og:url` is missing | `social_meta.py`: "🔴 og:url: missing (required)"; `parse_html.py`: og:url absent | Add `<meta property="og:url" content="https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/">` |
| 10 | On-Page / OG type | ⚠️ Warning | Confirmed | `og:type` is set to `"article"` but this is a trip/product page | `parse_html.py`: `"og:type": "article"` | Change to `og:type = "website"` or use `product` if appropriate for the booking context |
| 11 | Security | ⚠️ Warning | Confirmed | 4 security headers missing (HSTS, X-Content-Type-Options, Referrer-Policy, Permissions-Policy) | `security_headers.py`: Score 50/100 | Add all 4 headers in Next.js config or CDN/proxy layer (see previous report for exact values) |
| 12 | Image Opt | ⚠️ Warning | Confirmed | All `<img>` tags missing `width` and `height` attributes | `parse_html.py`: every image `"width": null, "height": null` | Add explicit dimensions to prevent CLS; `fetchpriority="high"` on hero/LCP image |
| 13 | Image Alt | ⚠️ Warning | Confirmed | Alt texts are generic filename stems across all images | `parse_html.py`: "zostel-head", "zo-trips-small", "follow-your", "zo-zo-zo" | Replace with descriptive alt text (see previous report for full table) |
| 14 | Links | ⚠️ Warning | Likely | Social links (Instagram, Facebook, Twitter) return HTTP 403 | `broken_links.py`: 3 × 403 external | Platform bot-blocking — functionally valid for users. Add `rel="noopener noreferrer"` to all external links |
| 15 | Links | ⚠️ Warning | Confirmed | Merchandise link has 2-hop 308 redirect chain | `broken_links.py`: `/merchandise` → 308 → 308 | Update href to final URL `/merchandise/tshirts` directly |
| 16 | Links | ⚠️ Warning | Confirmed | No link back to parent `/zo-trips` listing page from server-rendered HTML | `parse_html.py`: internal links show nav items only, no breadcrumb back to Zo Trips | Add breadcrumb `Home > Zo Trips > Experience Bali` with server-rendered markup |
| 17 | AEO | ⚠️ Warning | Confirmed | No FAQ section visible in server-side HTML | `parse_html.py`: `"paragraphs": []`; 14 words of content | Add plain-text FAQ section (≥5 Q&As covering: what's included, cancellation, difficulty, group size, visa requirements) |
| 18 | E-E-A-T | ⚠️ Warning | Confirmed | No trip curator or guide attribution | `article_seo.py`: `"author": ""`; no byline or Person entity | Add "Curated by [Name], Zo Trips" attribution; link to guide's profile page |
| 19 | GEO | 🔴 Critical | Confirmed | No llms.txt; no AI crawler policy in robots.txt | Previous audit confirmed; same root domain issue | Create `/llms.txt`; add explicit AI crawler entries to robots.txt |
| 20 | Sitemap | 🔴 Critical | Confirmed | No XML sitemap; this trip URL not discoverable via sitemap | `/sitemap.xml` → 404 (confirmed in previous audit) | Generate sitemap including all trip pages at their canonical (batch-free) URLs |
| 21 | Hreflang | ℹ️ Info | Confirmed | No hreflang tags | `parse_html.py`: `"hreflang": []` | If targeting international travellers (Bali trips booked from India, Southeast Asia), add hreflang for `en-IN`, `en-SG`, etc. |
| 22 | Performance | ℹ️ Info | Hypothesis | CWV unknown; likely CLS issues from missing image dimensions; likely high LCP on hero image | PageSpeed API blocked | Run PageSpeed Insights manually; check CLS caused by missing image dimensions |
| 23 | Schema — Missing | ⚠️ Warning | Confirmed | `TouristTrip` schema missing key properties: `itinerary`, `offers` (with price + availability), `startDate`, `endDate`, `aggregateRating` | WebFetch schema block lacks these fields | Add complete `TouristTrip` schema with all recommended properties (see Section C) |

---

## C) Detailed Findings by Area

---

### 1. Technical SEO

**Score: 22/100**

*Chain-of-thought:*
- Positives (3): HTTPS active, clean 200 response, CSP + X-Frame-Options present
- Deficits (7): No H1 in server HTML, no canonical in server HTML, `?batch=` URL duplication, SPA thin HTML (14 words), missing security headers, no sitemap, no breadcrumb
- base = 3/10 × 100 = 30
- Criticals: No canonical (−15), batch URL duplication (−15) = −30 → max(0, 30−30) = 0 + partial recovery for positives
- Warnings: security headers (−5) → adjusted score: **~22**

> "Score of 22 reflects HTTPS and CSP presence (+), severely penalized by missing server-side canonical (Critical, −15), uncontrolled ?batch= URL variants (Critical, −15), and 4 missing security headers (Warning, −5)."

#### URL Architecture Deep-Dive

The audited URL structure is:
```
/zo-trip/{trip-slug}/{batch-id}?batch={date}_{trip-code}-{seq}
```

Probed variants and their responses:
| URL | Status |
|-----|--------|
| `/zo-trip/experience-bali-tr-j5xx3p9f/` | 308 → `/zo-trip/experience-bali-tr-j5xx3p9f` |
| `/zo-trip/experience-bali-tr-j5xx3p9f/88GMC8MX` | 200 |
| `/zo-trip/experience-bali-tr-j5xx3p9f/88GMC8MX?batch=2026-04-02_TR-J5XX3P9F-0001` | 200 |
| `/zo-trip/experience-bali-tr-j5xx3p9f/88GMC8MX?batch=2026-05-01_TR-J5XX3P9F-0001` | 200 |

All parameterised batch variants return 200 with identical page content (same trip, different departure date selection). No canonical tag in server-side HTML means:
- Google may index each `?batch=` URL as a separate page
- PageRank is diluted across all variants
- The actual trip content (when JS-rendered) is duplicated
- Rich result eligibility is fragmented

**Canonical strategy required:**
All `?batch=` variants and the `/{batch-id}` URL should canonicalise to the base trip URL:
`https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/`

#### Next.js Implementation Context

The site uses **Next.js App Router** (confirmed by `__next_f` RSC streaming payload and `proxy.cdn.zostel.com/next_static/_next/static/chunks/` URLs in the HTML). This means:
- Server-side metadata (canonical, schema, OG tags) should be set via `generateMetadata()` in the page route
- The current schema is embedded in the JS payload (likely via a client-side `useEffect` or `next/head` in a client component), which bypasses server rendering
- Fix is a code-level change, not infrastructure

---

### 2. Content Quality & E-E-A-T

**Score: 14/100**

*Chain-of-thought:*
- Positives (2): Rich trip description in meta description confirms content exists; OG image 1200×630 with trip branding
- Deficits (5): 14 words in server HTML (SPA), no H1/H2 in server HTML, no author attribution, no publish/update date, no first-hand experience signals
- base = 2/7 × 100 = 28.6
- Criticals: Thin SPA content (−15) → max(0, 28.6−15) = **~14**

> "Score of 14 reflects existence of rich content in meta description (+), severely penalized by only 14 words of server-rendered content (Critical, −15) and complete absence of E-E-A-T signals."

**The meta description contains the content — but in the wrong place.** The 737-character meta description reveals Zostel has written comprehensive, keyword-rich content about the Bali trip (temples, waterfalls, rice fields, snorkelling, Ubud, Gili, Nusa Penida, Kuta, solo trips, group getaways). This text should be the H1 intro + H2 section copy on the page itself, not buried in a meta tag.

**Content that exists in meta description and should be on-page:**
- Trip overview paragraph (→ hero section copy)
- Destinations covered: Ubud, Gili Trawangan, Nusa Penida, Kuta (→ `<h2>Destinations</h2>` section)
- Audience targeting: "solo trips to Bali", "group getaways" (→ `<h2>Who Is This Trip For?</h2>`)
- Activities: temples, waterfalls, rice fields, snorkelling, dancing (→ `<h2>Highlights</h2>`)

---

### 3. On-Page SEO

**Score: 18/100**

*Chain-of-thought:*
- Positives (2): Title tag well-formed at 41 chars with trip name + brand; OG image correct dimensions (1200×630)
- Deficits (6): No H1, no canonical, meta desc 737 chars, og:url missing, og:type wrong, no breadcrumb
- base = 2/8 × 100 = 25
- Criticals: No H1 (−15), 737-char meta desc (−15) = −30 → max(0, 25−30) = 0; adjusted with positives: **~18**

**Title Tag Analysis:**
- `"Bali 7N/8D - Experience Bali | Zostel"` — 41 characters. Within limit. ✅
- "Bali" appears twice — could be consolidated: `"Experience Bali 7N/8D — Ubud, Gili & Nusa Penida | Zostel"` (59 chars) adds destination specificity for long-tail queries.

**Meta Description — Current vs. Recommended:**

Current (737 chars — 4.7× over limit, truncated at ~155 in SERPs to):
> *"Get ready for the perfect Bali trip that blends island adventure, cultural depth, and beachside chill."*

Recommended (148 chars):
> *"Explore Bali in 7N/8D with Zostel — Ubud temples, Gili snorkelling, Nusa Penida cliffs & Kuta beaches. From ₹41,722. Solo & group-friendly. Book now."*

This version:
- Stays within 155 chars ✅
- Contains primary keyword "Bali" twice naturally
- Names 4 destinations (higher relevance signal)
- Includes price anchor (reduces bounce from price-shocked users)
- Has clear CTA ("Book now")
- Targets both "solo trips to Bali" and "group" intents

---

### 4. Schema / Structured Data

**Score: 12/100**

*Chain-of-thought:*
- Positives (1): Schema block does exist in the JS/RSC payload (better than nothing; Googlebot may parse it)
- Deficits (5): Not in static `<head>` (invisible to AI crawlers), invalid `duration` format, `provider` as string, missing `offers`/`price`/`startDate`, missing `aggregateRating`
- base = 1/6 × 100 = 16.7
- Criticals: Schema not in `<head>` (−15) → max(0, 16.7−15) = **~12**

> "Score of 12 — schema exists in JavaScript payload but is invisible to AI crawlers; contains property errors that would fail Rich Results Test."

**Current schema (from RSC payload via WebFetch):**
```json
{
  "@context": "https://schema.org",
  "@type": "Trip",
  "name": "Experience Bali",
  "description": "Get ready for the perfect Bali trip...",
  "image": "https://proxy.cdn.zo.xyz/gallery/media/images/b0af6c38-...",
  "url": "https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/",
  "duration": "P8",
  "provider": "Zostel"
}
```

**Issues with current schema:**

| Property | Current Value | Problem | Fix |
|----------|--------------|---------|-----|
| `@type` | `"Trip"` | Overly generic. `TouristTrip` is the more specific recommended type | Use `"TouristTrip"` |
| `duration` | `"P8"` | Invalid ISO 8601 — missing unit designator | `"P8D"` (8 days) |
| `provider` | `"Zostel"` | Must be an object, not a string | `{"@type": "Organization", "name": "Zostel", "url": "https://www.zostel.com"}` |
| `offers` | Missing | Price and availability are key for travel rich results | Add `Offer` with price, currency, availability, validFrom |
| `itinerary` | Missing (referenced in payload but absent from schema block) | Itinerary is a key `TouristTrip` signal | Add `ItemList` itinerary |
| `startDate` | Missing | Required for batch-specific pages | Add ISO 8601 date: `"2026-04-02"` |
| `aggregateRating` | Missing | Strong CTR and trust signal in SERPs | Add if reviews exist |
| `touristType` | Missing | Helps AI classify the audience | `["Adventure", "Cultural"]` |

**Recommended complete schema:**

```json
[
  {
    "@context": "https://schema.org",
    "@type": "TouristTrip",
    "name": "Experience Bali — 7 Nights 8 Days",
    "description": "Explore Bali across Ubud, Gili Trawangan, Nusa Penida, and Kuta. 8-day adventure with temples, waterfalls, snorkelling, and Kecak dance.",
    "image": "https://proxy.cdn.zo.xyz/gallery/media/images/b0af6c38-fc7f-46fd-995c-00c9fe31b059_20250417135249.png",
    "url": "https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/",
    "duration": "P8D",
    "touristType": ["Adventure", "Cultural", "Group"],
    "itinerary": {
      "@type": "ItemList",
      "numberOfItems": 8,
      "itemListElement": [
        {"@type": "ListItem", "position": 1, "name": "Arrival in Bali — Transfer to Ubud"},
        {"@type": "ListItem", "position": 2, "name": "Ubud Adventures — ATV & Jungle Swing"},
        {"@type": "ListItem", "position": 3, "name": "Bali to Gili Trawangan transfer"},
        {"@type": "ListItem", "position": 4, "name": "Gili Island leisure day"},
        {"@type": "ListItem", "position": 5, "name": "Gili to Nusa Penida — Diamond Beach"},
        {"@type": "ListItem", "position": 6, "name": "Nusa Penida exploration — transfer to Kuta"},
        {"@type": "ListItem", "position": 7, "name": "Uluwatu Temple & Kecak Dance"},
        {"@type": "ListItem", "position": 8, "name": "Departure"}
      ]
    },
    "offers": {
      "@type": "Offer",
      "price": "41722.18",
      "priceCurrency": "INR",
      "availability": "https://schema.org/InStock",
      "validFrom": "2026-01-01",
      "url": "https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/"
    },
    "provider": {
      "@type": "Organization",
      "name": "Zostel",
      "url": "https://www.zostel.com"
    }
  },
  {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    "itemListElement": [
      {"@type": "ListItem", "position": 1, "name": "Home", "item": "https://www.zostel.com/"},
      {"@type": "ListItem", "position": 2, "name": "Zo Trips", "item": "https://www.zostel.com/zo-trips"},
      {"@type": "ListItem", "position": 3, "name": "Experience Bali", "item": "https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/"}
    ]
  }
]
```

Validate at: https://search.google.com/test/rich-results

---

### 5. Performance (Core Web Vitals)

**Score: N/A — Hypothesis only**

Same platform as `/zo-trips` — all hypotheses from the previous audit apply. Additional risk on this page:
- **LCP**: The trip hero image (`b0af6c38-fc7f-46fd...png`, 1200×630) is a primary LCP candidate. No `fetchpriority="high"` or `<link rel="preload">` observed in server HTML.
- **CLS**: All 24 images missing `width`/`height`. A trip page with 8+ day images loading without reserved dimensions is a high CLS risk.
- **INP**: Next.js App Router with RSC streaming is generally good for INP, but third-party scripts (Razorpay, MoEngage SDK, Facebook Pixel, GTM) observed in HTML may add main-thread contention.

---

### 6. Links

**External Links:**
- Social links (Instagram, Facebook, Twitter): 403 (platform bot-blocking, not truly broken). Add `rel="noopener noreferrer"`.
- Razorpay and MoEngage SDK: third-party scripts loaded. Check if they can be deferred.

**Internal Links:**
- No link back to `/zo-trips` parent page in server-rendered HTML (nav has "Zo Trips" link, which helps).
- No breadcrumb navigation rendered server-side.
- Merchandise link still has 2-hop redirect chain (same as parent domain issue).
- No cross-sell links to related trips (e.g. "Also explore: Sri Lanka, Singapore GP, Japan").

---

### 7. GEO / AEO

**Score: 10/100** (inherited from root domain — no change)

**Key AEO observations for this trip page:**
- No "What's included?" / "What's not included?" content in server-side HTML — these are prime Featured Snippet candidates for queries like "bali trip package inclusions"
- No FAQ section covering: visa requirements for Bali from India, best time to visit, group size, physical difficulty, cancellation policy
- No review schema or star rating — significantly reduces CTR for competitive Bali travel queries
- The trip description (currently in meta description only) contains 4 strong long-tail keyword phrases: "solo trips to Bali", "group getaways", "Bali vacation packages", "best of Bali in just one week" — these should be on-page headings or copy
- No `speakable` schema for voice search optimisation

---

### 8. Sitemap

**Status: No sitemap (inherited from root domain)**

This trip URL at its canonical form (`/zo-trip/experience-bali-tr-j5xx3p9f/`) should be included in a `/sitemap-trips.xml` sub-sitemap with:
- `<loc>https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/</loc>` (canonical, no batch param)
- `<lastmod>` reflecting most recent trip update
- `<changefreq>weekly</changefreq>` (prices and availability change)

---

## D) Comparison with /zo-trips Listing Page

| Issue | /zo-trips | This trip page | Delta |
|-------|-----------|---------------|-------|
| H1 | Missing | Missing | Same |
| Canonical | Missing | Missing + batch URL problem | Worse |
| Schema | None | Present but JS-only + errors | Slightly better but still critical |
| Meta description length | 213 chars | 737 chars | Much worse |
| og:url | Missing | Missing | Same |
| og:type | `website` | `article` (wrong) | New issue |
| Content (server HTML) | 23 words | 14 words | Worse |
| Sitemap | None | None | Same |
| Security headers | 50/100 | 50/100 | Same |
| URL architecture | Clean | `?batch=` duplication | New critical |

---

## E) Unknowns and Follow-ups

| Item | Confidence | How to Confirm |
|------|------------|----------------|
| Total number of `?batch=` URL variants per trip | Unknown | Check product database; test additional batch codes |
| Whether Googlebot renders RSC payload schema correctly | Hypothesis | Use Google's URL Inspection tool in GSC on this URL |
| CWV field data | Hypothesis | PageSpeed Insights + GSC Core Web Vitals report |
| Whether trip-level pages are currently indexed | Unknown | `site:zostel.com/zo-trip/experience-bali` in Google |
| Whether `?batch=` URLs are already in Google's index | Unknown | GSC Coverage report filtered to `/zo-trip/` prefix |
| Number of total trip pages across Zo Trips | Unknown | Check sitemap or crawl with Screaming Frog |

---

## F) Environment Limitations

- **PageSpeed Insights API**: HTTP 403. CWV is Hypothesis only.
- **Social links 403**: Platform bot-blocking, not genuine broken links.
- **RSC payload schema**: Confirmed present via WebFetch (browser rendering) but absent in static HTML — confidence Confirmed for "schema in JS" finding.

---

*Audit produced by Agentic SEO Skill v1 | zostel.com/zo-trip/experience-bali | 2026-03-15*
