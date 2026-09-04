# Contributing Guide

This document outlines standards and patterns for contributing to the RAG Frontend project.

## Code Style & Standards

### TypeScript

- ✅ **Always use TypeScript** - No `.js` files in `app/` or `components/`
- ✅ **Explicit types** - Type function parameters and return values
- ✅ **Avoid `any`** - Use specific types instead

Good:
```typescript
function formatMessage(message: string): string {
  return message.trim();
}
```

Bad:
```typescript
function formatMessage(message: any) {
  return message.trim();
}
```

### React Components

- ✅ **Use functional components** - No class components
- ✅ **Use TypeScript** - Define props with interfaces
- ✅ **Export named components** - Makes debugging easier

Good:
```typescript
interface MessageProps {
  content: string;
  timestamp: Date;
}

export function Message({ content, timestamp }: MessageProps) {
  return <div>{content}</div>;
}
```

### Styling

- ✅ **Tailwind CSS only** - No CSS modules, no styled-components
- ✅ **Utility classes** - Use Tailwind's pre-built utilities
- ✅ **Responsive design** - Use Tailwind breakpoints (`md:`, `lg:`, etc.)

Good:
```tsx
<div className="flex flex-col gap-4 md:flex-row">
  <button className="bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded">
    Click me
  </button>
</div>
```

Bad:
```tsx
<div style={{ display: 'flex' }}>
  <button style={{ backgroundColor: 'blue' }}>Click me</button>
</div>
```

---

## File Organization

### Component Naming

- **UI Components**: `components/name.tsx` (lowercase, no prefix)
  - Example: `components/button.tsx`, `components/textarea.tsx`

- **Assistant-UI Components**: `components/name.aui.tsx` (with `.aui` suffix)
  - Example: `components/thread.aui.tsx`, `components/reasoning.aui.tsx`

- **Hooks**: `hooks/use-name.ts` (with `use-` prefix, kebab-case)
  - Example: `hooks/use-attachment-src.ts`

- **Utilities**: `lib/name.ts` (lowercase)
  - Example: `lib/utils.ts`

### File Structure

```
components/
├── ui/                      # Base UI components
│   ├── button.tsx
│   └── textarea.tsx
├── name.aui.tsx            # Assistant-UI specific components
├── name.tsx                # Regular React components
└── tooltip-icon-button.tsx # Compound names (kebab-case)
```

---

## Working with API Routes

### Location

All LLM communication happens in `app/api/chat/route.ts`.

### Pattern

```typescript
export async function POST(req: Request) {
  try {
    // 1. Parse request
    const { messages, system, tools } = await req.json();

    // 2. Validate inputs
    if (!messages || !Array.isArray(messages)) {
      return new Response("Invalid messages", { status: 400 });
    }

    // 3. Call LLM / backend service
    const result = await yourLlmService.chat({
      messages,
      system,
    });

    // 4. Return response
    return result.toUIMessageStreamResponse();
  } catch (error) {
    console.error("Chat API error:", error);
    return new Response("Internal server error", { status: 500 });
  }
}
```

### Error Handling

Always:
- ✅ Log errors with context: `console.error("Operation failed:", error)`
- ✅ Return appropriate HTTP status codes (400, 500, etc.)
- ✅ Don't expose internal error details to frontend (for security)

---

## Working with Components

### Follow Assistant-UI Patterns

Study existing components like `thread.aui.tsx` before creating new ones.

### Props Definition

```typescript
interface MyComponentProps {
  value: string;
  onChange: (value: string) => void;
  disabled?: boolean;  // Optional props use ?
}

export function MyComponent({ value, onChange, disabled }: MyComponentProps) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}
```

### Composition

Prefer composition over prop drilling:

Good:
```tsx
<MessageContainer>
  <MessageHeader>...</MessageHeader>
  <MessageBody>...</MessageBody>
</MessageContainer>
```

Avoid:
```tsx
<Message 
  headerText="..." 
  headerIcon="..." 
  bodyText="..." 
  bodyClassName="..."
/>
```

---

## Environment Variables

