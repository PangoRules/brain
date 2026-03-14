# .NET in Neovim — Workflow Guide

## Project / Solution Management → `dotnet` CLI, always

```bash
# New class library / project
dotnet new classlib -n Commerce.NewLayer -o backend/src/Commerce.NewLayer

# Add to solution
dotnet sln backend/Commerce.sln add backend/src/Commerce.NewLayer/Commerce.NewLayer.csproj

# Add project reference
dotnet add backend/src/Commerce.Api/Commerce.Api.csproj \
  reference backend/src/Commerce.NewLayer/Commerce.NewLayer.csproj

# Add NuGet package
dotnet add package Dapper --project backend/src/Commerce.Repositories

# EF migration
dotnet ef migrations add AddOrderTable --project ../Commerce.Repositories
dotnet ef database update --project ../Commerce.Repositories
```

There's no GUI for any of this — CLI is the IDE.

---

## Creating a New Class → Just Make the File

OmniSharp picks up any `.cs` file automatically. In Neovim:

```
:e backend/src/Commerce.Services/Services/OrderService.cs
```

Then type your code — LSP autocomplete, go-to-definition, etc. all just work.

---

## Navigation

| Key | Action |
|-----|--------|
| `grd` | Go to definition |
| `gri` | Go to implementation (e.g. jump from interface to concrete class) |
| `grr` | Find all references (Telescope picker) |
| `grt` | Go to type definition (e.g. from variable to its type) |
| `grn` | Rename symbol across the project |
| `gra` | Code actions (add using, implement interface, extract method, fix hints) |
| `gO` | Document symbols (methods, classes in current file) |
| `gW` | Workspace symbols (all symbols across solution) |
| `K` | Hover docs (signature, summary comment) |
| `<C-k>` | Signature help (parameter hints while typing a call) |
| `<leader>sf` | Search files (Telescope) |
| `<leader>sg` | Grep across project |
| `[d` / `]d` | Jump to previous / next diagnostic |
| `<leader>q` | Send all diagnostics to quickfix list |

> **Note:** These are the Neovim 0.10+ built-in LSP bindings used by kickstart.nvim
> (`gr*` prefix). `gi` is a vim built-in ("go to last insert position") — NOT an
> LSP binding.

### Tips
- **`gri`** is the equivalent of "Go to Implementation" in VSCode — essential when
  you're on an interface method and want to jump to the concrete class.
- **`gW`** lets you type a class or method name and jump to it anywhere in
  the solution — faster than searching files manually.
- **`grr`** opens a Telescope picker with every reference; use `<C-q>` inside it to
  send all results to the quickfix list for bulk review.

---

## Code Actions (`<leader>ca`)

Replaces Ctrl+. from VSCode. Common ones OmniSharp surfaces:

| Situation | What to pick |
|-----------|-------------|
| Missing `using` | "Add using …" |
| Interface not implemented | "Implement interface" / "Implement abstract class" |
| Method not in interface | "Extract to interface" |
| `var` hint | "Use explicit type" |
| Generate constructor | "Generate constructor" |
| Generate property | "Generate get/set" |

---

## Diagnostics

| Key | Action |
|-----|--------|
| `]d` | Next diagnostic |
| `[d` | Previous diagnostic |
| `<leader>e` | Open floating diagnostic detail |
| `<leader>q` | All diagnostics → quickfix list |

Quickfix list navigation: `:cnext` / `:cprev` or `]q` / `[q`.

---

## Build / Run — Terminal Inside Neovim

Open a split terminal:

```
:split | terminal
```

Then run your normal commands (`dotnet run`, `dotnet build`, `npm run dev`, etc.).

Alternatively use tmux panes alongside nvim — the most common setup for
keeping the editor and multiple terminals visible at once.

---

## Formatting

`<leader>f` — runs csharpier on the current `.cs` file.

---

## Debugging

Plugins already configured in `lua/custom/plugins/vue_dotnet.lua`:
- `mfussenegger/nvim-dap` — DAP client
- `rcarriga/nvim-dap-ui` — UI panels (variables, call stack, watches, console)
- `netcoredbg` — .NET debug adapter (installed via Mason)

### Setup before first session

```bash
# Build so the dll exists
dotnet build backend/src/Commerce.Api
```

Open nvim from the **project root** (`commerce-vue-dotnet/`). The dll auto-detect
searches `backend/src/**/bin/Debug/net8.0/` and filters to the project-entry dll
(filename == project folder name). If multiple are found you'll get a prompt.

### Keybindings

| Key | Action |
|-----|--------|
| `<F5>` | Start session / Continue |
| `<F10>` | Step over |
| `<F11>` | Step into |
| `<F12>` | Step out |
| `<leader>db` | Toggle breakpoint on current line |
| `<leader>dB` | Conditional breakpoint (enter expression) |
| `<leader>du` | Toggle DAP UI manually |

The UI opens and closes automatically with the session.

### Typical debug session

```
1. <leader>db          → set breakpoints on the lines you care about
2. dotnet build        → rebuild if you changed code
3. <F5>                → starts netcoredbg, UI opens
                          (prompts for dll path if auto-detect finds multiple)
4. Hit your endpoint   → execution pauses at breakpoint
5. <F10> / <F11>       → step through code, watch variables in UI panels
6. <F5>                → continue to next breakpoint
7. <F5> (at end)       → session ends, UI closes
```

### Inspecting variables

In the DAP UI, the **Scopes** panel shows locals and `this` automatically.
To add a watch expression: focus the **Watches** panel and press `a`.

---

## Feature Development Loop

```
1. dotnet new / :e path/to/NewFile.cs    → create the class
2. nvim-vue opens the file               → OmniSharp attaches
3. <leader>ca                            → add missing usings, implement interface
4. gri / grd / grr                       → navigate as you build
5. <leader>f                             → format with csharpier
6. :split | terminal → dotnet build      → verify it compiles
7. :split | terminal → dotnet test       → run tests
8. <leader>db + <F5>                     → debug if needed
9. git add / commit                      → as normal
```

---

## Roslyn Analyzer Hints

The inline virtual text suggestions (e.g. "Use explicit type instead of var",
"Add param element to documentation comment") come from OmniSharp's Roslyn
analyzers. They are hint-level diagnostics — not errors or warnings.

- Use `<leader>ca` on the flagged line to apply the suggestion
- Silence specific rules project-wide via `.editorconfig`:
  ```ini
  [*.cs]
  dotnet_diagnostic.IDE0008.severity = none   # var → explicit type
  dotnet_diagnostic.IDE0161.severity = none   # block-scoped namespace
  ```
- They are harmless to ignore
