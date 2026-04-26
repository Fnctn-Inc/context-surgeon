# Context Surgeon

**Context Surgeon** is a Big Berlin Hack submission that turns scattered operating data into a source-grounded context repository for AI agents.

Agents should not act on stale, contradictory, or unverifiable context. Context Surgeon compiles messy source material into facts, conflicts, Markdown/VFS files, provenance, and surgical **Fact Patch** updates that preserve human edits.

- Live product: [contextsurgeon.fnctn.io](https://contextsurgeon.fnctn.io/)
- Public repository: [github.com/fnctn/context-surgeon](https://github.com/fnctn/context-surgeon)
- Demo route: [contextsurgeon.fnctn.io/demo](https://contextsurgeon.fnctn.io/demo)

## Hackathon Fit

Primary tracks:

- **Buena: The Context Engine**: property context compiler, source relevance, conflict detection, generated `context.md`, surgical updates, and agent pre-flight checks.
- **Qontext**: inspectable virtual file system, fact graph, provenance ledger, source references, and human review queue.

Official hackathon partner technologies used:

| Partner | Role |
|---|---|
| Google DeepMind / Gemini | Agent context-check reasoning and structured extraction adapter contract. |
| Tavily | Public/vendor enrichment and verification adapter. |
| Pioneer / Fastino | Schema-first source classification, relevance scoring, and extraction hints. |

Supporting infrastructure:

| System | Role |
|---|---|
| Composio | Live connector layer for email, CRM, files, support, collaboration, work tracking, and manual evidence. |
| Firebase Auth | Protects persisted workspace mutations and live-mode APIs. |
| Cloudflare Workers + D1 | Edge deployment and persisted workspace/audit state. |

## Demo Story

The sample property is `Sonnenallee 44, 12045 Berlin`.

The demo shows:

1. Load messy handover sources.
2. Compile sources into a fact ledger with source quotes and spans.
3. Detect contradictions before an agent acts.
4. Generate Markdown context files and machine exports.
5. Add a human note to `context.md`.
6. Ingest a new owner email.
7. Propose a minimal Fact Patch instead of regenerating the whole file.
8. Apply the patch while preserving the human note.
9. Show the downstream agent action plan change because context changed.

The memorable moment is **Fact Patch**: generated blocks update, stale facts are superseded, and human-written context remains intact.

## Product Surfaces

- `/` — marketing landing page with product proof, Qontext graph modal, live mode, and API section.
- `/demo` — public sample-data workbench and live-intake panel.
- `/login` — Firebase Auth sign-in/sign-up.
- `/api/*` — workspace, provider, Qontext proof, integration, and live-mode APIs.

## Live Mode API

The live API mirrors the workbench:

| Method | Route | Purpose | Auth |
|---|---|---|---|
| `GET` | `/api/live/connectors` | List supported connectors and connection state. | Public demo; Firebase for live state |
| `GET/POST` | `/api/live/connect` | Show contract or create a Composio OAuth connect link. | Firebase for `POST` |
| `GET/POST` | `/api/live/sync` | Show contract or sync selected connected systems. | Firebase for `POST` |
| `GET/POST` | `/api/live/upload` | Show contract or upload evidence files. | Firebase for `POST` |
| `GET/POST` | `/api/live/rules` | Inspect or activate scoped ingestion rules. | Firebase for live activation |
| `GET` | `/api/live/sources` | Inspect candidate-source output. | Firebase |

Example upload contract:

```bash
curl -X POST https://contextsurgeon.fnctn.io/api/live/upload \
  -H "Authorization: Bearer <FIREBASE_ID_TOKEN>" \
  -F "files=@owner-email.txt"
```

The response is a candidate-source payload with source metadata, confidence, fact hints, and next actions for review/compile/publish.

## Architecture

```text
demo_data/properties/sonnenallee-44/
  -> ingestion + chunking
  -> provider adapter contracts
       Gemini: context check + extraction contract
       Tavily: enrichment proof
       Pioneer/Fastino: classification hints
       Composio: connected-app candidate-source normalization
  -> fact ledger + source spans
  -> conflict detector
  -> Markdown virtual file system
  -> Fact Patch proposal/apply flow
  -> agent context health check
  -> Cloudflare D1 workspace store when deployed
```

Project layout:

```text
app/                              Next.js app routes, UI, and API routes
lib/context-surgeon/              Pure TypeScript context engine
lib/firebase/                     Firebase client config and server token verification
demo_data/properties/sonnenallee-44/
                                  Synthetic demo property artifacts
migrations/                       Cloudflare D1 schema
docs/hackathon/                   PRD, feature spec, demo script, release checklist
tests/                            Vitest coverage for critical product behavior
```

## Local Setup

Prerequisites:

- Node.js compatible with Next.js 16
- `pnpm` 10.x

Install dependencies:

```bash
pnpm install
```

Run locally:

```bash
pnpm dev
```

Open:

```text
http://localhost:3000
```

The public demo works without API keys. By default, local development can run in deterministic mock mode using bundled demo data.

## Environment Variables

Do not commit `.env`, `.env.local`, provider keys, Firebase secrets, Cloudflare secrets, or Composio credentials.

Use this as an example only:

```bash
# Provider mode
PROVIDER_MODE=mock
INTEGRATIONS_MODE=demo

# Google DeepMind / Gemini
GEMINI_API_KEY=<your-gemini-api-key>

# Tavily
TAVILY_API_KEY=<your-tavily-api-key>

# Pioneer / Fastino
PIONEER_API_KEY=<your-pioneer-api-key>

# Firebase Auth
FIREBASE_API_KEY=<your-firebase-web-api-key>
FIREBASE_AUTH_DOMAIN=<your-firebase-auth-domain>
FIREBASE_PROJECT_ID=<your-firebase-project-id>
FIREBASE_APP_ID=<your-firebase-app-id>

# Composio
COMPOSIO_API_KEY=<your-composio-api-key>
COMPOSIO_REDIRECT_URI=<your-composio-redirect-uri>
COMPOSIO_AUTH_CONFIG_GMAIL=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_OUTLOOK=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_HUBSPOT=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_SALESFORCE=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_GOOGLEDRIVE=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_ONEDRIVE=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_DROPBOX=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_SHAREPOINT=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_GOOGLESHEETS=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_NOTION=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_SLACK=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_MICROSOFT_TEAMS=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_ZENDESK=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_INTERCOM=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_JIRA=<your-composio-auth-config-id>
COMPOSIO_AUTH_CONFIG_LINEAR=<your-composio-auth-config-id>
```

For production, configure provider values as Cloudflare Worker secrets or environment bindings outside git.

## Verification

Run before recording or submitting:

```bash
pnpm lint
pnpm test
pnpm build
pnpm cf:build
```

Current test coverage includes:

- Human notes survive regeneration.
- Human edits inside generated blocks create patch conflicts.
- Roof status and invoice amount contradictions are detected.
- Source quotes map to source span metadata.
- Qontext-style exports are generated from the same fact ledger.
- Agent check changes after applying a Fact Patch.
- Public/protected API contracts reject unauthenticated mutations.
- Live API contracts for connectors, sync, upload, rules, and sources.

## Cloudflare Deployment

The app is configured for Cloudflare Workers through OpenNext:

- `open-next.config.ts`
- `wrangler.jsonc`
- `@opennextjs/cloudflare`
- D1 binding: `CONTEXT_SURGEON_DB`

Build for Cloudflare:

```bash
pnpm cf:build
```

Deploy:

```bash
pnpm cf:deploy
```

Apply D1 migrations:

```bash
pnpm wrangler d1 migrations apply context-surgeon-db --remote
```

## Submission Checklist

Hackathon requirements covered:

- Public deployed product.
- Public GitHub repository.
- Comprehensive README with setup instructions.
- Docs pack under `docs/hackathon/`.
- At least three partner technologies.
- 2-minute-safe demo path in `/demo`.
- Public sample-data demo works without sign-in.
- Protected live-mode APIs are Firebase-scoped.
- No committed env files or real API keys.

## Documentation

- [Hackathon docs pack](docs/hackathon/README.md)
- [Demo script](docs/hackathon/context-surgeon-demo-script.md)
- [Release checklist](docs/hackathon/context-surgeon-release-checklist.md)
- [Product strategy](docs/hackathon/context-surgeon-product-strategy.md)
- [PRD](docs/hackathon/context-surgeon-prd.md)
- [Feature spec](docs/hackathon/context-surgeon-feature-spec.md)
- [Live integrations API](docs/hackathon/context-surgeon-live-integrations-api.md)
- [Session handoff](docs/hackathon/session-handoff.md)
