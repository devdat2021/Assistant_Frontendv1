# RAG Frontend Chat Application

A Next.js-based chat interface for interacting with a RAG (Retrieval-Augmented Generation) backend using a locally-hosted LLM.

## Overview

This is a modern chat frontend built with:
- **Next.js 16.3.4** with App Router
- **TypeScript** for type safety
- **Tailwind CSS** for styling
- **assistant-ui** for chat UI components
- **Base UI** component library with Nova/Geist design system
- **Bun** as the package manager

The application streams real-time chat responses and is designed to integrate with a local LLM backend.

## Prerequisites

Before you start, ensure you have:

- **Node.js 18+** (or **Bun 1.0+** - this project uses Bun)
- **Git**
- A **local LLM** running (Ollama, vLLM, llama.cpp, or similar)
  - The local model should expose an OpenAI-compatible API endpoint
  - Default expected endpoint: `http://localhost:8000/v1/chat/completions` (customize in `.env.local`)

## Quick Start

### 1. Install Dependencies

```bash
bun install
```

### 2. Configure Environment Variables

Create a `.env.local` file in the project root:

```bash
# .env.local

# Local LLM Configuration
LOCAL_LLM_ENDPOINT=http://localhost:8000/v1/chat/completions
LOCAL_LLM_MODEL=your-model-name
```

**Configuration notes:**
- `LOCAL_LLM_ENDPOINT`: Full URL to your local model's OpenAI-compatible chat completions endpoint
- `LOCAL_LLM_MODEL`: Model identifier (e.g., `mistral`, `llama2`, `neural-chat`)

### 3. Start Your Local LLM

Before running the frontend, ensure your local model is running. Examples:

**Using Ollama:**
```bash
ollama serve
# Model available at http://localhost:11434/v1/chat/completions
```

**Using vLLM:**
```bash
python -m vllm.entrypoints.openai.api_server --model <model-name> --port 8000
```

**Using llama.cpp:**
```bash
./server -m <model-file> --port 8000
```

### 4. Start Development Server

```bash
bun run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## Development Scripts

```bash
bun run dev      # Start development server with hot reload
bun run build    # Build for production
bun run start    # Start production server
bun run lint     # Run ESLint
```

## Project Structure

```
chat-frontend/
├── app/
│   ├── page.tsx                 # Home page - renders Assistant component
│   ├── assistant.tsx            # Main assistant wrapper, runtime setup
│   ├── layout.tsx               # Root layout with TooltipProvider
│   ├── globals.css              # Global styles
│   └── api/
│       └── chat/
│           └── route.ts         # API route for chat completions
├── components/
│   ├── thread.aui.tsx           # Chat thread UI component
│   ├── attachment.aui.tsx       # File attachment handling
│   ├── follow-up-suggestions.aui.tsx
│   ├── reasoning.aui.tsx        # Reasoning/thinking display
│   ├── tool-fallback.aui.tsx
│   ├── tool-group.aui.tsx
│   ├── ui/                      # Base UI components
│   ├── file.tsx                 # File display
│   ├── image.tsx                # Image display
│   └── markdown-text.tsx        # Markdown rendering
├── hooks/                       # Custom React hooks
├── lib/                         # Utility functions
├── package.json
├── tsconfig.json
├── next.config.ts
├── .env.local                   # Environment variables (create this)
└── README.md
```

## How It Works

### Chat Flow

```
User Input (Chat Composer)
    ↓
components/thread.aui.tsx
    ↓
POST /api/chat/route.ts
    ↓
Forward Request to Local LLM
    ↓
Stream Response Back
    ↓
Display in Chat Thread
```

### Key Files

| File | Purpose |
|------|---------|
| `app/page.tsx` | Entry point - renders Assistant component |
| `app/assistant.tsx` | Initializes assistant-ui runtime, sets up `/api/chat` transport |
| `app/api/chat/route.ts` | Server endpoint that calls your local LLM |
| `components/thread.aui.tsx` | Chat UI - messages, composer, responses |

## Integrating Your RAG Backend

To add RAG retrieval, modify `/api/chat/route.ts`:

1. Query your RAG service for relevant documents
2. Inject retrieved context into the system prompt
3. Forward augmented prompt to local LLM
4. Stream response back to frontend

Example:
```typescript
// In /api/chat/route.ts
// 1. Retrieve context from RAG backend
const context = await ragBackend.retrieve(latestUserMessage);

// 2. Build augmented system prompt
const augmentedSystem = `${system}\n\nContext:\n${context}`;

// 3. Send to LLM with augmented prompt
const result = streamText({
  model: localLlm,
  system: augmentedSystem,
  messages: convertedMessages,
  // ...
});
```

## Troubleshooting

### "Could not connect to local LLM"
- Verify local model is running: `curl http://localhost:8000/v1/chat/completions`
- Check `.env.local` endpoint URL
- Review local model logs

### "Module not found" errors
- Run `bun install`
- Restart IDE/TypeScript server

### Port 3000 in use
- Dev server will use next available port (3001, etc.)
- Or kill process: `lsof -ti:3000 | xargs kill -9` (macOS/Linux)

### Build errors
- Run `bun run build` for detailed output
- Check TypeScript errors: files must have proper imports
- Clear cache: `rm -rf .next && bun run build`

## Technology Stack

| Tech | Version | Purpose |
|------|---------|---------|
| Next.js | 16.3.4 | Framework |
| React | 19.2.8 | UI library |
| TypeScript | 5 | Type safety |
| Tailwind CSS | 4 | Styling |
| assistant-ui | 0.15.18 | Chat UI |
| Base UI | 1.7.0 | Unstyled components |
| Lucide React | 1.40.0 | Icons |
| Bun | 1.0+ | Runtime/package manager |

## Team Guidelines

- ✅ **TypeScript required** - All code must be type-safe
- ✅ **Tailwind only** - Use Tailwind utilities, no custom CSS (except globals.css)
- ✅ **Follow assistant-ui patterns** - Study existing components
- ✅ **API route `/api/chat`** - Single entry point for LLM communication
- ✅ **Never commit secrets** - Use `.env.local` for local development
- ✅ **Test locally** - Always run `bun run dev` before committing

## Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [assistant-ui Docs](https://assistant-ui.com/)
- [Tailwind CSS Docs](https://tailwindcss.com/)
- [Vercel AI SDK](https://sdk.vercel.ai/)

## Next Steps

1. ✅ Frontend setup complete
2. ⏳ Verify local LLM is accessible
3. ⏳ Test basic chat functionality
4. ⏳ Integrate RAG backend retrieval
5. ⏳ Add authentication (if needed)
6. ⏳ Prepare for production deployment
