# AI Psychology Counselor — Business & AI Economics

_Last updated: 2026-09-09_

This document captures the initial business assumptions discussed before development. All figures are planning estimates and should be replaced with measured production data as soon as possible.

## 1. Product usage simulation

The cost model assumes **1,000 highly active daily users (DAU)**.

### 500 teenagers

Each teenager discusses roughly 10 different people per day: friends, classmates, crushes, former partners and similar social situations.

Simulated mix:

- 50% light conversations: short question, small excerpt, 2 AI turns
- 35% medium conversations: pasted chat + several questions, 3 AI turns
- 15% drama / deep conversations: longer chat + follow-ups, 4 AI turns

Estimated per teenager per day:

- ~69,250 input tokens
- ~9,175 output tokens

### 500 adults

Adults usually discuss one major relationship, but in much greater depth.

Simulated mix:

- 40% short relationship discussion, ~4 AI turns
- 40% normal deep discussion with messages/history, ~7 AI turns
- 20% heavy scenario such as divorce, infidelity or long-running conflict, ~12 AI turns

Estimated per adult per day:

- ~73,100 input tokens
- ~4,700 output tokens

### Total workload

Approximate combined workload:

- ~16,650 API calls/day
- ~71.2M input tokens/day
- ~6.94M output tokens/day
- ~2.14B input tokens/month
- ~208M output tokens/month

This deliberately models a product people actively use every day.

## 2. Current model shortlist

Model quality is currently benchmarked primarily using **EQ-Bench 4** because it is directly relevant to emotional and social dialogue.

| Model | EQ-Bench 4 | Estimated monthly AI COGS, no strong caching | Estimated monthly AI COGS, strong caching | Approx. cached COGS / active user |
|---|---:|---:|---:|---:|
| GPT-5.6 Luna | 1156 | ~$680 | ~$400 | ~$0.40 |
| MiMo 2.5 Pro | 1208 | ~$1,110 | ~$450 | ~$0.45 |
| Kimi K2.6 | 1202 | ~$2,860 | ~$1,650 | ~$1.65 |
| Claude Sonnet 5 | 1236 | ~$6,350 | ~$3,600 | ~$3.60 |

### Current interpretation

**MiMo 2.5 Pro** is the leading default-model candidate because its EQ-Bench score is materially higher than Luna while its projected cached cost is almost the same.

**GPT-5.6 Luna** is the best ultra-low-cost option and could be useful for a free tier, fallback or very simple requests.

**Kimi K2.6** is worth A/B testing, but on current benchmark/cost math it is less attractive than MiMo as the default engine.

**Claude Sonnet 5** should be treated as premium compute for Deep Analysis or complex cases rather than the default model for every message.

## 3. Why caching matters

Relationship conversations contain large amounts of repeated context:

- system instructions
- user profile
- relationship profile
- previous conversation history
- summaries of prior events

On each new message, much of that context is unchanged. Prompt caching lets the provider charge less for already-processed context.

The simulation estimated that roughly **70% of input context could potentially be reusable** in a well-designed chat architecture.

Business implication: caching is not a minor engineering optimization. It can materially change gross margin.

## 4. Monetization options

The strategic goal is for the product to remain genuinely usable for free.

### Option A — Advertising-only

Works best with Luna or MiMo.

A rough planning assumption of ~$4 rewarded-video eCPM implies that a low-cost model can plausibly be funded by a few voluntary ad views per active user per day. Kimi and especially Sonnet are too expensive to rely on advertising alone at this usage intensity.

### Option B — Free daily allowance + rewarded ads

Recommended for the free tier.

Example mechanic:

- user receives a meaningful free daily allowance
- once the allowance is exhausted, a rewarded ad unlocks more messages / analysis capacity

Advantages:

- light users remain nearly free to serve
- heavy users generate incremental revenue as they consume more AI
- no hard paywall

### Option C — Credits

Sell small credit packs for deeper or more expensive analysis.

Possible early price points:

- $1.99 small pack
- $4.99 larger pack

Credits can fund calls to Sonnet or other premium models while the everyday conversation remains on MiMo/Luna.

### Option D — Optional Plus

Suggested exploratory price range:

- $2.99–4.99/month

Possible benefits:

- no ads
- higher daily limits
- longer relationship memory
- more uploaded conversation history
- monthly Deep Analysis credits

The goal is not to force every user into a subscription.

### Option E — B2B / partnerships

Possible future revenue sources:

- therapists / clinics
- dating products
- employee-assistance programs
- white-label deployments
- partner placements

Private conversation contents should not be sold or used as a creepy behavioral advertising dataset.

## 5. Early revenue scenarios for 1,000 DAU

Using MiMo 2.5 Pro as the default and approximately ~$450/month AI COGS with effective caching:

| Scenario | Approx. monthly revenue | AI COGS | Contribution after AI only |
|---|---:|---:|---:|
| Weak | ~$360 | ~$450 | ~-$90 |
| Base | ~$725 | ~$450 | ~$275 |
| Good | ~$1,280 | ~$450 | ~$830 |
| Very good | ~$1,650 | ~$450 | ~$1,200 |

These are not profit figures. They exclude hosting, moderation, app-store fees, payment processing, marketing, customer support, salaries, legal/compliance and taxes.

### Business planning range

For 1,000 highly active DAU, the current rough target range is:

- **Revenue: ~$700–1,300/month**
- **AI COGS on MiMo: ~$450/month with effective caching**
- **Contribution after AI: ~$250–850/month**

If the same unit economics scaled linearly:

- 10,000 DAU -> roughly $7k–13k/month revenue
- 100,000 DAU -> roughly $70k–130k/month revenue

In reality, revenue mix, ad eCPM, geography, retention, payment conversion and usage intensity will change with scale, so these should not be treated as forecasts.

## 6. Recommended initial architecture

**Default:** MiMo 2.5 Pro

**Cheap fallback / free-tier alternative:** GPT-5.6 Luna

**A/B candidate:** Kimi K2.6

**Premium Deep Analysis:** Claude Sonnet 5

A practical routing strategy:

1. Most everyday questions -> MiMo
2. Very simple / high-volume traffic -> Luna if economically useful
3. Complex or high-value analysis -> Sonnet via credits / premium allowance
4. Kimi K2.6 -> A/B tested against MiMo before receiving meaningful production share

## 7. Core KPIs

The MVP should instrument these from day one:

- AI COGS / DAU
- AI COGS / MAU
- Cost per conversation
- Cost per AI turn
- Average input/output tokens per session
- Prompt-cache hit rate
- ARPDAU
- Ad ARPDAU
- Rewarded-ad completion rate
- Payer conversion
- ARPPU
- Credit purchase conversion
- D1 / D7 / D30 retention
- Sessions per DAU
- Messages per session
- Percentage of traffic routed to premium models
- Gross contribution after AI COGS

## 8. Reference links

Pricing and benchmark assumptions should be rechecked before implementation or launch because model pricing changes quickly.

- EQ-Bench 4: https://eqbench.com/
- OpenAI model pricing/docs: https://developers.openai.com/
- MiMo pricing/docs: https://mimo.mi.com/
- Moonshot / Kimi: https://platform.moonshot.ai/
- Anthropic pricing/models: https://www.anthropic.com/

## 9. Current decision

The current working business hypothesis is:

> Build a free-first relationship AI product around **MiMo 2.5 Pro** as the primary low-COGS model, monetize with **rewarded ads + credits + optional low-cost Plus**, and reserve **Claude Sonnet 5** for premium Deep Analysis.

This is the starting hypothesis to validate, not a permanent architectural commitment.
