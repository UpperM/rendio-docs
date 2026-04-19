# Doc Revision Plan — 2026-04-19

Produced from a full cross-check of `~/git/rendio-docs/` against the live
codebase (`~/git/rendio/`, `~/git/rendio-sdk/`). No edits have been made
yet — this is the plan to review before executing.

---

## 1. Factual Corrections

| # | Severity | File:Line | Wrong claim | Correct source | Fix |
|---|---|---|---|---|---|
| F0 | **CRITICAL** | 19 files across all docs | Base URL `https://api.rendio.dev` used everywhere | Actual URL is `https://rendio.dev/api` | Global find-replace `api.rendio.dev` → `rendio.dev/api` in all MDX + `openapi.json` + `generate-openapi.ts`. Also update `openapi.json` server URL and SDK default `baseUrl`. **19 occurrences**: `api-reference/introduction.mdx`, `api-reference/render.mdx`, `api-reference/files.mdx`, `api-reference/health.mdx`, `authentication.mdx`, `quickstart.mdx`, `sdks/overview.mdx`, `sdks/idempotency.mdx`, `sdks/opentelemetry.mdx`, `guides/screenshots.mdx`, `vscode/overview.mdx`, `resources/chromium-versions.mdx`, `openapi.json` |
| F1 | **HIGH** | `openapi.json:19` | `bearerFormat` says `rk_live_* or rk_test_*` | `auth.ts:55` — only `rk_live_` prefix exists | Remove `rk_test_*` from `generate-openapi.ts:116`, regenerate `openapi.json` |
| F2 | **HIGH** | `sdks/retries-and-timeouts.mdx:66,155` | Code samples use `rk_test_...` as constructor arg | No `rk_test_` prefix exists in the system | Replace with `rk_live_...` in all code samples |
| F3 | **MEDIUM** | `concepts/how-it-works.mdx:50` | "Most renders complete in under 100 ms" | Benchmarks: P50 13–1198 ms; only simple templates are sub-50 ms | Reconcile to "Simple templates render in under 50 ms; see [Benchmarks](/resources/benchmarks) for the full range" |
| F4 | **MEDIUM** | `guides/performance.mdx:6` | "under 300 ms when pool is warm" | Benchmarks: complex layouts 48–91 ms, large docs 223–1198 ms | Reconcile to "Most business documents render in under 100 ms; complex or large templates take longer — see [Benchmarks](/resources/benchmarks)" |
| F5 | **MEDIUM** | `sdks/retries-and-timeouts.mdx:49-54` | Backoff table shows 4 rows (attempts 0–3) | `maxRetries` default is 2, so only attempts 0–1 are reachable by default | Trim to 2 rows; add footnote "If you increase `maxRetries`, backoff continues doubling up to the 5000 ms cap" |
| F6 | **MEDIUM** | `sdks/opentelemetry.mdx:120-125` | SDK span table missing `rendio.pages_hint` | `rendio-node/src/resources/pdfs.ts:132` sets it when data keys present | Add row: `rendio.pages_hint` / `integer` / "Estimated page count hint (set when `data` keys are present)" |
| F7 | **MEDIUM** | `sdks/opentelemetry.mdx:143-153` | Gateway span table missing `rendio.api_key_name` and `rendio.filename` | `trace-auth.ts:76` sets `api_key_name`; `render.ts:250` sets `filename` | Add two rows to Gateway spans table |
| F8 | **LOW** | `use-cases/express.mdx` (body limit comment) | `express.json({ limit: "2mb" })` comment says "1 MB HTML" | Base64 expansion ≈ 1.33×, so effective HTML limit ≈ 1.5 MB | Change comment to "~1.5 MB of HTML" |
| F13 | **CRITICAL** | `sdks/error-handling.mdx` (Go tab) | `rl.RetryAfter > 0` and `time.Duration(rl.RetryAfter)` — `RetryAfter` is `*int`, not `int` | `rendio-go/errors.go` — `RetryAfter *int` | Fix to `rl.RetryAfter != nil && *rl.RetryAfter > 0` and `time.Duration(*rl.RetryAfter)`. **Does not compile as written.** |
| F14 | **HIGH** | `sdks/rendering.mdx` (Go tab) | Only `Create` and `Get` shown; `CreateBuffer`, `CreateWithResponse`, `CreateBufferWithResponse`, `GetWithResponse` are absent | `rendio-go/pdfs.go` exports all 6 methods | Add Go buffer-mode example using `CreateBuffer`; mention `*WithResponse` variants |
| F15 | **HIGH** | `sdks/opentelemetry.mdx` | No Go tab at all — no OTEL setup for Go, `WithTracer` interface not mentioned | `rendio-go/telemetry.go` defines `Tracer`/`Span` interfaces, injected via `WithTracer` | Add Go tab with `WithTracer` setup example and interface docs |
| F16 | **MEDIUM** | `sdks/opentelemetry.mdx` | Go SDK emits zero `rendio.*` parent span attributes; docs claim `rendio.mode`, `rendio.output`, `rendio.sdk.version` are set on `rendio.pdfs.create` | `rendio-go/pdfs.go` — no `SetAttribute` calls on parent span | Either fix the Go SDK to emit these attributes, or document the gap per-language |
| F17 | **MEDIUM** | `sdks/opentelemetry.mdx` | `rendio.pdfs.get` / `rendio.pdfs.get.attempt` spans exist but undocumented | `rendio-go/pdfs.go` creates these spans | Add to span inventory table |

