# SKILL: shadcn/ui Custom Registry — Full Setup & Management

## What This Skill Covers

This skill enables you to fully set up, build, and manage a custom shadcn/ui component registry — from scaffolding the project and writing components, to building the JSON output and guiding the user through hosting/deployment. When something requires manual action from the user (e.g. deploying to Vercel, pushing to GitHub), you will clearly call it out with a `🙋 USER ACTION REQUIRED` block.

---

## Mental Model

A shadcn registry is simply:
1. **Source components** living in a `registry/` folder in your project
2. A **`registry.json`** manifest that lists all items and their metadata
3. A **build step** (`npx shadcn build`) that reads `registry.json` and emits one JSON file per item into `public/r/[name].json`
4. A **static file server** (Next.js, any static host) that serves those JSON files over HTTP
5. Users install components via: `npx shadcn@latest add https://your-domain.com/r/component-name.json`

---

## Step 1 — Project Setup

### Option A: Start from the official template (recommended for new registries)

```bash
npx degit shadcn-ui/registry-template my-registry
cd my-registry
npm install
```

> This template uses **Next.js + Tailwind v4**. For Tailwind v3, use `shadcn-ui/registry-template-v3`.

### Option B: Add a registry to an existing project

Install the shadcn CLI:

```bash
npm install shadcn@latest
# or
pnpm add shadcn@latest
```

Add the build script to `package.json`:

```json
{
  "scripts": {
    "registry:build": "shadcn build"
  }
}
```

---

## Step 2 — Directory Structure

Always follow this structure. The style name (e.g. `default`, `new-york`) is arbitrary — pick one and be consistent:

```
my-registry/
├── registry.json                  ← manifest / entry point
├── registry/
│   └── default/                   ← your style name
│       ├── my-button/
│       │   └── my-button.tsx
│       ├── use-counter/
│       │   └── use-counter.ts
│       └── auth-page/
│           ├── page.tsx
│           └── login-form.tsx
├── public/
│   └── r/                         ← build output (auto-generated, do not edit)
│       ├── registry.json
│       ├── my-button.json
│       └── use-counter.json
└── package.json
```

**Important rules:**
- All registry source files MUST live under `registry/`
- Imports inside registry files must use `@/registry/...` paths (the build step transforms these)
- Never manually edit files in `public/r/` — they are generated

---

## Step 3 — registry.json (The Manifest)

Create `registry.json` at the root of the project:

```json
{
  "$schema": "https://ui.shadcn.com/schema/registry.json",
  "name": "acme",
  "homepage": "https://acme.com",
  "items": [
    {
      "name": "my-button",
      "type": "registry:ui",
      "title": "My Button",
      "description": "A custom branded button component.",
      "registryDependencies": ["button"],
      "dependencies": ["class-variance-authority"],
      "files": [
        {
          "path": "registry/default/my-button/my-button.tsx",
          "type": "registry:component"
        }
      ]
    }
  ]
}
```

### registry.json Top-Level Fields

| Field | Required | Description |
|-------|----------|-------------|
| `$schema` | Yes | Always `https://ui.shadcn.com/schema/registry.json` |
| `name` | Yes | Short slug for your registry (no spaces) |
| `homepage` | Yes | Public URL of your registry or site |
| `items` | Yes | Array of registry item definitions |

---

## Step 4 — Registry Item Types

### Item-level `type` (what kind of thing is this item?)

| Type | Use For |
|------|---------|
| `registry:ui` | Primitive UI components (buttons, inputs, etc.) |
| `registry:block` | Composed, multi-file UI blocks (forms, dashboards) |
| `registry:component` | Standalone app-level components |
| `registry:hook` | React hooks |
| `registry:lib` | Utility/helper libraries |
| `registry:page` | Full page components |
| `registry:theme` | CSS variable theme definitions |
| `registry:style` | Full style definition (extends or replaces shadcn defaults) |
| `registry:item` | Universal/framework-agnostic items (cursor rules, config files, etc.) |
| `registry:file` | Arbitrary files (env, config, etc.) |

### File-level `type` (what kind of file is this?)

| Type | Use For |
|------|---------|
| `registry:component` | `.tsx` / `.jsx` React components |
| `registry:hook` | Custom React hooks |
| `registry:lib` | Utility functions |
| `registry:page` | Next.js / framework page files |
| `registry:file` | Any other file (env, json, md, cursor rules) |

