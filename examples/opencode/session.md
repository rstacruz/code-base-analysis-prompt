---
createdAt: 2025-07-22T13:32:28Z
---

# Session Module

## Overview

The `Session` module (`packages/opencode/src/session/index.ts`) is the core component for managing user interaction sessions within the OpenCode application. It handles the creation, persistence, and lifecycle of sessions, including messages, tool calls, and snapshots. It integrates with various other modules to provide a comprehensive conversational AI experience.

## Architecture

The `Session` module uses `App.state` to manage its in-memory state, including active sessions, messages, and pending operations. Sessions are persisted to storage using the `Storage` module. It orchestrates interactions with language models via the `Provider` module, handles tool execution, and manages the flow of messages. The module also supports session sharing, summarization, and reverting to previous states. It defines various event types for session and message updates, which are published through the `Bus` module.

```mermaid
graph TD
    A["Session Module"] --> B{"Session.state (App.state)"}
    B --> C["sessions Map"]
    B --> D["messages Map"]
    B --> E["pending Map (AbortController)"]
    B --> F["queued Map (ChatInput)"]

    A --> G["Session.create()"]
    G --> H["Identifier.descending(\"session\")"]
    G --> I["Storage.writeJSON"]
    G --> J["Config.get()"]
    G --> K["Flag.OPENCODE_AUTO_SHARE"]
    G --> L["Session.share()"]
    G --> M["Bus.publish(Event.Updated)"]

    A --> N["Session.chat()"]
    N --> O["Identifier.ascending(\"message\")"]
    N --> P["App.info()"]
    N --> Q["ReadTool.execute()"]
    N --> R["LSP.documentSymbol()"]
    N --> S["Provider.getModel()"]
    N --> T["Provider.getSmallModel()"]
    N --> U["SystemPrompt.title()"]
    N --> V["SystemPrompt.environment()"]
    N --> W["SystemPrompt.custom()"]
    N --> X["Mode.get()"]
    N --> Y["Provider.tools()"]
    N --> Z["MCP.tools()"]
    N --> AA["streamText()"]
    N --> BB["createProcessor()"]
    BB --> CC["Snapshot.create()"]
    BB --> DD["updatePart()"]
    BB --> EE["updateMessage()"]
    FF["getUsage()"]

    A --> GG["Session.summarize()"]
    GG --> HH["SystemPrompt.summarize()"]
    GG --> AA
    GG --> BB

    A --> II["Session.share()"]
    II --> JJ["Share.create()"]
    II --> KK["Storage.writeJSON"]
    II --> LL["Share.sync()"]

    A --> MM["Session.unshare()"]
    MM --> NN["Storage.remove()"]
    MM --> OO["Share.remove()"]

    A --> PP["Session.remove()"]
    PP --> QQ["Session.abort()"]
    PP --> RR["Session.children()"]
    PP --> SS["Storage.removeDir()"]
    PP --> TT["Bus.publish(Event.Deleted)"]

    A --> UU["Session.revert()"]
    A --> VV["Session.unrevert()"]
    VV --> WW["Snapshot.restore()"]

    A --> XX["Session.messages()"]
    XX --> YY["Storage.list()"]
    XX --> ZZ["Storage.readJSON"]

    A --> AAA["Session.get()"]
    AAA --> BBB["Storage.readJSON"]

    A --> CCC["Session.list()"]
    CCC --> YY

    A --> DDD["Session.abort()"]

    A --> EEE["Session.initialize()"]
    EEE --> N
    EEE --> I
```

## Data Models

### Session.Info

Represents the core information about a user session.

**Schema:**

```typescript
export const Info = z
  .object({
    id: Identifier.schema("session"),
    parentID: Identifier.schema("session").optional(),
    share: z
      .object({
        url: z.string(),
      })
      .optional(),
    title: z.string(),
    version: z.string(),
    time: z.object({
      created: z.number(),
      updated: z.number(),
    }),
    revert: z
      .object({
        messageID: z.string(),
        part: z.number(),
        snapshot: z.string().optional(),
      })
      .optional(),
  })
  .openapi({
    ref: "Session",
  })
export type Info = z.output<typeof Info>
```

