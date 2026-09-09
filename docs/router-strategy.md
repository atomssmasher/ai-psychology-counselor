# Cost routing / cheaper inference strategy

Updated: 2026-09-09

## Goal

This branch is **not** about semantic model routing. The application can choose models itself.

The goal is to find ways to buy inference for our approved high-EQ models **cheaper than the model creator's standard direct API**, using:

- competing inference providers for open-weight models;
- marketplace/provider routing that picks the cheapest deployment of the exact same model;
- discounted processing tiers such as Flex or Batch;
- enterprise/volume discounts when publicly available.

Approved models:

- OpenAI GPT-5.6 Luna
- Xiaomi MiMo-V2.5-Pro
- Moonshot Kimi K2.6
- Anthropic Claude Sonnet 5

We do not accept substitution with lower-EQ models merely because they are cheaper.

## Key conclusion

### Open-weight models are where real provider arbitrage exists

MiMo-V2.5-Pro and Kimi K2.6 are available from multiple inference providers. Different hosts can sell the same model at materially different prices.

OpenRouter is useful here **not because it chooses a different model**, but because it exposes multiple providers of the *same exact model* and can route to the cheapest healthy provider.

OpenRouter charges a 5.5% fee when buying standard PAYG credits, so a provider discount must exceed that fee to create real savings.

### Proprietary models are different

GPT-5.6 Luna and Claude Sonnet 5 cannot generally be re-hosted independently. Public reseller prices are normally the same as or higher than direct API pricing. Savings mostly come from the model vendor's own discounted processing tiers (Flex / Batch), caching, or negotiated enterprise contracts.

## Current opportunities

### 1. MiMo-V2.5-Pro — strong real arbitrage opportunity

Xiaomi direct overseas PAYG:

- uncached input: $0.435 / 1M
- cache hit: $0.0036 / 1M
- output: $0.87 / 1M

OpenRouter currently lists GMICloud at a promotional 30% discount:

- uncached input: $0.3045 / 1M
- cache read: about $0.0028 / 1M
- output: $0.609 / 1M

Even after OpenRouter's 5.5% credit purchase fee, that is materially cheaper than Xiaomi direct while the underlying model is still MiMo-V2.5-Pro.

Other MiMo hosts on OpenRouter currently include Xiaomi, DigitalOcean, DeepInfra, NovitaAI, StreamLake and AtlasCloud. Price, uptime and throughput vary substantially.

Important: Xiaomi also sells extremely cheap Token Plan subscriptions, but their terms explicitly restrict Token Plan to coding/development-tool scenarios and prohibit using it as the backend of a non-coding custom application. Therefore it is **not valid for AI Psychology Counselor**.

Sources:

- Xiaomi PAYG: https://mimo.mi.com/docs/en-US/price/pay-as-you-go
- Xiaomi Token Plan and restriction: https://mimo.mi.com/docs/en-US/price/token-plan
- OpenRouter MiMo provider comparison: https://openrouter.ai/xiaomi/mimo-v2.5-pro/providers

### 2. Kimi K2.6 — strong provider-arbitrage opportunity

Moonshot/Kimi direct public price:

- uncached input: $0.95 / 1M
- cached input: $0.16 / 1M
- output: $4.00 / 1M

OpenRouter currently exposes around 20 providers for Kimi K2.6. Several are cheaper than Moonshot direct. Recent public provider tables have shown examples such as:

- Inceptron around $0.56 input / $3.39 output
- Decart around $0.66 input / $3.41 output, cached around $0.144
- some promotional/provider routes even lower on output for limited periods

Therefore Kimi is a good candidate for exact-model cheapest-provider routing.

Sources:

- Kimi official pricing: https://www.kimi.ai/resources/kimi-k2-6-pricing
- OpenRouter Kimi provider marketplace: https://openrouter.ai/moonshotai/kimi-k2.6/providers
- Decart pricing: https://cogito.decart.ai/pricing