---

## Step 5 — Full registry-item.json Reference

Every item in `registry.json` follows this schema:

```json
{
  "$schema": "https://ui.shadcn.com/schema/registry-item.json",
  "name": "hello-world",
  "type": "registry:block",
  "title": "Hello World",
  "description": "A hello world component with a button.",
  "author": "Your Name <you@example.com>",
  "registryDependencies": [
    "button",
    "@acme/other-item",
    "https://example.com/r/remote-dep.json"
  ],
  "dependencies": ["zod@^3.20.0", "motion"],
  "devDependencies": ["tw-animate-css"],
  "files": [
    {
      "path": "registry/default/hello-world/hello-world.tsx",
      "type": "registry:component"
    },
    {
      "path": "registry/default/hello-world/use-hello-world.ts",
      "type": "registry:hook"
    },
    {
      "path": "registry/default/hello-world/page.tsx",
      "type": "registry:page",
      "target": "app/hello/page.tsx"
    },
    {
      "path": "registry/default/hello-world/.env.example",
      "type": "registry:file",
      "target": "~/.env.local"
    }
  ],
  "cssVars": {
    "theme": {
      "font-heading": "Poppins, sans-serif"
    },
    "light": {
      "brand": "20 14.3% 4.1%"
    },
    "dark": {
      "brand": "20 14.3% 4.1%"
    }
  },
  "envVars": {
    "NEXT_PUBLIC_APP_URL": "http://localhost:3000",
    "MY_SECRET_KEY": ""
  },
  "docs": "https://example.com/docs/hello-world",
  "categories": ["forms", "auth"]
}
```

**Key rules:**
- `target` is only required for `registry:page` and `registry:file` types (tells CLI where to place the file)
- `envVars` are added to `.env.local`. Existing values are NOT overwritten. Only use for dev/example vars, never production secrets.
- `registryDependencies` can be: a plain shadcn component name (`"button"`), a namespaced name (`"@acme/component"`), or a full URL (`"https://example.com/r/editor.json"`)
- `dependencies` format: `"package-name"` or `"package-name@version"` (e.g. `"zod@^3.20.0"`)

---

## Step 6 — Writing Registry Components

### Rules for source files

1. **Import other registry items using `@/registry/...` paths:**
   ```tsx
   // ✅ correct
   import { Button } from "@/registry/default/button/button"
   import { cn } from "@/lib/utils"
   
   // ❌ wrong — don't use relative paths
   import { Button } from "../button/button"
   ```

2. **Import shadcn/ui primitives using `@/components/ui/...`:**
   ```tsx
   import { Button } from "@/components/ui/button"
   ```

3. **Export the component and its prop types:**
   ```tsx
   export type MyComponentProps = {
     label: string
     variant?: "default" | "outline"
   }
   
   export function MyComponent({ label, variant = "default" }: MyComponentProps) {
     // ...
   }
   ```

4. **Support dark mode** — use Tailwind's `dark:` variants
5. **Use TypeScript** — always include proper types
6. **Test both light and dark themes** before registering

### Example: A well-formed registry component

```tsx
// registry/default/metric-card/metric-card.tsx
import { cn } from "@/lib/utils"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import { Badge } from "@/components/ui/badge"

export type MetricCardProps = {
  title: string
  value: string | number
  trend?: "up" | "down" | "neutral"
  className?: string
}

export function MetricCard({ title, value, trend = "neutral", className }: MetricCardProps) {
  return (
    <Card className={cn("w-full", className)}>
      <CardHeader>
        <CardTitle className="text-sm font-medium text-muted-foreground">{title}</CardTitle>
      </CardHeader>
      <CardContent>
        <div className="flex items-center justify-between">
          <span className="text-2xl font-bold">{value}</span>
          {trend !== "neutral" && (
            <Badge variant={trend === "up" ? "default" : "destructive"}>
              {trend === "up" ? "↑" : "↓"}
            </Badge>
          )}
        </div>
      </CardContent>
    </Card>
  )
}
```

---

## Step 7 — Build the Registry

```bash
npm run registry:build
# or
npx shadcn build
```