**Overview:**

- `id`: Unique identifier for the session.
- `parentID`: Optional ID of the parent session, indicating a sub-session.
- `share`: Optional object containing sharing information (e.g., URL).
- `title`: A descriptive title for the session.
- `version`: The OpenCode version that created the session.
- `time`: Object containing creation and last update timestamps.
- `revert`: Optional object containing information for reverting the session to a previous state.

**Sources:** `packages/opencode/src/session/index.ts:44-69`

### Session.ShareInfo

Represents information related to session sharing.

**Schema:**

```typescript
export const ShareInfo = z
  .object({
    secret: z.string(),
    url: z.string(),
  })
  .openapi({
    ref: "SessionShare",
  })
export type ShareInfo = z.output<typeof ShareInfo>
```

**Overview:**

- `secret`: A secret key for managing the shared session.
- `url`: The URL where the session is shared.

**Sources:** `packages/opencode/src/session/index.ts:71-78`

### Session.Event

Defines various event types related to session lifecycle.

**Schema:**

```typescript
export const Event = {
  Updated: Bus.event(
    "session.updated",
    z.object({
      info: Info,
    }),
  ),
  Deleted: Bus.event(
    "session.deleted",
    z.object({
      info: Info,
    }),
  ),
  Idle: Bus.event(
    "session.idle",
    z.object({
      sessionID: z.string(),
    }),
  ),
  Error: Bus.event(
    "session.error",
    z.object({
      sessionID: z.string().optional(),
      error: MessageV2.Assistant.shape.error,
    }),
  ),
}
```

**Overview:**

- `Updated`: Published when a session's information is updated.
- `Deleted`: Published when a session is deleted.
- `Idle`: Published when a session becomes idle (e.g., after a chat response).
- `Error`: Published when an error occurs within a session.

**Sources:** `packages/opencode/src/session/index.ts:80-107`

### Session.ChatInput

Represents the input structure for a chat message within a session.

**Schema:**

```typescript
export const ChatInput = z.object({
  sessionID: Identifier.schema("session"),
  messageID: Identifier.schema("message").optional(),
  providerID: z.string(),
  modelID: z.string(),
  mode: z.string().optional(),
  tools: z.record(z.boolean()).optional(),
  parts: z.array(
    z.discriminatedUnion("type", [
      MessageV2.TextPart.omit({
        messageID: true,
        sessionID: true,
      })
        .partial({
          id: true,
        })
        .openapi({
          ref: "TextPartInput",
        }),
      MessageV2.FilePart.omit({
        messageID: true,
        sessionID: true,
      })
        .partial({
          id: true,
        })
        .openapi({
          ref: "FilePartInput",
        }),
    ]),
  ),
})
export type ChatInput = z.infer<typeof ChatInput>
```

**Overview:**

- `sessionID`: The ID of the session the message belongs to.
- `messageID`: Optional ID for the message.
- `providerID`: The ID of the language model provider to use.
- `modelID`: The ID of the language model to use.
- `mode`: Optional mode for the chat (e.g., "plan", "build").
- `tools`: Optional record of tools to enable/disable.
- `parts`: An array of message parts (text or file).

**Sources:** `packages/opencode/src/session/index.ts:300-329`

### MessageV2.Info

Represents a message in a session, either from a user or an assistant.

**Schema:**

```typescript
export const Info = z.discriminatedUnion("role", [User, Assistant]).openapi({
  ref: "Message",
})
export type Info = z.infer<typeof Info>
```

**Sources:** `packages/opencode/src/session/message-v2.ts:209-212`

### MessageV2.Part

Represents a part of a message, such as text, a file, a tool call, or a step in the AI's thought process.

**Schema:**