### 3. GPT-5.6 Luna — direct standard API is already cheap; Flex is the main discount

OpenAI standard Luna:

- input: $0.20 / 1M
- cached: $0.02 / 1M
- output: $1.20 / 1M

OpenRouter currently exposes an OpenAI Flex deployment at roughly half the standard price:

- input: $0.10 / 1M
- cached: $0.01 / 1M
- output: $0.60 / 1M

However Flex trades price for lower priority / longer and less predictable response times. This is attractive for non-urgent work but needs latency testing before using it for interactive chat.

This is not third-party rehosting; it is OpenAI's discounted processing tier surfaced through the marketplace.

Sources:

- OpenAI Luna standard pricing: https://developers.openai.com/api/docs/models/gpt-5.6-luna
- OpenRouter Luna providers: https://openrouter.ai/openai/gpt-5.6-luna-20260709
- OpenAI service tiers / Flex support: https://developers.openai.com/api/reference/cli/resources/responses/methods/create

### 4. Claude Sonnet 5 — no compelling real-time reseller discount found

Anthropic direct:

- input: $2 / 1M
- cached input: $0.20 / 1M
- output: $10 / 1M

AWS Bedrock, Google Vertex and Requesty generally expose the same upstream pricing or add markup. Requesty PAYG adds 5% unless using BYOK.

Anthropic's official Batch API gives a real 50% discount:

- batch input: $1 / 1M
- batch output: $5 / 1M

But Batch is asynchronous, so it is suitable for offline/deferred analyses, summaries, re-processing or scheduled deep reports — not normal live conversation.

Sources:

- Anthropic Sonnet 5 pricing: https://www.anthropic.com/claude/sonnet
- Anthropic full API pricing incl. Batch: https://platform.claude.com/docs/en/about-claude/pricing
- Requesty Sonnet 5 provider comparison: https://www.requesty.ai/model/anthropic/claude-sonnet-5

## Approximate impact on our 1000-DAU workload

Using the earlier workload estimate (~2.135B input tokens/month, ~208M output tokens/month, ~72% of input assumed cacheable):

| Model / route | Approx monthly AI cost | Comment |
|---|---:|---|
| Luna standard direct | ~$400 | baseline with good cache |
| Luna via OpenAI Flex pricing | ~$200 before marketplace fee; ~$211 incl. 5.5% OpenRouter credit fee | ~47% net saving, but slower/lower-priority |
| MiMo direct Xiaomi | ~$447 | baseline with good cache |
| MiMo via GMICloud promo on OpenRouter | ~$313 before fee; ~$330 incl. 5.5% fee | ~26% net saving |
| Kimi direct Moonshot | ~$1,647 | baseline with good cache |
| Kimi through a materially cheaper host | roughly ~$1.1K–1.4K depending provider/cache/discount | potentially ~15–35% saving |
| Sonnet 5 standard direct | ~$3,585 | baseline with good cache |
| Sonnet 5 Batch | roughly half (~$1,793) | asynchronous; not for normal live chat |

These are planning estimates, not guaranteed invoices. Provider promotions, cache behavior, tokenizers, availability and marketplace fees can change.

## Recommended architecture

1. **MiMo:** buy through a marketplace or directly from whichever reliable host is materially cheaper than Xiaomi; continuously compare provider price + uptime.
2. **Kimi:** same approach; open weights make price competition meaningful.
3. **Luna:** standard direct API for latency-sensitive chat; test Flex for non-urgent turns / background work if user experience allows it.
4. **Sonnet:** direct real-time API for premium live turns; Batch for deferred deep analyses where waiting is acceptable.

## Business rule

We use a marketplace only if it produces a **lower all-in COGS for the exact approved model** after platform fees and without unacceptable latency, reliability, privacy or model-quality degradation.

A gateway that merely adds convenience while charging more is not a cost optimization and is not a reason to adopt it.