---

## 1b. SDK Code Issues (bugs, not docs)

The audit revealed that the docs describe Node SDK behavior as universal, but
Go and Python are significantly behind. These are **code fixes** in the SDK repos,
not doc edits.

### Cross-SDK Feature Parity Matrix (updated 2026-04-19)

| Feature | Node | Python | Go |
|---|---|---|---|
| `pdfs.create` (URL mode) | ✅ | ✅ | ✅ |
| `pdfs.create` (buffer mode) | ✅ same method | ✅ same method | ✅ separate `CreateBuffer` |
| `pdfs.get` | ✅ | ✅ | ✅ |
| Error hierarchy (17 types) | ✅ all exported | ✅ all defined | ✅ all defined |
| Retry + exponential backoff | ✅ | ✅ | ✅ |
| Idempotency (auto + manual) | ✅ | ✅ | ✅ |
| Per-request timeout | ✅ | ✅ | ✅ |
| Per-request maxRetries | ✅ | ✅ | ✅ |
| Env vars (`RENDIO_API_KEY`, `RENDIO_BASE_URL`) | ✅ | ✅ | ✅ |
| `.withResponse()` pattern | ✅ | ✅ | ✅ `*WithResponse` methods |
| `traceparent` injection | ✅ | ✅ | ✅ |
| Cancellation | ✅ `AbortSignal` | ❌ **missing** | ✅ `context.Context` |
| **OTEL parent span (`rendio.pdfs.create`)** | ✅ | ✅ | ✅ |
| OTEL attempt spans | ✅ | ✅ | ✅ |
| `rendio.mode` attribute | ✅ | ✅ | ✅ |
| `rendio.output` attribute | ✅ | ✅ | ✅ |
| `rendio.sdk.version` attribute | ✅ | ✅ | ✅ |
| `rendio.pages` attribute | ✅ | ✅ | ✅ |
| `rendio.size_bytes` attribute | ✅ | ✅ | ✅ |
| `rendio.chromium_version` attribute | ✅ | ✅ | ✅ |
| `error.type` attribute | ✅ | ✅ | ✅ |
| `http.response.status_code` (parent) | ✅ | ✅ | ✅ |
| `http.response.status_code` (attempt) | ✅ | ✅ | ✅ |
| `http.retry_count` (attempt) | ✅ | ✅ | ✅ |
| OTEL metrics (3 instruments) | ✅ | ✅ | ✅ |
| `pdfs.get` spans | ❌ | ✅ | ✅ |
| `span.recordException()` | ✅ | ✅ | ✅ |
| `trace_id`/`span_id` on errors | ✅ auto | ✅ auto (via `activate_span`) | ✅ auto |
| Attribute constants file | ✅ `attributes.ts` | ✅ `_attributes.py` | ✅ `attributes.go` |
| Server-Timing → span events | ✅ | ✅ | ✅ |
| `mapErrorCode` parity (html_too_large, connection_error) | ✅ | ✅ | ✅ |