### Setup

Create `.env.local` in project root:
```
LOCAL_LLM_ENDPOINT=http://localhost:8000/v1/chat/completions
LOCAL_LLM_MODEL=mistral
```

### Accessing in Code

**Server-side only** (in `app/api/`):
```typescript
const endpoint = process.env.LOCAL_LLM_ENDPOINT;
```

**Never expose secrets in client code.**

---

## Git Workflow

### Branch Names

Use descriptive names:
- `feature/add-rag-retrieval`
- `fix/chat-scroll-bug`
- `docs/update-readme`

### Commit Messages

Be clear and specific:

Good:
- `Add RAG context to system prompt`
- `Fix message loading skeleton animation`
- `Update environment variable documentation`

Bad:
- `Update stuff`
- `Fix bug`
- `Changes`

### Pull Requests

1. Describe what changed and why
2. Reference any related issues
3. Ensure code passes linting: `bun run lint`
4. Test locally before pushing

---

## Testing Your Changes

### Before Committing

```bash
# 1. Check for lint errors
bun run lint

# 2. Test the feature manually
bun run dev
# Visit http://localhost:3000 and test

# 3. Check the build works
bun run build

# 4. Restart dev server (sometimes needed)
Ctrl+C
bun run dev
```

### Common Issues

| Issue | Solution |
|-------|----------|
| Component doesn't appear | Check import path, restart dev server |
| Styling not applied | Ensure using Tailwind classes, not CSS |
| TypeScript errors | Hover over error in editor for details |
| LLM not responding | Check `.env.local` endpoint, restart server |

---

## Code Review Checklist

Before submitting a PR, verify:

- ✅ All files use TypeScript (`.ts` or `.tsx`)
- ✅ No `console.log` statements left in code (use for debugging only)
- ✅ Functions have return types: `function name(): Type { }`
- ✅ Components have prop types defined
- ✅ Styling uses only Tailwind classes
- ✅ No environment variables exposed in client code
- ✅ Error handling is present in API routes
- ✅ Code follows patterns from existing components

---

## Common Patterns

### Using Environment Variables

```typescript
// ✅ Good - server-side only
export async function POST(req: Request) {
  const endpoint = process.env.LOCAL_LLM_ENDPOINT;
  // ...
}

// ❌ Bad - client-side (security risk)
export function MyComponent() {
  const apiKey = process.env.SECRET_KEY;  // NEVER do this
  // ...
}
```

### Handling Async Operations

```typescript
// ✅ Good - proper error handling
async function fetchResponse() {
  try {
    const response = await fetch(endpoint);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (error) {
    console.error("Fetch failed:", error);
    throw error;  // Let caller handle
  }
}
```

### Component Props

```typescript
// ✅ Good - explicit types
interface Props {
  label: string;
  value: string;
  onChange: (value: string) => void;
}

export function Input({ label, value, onChange }: Props) {
  return (
    <input value={value} onChange={(e) => onChange(e.target.value)} />
  );
}
```

---

## Performance Considerations

- Use `React.memo` for components that don't need frequent re-renders
- Avoid creating functions inside render/component bodies
- Use `useCallback` for event handlers passed as props
- Keep components focused on a single responsibility

---

## Debugging Tips

### Browser Console

1. Open DevTools: `F12` or `Cmd+Opt+I`
2. Check **Console** tab for errors
3. Check **Network** tab to see API requests/responses

### Server Logs

Dev server logs appear in your terminal. Look for:
- `✓` (success) or `⨯` (errors)
- API request details
- TypeScript compilation errors

### VSCode

- Hover over variables to see types
- `Ctrl+Space` for autocomplete
- `Ctrl+Click` to jump to definitions
- Red squiggles = TypeScript errors (hover for details)

---

## Getting Help

1. Check if similar code exists in the codebase
2. Read inline comments and documentation
3. Try the steps in SETUP.md troubleshooting section
4. Ask a teammate—don't get stuck for >15 minutes
5. Check the original PR/commit for context

---

## Questions?

Ask on the team channel or during standups. We're here to help! 🚀
