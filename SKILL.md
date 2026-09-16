---
name: render402
description: Discover, price, and call Render402 for MiniMax H3 text-to-video, reference-image, first/last-frame, or lip-sync generation using exact USDC payments over x402 v2 on Base.
---

# Render402

Use the public Render402 API at `https://api.render402.xyz`. It has one paid resource, `POST /v1/generate`, and requires no account for text-to-video. Treat payment, wallet authentication, and media upload as separate steps.

## Choose and price a generation

1. Read the live `GET /v1/models`, `GET /v1/capabilities`, and `GET /v1/pricing` responses. Do not infer currently supported combinations from examples in this skill.
2. Build one generation input using the exact mode contract in [the API reference](references/api.md). Use English for generation prompts.
3. Calculate an estimate from the live price schedule, then obtain the authoritative exact amount from the unpaid 402 challenge. Do not pay more than the user's stated budget.
4. Read the linked Terms and Content Policy before submitting content when they have not already been accepted in the surrounding workflow.

For inputs with images, frames, or audio, read and follow [the media upload workflow](references/upload.md) before building the generation request. Do not pass local paths, public URLs, upload IDs, or transfer URLs to `/v1/generate`; it accepts only ready canonical asset UUIDs. Upload preparation is free, and its wallet signature authenticates identity rather than authorizing USDC payment.

## Request and pay

- Generate a fresh UUID as `Idempotency-Key` for a new intent. Keep that key and the byte-equivalent request body stable across challenge, payment, and uncertain retries.
- Send the request without a payment proof to receive HTTP 402. This probe is read-only and does not move funds.
- Decode `PAYMENT-REQUIRED` as base64 JSON and verify all of these before signing: x402 version `2`, resource URL, Base network `eip155:8453`, official USDC asset, exact amount, recipient, timeout, and `exact` scheme.
- Use an x402 v2 EVM client to create `PAYMENT-SIGNATURE`, then retry the identical request. Do not convert it to legacy `X-Payment`, and do not send one unused authorization to multiple facilitators or resources.
- Immediately before creating a real authorization, surface the exact USDC amount, network, recipient, and resource. Obtain confirmation unless the user has already authorized this exact payment or an explicit maximum that covers it in the current task.
- Never request, print, log, or persist a private key or seed phrase. Prefer the wallet mechanism already chosen by the user. A wallet signature is not permission for later payments.

HTTP 202 means the order was accepted, not that the video is finished. Follow the absolute `status_url`; the embedded ticket is scoped and short-lived. On completion, return `generation.output.url`. If the paid retry has an uncertain outcome, inspect the same order with the same idempotency key and authorization instead of signing a replacement.

## Errors and stopping conditions

- Fix structural 4xx errors before requesting payment again.
- If the live price exceeds the approved ceiling, stop before signing.
- If payment or generation state is unknown, preserve the request, idempotency key, and non-secret response evidence; do not create a second order.
- If the status ticket expires, use the documented wallet-authenticated recovery path.
- Do not claim that a generation succeeded until the status response is terminal and includes the output URL.

Read [the API reference](references/api.md) for the stable public routes and mode shapes. Read [the media upload workflow](references/upload.md) whenever the request uses reference images, first/last frames, or lip-sync media.
