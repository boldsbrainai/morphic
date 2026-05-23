# Morphic — Agent Guide

AI-powered search engine with Generative UI. Stack: Next.js 15 App Router, Vercel AI SDK v4, TypeScript, Tailwind CSS, shadcn/ui. Package manager: **bun**.

See [README.md](README.md) for features and stack overview. See [CONTRIBUTING.md](CONTRIBUTING.md) for contribution workflow.

---

## Build & Dev Commands

```sh
bun install          # install dependencies
bun dev              # dev server (Next.js + Turbopack)
bun build            # production build
bun lint             # ESLint check
docker compose up -d # start SearXNG + Redis (optional)
```

No test runner is configured.

---

## Environment Setup

Full details: [docs/CONFIGURATION.md](docs/CONFIGURATION.md)

```sh
cp .env.local.example .env.local
```

Minimum required: `OPENAI_API_KEY` + `TAVILY_API_KEY`.

---

## Architecture

### Two Streaming Paths (`app/api/chat/route.ts`)

```
isToolCallSupported(model)
  ├─ true  → createToolCallingStreamResponse()   # OpenAI, Anthropic, DeepSeek, Groq, Fireworks
  └─ false → createManualToolStreamResponse()    # Ollama, Google
```

The manual path parses XML tool-call tags; update `lib/streaming/tool-execution.ts` when adding tools.

### Model Addressing

Always use `"provider:model-id"` format — e.g., `"openai:gpt-4o-mini"`, `"anthropic:claude-3-5-haiku-20241022"`.  
`getModel()` in `lib/utils/registry.ts` parses this string and routes to the correct provider SDK.

### Three Tools

| Tool          | File                        | Provider selection                                                              |
| ------------- | --------------------------- | ------------------------------------------------------------------------------- |
| `search`      | `lib/tools/search.ts`       | `SEARCH_API` env var → Tavily / Exa / SearXNG                                   |
| `retrieve`    | `lib/tools/retrieve.ts`     | Jina / Tavily Extract — **user-provided URLs only** (enforced in system prompt) |
| `videoSearch` | `lib/tools/video-search.ts` | Serper API — requires `SERPER_API_KEY`                                          |

### Shared UI State

`CHAT_ID = 'search'` (defined in `lib/constants/index.ts`) is passed as `id` to every `useChat()` call. All UI components share the same chat state through this constant.

### Search Mode

Controlled by the `search-mode` cookie. When active, `maxSteps: 5` enables multi-turn tool calling.

---

## Non-Obvious Conventions

- **`ExtendedCoreMessage` with `role: 'data'`** — tool results and related questions are stored as data annotations, not assistant messages.
- **DeepSeek-R1 reasoning** — models matching `/deepseek-r1/i` automatically receive `extractReasoningMiddleware({ tagName: 'think' })` in `lib/utils/registry.ts`.
- **`NEXT_PUBLIC_*` vars are client-bundled** — never place secrets in them.
- **`maxDuration = 30`** in `route.ts` — Vercel streaming timeout; complex multi-step chains may hit this limit.
- **Redis is optional** — chat history fails silently when Redis is not configured.
- **SearXNG advanced depth** calls `/api/advanced-search` internally — requires `NEXT_PUBLIC_BASE_URL` to be set.

---

## Adding a New AI Provider

Reference: `lib/utils/registry.ts`, `lib/types/models.ts`

1. Add provider instance to `createProviderRegistry()` in `lib/utils/registry.ts`.
2. Add model entries in `lib/types/models.ts`.
3. Update `isToolCallSupported()` if the provider handles tool calling natively.

## Adding a New Tool

Reference: existing tools in `lib/tools/`

1. Create tool file in `lib/tools/`.
2. Add Zod schema in `lib/schema/`.
3. Register in `lib/agents/researcher.ts` (tool-calling path).
4. Handle XML tag in `lib/streaming/tool-execution.ts` (manual path).
5. Add UI component in `components/` and wire it in `components/render-message.tsx`.

---

## Commit Conventions

See [CONTRIBUTING.md](CONTRIBUTING.md). Format: `feat:`, `fix:`, `docs:`, `chore:`, `refactor:`. Branch from `main` as `feat/your-feature-name`.
