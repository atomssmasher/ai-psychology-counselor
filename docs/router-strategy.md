# Router strategy research

Updated: 2026-09-09

## Goal

Do not call a single LLM directly for every request. Put a routing layer in front of the application that can choose the most appropriate model while **hard-restricting the candidate pool** to models that perform well for social / relationship dialogue.

Current approved model pool:

- OpenAI GPT-5.6 Luna
- Xiaomi MiMo-V2.5-Pro
- Moonshot Kimi K2.6
- Anthropic Claude Sonnet 5

The router must never silently expand this pool to lower-EQ models.

## Shortlist

### 1. OpenRouter Auto Router — strongest immediate MVP option

Why it fits:

- `openrouter/auto` automatically chooses a model per prompt.
- The Auto Router is powered by Not Diamond.
- It supports an `allowed_models` list/patterns, so we can restrict routing to our approved models only.
- It supports a 0–10 cost-vs-quality tradeoff.
- Multi-turn conversations support session stickiness so the model/provider can remain stable during a conversation and preserve cache hits.
- There is no extra fee specifically for Auto Router; the routed model is billed at its normal rate.
- OpenRouter supports all four approved models today.
- OpenRouter itself does not store prompts/responses unless prompt logging is explicitly enabled; ZDR/provider privacy controls are available.

Important pricing note: OpenRouter passes through model inference pricing but charges a 5.5% fee when buying standard PAYG credits. BYOK has its own fee rules.

Links:

- Auto Router and allowed models: https://openrouter.ai/docs/guides/routing/routers/auto-router
- Guardrails / model allowlists: https://openrouter.ai/docs/guides/features/guardrails/overview
- Pricing / fees: https://openrouter.ai/docs/faq
- Privacy / ZDR: https://openrouter.ai/docs/guides/features/zdr
- Luna: https://openrouter.ai/openai/gpt-5.6-luna
- MiMo 2.5 Pro: https://openrouter.ai/xiaomi/mimo-v2.5-pro
- Kimi K2.6: https://openrouter.ai/moonshotai/kimi-k2.6
- Sonnet 5: https://openrouter.ai/anthropic

Recommended first experiment:

`allowed_models = [Luna, MiMo 2.5 Pro, Kimi K2.6, Sonnet 5]`

Start with cost/quality tradeoff around the middle and inspect what percentage of traffic is sent to each model.

### 2. Not Diamond — best pure intelligent-routing technology

Why it fits:

- The API explicitly accepts the exact candidate LLM list for every routing decision.
- Quality is the default objective; it can also optimize for cost or latency.
- It supports continuous cost-vs-quality weighting.
- Its pre-trained Chat router can be used immediately.
- More importantly, a custom router can later be trained on domain-specific evaluation data.
- Arbitrary custom inference endpoints can be included, so lack of native support for one model does not necessarily block it.

Difference from OpenRouter: Not Diamond can be used only as the **decision engine**. It returns which model should be called; the application can then call that model through OpenRouter, Requesty, or direct APIs.

Links:

- Model routing overview: https://docs.notdiamond.ai/docs/what-is-model-routing
- Pre-trained Chat router: https://docs.notdiamond.ai/docs/quickstart-routing
- Model select API: https://docs.notdiamond.ai/reference/token_model_select_v2_modelrouter_modelselect_post
- Custom router training: https://docs.notdiamond.ai/docs/router-training-quickstart
- Custom model/endpoints: https://docs.notdiamond.ai/docs/routing-between-custom-models

Best long-term fit if we eventually want routing decisions optimized specifically for relationship conversations rather than generic tasks.

### 3. Requesty — best production gateway / business control layer

Why it fits:

- One OpenAI-compatible API for 600+ models.
- Supports approved-model whitelists.
- Supports routing policies, caching, fallbacks, latency routing, budget limits and analytics.
- Smart Routing can classify requests and select models according to policy.
- Supports all four approved models today.
- PAYG pricing is model cost + 5% markup; BYOK is offered with 0% Requesty markup on provider contracts according to its current pricing page.
- EU / US / AP regional gateways and PII controls are useful for sensitive relationship data.

Caveat: Requesty's most clearly documented production routing policies are fallback, load-balance and latency policies. Its Smart Routing exists, but for our use case I would test its semantic model-choice quality rather than assume it understands relationship difficulty correctly.

Links:

- Routing policies: https://www.requesty.ai/product/routing
- Smart Routing / gateway: https://www.requesty.ai/gateway
- Approved model governance: https://www.requesty.ai/product/governance
- Pricing: https://www.requesty.ai/pricing
- Luna: https://www.requesty.ai/models/openai/gpt-5.6-luna
- MiMo 2.5 Pro: https://www.requesty.ai/models/xiaomi/mimo-v2.5-pro
- Kimi K2.6: https://www.requesty.ai/models/moonshot/kimi-k2.6
- Sonnet 5: https://www.requesty.ai/model/anthropic/claude-sonnet-5

### 4. Portkey — strongest deterministic rule engine / gateway

Why it fits:

- Conditional routing based on our metadata and request parameters.
- Easy rules such as `free -> MiMo`, `deep_analysis -> Sonnet`, `long_context -> Luna`, `premium -> Kimi/Sonnet`.
- Fallbacks, load-balancing, retries, budgets, caching and guardrails.
- Open-source gateway can be self-hosted.

Caveat: Portkey is primarily a rules/control-plane solution. It is not the first choice if the desired behavior is "read the user's message and intelligently decide which of four models will answer best" without our application providing useful routing metadata.

Links:

- Conditional routing: https://portkey.ai/docs/product/ai-gateway/conditional-routing
- Gateway: https://portkey.ai/docs/product/ai-gateway
- Pricing: https://portkey.ai/pricing

### 5. Martian Model Router — interesting, but needs more validation before adoption

Why it is interesting:

- Martian's core product is dynamic model routing across many models.
- The current gateway exposes 286 models across 48 providers, including OpenAI, Anthropic, Moonshot and Xiaomi.
- Martian publishes routing research and customer claims around quality/cost improvements.

Why it is not first choice yet:

- Public documentation is much clearer for the general gateway than for the current commercial intelligent-router configuration/pricing.
- Exact support and routing behavior for our four-model allowlist should be verified in the dashboard before architecture depends on it.

Links:

- Martian: https://withmartian.com/
- Gateway models: https://gateway-docs.withmartian.com/api-reference/models
- Router research: https://withmartian.com/post/introducing-routerbench

## Recommendation

### MVP

Use **OpenRouter Auto Router** with a hard allowlist of only:

1. MiMo-V2.5-Pro
2. GPT-5.6 Luna
3. Kimi K2.6
4. Claude Sonnet 5

Reasons: lowest integration work, all four models are already available, automatic semantic model selection, hard model restrictions, session stickiness, caching/provider failover and unified billing.

### Second experiment

Run **Not Diamond as the decision layer**, using the same four candidate models. It can return a model recommendation, while actual inference is still served by OpenRouter or Requesty.

This lets us compare:

- OpenRouter Auto decision
- Not Diamond decision
- fixed MiMo baseline

Measure model distribution, cost per conversation, latency, retention and user feedback.

### Production-control alternative

If routing needs become mostly business rules rather than semantic intelligence, use **Requesty or Portkey** as the control layer.

## Critical product rule

The approved model list is a hard constraint. A router may optimize *inside* the list, but it may not introduce an unapproved model because it is cheaper, faster, trending, or newly released.