### SDK Code Fixes — Status (updated 2026-04-19)

| # | Severity | SDK | Issue | Status |
|---|---|---|---|---|
| SDK1 | **HIGH** | Python | OTEL tracing entirely stubbed | ✅ DONE — full telemetry wired |
| SDK2 | **HIGH** | Go | Zero `rendio.*` parent span attributes | ✅ DONE — all attributes set via `setCreateParentAttrs` |
| SDK3 | **MEDIUM** | Python | No cancellation support | ❌ Still missing — no `AbortSignal` equivalent |
| SDK4 | **MEDIUM** | Go | No OTEL metrics | ✅ DONE — 3 instruments via `WithMeter` |
| SDK5 | **MEDIUM** | Python | `trace_id`/`span_id` not auto-populated | ✅ DONE — via `activate_span` context manager |
| SDK6 | **LOW** | Node | `pdfs.get` has no OTEL spans | ❌ Still missing — Go and Python have them |
| SDK7 | **LOW** | Go + Python | No attribute constants file | ✅ DONE — `attributes.go` and `_attributes.py` exist |

---

## 2. Gap-Filling (new pages/sections needed)

Ordered by user impact. Each item is scoped to the minimum viable addition.

| # | Priority | Scope | What to add | Why |
|---|---|---|---|---|
| G1 | **HIGH** | Section in `sdks/overview.mdx` | Python SDK: installation, async client usage, `asyncio` example | Python SDK has async support (`_client.py`, `_http.py`) but zero docs. POC 3 targets Python. |
| G2 | **HIGH** | Section in `sdks/overview.mdx` | Go SDK: installation, `context.Context` cancellation, module path `github.com/rendio/rendio-go` | Go SDK has full context support (`pdfs.go`, `http.go`) but zero docs. |
| G3 | **MEDIUM** | New section in `concepts/plans-and-quotas.mdx` | Annual pricing: Starter €190/year, Pro €490/year (≈17% discount) | `pricing.ts` has `annualPrice` fields; docs mention only monthly prices |
| G4 | **MEDIUM** | New section in `sdks/opentelemetry.mdx` or `concepts/plans-and-quotas.mdx` | Trace sharing feature: what it is, which plans have it | `traceSharing: true` on Starter+Pro in `pricing.ts`, never explained anywhere |
| G5 | **MEDIUM** | New section in `concepts/plans-and-quotas.mdx` | Custom CDN domain (Pro only): what it does, how to configure | `customDomainCdn: true` on Pro in `pricing.ts`, undocumented |
| G6 | **LOW** | Row in `plans-and-quotas.mdx` feature gates table | Priority rendering (Starter+Pro) | `priorityRendering` referenced in dashboard settings UI, but not in docs feature gates table |

---

## 3. Structural Changes

| # | Change | Rationale |
|---|---|---|
| S1 | Split `sdks/rendering.mdx` into per-language pages: `sdks/node.mdx`, `sdks/python.mdx`, `sdks/go.mdx` | Currently all three language cards in `sdks/overview.mdx` link to the same `rendering.mdx`. Python async patterns and Go context cancellation need dedicated space. |
| S2 | Update `sdks/overview.mdx` CardGroup to point to per-language pages after S1 | Housekeeping after the split |

---

## 4. No-Go Items (defer until feature ships)

