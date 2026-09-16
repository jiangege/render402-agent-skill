# Render402 public API reference

The live contract is authoritative:

- Product: `https://render402.xyz`
- OpenAPI: `https://api.render402.xyz/openapi.json`
- x402 manifest: `https://api.render402.xyz/.well-known/x402`
- Models: `GET https://api.render402.xyz/v1/models`
- Capabilities: `GET https://api.render402.xyz/v1/capabilities`
- Pricing: `GET https://api.render402.xyz/v1/pricing`
- Generation: `POST https://api.render402.xyz/v1/generate`

## Generation envelope

Send JSON shaped as `{ "generation": { ... } }` and an `Idempotency-Key` header.

Mode-specific required fields:

- `text`: `mode`, non-empty English `prompt`; no images.
- `reference`: `mode`, non-empty English `prompt`, `image_ids` containing one to nine ready asset UUIDs.
- `frames`: `mode`, non-empty English `prompt`, `first_frame_id`, and `last_frame_id`.
- `lipsync`: `mode`, exactly one ready UUID in `image_ids` and one in `audio_ids`; the prompt is empty or omitted.

Common fields include `model_id`, `duration`, `resolution`, `aspect_ratio`, and `seed`, subject to the live capability matrix. Do not manufacture combinations by taking a Cartesian product of independent-looking values.

## x402 v2 exchange

An unpaid valid generation request returns HTTP 402 and a base64-encoded `PAYMENT-REQUIRED` response header. A paid retry supplies `PAYMENT-SIGNATURE`. A successful settlement response may include `PAYMENT-RESPONSE`.

Render402 uses Base mainnet (`eip155:8453`), official Base USDC, the `exact` EVM scheme, and EIP-3009 authorization. The challenge amount is authoritative because media use and selected capability affect the price.

The currently configured facilitator is an implementation detail. Honor the challenge and canonical resource; never replay the same unused authorization through a second facilitator in parallel.

## Accepted response

HTTP 202 contains absolute status and output-following information. Poll only the returned `status_url`. Completion is authoritative only when the status payload is terminal and exposes `generation.output.url`.
