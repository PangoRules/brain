---
title: CookHomie Roadmap
tags: [project, cookhomie, roadmap]
---

> **Navigation:** [[00-CookHomie|Overview]] → [[01-Architecture]] → [[02-DataModel]] → [[03-RepoStructure]] → 04 Roadmap

## Version 1 — MVP (Current)

**Goal:** Working local web app with inventory, recipes, shopping list, and MCP tools.

### Milestone 1 — Foundation
- [ ] Create monorepo folder structure
- [ ] Set up Docker Compose (PostgreSQL + API + Web + MCP)
- [ ] C# Domain layer (entities, enums, interfaces)
- [ ] C# Application layer (use cases, DTOs)
- [ ] C# Infrastructure (EF Core DbContext, repositories, migrations)
- [ ] C# WebApi (controllers, DI wiring, Program.cs)

### Milestone 2 — Frontend
- [ ] Nuxt 3 scaffold with default layout (sidebar nav)
- [ ] Dashboard page (expiring items, recipe matches, shopping count)
- [ ] Inventory page + AddItemModal
- [ ] Recipes page + RecipeCard + AddRecipeModal
- [ ] Recipe detail page (ingredients with in-stock highlighting)
- [ ] Shopping list page

### Milestone 3 — MCP Server
- [ ] FastMCP scaffold + api_client.py
- [ ] get_inventory tool
- [ ] get_expiring_items tool
- [ ] suggest_recipes tool
- [ ] get_missing_ingredients tool
- [ ] build_shopping_list tool
- [ ] add_to_shopping_list tool
- [ ] system.md prompt

### Milestone 4 — Integration
- [ ] End-to-end: add item → suggest recipe → build shopping list
- [ ] Wire MCP server into Claude Code / opencode
- [ ] docker-compose.dev.yml with hot reload

---

## Version 2 — Intelligence

- Expiration tracking with alerts
- Priority-to-use items (use what expires first)
- Weekly meal planning
- Food preferences and dislikes
- Leftover suggestions
- API key auth for multi-device access

---

## Version 3 — Automation

- Barcode scanning or easier entry
- AI-assisted parsing from grocery receipts
- Budget-aware planning
- Nutrition-aware planning
- Tighter integration with other HomieOS modules (GymHomie → protein goals)

---

## 🔗 Related

- [[00-CookHomie|Overview]]
- [[Notes/Ideas/HomieOS|HomieOS]]
