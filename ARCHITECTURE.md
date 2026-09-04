# Architecture Overview

This document explains the structure and flow of the RAG Frontend application.

## System Architecture

```
┌─────────────────────────────────────────────┐
│     Browser (Client)                        │
│  ┌─────────────────────────────────────┐   │
│  │  React Components (assistant-ui)    │   │
│  │  - Thread (chat display)            │   │
│  │  - Composer (input)                 │   │
│  │  - Messages (rendering)             │   │
│  └──────────────┬──────────────────────┘   │
│                 │                           │
│                 │ HTTP POST /api/chat       │
│                 ↓                           │
└──────────────────────────────────────────────┘
                   │
                   ↓
┌──────────────────────────────────────────────┐
│  Next.js Server (Backend)                   │
│  ┌──────────────────────────────────────┐   │
│  │  /api/chat/route.ts                  │   │
│  │  - Receives chat messages            │   │
│  │  - Calls Local LLM                   │   │
│  │  - Streams response                  │   │
│  └──────────────┬───────────────────────┘   │
│                 │                           │
│                 │ HTTP POST                 │
│                 ↓                           │
└──────────────────────────────────────────────┘
                   │
                   ↓
┌──────────────────────────────────────────────┐
│  Local LLM (External Service)               │
│  - Ollama, vLLM, llama.cpp, etc.           │
│  - OpenAI-compatible API                   │
└──────────────────────────────────────────────┘
```

## Detailed Message Flow

### 1. User Types Message

```
┌─────────────────────────────────────────┐
│ Browser                                 │
│ ┌───────────────────────────────────┐   │
│ │ <ComposerPrimitive />             │   │
│ │ (text input from thread.aui.tsx)  │   │
│ └──────────────┬────────────────────┘   │
│                │                         │
│                ↓                         │
│  onSubmit fires, sends to runtime       │
└────────────────┬──────────────────────────┘
                 │
                 ↓
        Runtime collects message
                 │
                 ↓
        Prepares POST request
```

### 2. Frontend Sends to Backend

```
POST /api/chat
Content-Type: application/json

{
  "messages": [
    {
      "role": "user",
      "content": "Hello!",
      "id": "msg-123"
    }
  ],
  "system": "You are a helpful assistant",
  "tools": {}
}
```

See `app/assistant.tsx` lines 8-13 for transport configuration.

### 3. Backend Processes

```
/api/chat/route.ts
  ↓
[1] Parse request JSON
  ↓
[2] Convert messages format (convertToModelMessages)
  ↓
[3] Call Local LLM API
  ↓
[4] Stream response
  ↓
[5] Return to frontend
```

### 4. Frontend Receives & Displays

```
Browser receives streamed response
  ↓
<Thread /> processes chunks
  ↓
Updates message state
  ↓
Re-renders with new content
  ↓
User sees response in chat
```

---

## Component Hierarchy

```
Root
│
├─ app/layout.tsx (RootLayout)
│  └─ TooltipProvider (required by assistant-ui)
│     └─ RenderProps {children}
│
└─ app/page.tsx (Home)
   └─ <Assistant />
      │
      ├─ AssistantRuntimeProvider
      │  └─ useChatRuntime() configuration
      │     └─ AssistantChatTransport to /api/chat
      │
      └─ <Thread />
         │
         ├─ ThreadPrimitive (core chat UI)
         │  │
         │  ├─ Message List
         │  │  ├─ UserMessage
         │  │  └─ AssistantMessage
         │  │     ├─ Text content
         │  │     ├─ Tool calls (rendered with ToolGroup)
         │  │     └─ Reasoning (if present)
         │  │
         │  ├─ Composer (input area)
         │  │  ├─ ComposerPrimitive
         │  │  ├─ ComposerAttachments
         │  │  ├─ ComposerAddAttachment
         │  │  └─ Send button (ActionBarPrimitive)
         │  │
         │  └─ Actions
         │     ├─ Copy
         │     ├─ Edit
         │     ├─ Regenerate
         │     └─ More options
         │
         └─ Welcome (empty state)
            └─ Shows when no messages yet
```

---

## Key Files & Responsibilities

### Frontend Layer

| File | Purpose | Responsibility |
|------|---------|-----------------|
| `app/page.tsx` | Entry point | Render Assistant |
| `app/assistant.tsx` | Runtime setup | Configure AI SDK, transport |
| `components/thread.aui.tsx` | Chat UI | Display messages, handle user input |
| `app/layout.tsx` | Root layout | Provide TooltipProvider |

### Backend Layer

| File | Purpose | Responsibility |
|------|---------|-----------------|
| `app/api/chat/route.ts` | LLM proxy | Forward requests to local LLM |

### Utilities & Components

| File | Purpose |
|------|---------|
| `components/ui/*` | Base UI elements (button, textarea, etc.) |
| `components/attachment.aui.tsx` | File uploads |
| `components/reasoning.aui.tsx` | Reasoning/thinking display |
| `components/tool-*.aui.tsx` | Tool call rendering |
| `lib/utils.ts` | Helper functions |
| `hooks/use-*.ts` | Custom React hooks |

---

## Data Flow: Message

### User Message Structure

```typescript
interface UIMessage {
  id: string;
  role: "user" | "assistant";
  content: string;  // or array of parts
}
```

