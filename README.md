# AI Psychology Counselor

AI-powered relationship and social-situation analysis product.

## Product idea

A consumer app where users can discuss relationships with friends, parents, partners, spouses, dates, classmates and other people. Users can paste conversations, describe situations, ask what another person may mean, compare interpretations and get practical recommendations.

Target audience: teenagers through adults.

## Current AI strategy

Primary model candidates are selected mainly by **EQ-Bench 4** quality plus API economics.

| Role | Model | EQ-Bench 4 | Planning AI COGS for 1,000 very active DAU / month |
|---|---|---:|---:|
| Ultra-cheap | GPT-5.6 Luna | 1156 | ~$400–680 |
| Default / best current price-performance candidate | MiMo 2.5 Pro | 1208 | ~$450–1,100 |
| Mid-tier | Kimi K2.6 | 1202 | ~$1,650–2,860 |
| Premium / Deep Analysis | Claude Sonnet 5 | 1236 | ~$3,600–6,350 |

The lower end assumes effective prompt caching; the upper end is closer to uncached usage under the heavy-load simulation documented in `docs/business-model.md`.

## Current product architecture hypothesis

- **Default free model:** MiMo 2.5 Pro
- **Cheapest fallback / free-tier option:** GPT-5.6 Luna
- **Mid-tier candidate for A/B tests:** Kimi K2.6
- **Premium Deep Analysis:** Claude Sonnet 5

Expensive models should not answer every ordinary message. They should be reserved for high-value or complex requests.

## Monetization hypothesis

The product should remain usable for free.

Recommended mix:

- Freemium
- Rewarded ads for additional usage
- Credits for premium / deep analyses
- Optional Plus plan around $2.99–4.99
- Potential B2B / white-label partnerships later

## Current business target

For **1,000 highly active daily users**, a reasonable early planning range is:

- Revenue: **~$700–1,300/month**
- MiMo AI COGS: **~$450/month with strong caching**
- Contribution after AI: **~$250–850/month** before servers, acquisition, store fees, taxes, salaries and other operating costs

This is a deliberately heavy usage scenario, not a promise of actual unit economics.

## Core KPIs

- AI COGS / DAU
- Cost per conversation
- ARPDAU
- Ad ARPDAU
- Payer conversion
- D7 retention
- D30 retention
- Share of requests escalated to premium models

See [`docs/business-model.md`](docs/business-model.md) for assumptions and monetization scenarios.