**Options:**
```bash
# Custom input manifest
npx shadcn build ./path/to/registry.json

# Custom output directory
npx shadcn build --output ./dist/r

# Build and watch for changes
npx shadcn build --watch
```

**What this does:**
- Reads `registry.json`
- Parses and transforms each source file (rewrites import paths)
- Emits `public/r/[name].json` for each item, with file contents inlined
- Emits `public/r/registry.json` (the index)

After building, verify the output:
```bash
cat public/r/my-button.json  # Should contain the full JSON with file content
```

---

## Step 8 — Local Development & Testing

Start the dev server:
```bash
npm run dev
```

Your registry is now available at:
- Index: `http://localhost:3000/r/registry.json`
- Items: `http://localhost:3000/r/[name].json`

**Test installing locally from another project:**
```bash
# In a different project that has shadcn initialized
npx shadcn@latest add http://localhost:3000/r/my-button.json
```

---

## Step 9 — Namespaces (Optional but Recommended)

Namespaces let users install with `@yourname/component-name` syntax instead of full URLs. Configure in the **consumer's** `components.json`:

```json
{
  "registries": {
    "@acme": "https://registry.acme.com/r/{name}.json"
  }
}
```

Then users can install with:
```bash
npx shadcn@latest add @acme/my-button
npx shadcn@latest add @acme/my-button @acme/metric-card
```

The `{name}` placeholder is replaced with the component name automatically.

---

## Step 10 — Authentication (Private Registries)

For private/internal registries, add auth config in `components.json` on the consumer side:

```json
{
  "registries": {
    "@internal": {
      "url": "https://internal.company.com/r/{name}.json",
      "headers": {
        "Authorization": "Bearer ${INTERNAL_TOKEN}",
        "X-API-Key": "${API_KEY}"
      }
    }
  }
}
```

The `${ENV_VAR}` syntax reads from the user's environment variables at install time.

**Auth methods supported:**
- Bearer token (`Authorization: Bearer ${TOKEN}`)
- API key (`X-API-Key: ${KEY}`)
- Basic auth (`Authorization: Basic ${BASE64_CREDENTIALS}`)
- Query parameter (add to URL: `https://example.com/r/{name}.json?token=${TOKEN}`)

---

## Step 11 — Hosting & Publishing

### 🙋 USER ACTION REQUIRED — Deployment

The agent can scaffold everything, write all the code, and build the registry. But **you must deploy it yourself**. Here's what to do:

