# Setup Guide for Team Members

Follow these steps to get the project running on your local machine.

## Step 1: Prerequisites Check

Ensure you have:
- **Bun 1.0+** installed → Run `bun --version`
- **Git** installed → Run `git --version`
- A **local LLM running** (Ollama, vLLM, etc.)

If you don't have Bun:
```bash
curl -fsSL https://bun.sh/install | bash
# Then restart your terminal
```

## Step 2: Clone and Navigate to Project

```bash
cd chat-frontend
```

## Step 3: Install Dependencies

```bash
bun install
```

This installs all packages listed in `package.json`. Should take 30-60 seconds.

## Step 4: Create Environment File

Create `.env.local` in the `chat-frontend/` directory:

```bash
# .env.local

# Your local LLM endpoint (must be OpenAI-compatible)
LOCAL_LLM_ENDPOINT=http://localhost:8000/v1/chat/completions

# The model name your LLM recognizes
LOCAL_LLM_MODEL=mistral
```

**Find your endpoint by asking your team or checking your LLM's documentation.**

Example endpoints:
- Ollama: `http://localhost:11434/v1/chat/completions`
- vLLM: `http://localhost:8000/v1/chat/completions`
- llama.cpp: `http://localhost:8000/v1/chat/completions`

## Step 5: Verify Your Local LLM is Running

Before starting the frontend, ensure your model is up:

```bash
# Test the endpoint
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "mistral",
    "messages": [{"role": "user", "content": "Hello"}],
    "max_tokens": 10
  }'
```

If you get a response (even an error from the model), the endpoint is working. 

**If you get "Connection refused"**: Your LLM isn't running. Start it first.

## Step 6: Start the Development Server

```bash
bun run dev
```

You should see:
```
✓ Ready in XXXms
- Local:         http://localhost:3000
```

## Step 7: Open in Browser

Go to [http://localhost:3000](http://localhost:3000)

You should see the chat interface. Try typing a message!

## Step 8: Verify It Works

Type: `Hello, what's your name?`

Expected: The message appears in the chat, then a response comes back from your LLM.

---

## Troubleshooting

### "ECONNREFUSED" or Connection Error

Your frontend can't reach the local LLM.

**Fix:**
1. Verify LLM is running in another terminal
2. Check `.env.local` has correct `LOCAL_LLM_ENDPOINT`
3. Test with curl (Step 5 above)
4. Restart dev server: `Ctrl+C`, then `bun run dev`

### "Port 3000 already in use"

Another process is using port 3000.

**Fix:**
```bash
# macOS/Linux
lsof -ti:3000 | xargs kill -9

# Windows (PowerShell)
Get-Process -Id (Get-NetTCPConnection -LocalPort 3000).OwningProcess | Stop-Process -Force
```

Then restart: `bun run dev`

### TypeScript or Build Errors

```bash
# Clear cache
rm -rf .next

# Reinstall
bun install

# Try again
bun run dev
```

### "Cannot find module X"

Ensure all dependencies installed:
```bash
bun install
```

If still broken, delete and reinstall:
```bash
rm -rf node_modules
rm -rf bun.lockb
bun install
```

---

## Development Workflow

### Daily Development

```bash
# 1. Navigate to project
cd chat-frontend

# 2. Start dev server
bun run dev

# 3. Open browser to http://localhost:3000
# (Browser auto-refreshes as you edit files)

# 4. Code! Edit files in app/ or components/
```

### Building for Production

```bash
bun run build    # Creates optimized production build
bun run start    # Starts production server
```

### Code Quality

```bash
bun run lint     # Runs ESLint to check code style
```

---

## Project Structure Overview

| Folder | Contains |
|--------|----------|
| `app/` | Page components and API routes |
| `app/api/chat/` | Backend logic for LLM calls |
| `components/` | React components for the UI |
| `lib/` | Utility functions |
| `hooks/` | Custom React hooks |
| `public/` | Static files (images, favicons, etc.) |

---

## Key Concepts

### How Messages Flow

```
You type → Thread component → POST /api/chat
                                      ↓
                            Calls your local LLM
                                      ↓
                            Returns streamed response
                                      ↓
                            Displays in chat
```

### Files You'll Probably Edit

- **`app/api/chat/route.ts`** - LLM integration logic (most changes here)
- **`components/thread.aui.tsx`** - Chat UI components
- **`app/assistant.tsx`** - Runtime configuration

### Files You Usually Don't Touch

- **`components/ui/*`** - Base UI primitives (unless styling)
- **`package.json`** - Dependencies (only if adding new packages)
- **`tsconfig.json`** - TypeScript config (rarely changes)

---

## Common Tasks

### Add a New Component

1. Create file: `components/my-component.tsx`
2. Follow existing component patterns (use TypeScript)
3. Import in the file that needs it
4. Add Tailwind classes for styling

### Modify the Chat Backend

1. Edit `app/api/chat/route.ts`
2. Test by typing in the chat (dev server auto-reloads)
3. Check browser console for errors

### Update Environment Variables

1. Edit `.env.local`
2. Restart dev server: `Ctrl+C`, then `bun run dev`

---

## Getting Help

1. **Check existing code** - Look at similar components to understand patterns
2. **Read the comments** - Code has helpful comments explaining why
3. **Check errors carefully** - TypeScript errors point to exact problems
4. **Restart dev server** - Many issues resolve with a fresh start
5. **Ask teammates** - Don't spend >15 min stuck, ask for help

---

## Next Steps After Setup

1. ✅ Verify you can send a message and get a response
2. ⏳ Read through `app/api/chat/route.ts` to understand how it works
3. ⏳ Look at `components/thread.aui.tsx` to see the chat UI
4. ⏳ Start working on your assigned task

Welcome to the team! 🚀