```typescript
export const Part = z
  .discriminatedUnion("type", [TextPart, FilePart, ToolPart, StepStartPart, StepFinishPart, SnapshotPart])
  .openapi({
    ref: "Part",
  })
export type Part = z.infer<typeof Part>
```

**Sources:** `packages/opencode/src/session/message-v2.ts:199-204`

### Mode.Info

Represents the configuration for a specific operational mode (e.g., "build", "plan").

**Schema:**

```typescript
export const Info = z
  .object({
    name: z.string(),
    model: z
      .object({
        modelID: z.string(),
        providerID: z.string(),
      })
      .optional(),
    prompt: z.string().optional(),
    tools: z.record(z.boolean()),
  })
  .openapi({
    ref: "Mode",
  })
export type Info = z.infer<typeof Info>
```

**Sources:** `packages/opencode/src/session/mode.ts:6-19`

## Features

### Create Session (`Session.create`)

Creates a new session, generates a unique ID, sets initial metadata, and persists it to storage. It also handles automatic session sharing if configured.

**Call graph analysis:**

- `Session.create` → `Identifier.descending`
- `Session.create` → `Storage.writeJSON`
- `Session.create` → `Config.get`
- `Session.create` → `Flag.OPENCODE_AUTO_SHARE`
- `Session.create` → `Session.share`
- `Session.create` → `Bus.publish(Event.Updated)`

**Code example:**

```typescript
// packages/opencode/src/session/index.ts:120-147
export async function create(parentID?: string) {
  const result: Info = {
    id: Identifier.descending("session"),
    version: Installation.VERSION,
    parentID,
    title: (parentID ? "Child session - " : "New Session - ") + new Date().toISOString(),
    time: {
      created: Date.now(),
    },
  }
  log.info("created", result)
  state().sessions.set(result.id, result)
  await Storage.writeJSON("session/info/" + result.id, result)
  const cfg = await Config.get()
  if (!result.parentID && (Flag.OPENCODE_AUTO_SHARE || cfg.share === "auto"))
    share(result.id)
      .then((share) => {
        update(result.id, (draft) => {
          draft.share = share
        })
      })
      .catch(() => {
        // Silently ignore sharing errors during session creation
      })
  Bus.publish(Event.Updated, {
    info: result,
  })
  return result
}
```

**Sources:** `packages/opencode/src/session/index.ts:120-147`

### Chat Interaction (`Session.chat`)

Handles the core chat interaction logic. It processes user messages, integrates with language models, executes tools, and manages the streaming of assistant responses. It also includes logic for message queuing, session summarization, and error handling.

**Call graph analysis:**

- `Session.chat` → `Identifier.ascending`
- `Session.chat` → `ReadTool.execute`
- `Session.chat` → `LSP.documentSymbol`
- `Session.chat` → `Provider.getModel`
- `Session.chat` → `Provider.getSmallModel`
- `Session.chat` → `SystemPrompt.title`
- `Session.chat` → `SystemPrompt.environment`
- `Session.chat` → `SystemPrompt.custom`
- `Session.chat` → `Mode.get`
- `Session.chat` → `Provider.tools`
- `Session.chat` → `MCP.tools`
- `Session.chat` → `streamText`
- `Session.chat` → `createProcessor`
- `Session.chat` → `Snapshot.create`
- `Session.chat` → `updatePart`
- `Session.chat` → `updateMessage`
- `Session.chat` → `getUsage`

**Code example (simplified):**

