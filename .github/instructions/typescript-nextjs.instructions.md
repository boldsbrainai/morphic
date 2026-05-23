---
description: "Use when writing TypeScript or Next.js code in this project. Keywords: component, hook, server action, model, cookie, chat, API route, type, import"
applyTo: "**/*.tsx,**/*.ts"
---

## Server vs Client Components

- Files in `app/` are React Server Components by default — add `'use client'` only when using hooks, event handlers, or browser APIs.
- Server Actions in `lib/actions/` must start with `'use server'`.

## CHAT_ID Constant

- Never hardcode the string `'search'` — always import `CHAT_ID` from `lib/constants/index.ts`.
- Pass it as `id` to every `useChat()` call: `useChat({ id: CHAT_ID })`.

## Model Addressing

- Always reference models as `"provider:model-id"` strings (e.g., `"openai:gpt-4o-mini"`, `"anthropic:claude-3-5-haiku-20241022"`).
- Never use bare model names. `getModel()` in `lib/utils/registry.ts` parses this format.

## Class Merging

- Use `cn()` from `lib/utils/index.ts` for conditional class merging — no inline styles.
- Tailwind CSS only.

## Environment Variables

- Never put secrets in `NEXT_PUBLIC_*` variables — they are bundled into the client.
- `NEXT_PUBLIC_*` is only for non-secret config that must be visible in the browser.

## Types

- Use `ExtendedCoreMessage` (from `lib/types/`) as the message type — it extends the AI SDK's `CoreMessage` with `role: 'data'`.
- Use `import type { ... }` for type-only imports.
- All Zod schemas go in `lib/schema/`.

## Path Aliases

- Use `@/` for all imports instead of relative paths (`@/lib/utils`, `@/components/ui/button`).

## Component Structure

- shadcn/ui components → `components/ui/`
- Feature components → `components/`
