This is the HumanLayer Web UI (WUI) - a desktop application for managing AI agent approvals and sessions.

The WUI connects to the HumanLayer daemon (hld) to provide a graphical interface for monitoring Claude Code sessions and responding to approval requests. It's built with Tauri for desktop packaging and React for the interface.

When the WUI is running, logs are written to:

- Development: `~/.humanlayer/logs/wui-{branch-id}/codelayer.log` (e.g., `wui-eng-1784/codelayer.log`)
- Production: Platform-specific directories:
  - macOS: `~/Library/Logs/dev.humanlayer.wui/`
  - Windows: `%APPDATA%\dev.humanlayer.wui\logs\`
  - Linux: `~/.config/dev.humanlayer.wui/logs/`

Logs include output from the WUI backend, daemon stderr (prefixed with [Daemon]), and frontend console logs (prefixed with [Console]). The log files automatically rotate at 50MB. The application hot-reloads automatically when you make changes to the code - you cannot manually restart it.

The WUI communicates with the daemon via JSON-RPC over a Unix socket at ~/.humanlayer/daemon.sock. All session and approval data comes from the daemon - the WUI is purely a presentation layer.

To regenerate TypeScript types from the hld-sdk after OpenAPI spec changes:

- Run `make generate-sdks` from the root directory

For UI development, we use Radix UI components styled with Tailwind CSS. State management is handled by Zustand. The codebase follows React best practices with TypeScript for type safety.

## Tips and Tricks

- DO prefer ShadCN components over custom components. If a ShadCN equivalent exists that we haven't added yet, go ahead and add it. (e.g. `bunx --bun shadcn@latest add accordion`)
- DO prefer `tailwind`-based styling over other types of styling
- DO use `zustand` for managing global state. In a number of cases we've used internal React state management, but as the application scales we'll want to push more of that state into `zustand`.
- DO verify your changes with `bun run lint` and `bun run typecheck`.
- DO provide a manual list of steps for a human to test new UI changes.

## Guidelines

- In React 19, ref is now available as a standard prop for functional components, eliminating the need to wrap components with forwardRef.
- forwardRef is now depricated, NEVER use it. use ref instead.

## Testing

We use Bun's built-in test runner for unit tests. Run tests with `bun test`.

- Store tests are located in `src/AppStore.test.ts`
- Tests are critical for complex state management logic like keyboard shortcuts and selection behavior
- When modifying store methods, write tests FIRST to verify the expected behavior
- Use test-driven development (TDD) for store changes: write failing tests, then implement the fix

## Keyboard Shortcuts & Selection Management

The WUI implements vim-style keyboard navigation with complex selection behavior:

- `j/k` - Navigate down/up through sessions
- `shift+j/shift+k` - Bulk selection with anchor-based range selection
- `x` - Toggle individual selection
- `e` - Archive/unarchive sessions

Selection behavior is managed through the AppStore with these key methods:

- `bulkSelect(sessionId, direction)` - Main entry point for shift+j/k shortcuts
- `selectRange()` - Creates new selection ranges
- `addRangeToSelection()` - Adds ranges to existing selections
- `updateCurrentRange()` - Modifies existing ranges (pivot behavior)

The selection system uses "stateless anchor management" - anchors are calculated dynamically based on the current position within a selection range, not stored in state. This prevents synchronization issues.

## Responsive Design Patterns

The WUI uses Tailwind CSS responsive utilities for layout changes instead of JavaScript-based resize detection:

### Breakpoints

- `sm:` (640px) - Small screens, used for dialogs and text alignment
- `md:` (768px) - Medium screens, used for grid layouts
- `lg:` (1024px) - Large screens, used for major layout changes like sidebars
- `xl:` (1280px) - Extra large screens (rarely used)
- `2xl:` (1536px) - 2X large screens (rarely used)

### Example: SessionDetail Responsive Layout

The ActiveSession component uses pure CSS for responsive behavior:

- Mobile (<1024px): Single column layout, TodoWidget hidden
- Desktop (≥1024px): Two column layout with TodoWidget sidebar (80/20 split)

Implementation uses Tailwind utilities:

```tsx
// Container switches from column to row at lg breakpoint
<div className="flex flex-col lg:flex-row">

// TodoWidget only visible on large screens
<Card className="hidden lg:flex lg:w-1/5">
```

This approach eliminates JavaScript resize listeners, reduces React re-renders, and improves SSR compatibility.
