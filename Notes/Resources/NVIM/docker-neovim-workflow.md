# Docker in Neovim — Workflow Guide

LSP setup: **dockerls** (Dockerfile) + **docker_compose_language_service** (docker-compose.yml).
Both are always active regardless of which `NVIM_PROFILE` is set.

---

## Quick Reference — `<leader>D` Keybindings

| Key | What happens | Mode |
|-----|-------------|------|
| `<leader>Dl` | Open **lazydocker** TUI in a float | float terminal |
| `<leader>Di` | `docker compose --profile infra up -d` (postgres + minio) | background + notify |
| `<leader>Da` | `docker compose --profile infra --profile app up` | float terminal |
| `<leader>Db` | Same as above but with `--build` (rebuild images) | float terminal |
| `<leader>Ds` | `docker compose down` (stop everything) | background + notify |
| `<leader>Df` | `docker compose logs -f` (follow all logs) | float terminal |
| `<leader>Dp` | `docker compose ps` (container status) | float terminal |
| `<leader>De` | Open `.env` in current project | buffer edit |

> **Float terminal**: auto-closes when the process exits. Press `q` or `<C-c>` then `exit` to close manually.
> **Background + notify**: runs silently, shows a notification on start and on finish/failure.

`<leader>D?` — press `<leader>D` then wait to see the which-key popup listing all bindings.

---

## Typical Daily Workflow

### Starting development

```
1. <leader>Di     → start postgres + minio in background (no terminal clutter)
                     notification: "Docker: starting infra… / done"
2. cd frontend && npm run dev     (in a terminal)
3. cd backend/src/Commerce.Api && dotnet run   (in another terminal or split)
```

### Starting the full Docker stack (all services in containers)

```
1. <leader>Da     → opens float terminal running the full stack
                     logs stream in the float; Ctrl-C to stop
```

### After changing a Dockerfile or service

```
1. Edit Dockerfile / docker-compose.yml  (LSP + treesitter active)
2. <leader>Db     → rebuild + start (float terminal, streams output)
```

### Checking what's running

```
<leader>Dp        → float showing docker compose ps output, closes automatically
```

### Shutting down

```
<leader>Ds        → docker compose down, background, notification on done
```

### Editing environment variables

```
<leader>De        → opens .env in the current working directory
```

---

## LSP Features in Dockerfiles

**dockerls** attaches automatically when you open any file with `dockerfile` filetype
(files named `Dockerfile`, `Dockerfile.dev`, etc.).

| Key | Action |
|-----|--------|
| `K` | Hover docs — explains the instruction (`FROM`, `RUN`, `COPY`, `EXPOSE`, etc.) |
| `<leader>ca` | Code actions — rarely needed, but available |
| `]d` / `[d` | Jump to next / previous diagnostic (e.g. invalid instruction) |
| `<leader>e` | Float with full diagnostic detail |

### What dockerls provides

- **Syntax validation**: flags unknown instructions, bad `EXPOSE` values, etc.
- **Hover docs**: press `K` on `FROM`, `RUN`, `COPY`, `ARG`, `ENV` to read what each instruction does
- **Completions**: `<C-space>` (or your trigger) offers valid Dockerfile instructions

### Example

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build   ← K here: explains FROM multi-stage
WORKDIR /app
COPY *.csproj .
RUN dotnet restore                                 ← K here: explains RUN
```

---

## LSP Features in docker-compose.yml

**docker_compose_language_service** attaches automatically to files named
`docker-compose.yml`, `docker-compose.yaml`, `compose.yml`, `compose.yaml`.

| Key | Action |
|-----|--------|
| `K` | Hover docs — explains the key (`services`, `depends_on`, `healthcheck`, etc.) |
| `<C-space>` | Completions — valid service keys, image names from Docker Hub, condition values |
| `]d` / `[d` | Diagnostics — invalid structure, unknown keys |
| `<leader>e` | Full diagnostic detail |

### What the compose server provides

- **Schema validation**: flags unknown keys, wrong value types
- **Hover docs**: `K` on `depends_on.condition` → explains `service_healthy` vs `service_started`
- **Completions**: typing `condition:` offers `service_started`, `service_healthy`, `service_completed_successfully`
- **Service name awareness**: completing `depends_on` offers the names of other services defined in the file

### Example

```yaml
services:
  api:
    depends_on:
      postgres:
        condition: service_healthy   ← K here: explains what service_healthy means
    ports:
      - "8080:8080"                  ← completion offered after typing "- "
```

---

## Lazydocker TUI (`<leader>Dl`)

Lazydocker is a full terminal UI for Docker. It opens in a floating window over nvim.

### Layout

```
┌─────────────────────────────────────────────────────┐
│  [C] Containers  [I] Images  [V] Volumes  [N] Nets  │
├──────────────────┬──────────────────────────────────┤
│                  │  Logs / Stats / Config / Env      │
│  Container list  │                                  │
│                  │                                  │
└──────────────────┴──────────────────────────────────┘
```

### Navigation inside lazydocker

| Key | Action |
|-----|--------|
| `↑` / `↓` or `j` / `k` | Move between containers |
| `[` / `]` | Switch between panel tabs (Logs, Stats, Config, Env, Top) |
| `enter` | Expand / exec into container |
| `e` | Exec a shell inside the container |
| `l` | View logs for selected container |
| `s` | Stop container |
| `r` | Restart container |
| `d` | Remove container |
| `D` | Remove container + its volumes |
| `u` | Pull latest image |
| `b` | Rebuild service (docker compose up --build) |
| `m` | View container stats (CPU, memory) |
| `?` | Help / all keybindings |
| `q` | Quit lazydocker (closes the float in nvim) |

### Switching panels

Press `1`–`4` at the top to jump between Containers, Images, Volumes, Networks.

---

## Treesitter — Syntax Highlighting

Both `dockerfile` and `yaml` grammars are installed. Syntax highlighting,
indentation, and text objects are active automatically when you open the file.

### Useful text objects (if you have treesitter text objects configured)

| Key | Selects |
|-----|---------|
| `vib` / `dib` | Inside a YAML block |
| `%` | Jump between matching brackets in compose files |

---

## Diagnostics in Both File Types

```
]d / [d      → jump to next / previous diagnostic
<leader>e    → float with full error message
<leader>q    → all diagnostics → quickfix list (:cnext / :cprev to walk them)
```

---

## This Project's Docker Profiles

The `docker-compose.yml` in this project uses two profiles:

| Profile | Services |
|---------|----------|
| `infra` | `postgres` (port 5432), `minio` (ports 9000, 9001) |
| `app` | `frontend` (port 5173), `backend` (port 8080) |

The keybindings already encode the right profile flags:

- `<leader>Di` → infra only (for local frontend/backend dev)
- `<leader>Da` → full stack (everything in Docker)
- `<leader>Db` → full stack with image rebuild

---

## Verification Commands

Run these inside nvim to confirm everything is working:

```
:LspInfo        → shows dockerls / docker_compose_language_service as Active Clients
:Mason          → shows dockerfile-language-server and docker-compose-language-service installed
:TSBufInfo      → shows dockerfile or yaml grammar active on current buffer
:ConformInfo    → (no formatter configured for these filetypes — not needed)
```
