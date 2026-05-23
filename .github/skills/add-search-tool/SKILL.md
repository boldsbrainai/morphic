---
name: add-search-tool
description: 'Guide for adding a new search or retrieval tool to the Morphic codebase. Use when asked to: add tool, new search tool, new retrieval tool, add web search, add tool to morphic, implement tool.'
---

# Adding a New Tool to Morphic

Morphic has **two streaming paths** — both must be updated for every new tool.

| Path | Models | Entry point |
|------|--------|-------------|
| Tool-calling | OpenAI, Anthropic, DeepSeek, Groq, Fireworks | `lib/agents/researcher.ts` |
| Manual | Ollama, Google | `lib/agents/manual-researcher.ts` + `lib/streaming/tool-execution.ts` |

---

## Step 1 — Create the tool file

`lib/tools/<tool-name>.ts`

```ts
import { tool } from 'ai'
import { myToolSchema } from '../schema/<tool-name>'

export const myTool = tool({
  description: 'What this tool does.',
  parameters: myToolSchema,
  execute: async ({ query }) => {
    // call external API, return results
    return { results: [] }
  }
})
```

## Step 2 — Create the Zod schema

`lib/schema/<tool-name>.tsx`

```ts
import { z } from 'zod'

export const myToolSchema = z.object({
  query: z.string().describe('Search query')
})

export type MyToolResult = { results: Array<{ title: string; url: string }> }
```

## Step 3 — Register in the tool-calling path

`lib/agents/researcher.ts` — add to the `tools` object:

```ts
import { myTool } from '../tools/<tool-name>'

// inside tools: { ... }
myTool,
```

## Step 4 — Add XML description to the manual path system prompt

`lib/agents/manual-researcher.ts` — append inside the tools description block:

```
<tool_description>
<tool_name>myTool</tool_name>
<description>What this tool does.</description>
<parameters>
  <parameter><name>query</name><type>string</type><description>Search query</description></parameter>
</parameters>
</tool_description>
```

## Step 5 — Handle the XML tool call in manual execution

`lib/streaming/tool-execution.ts` — add a `case` to `executeToolCall()`:

```ts
import { myTool } from '../tools/<tool-name>'

// inside switch (toolName):
case 'myTool': {
  const args = myToolSchema.parse(parsedArgs)
  return await myTool.execute!(args, { messages: [], toolCallId: '' })
}
```

## Step 6 — Create the UI component

`components/<tool-name>-section.tsx`

```tsx
import { MyToolResult } from '@/lib/schema/<tool-name>'

export function MyToolSection({ result }: { result: MyToolResult }) {
  return (
    <div>
      {result.results.map((r, i) => (
        <a key={i} href={r.url}>{r.title}</a>
      ))}
    </div>
  )
}
```

## Step 7 — Register the UI component in the message renderer

`components/render-message.tsx` — add a case in the tool invocation dispatch:

```tsx
import { MyToolSection } from './my-tool-section'

// inside the toolInvocations map / switch:
case 'myTool':
  return <MyToolSection result={toolInvocation.result} />
```

> Complete all 7 steps above for both streaming paths.