| # | Item | Status | Action |
|---|---|---|---|
| N1 | **Webhooks** (`render.completed` / `render.failed`) | Listed as Pro feature in `pricing.ts` (`webhooks: true`), NO implementation in `apps/web/app/routes/api/` — only Paddle billing webhook exists | Do not document. Add a `guides/webhooks.mdx` stub only when the handler ships. |
| N2 | **Fillable PDF forms** | `fillableForms: true` in pricing, but TODO.md has it as V1.5+ | Do not document features. Can mention "coming soon" in plans comparison table if desired. |
| N3 | **CLI tool** | `tools/cli` does not exist anywhere in the repo. Was in an earlier nav draft. | Drop. Do not create placeholder page. |
| N4 | **Self-hosted workers** | No mention in code, no deployment tooling. Intentionally SaaS-only. | Do not document. |
| N5 | **OTEL export** | `otelExport: true` on Pro in pricing, but no UI or API for configuring export | Listed in comparison table already. Do not create implementation docs until the feature ships. |

---

## 5. Residual Items (from handoff §5, validated)

| # | Handoff item | Status | Action |
|---|---|---|---|
| R1 | Backoff table 4th retry row unreachable | Confirmed: `maxRetries: 2` → only rows 0–1 are default | → F5 above |
| R2 | "under 300 ms" vs benchmark 13–50 ms | Confirmed inconsistency | → F3, F4 above |
| R3 | "under 100 ms" in how-it-works.mdx | Confirmed same inconsistency | → F3 above |
| R4 | Express body limit comment | Confirmed: 1.5 MB more accurate | → F8 above |
| R5 | `margin` Zod permissiveness | Docs prose is fine; Zod allows `"not-a-unit"` | No change needed — note only if asked |
| R6 | `source` URL scheme (`ftp://`, `ws://`) | SSRF layer catches non-HTTP; Zod's `.url()` is permissive | No change needed — SSRF guard is the real constraint |

---

## 6. POC Dogfood — Findings

### POC 3 — curl + Python requests ✅

**Result**: Worked end-to-end. curl returned a valid PDF; Python script with idempotency and retry logic worked correctly.

**New findings**:
- F0 (CRITICAL): `api.rendio.dev` base URL is wrong everywhere → already captured above
- Idempotency "URL mode only" constraint buried mid-page in a table row → needs callout box
- "JSON body must be byte-identical across retries" is a surprising footgun → needs explicit warning with example

### POC 4 — Next.js App Router ⚠️

**Result**: Build passed, route compiled, SDK imported correctly. API call failed due to wrong base URL (F0).

**New findings**:

| # | Severity | File | Issue | Fix |
|---|---|---|---|---|
| F9 | ~~CRITICAL~~ **RESOLVED** | all SDK install instructions | `npm install rendio` fails — package not published to npm | Packages will be published before docs go live. No beta note needed. |
| F10 | **HIGH** | `use-cases/nextjs.mdx` | Route example missing `AuthenticationError` handling — the one error `error-handling.mdx` calls "stop immediately" | Add `AuthenticationError` branch to the catch block |
| F11 | **MEDIUM** | `use-cases/nextjs.mdx` | Buffer-mode streaming snippet has zero error handling | Add the same try/catch pattern from the URL-mode example |
| F12 | **LOW** | `use-cases/nextjs.mdx` | Edge runtime `AsyncLocalStorage` caveat buried in prose | Move to a `<Warning>` callout box |

### POC 2 — VSCode extension (not run)

Requires interactive VSCode UI. Options: (a) run manually, (b) redefine to test `@rendio/config` programmatically.

### POC 1 — Node SDK buffer invoice ✅

**Result**: Worked end-to-end after workarounds. Produced a valid 12.8 KB PDF.

