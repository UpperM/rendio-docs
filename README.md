# Rendio Documentation

Public Mintlify source for the Rendio docs, published at <https://rendio.dev/docs>.

## Local preview

```bash
npm install -g mintlify
mintlify dev
```

Opens <http://localhost:3000>. Edits to `.mdx` files hot-reload.

## Structure

```
docs.json                 # Navigation, theme, OpenAPI wiring
openapi.json              # Generated from Zod schemas in the rendio repo
introduction.mdx          # Landing page
quickstart.mdx            # Hello-world in Node / Python / Go / curl
authentication.mdx        # API keys, formats, env setup
api-reference/            # Endpoint reference + cross-cutting concerns
sdks/                     # Per-language SDK guides (Node, Python, Go)
concepts/                 # Lifecycle, plans, idempotency, retention
guides/                   # Task-oriented recipes
snippets/                 # Reusable MDX fragments imported across pages
```

## OpenAPI contract

`openapi.json` is **generated** — do not edit by hand. It is produced from the
Zod schemas in the private `rendio` repo (`apps/web/scripts/generate-openapi.ts`)
and synced here by a GitHub Action on every merge to `main`.

## Deployment

Mintlify serves this content at `rendio.mintlify.dev`. A Cloudflare Worker on
`rendio.dev` proxies `/docs/*` to Mintlify so the public URL is
`https://rendio.dev/docs`.

## Contributing

Typos, clarifications, missing examples — PRs welcome. For content that
depends on the API contract (endpoint params, error codes, response shapes),
please open an issue in the main rendio repo instead — those changes flow
through the generated `openapi.json`.