### Message Journey

```
1. User types in <Composer />
   └─ Captured by thread runtime

2. Message added to state
   └─ Frontend re-renders

3. POST /api/chat
   ├─ Payload includes all messages
   ├─ System prompt (if set)
   └─ Tools (if any)

4. Server receives & converts format
   └─ Uses @ai-sdk utilities

5. Local LLM processes
   └─ Streams response chunks

6. Frontend receives stream
   ├─ Adds to message history
   ├─ Re-renders incrementally
   └─ User sees response appearing in real-time
```

---

## Runtime Configuration

See `app/assistant.tsx`:

```typescript
const runtime = useChatRuntime({
  // When to auto-send the next message
  sendAutomaticallyWhen: lastAssistantMessageIsCompleteWithToolCalls,
  
  // How to reach the backend
  transport: new AssistantChatTransport({
    api: "/api/chat",  // Relative URL in Next.js
  }),
});
```

### Transport

- **Type**: `AssistantChatTransport`
- **Endpoint**: `/api/chat` (handled by `app/api/chat/route.ts`)
- **Method**: POST
- **Format**: Streaming JSON

---

## LLM Integration Points

### Current Setup

```
Frontend (/api/chat/route.ts)
  ↓
  ├─ Reads: LOCAL_LLM_ENDPOINT
  ├─ Reads: LOCAL_LLM_MODEL
  ↓
Calls your local LLM
  ├─ Uses @ai-sdk/openai (OpenAI-compatible)
  ├─ Sends: messages, system, tools
  ↓
Streams response
  ↓
Returned to frontend
```

### Where to Add RAG

Modify `/api/chat/route.ts` POST function:

```typescript
export async function POST(req: Request) {
  const { messages, system } = await req.json();

  // 1. Retrieve context from RAG
  const userMessage = messages[messages.length - 1];
  const ragContext = await ragBackend.retrieve(userMessage.content);

  // 2. Augment system prompt
  const augmentedSystem = `${system}\n\nContext:\n${ragContext}`;

  // 3. Send to LLM (unchanged)
  const result = streamText({
    model: openai(process.env.LOCAL_LLM_MODEL),
    messages: await convertToModelMessages(messages),
    system: augmentedSystem,
    // ...
  });

  return result.toUIMessageStreamResponse();
}
```

---

## State Management

### Message State

Managed by assistant-ui runtime:
- Stored in `AssistantRuntimeProvider`
- Automatically persists to local browser storage
- Shared across all components via context

### UI State

Managed by component-level React hooks:
- Message expansion (reasoning, tool calls)
- Loading states
- Error states

---

## Styling Architecture

### Layer 1: Tailwind CSS
- Utility classes for layout, spacing, colors
- Applied directly to JSX elements

### Layer 2: Base UI
- Unstyled accessible primitives
- Composed in assistant-ui components
- Styled with Tailwind on top

### Layer 3: Custom Components
- Built from Base UI + Tailwind
- Follow assistant-ui patterns
- Example: `components/ui/button.tsx`

---

## Error Handling

### Frontend Errors
- Display in chat as error messages
- User can retry
- Logged to browser console

### Backend Errors
- Try-catch in `/api/chat/route.ts`
- Log to server console
- Return HTTP error status
- Frontend shows generic error message

---

## Performance Considerations

### Streaming
- Response appears incrementally
- Better UX than waiting for full response
- Uses Server-Sent Events (SSE) or chunked transfer

### Message Rendering
- Only changed messages re-render (React reconciliation)
- Old messages stay in DOM
- Scroll position managed by assistant-ui

### Local Storage
- Chat history persists across page reloads
- Automatic, no manual configuration needed

---

## Security Notes

- ✅ API key in server-side `.env.local` only
- ✅ No secrets in client-side code
- ✅ Local LLM endpoint assumed to be trusted (local network)
- ⚠️ Add authentication if exposing to internet later

---

## Deployment Architecture

### Development
```
localhost:3000 → localhost:8000 (local LLM)
```

### Production
```
Web Server (Next.js)
  ↓
RAG Backend
  ↓
LLM Service
```

Requires:
- Environment variables for production LLM endpoint
- Authentication to RAG backend
- CORS configuration if cross-domain
- Error logging/monitoring

---

## Technology Stack Decisions

| Choice | Why |
|--------|-----|
| Next.js App Router | Server components, built-in API routes |
| TypeScript | Type safety, better IDE support |
| Tailwind CSS | Fast styling, no context switching |
| assistant-ui | Ready-made chat UI, less work to build |
| Base UI | Unstyled, accessible, lightweight |
| Streaming | Better UX than waiting for full response |
| Local LLM | Privacy, no API costs, full control |

---

## Future Extensibility

Current design supports:
- ✅ Adding RAG retrieval (modify `/api/chat/route.ts`)
- ✅ Adding tool calls (already supported, just define in runtime)
- ✅ Adding authentication (add middleware to `/api/chat/route.ts`)
- ✅ Adding chat history persistence (database integration in route)
- ✅ Adding analytics (log events in route)

Not supported yet (would require changes):
- ❌ Multiple chat sessions/threads (would need database)
- ❌ User management (would need auth system)
- ❌ Collaborative editing (would need WebSockets)