```typescript
// packages/opencode/src/session/index.ts:331-700 (main chat logic)
export async function chat(
  input: z.infer<typeof ChatInput>,
): Promise<{ info: MessageV2.Assistant; parts: MessageV2.Part[] }> {
  const l = log.clone().tag("session", input.sessionID)
  l.info("chatting")

  const userMsg: MessageV2.Info = {
    id: input.messageID ?? Identifier.ascending("message"),
    role: "user",
    sessionID: input.sessionID,
    time: {
      created: Date.now(),
    },
  }

  const app = App.info()
  const userParts = await Promise.all(
    input.parts.map(async (part): Promise<MessageV2.Part[]> => {
      if (part.type === "file") {
        const url = new URL(part.url)
        switch (url.protocol) {
          case "file:":
            // have to normalize, symbol search returns absolute paths
            // Decode the pathname since URL constructor doesn't automatically decode it
            const pathname = decodeURIComponent(url.pathname)
            const relativePath = pathname.replace(app.path.cwd, ".")
            const filePath = path.join(app.path.cwd, relativePath)

            if (part.mime === "text/plain") {
              let offset: number | undefined = undefined
              let limit: number | undefined = undefined
              const range = {
                start: url.searchParams.get("start"),
                end: url.searchParams.get("end"),
              }
              if (range.start != null) {
                const filePath = part.url.split("?")[0]
                let start = parseInt(range.start)
                let end = range.end ? parseInt(range.end) : undefined
                // some LSP servers (eg, gopls) don't give full range in
                // workspace/symbol searches, so we'll try to find the
                // symbol in the document to get the full range
                if (start === end) {
                  const symbols = await LSP.documentSymbol(filePath)
                  for (const symbol of symbols) {
                    let range: LSP.Range | undefined
                    if ("range" in symbol) {
                      range = symbol.range
                    } else if ("location" in symbol) {
                      range = symbol.location.range
                    }
                    if (range?.start?.line && range?.start?.line === start) {
                      start = range.start.line
                      end = range?.end?.line ?? start
                      break
                    }
                  }
                  offset = Math.max(start - 2, 0)
                  if (end) {
                    limit = end - offset + 2
                  }
                }
              }
              const args = { filePath, offset, limit }
              const result = await ReadTool.execute(args, {
                sessionID: input.sessionID,
                abort: new AbortController().signal,
                messageID: userMsg.id,
                metadata: async () => {},
              })
              return [
                {
                  id: Identifier.ascending("part"),
                  messageID: userMsg.id,
                  sessionID: input.sessionID,
                  type: "text",
                  synthetic: true,
                  text: `Called the Read tool with the following input: ${JSON.stringify(args)}`,
                },
                {
                  id: Identifier.ascending("part"),
                  messageID: userMsg.id,
                  sessionID: input.sessionID,
                  type: "text",
                  synthetic: true,
                  text: result.output,
                },
                {
                  ...part,
                  id: part.id ?? Identifier.ascending("part"),
                  messageID: userMsg.id,
                  sessionID: input.sessionID,
                },
              ]
            }

            let file = Bun.file(filePath)
            FileTime.read(input.sessionID, filePath)
            return [
              {
                id: Identifier.ascending("part"),
                messageID: userMsg.id,
                sessionID: input.sessionID,
                type: "text",
                text: `Called the Read tool with the following input: {\"filePath\":\"${pathname}\"}`,
                synthetic: true,
              },
              {
                id: part.id ?? Identifier.ascending("part"),
                messageID: userMsg.id,
                sessionID: input.sessionID,
                type: "file",
                url: `data:${part.mime};base64,` + Buffer.from(await file.bytes()).toString("base64"),
                mime: part.mime,
                filename: part.filename!,
                source: part.source,
              },
            ]
        }
      }
      return [
        {
          id: Identifier.ascending("part"),
          ...part,
          messageID: userMsg.id,
          sessionID: input.sessionID,
        },
      ]
    }),
  ).then((x) => x.flat())
  if (input.mode === "plan")
    userParts.push({
      id: Identifier.ascending("part"),
      messageID: userMsg.id,
      sessionID: input.sessionID,
      type: "text",
      text: PROMPT_PLAN,
      synthetic: true,
    })

  await updateMessage(userMsg)
  for (const part of userParts) {
    await updatePart(part)
  }

  if (isLocked(input.sessionID)) {
    return new Promise((resolve) => {
      const queue = state().queued.get(input.sessionID) ?? []
      queue.push({
        input: input,
        message: userMsg,
        parts: userParts,
        processed: false,
        callback: resolve,
      })
      state().queued.set(input.sessionID, queue)
    })
  }

  const model = await Provider.getModel(input.providerID, input.modelID)
  let msgs = await messages(input.sessionID)
  const session = await get(input.sessionID)

  if (session.revert) {
    const trimmed = []
    for (const msg of msgs) {
      if (
        msg.info.id > session.revert.messageID ||
        (msg.info.id === session.revert.messageID && session.revert.part === 0)
      ) {
        await Storage.remove("session/message/" + input.sessionID + "/" + msg.info.id)
        await Bus.publish(MessageV2.Event.Removed, {
          sessionID: input.sessionID,
          messageID: msg.info.id,
        })
        continue
      }

      if (msg.info.id === session.revert.messageID) {
        if (session.revert.part === 0) break
        msg.parts = msg.parts.slice(0, session.revert.part)
      }
      trimmed.push(msg)
    }
    msgs = trimmed
    await update(input.sessionID, (draft) => {
      draft.revert = undefined
    })
  }

  const previous = msgs.filter((x) => x.info.role === "assistant").at(-1)?.info as MessageV2.Assistant
  const outputLimit = Math.min(model.info.limit.output, OUTPUT_TOKEN_MAX) || OUTPUT_TOKEN_MAX

  // auto summarize if too long
  if (previous && previous.tokens) {
    const tokens =
      previous.tokens.input + previous.tokens.cache.read + previous.tokens.cache.write + previous.tokens.output
    if (model.info.limit.context && tokens > Math.max((model.info.limit.context - outputLimit) * 0.9, 0)) {
      await summarize({
        sessionID: input.sessionID,
        providerID: input.providerID,
        modelID: input.modelID,
      })
      return chat(input)
    }
  }

  using abort = lock(input.sessionID)

  const lastSummary = msgs.findLast((msg) => msg.info.role === "assistant" && msg.info.summary === true)
  if (lastSummary) msgs = msgs.filter((msg) => msg.info.id >= lastSummary.info.id)

  if (msgs.length === 1 && !session.parentID) {
    const small = (await Provider.getSmallModel(input.providerID)) ?? model
    generateText({
      maxOutputTokens: small.info.reasoning ? 1024 : 20,
      providerOptions: {
        [input.providerID]: small.info.options,
      },
      messages: [
        ...SystemPrompt.title(input.providerID).map(
          (x): ModelMessage => ({
            role: "system",
            content: x,
          }),
        ),
        ...MessageV2.toModelMessage([
          {
            info: {
              id: Identifier.ascending("message"),
              role: "user",
              sessionID: input.sessionID,
              time: {
                created: Date.now(),
              },
            },
            parts: userParts,
          },
        ]),
      ],
      model: small.language,
    })
      .then((result) => {
        if (result.text)
          return Session.update(input.sessionID, (draft) => {
            draft.title = result.text
          })
      })
      .catch(() => {})
  }

  const mode = await Mode.get(input.mode ?? "build")
  let system = input.providerID === "anthropic" ? [PROMPT_ANTHROPIC_SPOOF.trim()] : []
  system.push(...(mode.prompt ? [mode.prompt] : SystemPrompt.provider(input.modelID)))
  system.push(...(await SystemPrompt.environment()))
  system.push(...(await SystemPrompt.custom()))
  // max 2 system prompt messages for caching purposes
  const [first, ...rest] = system
  system = [first, rest.join("\n")]

  const assistantMsg: MessageV2.Info = {
    id: Identifier.ascending("message"),
    role: "assistant",
    system,
    path: {
      cwd: app.path.cwd,
      root: app.path.root,
    },
    cost: 0,
    tokens: {
      input: 0,
      output: 0,
      reasoning: 0,
      cache: { read: 0, write: 0 },
    },
    modelID: input.modelID,
    providerID: input.providerID,
    time: {
      created: Date.now(),
    },
    sessionID: input.sessionID,
  }
  await updateMessage(assistantMsg)
  const tools: Record<string, AITool> = {}

  const processor = createProcessor(assistantMsg, model.info)

  for (const item of await Provider.tools(input.providerID)) {
    if (mode.tools[item.id] === false) continue
    if (input.tools?.[item.id] === false) continue
    if (session.parentID && item.id === "task") continue
    tools[item.id] = tool({
      id: item.id as any,
      description: item.description,
      inputSchema: item.parameters as ZodSchema,
      async execute(args, options) {
        const result = await item.execute(args, {
          sessionID: input.sessionID,
          abort: abort.signal,
          messageID: assistantMsg.id,
          metadata: async (val) => {
            const match = processor.partFromToolCall(options.toolCallId)
            if (match && match.state.status === "running") {
              await updatePart({
                ...match,
                state: {
                  title: val.title,
                  metadata: val.metadata,
                  status: "running",
                  input: args,
                  time: {
                    start: Date.now(),
                  },
                },
              })
            }
          },
        })
        return result
      },
      toModelOutput(result) {
        return {
          type: "text",
          value: result.output,
        }
      },
    })
  }

  for (const [key, item] of Object.entries(await MCP.tools())) {
    if (mode.tools[key] === false) continue
    const execute = item.execute
    if (!execute) continue
    item.execute = async (args, opts) => {
      const result = await execute(args, opts)
      const output = result.content
        .filter((x: any) => x.type === "text")
        .map((x: any) => x.text)
        .join("\n\n")

      return {
        output,
      }
    }
    item.toModelOutput = (result) => {
      return {
        type: "text",
        value: result.output,
      }
    }
    tools[key] = item
  }

  const stream = streamText({
    onError() {},
    async prepareStep({ messages }) {
      const queue = (state().queued.get(input.sessionID) ?? []).filter((x) => !x.processed)
      if (queue.length) {
        for (const item of queue) {
          if (item.processed) continue
          messages.push(
            ...MessageV2.toModelMessage([
              {
                info: item.message,
                parts: item.parts,
              },
            ]),
          )
          item.processed = true
        }
        assistantMsg.time.completed = Date.now()
        await updateMessage(assistantMsg)
        Object.assign(assistantMsg, {
          id: Identifier.ascending("message"),
          role: "assistant",
          system,
          path: {
            cwd: app.path.cwd,
            root: app.path.root,
          },
          cost: 0,
          tokens: {
            input: 0,
            output: 0,
            reasoning: 0,
            cache: { read: 0, write: 0 },
          },
          modelID: input.modelID,
          providerID: input.providerID,
          time: {
            created: Date.now(),
          },
          sessionID: input.sessionID,
        })
        await updateMessage(assistantMsg)
      }
      return {
        messages,
      }
    },
    maxRetries: 10,
    maxOutputTokens: outputLimit,
    abortSignal: abort.signal,
    stopWhen: stepCountIs(1000),
    providerOptions: {
      [input.providerID]: model.info.options,
    },
    messages: [
      ...system.map(
        (x): ModelMessage => ({
          role: "system",
          content: x,
        }),
      ),
      ...MessageV2.toModelMessage(msgs),
    ],
    temperature: model.info.temperature ? 0 : undefined,
    tools: model.info.tool_call === false ? undefined : tools,
    model: wrapLanguageModel({
      model: model.language,
      middleware: [
        {
          async transformParams(args) {
            if (args.type === "stream") {
              // @ts-expect-error
              args.params.prompt = ProviderTransform.message(args.params.prompt, input.providerID, input.modelID)
            }
            return args.params
          },
        },
      ],
    }),
  })
  const result = await processor.process(stream)
  const queued = state().queued.get(input.sessionID) ?? []
  const unprocessed = queued.find((x) => !x.processed)
  if (unprocessed) {
    unprocessed.processed = true
    return chat(unprocessed.input)
  }
  for (const item of queued) {
    item.callback(result)
  }
  state().queued.delete(input.sessionID)
  return result
}
```

**Sources:** `packages/opencode/src/session/index.ts:331-700`

### Summarize Session (`Session.summarize`)

Generates a concise summary of a session's conversation history, useful for continuing long conversations or for generating titles.

**Call graph analysis:**

- `Session.summarize` → `lock`
- `Session.summarize` → `messages`
- `Session.summarize` → `Provider.getModel`
- `Session.summarize` → `App.info`
- `Session.summarize` → `SystemPrompt.summarize`
- `Session.summarize` → `createProcessor`
- `Session.summarize` → `streamText`

**Code example:**

```typescript
// packages/opencode/src/session/index.ts:800-846
export async function summarize(input: { sessionID: string; providerID: string; modelID: string }) {
  using abort = lock(input.sessionID)
  const msgs = await messages(input.sessionID)
  const lastSummary = msgs.findLast((msg) => msg.info.role === "assistant" && msg.info.summary === true)
  const filtered = msgs.filter((msg) => !lastSummary || msg.info.id >= lastSummary.info.id)
  const model = await Provider.getModel(input.providerID, input.modelID)
  const app = App.info()
  const system = SystemPrompt.summarize(input.providerID)

  const next: MessageV2.Info = {
    id: Identifier.ascending("message"),
    role: "assistant",
    sessionID: input.sessionID,
    system,
    path: {
      cwd: app.path.cwd,
      root: app.path.root,
    },
    summary: true,
    cost: 0,
    modelID: input.modelID,
    providerID: input.providerID,
    tokens: {
      input: 0,
      output: 0,
      reasoning: 0,
      cache: { read: 0, write: 0 },
    },
    time: {
      created: Date.now(),
    },
  }
  await updateMessage(next)

  const processor = createProcessor(next, model.info)
  const stream = streamText({
    maxRetries: 10,
    abortSignal: abort.signal,
    model: model.language,
    messages: [
      ...system.map(
        (x): ModelMessage => ({
          role: "system",
          content: x,
        }),
      ),
      ...MessageV2.toModelMessage(filtered),
      {
        role: "user",
        content: [
          {
            type: "text",
            text: "Provide a detailed but concise summary of our conversation above. Focus on information that would be helpful for continuing the conversation, including what we did, what we're doing, which files we're working on, and what we're going to do next.",
          },
        ],
      },
    ],
  })

  const result = await processor.process(stream)
  return result
}
```

**Sources:** `packages/opencode/src/session/index.ts:800-846`

### Session Sharing (`Session.share` and `Session.unshare`)

Allows users to share their sessions, making them accessible via a unique URL. It persists sharing information and synchronizes session data with the sharing service.

**Call graph analysis (`Session.share`):**

- `Session.share` → `Config.get`
- `Session.share` → `Session.get`
- `Session.share` → `Share.create`
- `Session.share` → `Session.update`
- `Session.share` → `Storage.writeJSON`
- `Session.share` → `Share.sync`
- `Session.share` → `Session.messages`

**Code example (`Session.share`):**

```typescript
// packages/opencode/src/session/index.ts:159-187
export async function share(id: string) {
  const cfg = await Config.get()
  if (cfg.share === "disabled") {
    throw new Error("Sharing is disabled in configuration")
  }

  const session = await get(id)
  if (session.share) return session.share
  const share = await Share.create(id)
  await update(id, (draft) => {
    draft.share = {
      url: share.url,
    }
  })
  await Storage.writeJSON<ShareInfo>("session/share/" + id, share)
  await Share.sync("session/info/" + id, session)
  for (const msg of await messages(id)) {
    await Share.sync("session/message/" + id + "/" + msg.info.id, msg.info)
    for (const part of msg.parts) {
      await Share.sync("session/part/" + id + "/" + msg.info.id + "/" + part.id, part)
    }
  }
  return share
}
```

**Sources:** `packages/opencode/src/session/index.ts:159-187`

### Session Deletion (`Session.remove`)

Deletes a session and all its associated data, including child sessions, messages, and shared information. It also publishes a `session.deleted` event.

**Call graph analysis:**

- `Session.remove` → `Session.abort`
- `Session.remove` → `Session.get`
- `Session.remove` → `Session.children`
- `Session.remove` → `Session.unshare`
- `Session.remove` → `Storage.remove`
- `Session.remove` → `Storage.removeDir`
- `Session.remove` → `Bus.publish(Event.Deleted)`

**Code example:**

```typescript
// packages/opencode/src/session/index.ts:250-275
export async function remove(sessionID: string, emitEvent = true) {
  try {
    abort(sessionID)
    const session = await get(sessionID)
    for (const child of await children(sessionID)) {
      await remove(child.id, false)
    }
    await unshare(sessionID).catch(() => {})
    await Storage.remove(`session/info/${sessionID}`).catch(() => {})
    await Storage.removeDir(`session/message/${sessionID}/`).catch(() => {})
    state().sessions.delete(sessionID)
    state().messages.delete(sessionID)
    if (emitEvent) {
      Bus.publish(Event.Deleted, {
        info: session,
      })
    }
  } catch (e) {
    log.error(e)
  }
}
```

**Sources:** `packages/opencode/src/session/index.ts:250-275`

### Session Reversion (`Session.revert` and `Session.unrevert`)

Provides functionality to revert a session to a previous state, effectively undoing messages and tool calls. `Session.unrevert` undoes the reversion.

**Call graph analysis (`Session.unrevert`):**

- `Session.unrevert` → `Session.get`
- `Session.unrevert` → `Snapshot.restore`
- `Session.unrevert` → `Session.update`

**Code example (`Session.unrevert`):}

```typescript
// packages/opencode/src/session/index.ts:789-797
export async function unrevert(sessionID: string) {
  const session = await get(sessionID)
  if (!session) return
  if (!session.revert) return
  if (session.revert.snapshot) await Snapshot.restore(sessionID, session.revert.snapshot)
  update(sessionID, (draft) => {
    draft.revert = undefined
  })
}
```

**Sources:** `packages/opencode/src/session/index.ts:789-797`

## Dependencies

- `path`: Node.js built-in module for path manipulation.
- `decimal.js`: For precise decimal arithmetic (e.g., cost calculation).
- `zod`: For schema definition and validation.
- `ai`: AI SDK for language model interactions, tool definitions, and streaming.
- `../app/app`: For managing session state (`App.state`).
- `../bus`: For publishing session and message events.
- `../config/config`: For accessing configuration settings.
- `../flag/flag`: For checking feature flags (e.g., `OPENCODE_AUTO_SHARE`).
- `../id/id`: For generating unique identifiers.
- `../installation`: For accessing application version information.
- `../mcp`: For interacting with MCP servers and their tools.
- `../provider/provider`: For accessing language models and tools.
- `../provider/transform`: For transforming messages for specific providers.
- `../provider/models`: For language model metadata.
- `../share/share`: For session sharing functionalities.
- `../snapshot`: For creating and restoring session snapshots.
- `../storage/storage`: For persisting session data.
- `../util/log`: For logging events.
- `../util/error`: For creating named error types.
- `../session/system`: For system prompts.
- `../file/time`: For file access time tracking.
- `./message-v2`: For message-related data models and utilities.
- `./mode`: For mode-specific configurations.
- `../lsp`: For Language Server Protocol functionalities (e.g., document symbols).
- `../tool/read`: For the `ReadTool` used in chat interactions.

**Sources:** `packages/opencode/src/session/index.ts:1-40`

## Consumers

The `Session` module is a central hub consumed by various parts of the application that involve user interaction and AI capabilities. This includes the `Server` module (for API endpoints), CLI commands that initiate or manage sessions, and potentially the TUI or web interface for displaying conversational history and managing session states.

**Sources:** `packages/opencode/src/session/index.ts` (implicit from exports)
