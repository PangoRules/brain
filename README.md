# 🧠 Pango Vault System (Obsidian Knowledge OS)

## Overview

This vault is structured as a lightweight personal operating system
built on:

-   📁 [[PARA Framework]]-inspired folder structure
-   🧩 Structured templates
-   🔍 Dataview-powered dashboards
-   ⚡ Quick capture workflows
-   🔁 Review-based organization

Goal:

> Capture fast → Clarify intentionally → Link deeply → Execute clearly →
> Archive cleanly.

------------------------------------------------------------------------

# 📂 Folder Structure

    .
    ├── Archives
    ├── Daily
    ├── Inbox
    ├── Notes
    │   ├── Areas
    │   ├── Ideas
    │   ├── People
    │   ├── Projects
    │   └── Resources
    └── Templates

------------------------------------------------------------------------

## 📥 Inbox

Fast capture zone. No organizing. Just dump ideas, notes, links,
thoughts.

## 📅 Daily

Operational thinking space for planning, execution, and reflection.

## 📁 Notes

### 🚀 Projects

Short-term goal-driven efforts with structure: - Objective - Outcome -
Tasks - Related links

### 🧭 Areas

Long-term responsibilities (Career, Health, Finance, Learning, etc.).

### 💡 Ideas

Raw thoughts that evolve: raw → validated → active → archived

### 📚 Resources

Reference material (books, docs, research, articles).

### 👥 People

Relationship intelligence system: - Context - Conversations -
Opportunities - Linked projects

## 🗄 Archives

Completed or inactive notes to keep workspace clean.

------------------------------------------------------------------------

# 🧩 Templates

Located in `Templates/`

-   template_daily
-   template_project
-   template_idea
-   template_resource

Templates standardize note creation and reduce friction.

------------------------------------------------------------------------

# 🔌 Plugins Used

## Core System

### Dataview
Dynamic dashboards and project tracking.

Use this syntax:

```dataview
table start, due
from "Notes/Projects"
where status != "completed"
```

---

### Templater
Dynamic template variables such as:

<% tp.date.now("YYYY-MM-DD") %>

Used for:
- Auto-filling dates
- Auto-filling titles
- Template automation

---

### Periodic Notes
Auto-manages daily notes inside the `Daily/` folder.

---

### QuickAdd
Rapid note creation macros for:
- New Idea
- New Project
- New Resource
- Quick Capture

---

### Calendar
Visual navigation for daily notes.

---

### Kanban
Optional visual task boards inside project notes.

---

### Breadcrumbs
Hierarchy and structural visualization between notes.

---

### Juggl
Enhanced graph view for exploring relationships.

---

### Folder Notes
Allows folders to behave like index notes.

---

### Outliner
Improves bullet workflow and structured thinking.

---

### Tag Wrangler
Tag cleanup and management.

---

# 🏠 Home Dashboard

`Home.md` acts as the command center.

It dynamically displays:

- 🔥 Active Projects
- 💡 Idea Backlog
- 🧠 Resources

Example Dataview block:

```dataview
list
from "Notes/Ideas"
where status = "raw"
sort file.ctime desc
```

---

# 🔄 Workflow

1️⃣ Capture → Inbox  
2️⃣ Clarify → Move to proper folder  
3️⃣ Execute → Work from Daily + Projects  
4️⃣ Review → Weekly cleanup and linking  
5️⃣ Archive → Move completed work  

---

# 📌 Rules

- Capture first, organize later.
- Link generously.
- Review weekly.
- Archive aggressively.
- Keep YAML minimal.
- Avoid metadata bloat.

---

# 🚀 Purpose

This vault is not a notebook.

It is a:

- Thinking system  
- Execution system  
- Knowledge base  
- Relationship tracker  
- Personal operating system  

---

**Pango OS v1**