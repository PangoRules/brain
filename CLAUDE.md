# CLAUDE.md — Pango OS v1 (Obsidian Knowledge OS)

## What This Project Is

A personal knowledge operating system built on Obsidian. Not just a notebook — a full thinking, execution, and knowledge management system using the **PARA Framework** (Projects, Areas, Resources, Archives).

**Core mantra:** Capture fast → Clarify intentionally → Link deeply → Execute clearly → Archive cleanly.

---

## Directory Structure

```
/home/pango/PangoObsidianVault/Pango/
├── Home.md              # Command center dashboard (Dataview queries for active work)
├── README.md            # System documentation
├── CLAUDE.md            # This file
├── Daily/               # Operational journal (YYYY-MM-DD.md files)
├── Inbox/               # Fast capture zone (dump first, organize later)
├── Archvies/            # Completed/inactive notes (note: folder name has typo on disk)
├── Assets/              # Images, attachments
├── Templates/           # 5 structured templates (project, daily, idea, resource, book)
├── Notes/
│   ├── Projects/        # Active goal-driven work (e.g., python/mini-search-engine/)
│   ├── Areas/           # Long-term responsibilities (NLP, WebCrawling, bash, Learning)
│   ├── Ideas/           # Raw thoughts (raw → validated → active → archived)
│   ├── Resources/       # Reference material (books, docs, snippets)
│   └── People/          # Relationship intelligence
└── .obsidian/           # Obsidian config, plugins, theme
```

---

## Active Content (as of 2026-03)

### Projects
- **Notes/Projects/python/mini-search-engine/** — Python web crawler + search engine project
  - Files: features, structure, DB models, roadmap (00–04 numbered files)

### Areas
- **Notes/Areas/NLP/** — NLP theory (Tokenization, TF-IDF, Stemming, Inverted Index, Stop Words, Corpus, Normalizing Text) + Python implementations
- **Notes/Areas/WebCrawling/** — Web crawling concepts (HTTP Requests, HTML Parsing, URL Frontier, Extracting Links, Crawl Depth, Duplicate Avoidance, Robots.txt, URL Normalization) + Python implementations
- **Notes/Areas/bash/exercises/** — Shell scripting practice (beginner → intermediate → interview-style → mini-projects)

### Resources
- **Notes/Resources/Books/The_Linux_Command_Line/** — 36-chapter breakdown
- **Notes/Resources/Python/** — Reference snippets (dicts, dataclasses, regex, file I/O, Counter, string methods)
- **Notes/Resources/NVIM/** — Neovim workflow guides (Docker, .NET, Vue)
- **Notes/Resources/CS_Fundamentals/** — CS theory (Big O Notation, BFS/DFS)

### Daily Notes
- Located in `Daily/`, format: `YYYY-MM-DD.md`

---

## Key Conventions

### YAML Frontmatter
- **Projects:** `title`, `status` (active/completed), `start`, `due`, `tags: [project]`
- **Daily:** `date`, `tags: [daily]`
- **Ideas:** `status: raw`, `tags: [idea]`, `created`
- **Resources:** `tags: [resource]`, `source`, `type`, `author`, `status`
- **Books:** `tags: [resource, book]`, `author`, `status: in-progress`, `area`, `started`, `finished`

### File Naming
- Projects use numbered prefixes: `00-`, `01-`, `02-` for sequence
- Daily notes: `YYYY-MM-DD.md`
- Concepts + paired implementation (e.g., `Tokenization.md` + `Tokenization_in_Python.md`)
- Resource files: `Descriptive_Names_With_Underscores.md`

### Linking
- `[[Note]]` for internal links, `[[Folder/Note|Alias]]` for aliased links
- Hub notes per domain (e.g., `NLP.md`, `Learning Area.md`)
- Link generously across concepts and implementations

---

## Plugins in Use

| Plugin | Purpose |
|--------|---------|
| **Dataview** | Dynamic dashboards — queries projects, filters by status, sorts by date |
| **Templater** | Auto-fills dates (`2026-03-08`), titles, on file creation |
| **Periodic Notes** | Auto-creates daily notes in `Daily/` folder |
| **QuickAdd** | Macros for one-click creation: New Idea, New Project, New Resource |
| **Folder Notes** | Folders can act as index notes |
| **Calendar** | Visual date picker for daily notes |
| **Kanban** | Visual task boards inside project notes |
| **Juggl** | Enhanced graph view for relationship exploration |
| **Outliner** | Better bullet/hierarchical structure |
| **Tag Wrangler** | Tag cleanup and management |

**Theme:** Things

---

## Dataview Query Patterns

```dataview
table start, due
from "Notes/Projects"
where status != "completed"
sort due asc
```

```dataview
list
from "Notes/Ideas"
where status = "raw"
sort file.ctime desc
```

---

## Workflow

1. **Capture** → Drop into `Inbox/`
2. **Clarify** → Move to proper folder (Project/Area/Resource/Idea)
3. **Execute** → Work from `Daily/` + `Notes/Projects/`
4. **Review** → Weekly cleanup and linking
5. **Archive** → Move completed work to `Archives/`

---

## Rules

- Capture first, organize later
- Link generously
- Review weekly, archive aggressively
- Keep YAML minimal — avoid metadata bloat
- Notes are actionable, not just reference

---

## Git Workflow for Notes

This vault is a git repo. When helping add new notes:

1. **Check current branch** with `git branch`
2. **If on `main`**, create and checkout a new branch before creating any files:
   - Branch naming: `notes/<area-or-topic>` (e.g., `notes/nlp-word2vec`, `notes/bash-arrays`, `notes/webcrawling-sitemaps`)
   - Use kebab-case, keep it short and descriptive
3. **Create the note(s)** on that branch
4. **Commit** the new files with a clear message (no `Co-Authored-By` tag)
5. **Do not push** unless the user asks

If already on a feature branch (not `main`), just commit there — no new branch needed.

---

## Claude Permissions (.claude/settings.local.json)

WebFetch is allowed for these domains:
- `realpython.com`, `www.w3schools.com`, `medium.com`, `www.geeksforgeeks.org`
- `www.stratascratch.com`, `dataknowsall.com`, `www.pingcap.com`, `devtoolbox.dedyn.io`
- WebSearch is also enabled

---

## How to Help the User

- When asked to create notes, follow the YAML frontmatter conventions above
- Place files in the correct PARA folder based on content type
- Use `[[wikilinks]]` for internal references
- Keep new notes minimal — no metadata bloat
- When building on existing content, check `Notes/Areas/NLP/`, `Notes/Areas/WebCrawling/`, and `Notes/Projects/` for context
- The mini-search-engine project connects NLP theory + WebCrawling concepts to Python implementation
- NLP and WebCrawling areas both follow the same pattern: `Concepts/` for theory, `Python/` for implementation
