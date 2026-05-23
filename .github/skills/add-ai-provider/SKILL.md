---
name: add-ai-provider
description: 'Add a new AI provider or LLM model to the Morphic project. Use when asked to: add provider, new AI provider, integrate LLM provider, register provider, add model, new model, connect to a new LLM, support a new API. Covers SDK install, registry setup, model list, streaming path, tool-call config, and environment variable.'
---

# Add AI Provider Skill

Guides you through registering a new AI provider (and its models) in Morphic end-to-end.

## Step 1 — Install the provider SDK

```sh
bun add @ai-sdk/<provider>
# e.g. bun add @ai-sdk/mistral
```

## Step 2 — Register the provider in `lib/utils/registry.ts`

Inside `createProviderRegistry()`, import and add the provider instance:

```ts
import { createMistral } from '@ai-sdk/mistral'

// inside createProviderRegistry():
mistral: createMistral({ apiKey: process.env.MISTRAL_API_KEY }),
```

For providers that require `simulateStreaming` (e.g. Ollama):

```ts
ollama: createOllama({ simulateStreaming: true }),
```

## Step 3 — Add model entries in `lib/types/models.ts`

Append to the exported models array. Shape must be exact:

```ts
{
  id: 'mistral:mistral-large-latest',   // ALWAYS "provider:model-id"
  name: 'Mistral Large',
  provider: 'Mistral',
  providerId: 'mistral',
  enabled: true,
  toolCallType: 'native',               // 'native' | 'manual' | 'none'
}
```

`toolCallType` values:
- `'native'` — provider handles tool calling via the SDK (OpenAI, Anthropic, Groq, Fireworks, DeepSeek, Mistral…)
- `'manual'` — XML-tag parsing path (Ollama, Google)
- `'none'` — no tool calling support

## Step 4 — Update `isToolCallSupported()` in `lib/utils/registry.ts`

This function controls which streaming path is used. Add the new provider:

```ts
// Native tool-calling → returns true
return provider === 'openai' || provider === 'anthropic' || provider === 'mistral' /* add here */

// Manual path (Ollama, Google) → returns false — already handled
```

## Step 5 — Update `getToolCallModel()` (manual path only)

Only needed if `toolCallType === 'manual'`. Point it to the helper model that will parse XML tool calls:

```ts
if (provider === 'google') return 'google:gemini-2.0-flash'
// add your provider case here if required
```

## Step 6 — Add the API key to `.env.local.example`

```sh
# .env.local.example
MISTRAL_API_KEY=          # Mistral — https://console.mistral.ai/
```

Then add to your own `.env.local`:

```sh
MISTRAL_API_KEY=your-key-here
```

## Step 7 (optional) — Reasoning middleware

Models matching `/deepseek-r1/i` automatically receive `extractReasoningMiddleware({ tagName: 'think' })` in `getModel()`. For other reasoning models, add a similar regex check in `lib/utils/registry.ts` → `getModel()`.