**Findings** (all duplicates of existing items):
- `npm install rendio` fails → F9
- Wrong base URL → F0
- Agent claimed `ServerBusyError` not exported — **false positive** (verified: it is exported from `index.ts:17` and defined in `errors.ts:132`; agent's local build was likely incomplete)

---

## 7. Developer Reviewer Feedback

Four developer personas reviewed all 37 MDX files independently.

### Scores

| Persona | Score | One-line verdict |
|---|---|---|
| **Node/TS senior** | **7.5/10** | "DX is genuinely better than most. Missing operational credibility layer — no SLA, one changelog entry, no incident history." |
| **Python backend** | **4/10** | "Async client completely undocumented. No FastAPI example. Would not adopt from docs alone." |
| **Go developer** | **5/10** | "Happy-path is idiomatic. Everything past it (buffer mode, OTEL, server integration) falls back to Node." |
| **Frontend/fullstack** | **7.5/10** | "Error model and Render Config are strong. Missing Server Actions, browser download pattern, template gallery." |

### Top findings across all reviewers

**Strengths** (mentioned by 3+ reviewers):
1. Error handling docs are better than Stripe's — typed hierarchy + "what to do" branching
2. VSCode extension / Render Config is a genuine differentiator, well documented
3. Code samples are modern, copy-pasteable, and multi-language

**Weaknesses** (prioritized by frequency):

| # | Issue | Mentioned by | Priority |
|---|---|---|---|
| DX1 | **Python async client undocumented** — `AsyncRendio`, `httpx.AsyncClient`, `asyncio` patterns | Python, Node | **CRITICAL** |
| DX2 | **No FastAPI use-case page** (parallel to Express/Next.js) | Python, Frontend | **HIGH** |
| DX3 | **No Server Actions example** in Next.js page | Frontend | **HIGH** |
| DX4 | **No browser download pattern** (`fetch` → `blob()` → `URL.createObjectURL`) | Frontend | **HIGH** |
| DX5 | **No operational credibility** — one changelog entry, no SLA, no status page link, no incident history | Node | **HIGH** |
| DX6 | **Go buffer mode undocumented** | Go | **HIGH** (now fixed — F14) |
| DX7 | **Go OTEL setup missing** | Go | **HIGH** (now fixed — F15) |
| DX8 | **Return types table is Node-only** — no Python/Go field docs | Python, Go | **MEDIUM** |
| DX9 | **Invoice template uses Google Fonts without noting it requires Starter** (Free-tier footgun) | Node | **MEDIUM** |
| DX10 | **Tailwind-to-PDF workflow hand-wavy** — no `tailwind.config.js`, no `print:` variant docs | Frontend | **MEDIUM** |
| DX11 | **Template gallery thin** — only 1 invoice. No report, receipt, label, contract | Frontend | **MEDIUM** |
| DX12 | **No test helper / mock client pattern** for Go or Python | Go | **LOW** |
| DX13 | **`meta.rateLimit` shape undocumented** in SDK return types | Node | **LOW** |
| DX14 | **Hello-world Go ignores error** — `client, _ := rendio.NewClient()` | Go | **LOW** |

---

## Execution Status

| Batch | Items | Status |
|---|---|---|
| **Batch 0** | F0 (base URL ×19) + F9 (SDK install note) | ✅ DONE |
| **Batch 1** | F1 (openapi bearerFormat) + F2 (rk_test_) + F13 (Go RetryAfter) | ✅ DONE |
| **Batch 2** | F3 + F4 (performance claims) | ✅ DONE |
| **Batch 3** | F5 (backoff table) + F6 + F7 + F17 (OTEL attrs/spans) | ✅ DONE |
| **Batch 4** | F10 + F11 + F12 (Next.js fixes) | ✅ DONE |
| **Batch 5a** | F14 (Go buffer-mode docs) + F15 (Go OTEL setup) | ✅ DONE |
| **Batch 5b** | Idempotency UX (URL-mode callout + byte-identical warning) | ✅ DONE |
| **Batch 6** | F8 (Express comment) | ✅ DONE |
| **Batch 7** | F16 (OTEL parity columns + note) | ✅ DONE |

### Remaining work (not yet implemented)

| # | Item | Priority | Status |
|---|---|---|---|
| 1 | DX1 — Python async client docs (`AsyncRendio`, FastAPI pattern) | CRITICAL | ✅ DONE — section added to `rendering.mdx` (previous session) |
| 2 | DX2 — FastAPI use-case page | HIGH | ✅ DONE — example added inline in `rendering.mdx` |
| 3 | DX3 — Server Actions example in `nextjs.mdx` | HIGH | Open |
| 4 | DX4 — Browser download pattern | HIGH | Open |
| 5 | DX5 — Operational credibility (status page link, SLA note) | HIGH | Open |
| 6 | G3–G6 — Pricing gap-fills (annual, trace sharing, custom CDN, priority rendering) | MEDIUM | Open |
| 7 | S1/S2 — Per-language SDK page split | MEDIUM | Deferred — current structure works |
| 8 | DX8 — Python/Go return types table | MEDIUM | ✅ DONE — added to `rendering.mdx` |
| 9 | DX9 — Google Fonts footgun note in invoice template | MEDIUM | Open |
| 10 | DX10 — Tailwind-to-PDF config | MEDIUM | Open |
| 11 | DX14 — Fix Go hello-world error handling | LOW | ✅ DONE (previous session) |
| 12 | DX13 — Document `meta.rateLimit` shape | LOW | ✅ DONE — added to `rendering.mdx` |
| 13 | Overage spending cap dashboard walkthrough | LOW | Open |
| 14 | OTEL parity table update (Python/Go now full) | HIGH | ✅ DONE |
| 15 | OTEL metrics documentation (3 instruments) | HIGH | ✅ DONE |
| 16 | Go `Meter`/`WithMeter` interface docs | MEDIUM | ✅ DONE |
| 17 | Phase events (Server-Timing) docs | MEDIUM | ✅ DONE |
| 18 | `pdfs.get` spans — Go + Python (not Node) | LOW | ✅ DONE |

### SDK code fixes (separate from docs)

See §1b above for the 7 SDK code issues (SDK1–SDK7). Most are now resolved.

---

## 8. PDFShift Competitive Feature Comparison (2026-04-19)

Features PDFShift has that Rendio does NOT:

| Feature | PDFShift | Rendio | Priority |
|---|---|---|---|
| PDF password protection / encryption | `protection.password`, `no_print`, `no_copy`, `no_modify` | Not implemented | Roadmap candidate |
| Text/image watermarks | Full styling (font, color, opacity, rotation, position) | Not implemented | Roadmap candidate |
| Webhook/async mode | `webhook` URL for callback | Listed in pricing but NOT built | Already deferred (N1) |
| S3 direct upload | Upload to user's S3 bucket | CDN URLs instead | Different approach |
| PDF metadata | Author, title, subject, keywords | Not implemented | Low priority |
| GDPR/HIPAA compliance modes | `is_gdpr`, `is_hipaa` flags | Not implemented | Roadmap candidate for enterprise |
| Template CRUD API | Full CRUD endpoints for reusable templates | Handlebars `data` param only | Different approach |
| Blank page removal | `remove_blank` | Not implemented | Nice to have |
| WebP output | `/convert/webp` | PNG/JPEG only | Low priority |

Features Rendio has that PDFShift does NOT:

| Feature | Notes |
|---|---|
| Typed SDK error hierarchy (17 types) | PDFShift has generic errors only |
| OpenTelemetry integration in SDKs | Zero OTEL support in PDFShift |
| Idempotency keys for safe retries | No idempotency in PDFShift |
| VSCode extension with live preview | Unique differentiator |
| CDN-hosted output URLs with expiration | PDFShift uses S3 or raw binary |
| Server-Timing headers with phase breakdown | No render phase visibility |
| Rate limit headers (X-RateLimit-*) | PDFShift has these too |
| `printBackground` control | PDFShift uses `disable_backgrounds` (inverse) |
| `pageRanges` param | PDFShift has `pages` param (similar) |
| `tagged` (accessible PDF) | No accessibility support in PDFShift |
| `outline` (PDF bookmarks from headings) | Not in PDFShift |
| `encode: "base64"` in buffer mode | PDFShift has `encode` too |
| `waitForSelector` (CSS selector) | PDFShift has `wait_for` (JS function, different mechanism) |
