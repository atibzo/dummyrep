# SEO Action Plan — Experience Bali Trip Page
## URL: https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/88GMC8MX?batch=2026-04-02_TR-J5XX3P9F-0001
## Canonical: https://www.zostel.com/zo-trip/experience-bali-tr-j5xx3p9f/
## Date: 2026-03-15
## Score: 19/100 (CRITICAL) → Target: 68/100 within 90 days

> Most fixes below are **code-level changes in the Next.js App Router** — specifically adding/correcting `generateMetadata()` output and moving schema into server-rendered `<head>`. The architectural issue (SPA / no SSR for trip content) requires a larger sprint.

---

## Priority Matrix

| # | Action | Impact | Effort | Type |
|---|--------|--------|--------|------|
| P1 | Add canonical tag (server-side) via `generateMetadata()` | Critical | Low | Quick Win |
| P2 | Handle `?batch=` as non-indexable param + GSC configuration | Critical | Low | Quick Win |
| P3 | Add H1 tag (server-side) | High | Low | Quick Win |
| P4 | Fix meta description to ≤155 chars | High | Low | Quick Win |
| P5 | Move schema to server-side `<script type="application/ld+json">` | High | Low | Quick Win |
| P6 | Fix schema errors (duration, provider, add offers/itinerary) | High | Low | Quick Win |
| P7 | Add `og:url` meta tag | Medium | Low | Quick Win |
| P8 | Fix `og:type` from `article` to `website` | Medium | Low | Quick Win |
| P9 | Trim `og:description` + `twitter:description` to ≤200 chars | Medium | Low | Quick Win |
| P10 | Add width/height to all `<img>` tags (CLS fix) | High | Low | Quick Win |
| P11 | Add server-side `BreadcrumbList` schema + visible breadcrumb nav | Medium | Low | Quick Win |
| P12 | Pre-render trip content via Next.js ISR (itinerary, inclusions, highlights) | Critical | High | Strategic |
| P13 | Add FAQ section (visa, cancellation, difficulty, group size) | High | Medium | Strategic |
| P14 | Add trip curator attribution (E-E-A-T) | Medium | Medium | Strategic |
| P15 | Add cross-sell internal links to related trips | Medium | Low | Quick Win |
| P16 | Add `aggregateRating` schema once reviews are collected | High | Low | Quick Win |
| P17 | Fix security headers (HSTS, X-Content-Type-Options, etc.) | Medium | Low | Quick Win |
| P18 | Create sitemap including canonical trip URLs | High | Medium | Quick Win |
| P19 | Add `rel="noopener noreferrer"` to all external links | Low | Low | Maintenance |
| P20 | Fix merchandise redirect chain | Low | Low | Maintenance |

---

## Phase 1 — Immediate Blockers (< 2 hours, all in Next.js `generateMetadata()`)

All fixes in this phase are changes to a single function in the trip page route file. They will fix **5 Critical** and **3 Warning** findings in one PR.

### 1.1 Fix `generateMetadata()` for Trip Pages

In the Next.js App Router file for trip pages (likely `app/zo-trip/[slug]/[batchId]/page.tsx` or similar), update the `generateMetadata` export:

```typescript
// app/zo-trip/[slug]/[batchId]/page.tsx (or equivalent)

export async function generateMetadata(
  { params, searchParams }: { params: { slug: string; batchId: string }; searchParams: { batch?: string } }
): Promise<Metadata> {
  const trip = await fetchTripBySlug(params.slug);  // server-side fetch

  // Canonical always points to clean base URL — ignores batchId and ?batch= param
  const canonicalUrl = `https://www.zostel.com/zo-trip/${params.slug}/`;

  return {
    title: `${trip.name} ${trip.duration} | Zostel`,  // e.g. "Experience Bali 7N/8D | Zostel"
    description: buildTripMetaDescription(trip),  // ≤155 chars (see 1.2)

    alternates: {
      canonical: canonicalUrl,  // ← FIXES CANONICALIZATION
    },

    openGraph: {
      title: `${trip.name} ${trip.duration} | Zostel`,
      description: buildTripOgDescription(trip),  // ≤200 chars (see 1.3)
      url: canonicalUrl,  // ← ADDS og:url
      type: 'website',  // ← FIXES og:type (was "article")
      siteName: 'Zostel',
      images: [
        {
          url: trip.heroImage,
          width: 1200,
          height: 630,
          alt: `${trip.name} — ${trip.duration} trip by Zostel`,
        },
      ],
    },

    twitter: {
      card: 'summary_large_image',
      site: '@zostelhostel',
      title: `${trip.name} ${trip.duration} | Zostel`,
      description: buildTripOgDescription(trip),  // reuse ≤200 chars
      images: [trip.heroImage],
    },
  };
}
```

### 1.2 Meta Description Builder (≤155 chars)

```typescript
function buildTripMetaDescription(trip: Trip): string {
  // Format: "{Name} — {duration} covering {destinations}. From ₹{price}. {audience}. Book with Zostel."
  // Example (148 chars):
  // "Explore Bali in 7N/8D with Zostel — Ubud temples, Gili snorkelling, Nusa Penida cliffs & Kuta beaches. From ₹41,722. Solo & group-friendly. Book now."
  const destinations = trip.destinations.slice(0, 4).join(', ');
  const price = formatINR(trip.startingPrice);
  return `Explore ${trip.name} in ${trip.duration} with Zostel — ${destinations}. From ${price}. ${trip.audienceTag}. Book now.`.slice(0, 155);
}
```

### 1.3 OG/Twitter Description Builder (≤200 chars)

```typescript
function buildTripOgDescription(trip: Trip): string {
  // Use first 200 chars of the full description + ellipsis
  return trip.description.slice(0, 197) + '...';
}
```

---

## Phase 2 — Schema Fix (< 1 hour)

### 2.1 Move Schema to Server-Rendered `<head>`

In the trip page component, replace any client-side schema injection (`useEffect`, `next/head` in client component) with a server-rendered `<script>` block:

```tsx
// In the server component (not a 'use client' component)
import Script from 'next/script';

