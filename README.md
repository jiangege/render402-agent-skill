# Render402 Agent Skill

An installable Agent Skill for discovering, pricing, and calling [Render402](https://render402.xyz), a MiniMax H3 video generation API paid per request with USDC over x402 v2 on Base.

## Install

After this repository is published:

```sh
npx skills add jiangege/render402-agent-skill
```

Then ask your agent to use `$render402` for text-to-video, reference-image, first-and-last-frame, or lip-sync generation.

The skill can inspect public catalogs and request a payment quote without moving funds. It requires confirmation before creating a real USDC authorization unless the current task already includes an exact payment approval or a sufficient explicit spending ceiling.

## Public API

- Product: <https://render402.xyz>
- OpenAPI: <https://api.render402.xyz/openapi.json>
- x402 discovery: <https://api.render402.xyz/.well-known/x402>
- Paid resource: `POST https://api.render402.xyz/v1/generate`

No wallet keys, API credentials, user prompts, or private media are included in this repository.
