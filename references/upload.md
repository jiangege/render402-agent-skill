# Render402 media upload workflow

Use this workflow before `POST /v1/generate` for `reference`, `frames`, and `lipsync` modes. Uploading and normalization are free; the wallet signature proves control of an address but does not authorize an x402 payment.

The live OpenAPI document at `https://api.render402.xyz/openapi.json` is authoritative. Preserve one wallet session across the steps below, keep each upload's idempotency key stable across retries, and never expose a private key or seed phrase.

## 1. Create a wallet session

1. `POST /v1/auth/challenge` with JSON `{ "address": "0x..." }`.
2. Sign the exact returned `message` as an EIP-191 personal message. With an injected EVM wallet, UTF-8 encode the message, hex encode those bytes, and call `personal_sign` with `[messageHex, address]`. With viem, call `account.signMessage({ message })`.
3. `POST /v1/auth/session` with `{ "address": "0x...", "signature": "0x..." }`.
4. Keep the returned token in memory and send it as `Authorization: Bearer <token>` to every upload API request. Do not log or persist the token beyond the task.

The challenge is one-use and expires. Request a new challenge rather than replaying an expired message.

## 2. Describe the file and create the upload

Before calling the API, determine the exact byte length and lowercase SHA-256 hex digest of the original file. Files must be between 1 byte and 20 MiB.

Send `POST /v1/uploads` with:

- `Authorization: Bearer <token>`
- a fresh `Idempotency-Key` of 8–128 characters matching `^[a-zA-Z0-9_.:-]+$`
- `Content-Type: application/json`
- JSON fields `media_type`, `content_type`, `bytes`, and `sha256`

Use `media_type: "image"` for PNG, JPEG, or WebP. Use `media_type: "audio"` for MP3, WAV, FLAC, M4A/MP4 audio, or an accepted MP4 source. Obtain the exact accepted MIME list from the live OpenAPI schema instead of relabeling unsupported content.

Example request body:

```json
{
  "media_type": "image",
  "content_type": "image/png",
  "bytes": 123456,
  "sha256": "64-lowercase-hex-characters"
}
```

The response contains an upload `id`, `status_url`, `complete_url`, and—while the upload is receiving—a short-lived `transfer` object. Reusing the same idempotency key with the same file description recovers the same upload state.

## 3. Transfer the original bytes

When `transfer` is present, immediately send the raw file bytes to `transfer.url`:

- use `transfer.method`, which is `PUT`;
- copy every header from `transfer.headers` exactly, including `content-type` and `x-cos-forbid-overwrite: true`;
- do not add the Bearer token, cookies, referrer, JSON encoding, multipart encoding, or x402 headers to the signed transfer URL;
- do not follow redirects;
- finish before `transfer.expires_at`.

Treat HTTP 2xx as success. HTTP 409 may mean the no-overwrite transfer already completed; continue with confirmation and let the API verify the object. For other failures, do not create a new upload immediately—retry or recover with the same idempotency key while the transfer capability remains valid.

## 4. Confirm and normalize

After the PUT finishes, send an authenticated `POST` to the returned `complete_url` (equivalent to `/v1/uploads/{id}/complete`). Then poll the absolute authenticated `status_url` using the Bearer token.

Honor `retry_after_seconds` when present. Continue through `queued` and `processing`. Stop on `failed` and report `error.code` and `error.message`; do not submit generation with a failed upload.

The upload is usable only when:

```json
{
  "status": "succeeded",
  "asset": {
    "id": "canonical-asset-uuid",
    "type": "image",
    "expires_at": "..."
  }
}
```

Use `asset.id`, not the upload `id`, in `image_ids`, `audio_ids`, `first_frame_id`, or `last_frame_id`. Submit generation before `asset.expires_at`.

## 5. Map assets to generation modes

- `reference`: upload one to nine images and place the ready asset UUIDs in `generation.image_ids`.
- `frames`: upload exactly one first-frame image and one last-frame image; place their UUIDs in `generation.first_frame_id` and `generation.last_frame_id`.
- `lipsync`: upload exactly one image and one audio source; place their UUIDs in `generation.image_ids` and `generation.audio_ids`.

Only after every required asset is ready should the agent request the paid `/v1/generate` challenge. The upload wallet session and the x402 payment authorization are separate credentials and must not be treated as interchangeable consent.