export default async function TripPage({ params, searchParams }) {
  const trip = await fetchTripBySlug(params.slug);
  const schema = buildTripSchema(trip, searchParams.batch);

  return (
    <>
      <Script
        id="trip-schema"
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(schema) }}
        strategy="beforeInteractive"  // ensures it's in <head>, not deferred
      />
      {/* ... page content ... */}
    </>
  );
}
```

### 2.2 Complete Schema Builder

```typescript
function buildTripSchema(trip: Trip, batchDate?: string) {
  return [
    {
      "@context": "https://schema.org",
      "@type": "TouristTrip",
      "name": `${trip.name} — ${trip.durationLabel}`,
      "description": trip.description.slice(0, 300),
      "image": trip.heroImage,
      "url": `https://www.zostel.com/zo-trip/${trip.slug}/`,
      "duration": `P${trip.durationDays}D`,  // ← WAS "P8" (invalid), now "P8D"
      "touristType": trip.tags,  // e.g. ["Adventure", "Cultural", "Group"]
      "itinerary": {
        "@type": "ItemList",
        "numberOfItems": trip.itinerary.length,
        "itemListElement": trip.itinerary.map((day, i) => ({
          "@type": "ListItem",
          "position": i + 1,
          "name": day.title,
        })),
      },
      "offers": {
        "@type": "Offer",
        "price": trip.startingPrice.toString(),
        "priceCurrency": "INR",
        "availability": "https://schema.org/InStock",
        "validFrom": new Date().toISOString().split('T')[0],
        "url": `https://www.zostel.com/zo-trip/${trip.slug}/`,
      },
      "provider": {  // ← WAS "Zostel" (string), now Organization object
        "@type": "Organization",
        "name": "Zostel",
        "url": "https://www.zostel.com",
      },
      ...(trip.reviewCount > 0 && {
        "aggregateRating": {
          "@type": "AggregateRating",
          "ratingValue": trip.averageRating.toString(),
          "reviewCount": trip.reviewCount.toString(),
          "bestRating": "5",
          "worstRating": "1",
        },
      }),
    },
    {
      "@context": "https://schema.org",
      "@type": "BreadcrumbList",
      "itemListElement": [
        { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://www.zostel.com/" },
        { "@type": "ListItem", "position": 2, "name": "Zo Trips", "item": "https://www.zostel.com/zo-trips" },
        { "@type": "ListItem", "position": 3, "name": trip.name, "item": `https://www.zostel.com/zo-trip/${trip.slug}/` },
      ],
    },
  ];
}
```

---

## Phase 3 — Strategic Improvements (Month 1–3)

### 3.1 Pre-render Trip Content via Next.js ISR

This is the highest-impact fix. The current RSC streaming approach is not surfacing trip content in server HTML.

```typescript
// Force static generation with revalidation
export async function generateStaticParams() {
  const trips = await fetchAllTripSlugs();
  return trips.map(trip => ({ slug: trip.slug }));
}

// Revalidate every 6 hours (prices/availability may change)
export const revalidate = 21600;
```

**Minimum content to pre-render (target: 600–800 words):**
1. `<h1>` — Trip name + duration (e.g. "Experience Bali — 7 Nights 8 Days")
2. `<h2>Trip Overview</h2>` — 100–150 word intro paragraph
3. `<h2>Itinerary</h2>` — Day-by-day breakdown (8 items × ~30 words = 240 words)
4. `<h2>Highlights</h2>` — Bullet list of 6–8 experiences
5. `<h2>What's Included / Excluded</h2>` — Two columns
6. `<h2>Frequently Asked Questions</h2>` — 5–7 Q&As
7. `<h2>About Your Trip Curator</h2>` — 50–80 word bio with link to guide profile

### 3.2 FAQ Section Content (AEO & Featured Snippets)

Add these Q&As as server-rendered text (NOT FAQPage schema — that is restricted to govt/health sites):

| Question | Target query |
|---------|-------------|
| Do I need a visa for Bali from India? | "bali visa for indian passport" |
| What is the best time to visit Bali? | "best time to visit bali" |
| Is this trip suitable for solo travellers? | "solo trip to bali" |
| What is the group size for Zo Trips? | "zo trips group size" |
| What is the cancellation policy? | "zostel trip cancellation policy" |
| Are flights included in the package? | "bali trip package india with flights" |
| What is the physical difficulty level? | "bali trip difficulty level" |

### 3.3 Google Search Console — Batch URL Parameter

1. In GSC → Legacy Search Console → URL Parameters (if available), mark `batch` as a parameter that "doesn't change the content" → "No URLs"
2. Alternatively, ensure all `?batch=` URLs return a server-side canonical pointing to the clean URL (Phase 1 fix handles this programmatically)

### 3.4 Add Cross-Sell Internal Links

On each trip page, add a "You might also like" section with 3–4 related trips:
```html
<section>
  <h2>Explore More Zo Trips</h2>
  <ul>
    <li><a href="/zo-trip/experience-sri-lanka-...">Experience Sri Lanka — 8N/9D</a></li>
    <li><a href="/zo-trip/experience-japan-...">Experience Japan — 9N/10D</a></li>
    <li><a href="/zo-trips">See all Zo Trips →</a></li>
  </ul>
</section>
```
This improves internal link depth, reduces orphan pages, and increases session depth/conversions.

---

## Phase 4 — Maintenance (Ongoing)

| Task | Frequency |
|------|-----------|
| Validate schema in Rich Results Test after each trip update | On change |
| Check GSC Coverage for `?batch=` URL indexation | Monthly |
| Update `aggregateRating` in schema as reviews accumulate | Weekly |
| Monitor `site:zostel.com/zo-trip/` for unexpected indexation | Monthly |
| Add new trip slugs to sitemap on publication | Automated on deploy |

---

## Expected Impact by Phase

| Phase | Actions | Score Lift | Key Metric |
|-------|---------|-----------|-----------|
| Phase 1 | Canonical, H1, meta desc fix, og:url, og:type | +10 pts | Technical + On-Page |
| Phase 2 | Schema to `<head>` + fix errors | +15 pts | Schema |
| Phase 3 | ISR pre-render + FAQ + cross-sells | +22 pts | Content + Links |
| **Total** | All phases | **~47 pts** | **~66–68/100** |

---

## Platform-Wide Fixes (Apply to ALL Trip Pages)

The following fixes should be applied once at the platform level and will fix all current and future trip pages:

| Fix | Where | Impact |
|-----|-------|--------|
| `generateMetadata()` with canonical | Next.js App Router page file | Canonical for all trips |
| `?batch=` → canonical strips param | Same | Batch URL deduplication |
| Schema server-render via `<Script strategy="beforeInteractive">` | Trip page server component | Schema visible to all crawlers |
| Image `width`/`height` in shared `TripImage` component | Shared component | CLS fix across all trips |
| Security headers in `next.config.js` `headers()` | Next.js config | All pages at once |
| Sitemap generation script covering `/zo-trip/*/` | Build pipeline | All trip pages in sitemap |
| Fix meta description length cap at 155 chars | `buildTripMetaDescription()` | All trip pages |

---

*Action plan produced by Agentic SEO Skill v1 | zostel.com/zo-trip/experience-bali | 2026-03-15*
