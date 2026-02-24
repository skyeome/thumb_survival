# AGENTS.md — thumb_survival

Next.js 16 app (React 19, TypeScript 5, Tailwind CSS v4, shadcn/ui radix-nova). Package manager: **pnpm**.

## Build / Dev / Lint Commands

```bash
pnpm dev          # Start dev server (next dev)
pnpm build        # Production build (next build)
pnpm start        # Serve production build
pnpm lint         # ESLint (flat config, core-web-vitals + typescript)
```

No test framework is currently installed. If tests are added, prefer **Vitest** (aligns with the Vite-era tooling in this stack).

## Project Structure

```
app/              # Next.js App Router (layouts, pages, globals.css)
components/       # Shared React components
  ui/             # shadcn/ui primitives (button, card, input, select, etc.)
lib/              # Utilities (cn helper, shared logic)
public/           # Static assets (SVGs)
```

Path alias: `@/*` maps to project root (`./`). Always use `@/` imports — never relative paths across directories.

## TypeScript

- **Strict mode** is on (`"strict": true` in tsconfig.json).
- Target: ES2017, module: ESNext, moduleResolution: bundler.
- Never use `as any`, `@ts-ignore`, or `@ts-expect-error`.
- Prop types use `React.ComponentProps<"element">` extended with additional props — not separate interface definitions.

```tsx
// ✅ Correct
function Card({ className, size = "default", ...props }: React.ComponentProps<"div"> & { size?: "default" | "sm" }) {

// ❌ Wrong
interface CardProps { className?: string; size?: "default" | "sm" }
function Card({ className, size = "default", ...props }: CardProps) {
```

## Code Style

### Formatting

- **No semicolons** — the codebase omits them consistently.
- **Double quotes** for strings and imports.
- **2-space indentation**.
- Trailing commas in multi-line arrays/objects.

### Functions & Components

- Use `function` declarations for components — not arrow functions.
- Exported components use named exports (not default), except Next.js pages/layouts which use `export default function`.
- Mark client components with `"use client"` directive at the top of the file.

```tsx
// ✅ Named export, function declaration
export function MyComponent() { ... }

// ✅ Page files use default export
export default function Page() { ... }

// ❌ Arrow function component
export const MyComponent = () => { ... }
```

### Imports

Group imports in this order, separated by blank lines:

1. `"use client"` directive (if needed)
2. React (`import * as React from "react"`)
3. External libraries (next, radix-ui, class-variance-authority, lucide-react)
4. Internal modules (`@/components/...`, `@/lib/...`)

```tsx
"use client"

import * as React from "react"

import { cva, type VariantProps } from "class-variance-authority"
import { Slot } from "radix-ui"

import { cn } from "@/lib/utils"
```

Always import React as `import * as React from "react"` (namespace import) — not `import React from "react"`.

### Naming

- **Components**: PascalCase (`Button`, `CardHeader`, `ComponentExample`).
- **Files**: kebab-case (`component-example.tsx`, `alert-dialog.tsx`).
- **CSS variables**: kebab-case with `--` prefix (`--color-primary`, `--font-sans`).
- **data attributes**: `data-slot` for component identity, `data-variant`/`data-size` for variant state.

## Styling (Tailwind CSS v4 + shadcn/ui)

- Tailwind v4 uses CSS-first config via `app/globals.css` — no `tailwind.config.ts`.
- Theme tokens are CSS custom properties in oklch color space.
- Use the `cn()` utility from `@/lib/utils` to merge class names:

```tsx
import { cn } from "@/lib/utils"

className={cn("base-classes", conditional && "conditional-class", className)}
```

- Use `cva` (class-variance-authority) for components with multiple variants.
- Dark mode: `.dark` class on ancestor. Use `dark:` prefix in Tailwind classes.
- Design tokens: Use semantic names (`bg-primary`, `text-muted-foreground`, `border-input`), not raw colors.

## shadcn/ui Component Conventions

- Style: **radix-nova** (from `components.json`).
- RSC-enabled (`"rsc": true`).
- Icon library: **lucide-react**.
- Components live in `components/ui/`. Install new ones via:

```bash
pnpm dlx shadcn@latest add <component-name>
```

- UI components follow this pattern:
  - `data-slot="component-name"` attribute on root element for targeting.
  - Spread `...props` last on the element.
  - Accept `className` and merge with `cn()`.
  - Destructure specific props, pass rest via `...props`.
  - Use `asChild` pattern with `Slot.Root` from radix-ui for polymorphic components.

```tsx
function Input({ className, type, ...props }: React.ComponentProps<"input">) {
  return (
    <input
      type={type}
      data-slot="input"
      className={cn("base-styles", className)}
      {...props}
    />
  )
}

export { Input }
```

## Error Handling

- No empty catch blocks.
- Use Next.js `error.tsx` boundary files for page-level errors.
- Form validation: use `aria-invalid` attribute — UI components already style for it.

## Key Dependencies

| Package | Version | Purpose |
|---|---|---|
| next | 16.1.6 | App framework (App Router) |
| react / react-dom | 19.2.3 | UI library |
| tailwindcss | ^4 | Styling (CSS-first config) |
| radix-ui | ^1.4.3 | Headless UI primitives |
| @base-ui/react | ^1.2.0 | Base UI components |
| class-variance-authority | ^0.7.1 | Variant-based className utility |
| clsx + tailwind-merge | latest | Class merging (via `cn()`) |
| lucide-react | ^0.575.0 | Icons |
| shadcn | ^3.8.5 | Component CLI (devDependency) |
| tw-animate-css | ^1.4.0 | Animation utilities |

## Gotchas

- This is Next.js 16 with React 19 — use the latest APIs (e.g., `React.ComponentProps` not `React.FC`).
- `React.FC` is **not used** in this codebase. Don't introduce it.
- Tailwind v4 does NOT use `tailwind.config.ts`. All theme config is in `globals.css` via `@theme inline`.
- The `@custom-variant dark (&:is(.dark *))` line enables class-based dark mode.
- Images in components use standard `<img>` tags — use `next/image` for optimized images in production features.
- Font setup: Inter (--font-sans), Geist Sans, Geist Mono — loaded via `next/font/google` in root layout.
