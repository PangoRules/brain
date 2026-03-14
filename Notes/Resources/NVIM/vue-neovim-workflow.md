# Vue 3 + TypeScript in Neovim — Workflow Guide

LSP setup: **ts_ls** (TypeScript inside `.vue` files) + **vue_ls** (template syntax).
Both run together — TypeScript errors show in templates, props are type-checked, etc.

---

## Creating a New Feature — Project Order

Follow the layer order from the bottom up:

```
1. types/api/{domain}Types.ts       → define request/response interfaces
2. services/{domain}Api.ts          → API client using ApiResult pattern
3. {domain}Api.spec.ts              → service tests
4. composables/use{Domain}.ts       → reactive state wrapping the service
5. use{Domain}.spec.ts              → composable tests
6. components/{domain}/Foo.vue      → UI components
7. pages/{route}.vue                → page (auto-routed by filename)
8. i18n/locales/en/*.json           → English strings
   i18n/locales/es/*.json           → Spanish strings
```

### Creating files

Just open the path — Neovim creates the file, vue_ls/ts_ls attach automatically:

```
:e frontend/src/services/ordersApi.ts
:e frontend/src/composables/useOrders.ts
:e frontend/src/components/orders/OrderCard.vue
:e frontend/src/pages/orders/index.vue
```

File-based routing: `pages/orders/index.vue` → `/orders`,
`pages/orders/[id].vue` → `/orders/:id`.

---

## Navigation

| Key | Action |
|-----|--------|
| `gd` | Go to definition — works in templates too (component → its file) |
| `gD` | Go to declaration |
| `gi` | Go to implementation (interface → concrete class) |
| `go` | Go to type definition (variable → its type) |
| `gr` | Find all references — finds usages across `.vue` and `.ts` files |
| `K` | Hover docs — shows prop types, JSDoc, inferred types |
| `<C-k>` | Signature help — parameter hints while typing a function call |
| `<leader>rn` | Rename symbol — renames across all files including templates |
| `<leader>ca` | Code actions — add import, fix TS error, refactor |
| `<leader>sf` | Search files (Telescope) |
| `<leader>sg` | Grep across project |
| `<leader>ss` | Document symbols — all exports/functions in current file |
| `<leader>sS` | Workspace symbols — find any component/composable by name |
| `[d` / `]d` | Jump to previous / next diagnostic |
| `<leader>q` | All diagnostics → quickfix list |

### Tips

- **`gd` on a component in a template** (e.g. `<ProductCard`) jumps straight to
  `ProductCard.vue` — same as Ctrl+click in VSCode.
- **`K` on a prop** shows the TypeScript type — useful when using a Vuetify
  component and you want to know what a prop expects.
- **`<leader>rn`** on a composable function renames it everywhere — `.vue` templates,
  `.ts` files, test files.
- **`gr` + `<C-q>`** inside the Telescope picker dumps all references to the
  quickfix list, great for auditing where a composable is used.

---

## Code Actions (`<leader>ca`)

| Situation | What to pick |
|-----------|-------------|
| Missing import in `.ts` file | "Add import from …" |
| `defineProps` missing a prop | Fix TS error suggestion |
| Unused import | "Remove unused imports" |
| Infer return type | "Add return type annotation" |
| Convert to `<script setup>` | Available if on legacy Options API |

> **Auto-imports note**: this project uses `unplugin-auto-import` and
> `unplugin-vue-components`. Vue APIs (`ref`, `computed`, `onMounted`) and
> components in `src/components/` are auto-imported — you don't need to write
> import statements for them. ts_ls still type-checks them correctly.

---

## TypeScript in Vue Files

The hybrid mode means full TS support inside `<script setup lang="ts">` and
**in the template**. You get:

- Type errors on wrong prop values in templates
- Autocomplete for component props
- `K` hover on template variables shows their type
- `gd` on a ref/computed in template jumps to where it's defined in `<script>`

### Patterns used in this project

```typescript
// Props
defineProps<{ title: string; items: Product[] }>()

// Emits
defineEmits<{ submit: [data: OrderForm]; cancel: [] }>()

// ApiResult pattern — always handle both branches
const result = await ordersApi.getOrders()
if (result.success) {
  orders.value = result.data
} else {
  error.value = result.error
}
```

---

## Diagnostics

| Key | Action |
|-----|--------|
| `]d` | Next diagnostic (TS error, type mismatch, etc.) |
| `[d` | Previous diagnostic |
| `<leader>e` | Float with full diagnostic message |
| `<leader>q` | All diagnostics → quickfix list |

TypeScript errors in templates show as diagnostics just like in `.ts` files.

---

## Formatting

`<leader>f` — runs prettier on the current file.

Works on: `.vue`, `.ts`, `.js`, `.json`, `.css`

---

## Dev Server & Tests

```bash
# Dev server (port 5173)
cd frontend && npm run dev

# Tests in watch mode
npm run test

# Run tests once (CI / quick check)
npm run test:run

# Coverage report (target: 75%+)
npm run test:coverage

# Type check + build
npm run build
```

Open a split terminal inside nvim for these:

```
:split | terminal
```

---

## i18n Workflow

All user-facing strings live in locale files — never hardcode text in components.

```
frontend/src/i18n/locales/en/    ← English (one file per domain)
frontend/src/i18n/locales/es/    ← Spanish
```

### Adding a new string

1. Add the key to `en/{domain}.json` and `es/{domain}.json`
2. Use it in the component:
   ```vue
   {{ $t('orders.emptyState') }}
   ```
3. For TypeScript files: `const { t } = useI18n()`

### Navigating between locale files

```
<leader>sf  → type "en/orders" or "es/orders" to jump to the locale file
```

---

## Debugging

Frontend debugging is **browser-based** — Vue Devtools gives you everything
nvim-dap would give you for component state:

- **Vue Devtools** (browser extension) — component tree, props, emits, Pinia stores
- **Browser DevTools** → Sources tab — set breakpoints in TS (Vite serves source maps)

For debugging Vitest tests directly in Neovim, you can run:

```bash
npx vitest --reporter=verbose --run src/composables/useOrders.spec.ts
```

nvim-dap + `vscode-js-debug` can attach to a Node process if you ever need
step-through debugging of a Vite SSR setup or a Node script, but for this
project the browser is sufficient.

---

## Feature Development Loop

```
1. :e src/types/api/orderTypes.ts        → define the types
2. :e src/services/ordersApi.ts          → write the API service
3. npm run test:run                      → green on service tests
4. :e src/composables/useOrders.ts       → wrap with reactive state
5. npm run test:run                      → green on composable tests
6. :e src/components/orders/OrderCard.vue → build the component
7. :e src/pages/orders/index.vue         → wire up the page
8. gd / gr / K / <leader>ca             → navigate and fix TS errors
9. <leader>f                             → format
10. npm run build                        → type check passes
11. git add / commit
```

---

## File Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `OrderCard.vue` |
| Composables | camelCase, `use` prefix | `useOrders.ts` |
| Services | camelCase, `Api` suffix | `ordersApi.ts` |
| Types | camelCase, `Types` suffix | `orderTypes.ts` |
| Tests | Same as source + `.spec.ts` | `useOrders.spec.ts` |
| Pages | kebab-case or `[param]` | `index.vue`, `[id].vue` |