**Option A: Deploy to Vercel (easiest, recommended)**
1. Push your registry project to a GitHub repository
2. Go to [vercel.com](https://vercel.com) → "Add New Project"
3. Import your GitHub repo
4. Vercel auto-detects Next.js — just click "Deploy"
5. Your registry will be live at `https://your-project.vercel.app/r/[name].json`

**Option B: Deploy to Netlify**
1. Push to GitHub
2. Go to [netlify.com](https://netlify.com) → "Add new site" → "Import from Git"
3. Set build command: `npm run registry:build && npm run build`
4. Set publish directory: `out` or `.next` (for Next.js)
5. Deploy

**Option C: GitHub Pages (static only, no Next.js SSR)**
1. Run `npm run registry:build` locally
2. Copy the `public/r/` folder to your GitHub Pages branch
3. Items will be at `https://username.github.io/repo-name/r/[name].json`

**Option D: Any static host (Cloudflare Pages, S3, etc.)**
- Build locally, then upload the `public/r/` directory
- Make sure the server sets `Content-Type: application/json` for `.json` files
- Enable CORS headers if needed: `Access-Control-Allow-Origin: *`

### After deploying

Update `registry.json` with your real production URL:
```json
{
  "name": "acme",
  "homepage": "https://your-actual-domain.com"
}
```

Rebuild and redeploy.

---

## Step 12 — Listing on the Official Registry Index (Optional)

### 🙋 USER ACTION REQUIRED — Submit to shadcn index

To appear in `npx shadcn search` results:

1. Fork [shadcn-ui/ui](https://github.com/shadcn-ui/ui) on GitHub
2. Add your registry entry to `apps/v4/registry/directory.json`
3. Run `pnpm registry:build` inside the repo to update `registries.json`
4. Open a Pull Request

**Requirements for acceptance:**
- Registry must be open source and publicly accessible
- Must serve valid JSON conforming to the registry schema
- Must be a flat registry: items at `/r/[name].json`, index at `/r/registry.json` (or `/registry.json`)

---

## Common Patterns

### Pattern: Multi-file block

```json
{
  "name": "data-table",
  "type": "registry:block",
  "title": "Data Table",
  "description": "A full-featured sortable, filterable data table.",
  "registryDependencies": ["table", "button", "input"],
  "dependencies": ["@tanstack/react-table"],
  "files": [
    {
      "path": "registry/default/data-table/data-table.tsx",
      "type": "registry:component"
    },
    {
      "path": "registry/default/data-table/data-table-toolbar.tsx",
      "type": "registry:component"
    },
    {
      "path": "registry/default/data-table/use-data-table.ts",
      "type": "registry:hook"
    },
    {
      "path": "registry/default/data-table/columns.tsx",
      "type": "registry:component"
    }
  ]
}
```

### Pattern: Page with route

```json
{
  "name": "dashboard-page",
  "type": "registry:page",
  "files": [
    {
      "path": "registry/default/dashboard-page/page.tsx",
      "type": "registry:page",
      "target": "app/dashboard/page.tsx"
    },
    {
      "path": "registry/default/dashboard-page/components/stats.tsx",
      "type": "registry:component"
    }
  ]
}
```

### Pattern: Custom theme

```json
{
  "name": "brand-theme",
  "type": "registry:theme",
  "cssVars": {
    "theme": {
      "font-heading": "Cal Sans, sans-serif",
      "radius": "0.375rem"
    },
    "light": {
      "primary": "262.1 83.3% 57.8%",
      "primary-foreground": "210 20% 98%"
    },
    "dark": {
      "primary": "263.4 70% 50.4%",
      "primary-foreground": "210 20% 98%"
    }
  }
}
```

### Pattern: Universal item (framework-agnostic, e.g. Cursor rules)

```json
{
  "name": "cursor-rules",
  "type": "registry:item",
  "files": [
    {
      "path": "registry/default/cursor-rules/react.mdc",
      "type": "registry:file",
      "target": ".cursor/rules/react.mdc"
    }
  ]
}
```

### Pattern: Remote dependency on another custom registry

```json
{
  "name": "my-form",
  "type": "registry:block",
  "registryDependencies": [
    "button",
    "input",
    "https://ui.other-registry.com/r/fancy-select.json"
  ]
}
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `public/r/` output is empty | Make sure `registry.json` `items` are not empty; run `npx shadcn build` |
| Install fails with "not found" | Verify the URL is publicly accessible, returns valid JSON, has correct CORS headers |
| Import paths break after install | Ensure source files use `@/registry/...` not relative paths |
| `registryDependencies` not installing | Use exact shadcn component names (lowercase-hyphenated: `"button"`, `"data-table"`) |
| CLI can't detect framework | Add explicit `target` to all files to make the item universal |
| Version conflicts | Pin versions in `dependencies`: `"zod@^3.20.0"` |
| `.env` vars already set | `envVars` never overwrites existing values — this is by design |

---

## Checklist Before Publishing

- [ ] `registry.json` has `$schema`, `name`, and `homepage`
- [ ] All items have `name`, `type`, `title`, `description`, and `files`
- [ ] All `files` have `path` and `type`
- [ ] `registry:page` and `registry:file` entries have `target`
- [ ] All `registryDependencies` are correct (names or URLs)
- [ ] All npm `dependencies` are listed
- [ ] Imports in source files use `@/registry/...` paths
- [ ] `npx shadcn build` runs without errors
- [ ] `public/r/[name].json` files exist and contain `files[].content`
- [ ] Local install test passes: `npx shadcn@latest add http://localhost:3000/r/[name].json`
- [ ] Deployed URL is accessible and returns JSON

---

## Official Resources

- Docs: https://ui.shadcn.com/docs/registry
- Getting started: https://ui.shadcn.com/docs/registry/getting-started
- registry.json schema: https://ui.shadcn.com/schema/registry.json
- registry-item.json schema: https://ui.shadcn.com/schema/registry-item.json
- Official template: https://github.com/shadcn-ui/registry-template
- Examples: https://ui.shadcn.com/docs/registry/examples
- Namespaces: https://ui.shadcn.com/docs/registry/namespace
- Authentication: https://ui.shadcn.com/docs/registry/authentication