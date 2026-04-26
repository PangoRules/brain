---
title: CookHomie
status: active
start: 2026-04-25
tags: [project, homie-os, cookhomie]
---

> **Navigation:** 00 Overview → [[01-Architecture]] → [[02-DataModel]] → [[03-RepoStructure]] → [[04-Roadmap]]

CookHomie is the food and kitchen module of [[Notes/Ideas/HomieOS|HomieOS]].

A kitchen-aware AI assistant that knows what you have, what you like, what's expiring, and what you can cook next. Not a generic recipe app — a personal kitchen intelligence layer.

## 🎯 Objective

Build a local-first web app that tracks kitchen inventory, suggests recipes, generates shopping lists, and exposes all of this as MCP tools so any AI client can interact with your kitchen data.

## 🗂️ Outcome

A fully functional v1 with:
- Inventory management (fridge, freezer, pantry, spices)
- Recipe browser with ingredient match %
- Shopping list with auto-add from recipes
- Python MCP server exposing kitchen tools to AI clients
- Everything running locally via Docker Compose

## 🧩 Stack

| Layer | Tech |
|-------|------|
| Frontend | Nuxt 3 · Vue 3 |
| Backend | ASP.NET Core 9 · Clean Architecture |
| Database | PostgreSQL · EF Core |
| AI Gateway | Python MCP Server (FastMCP) |
| Infra | Docker Compose |

## 🧩 Tasks

- [ ] Scaffold monorepo structure
- [ ] Set up Docker Compose (PostgreSQL + API + Web + MCP)
- [ ] Build C# Domain + Application layers
- [ ] Build C# Infrastructure (EF Core + migrations)
- [ ] Build C# WebApi controllers
- [ ] Build Nuxt frontend (Dashboard, Inventory, Recipes, Shopping)
- [ ] Build Python MCP server with 6 tools
- [ ] End-to-end test: add item → suggest recipe → build shopping list

## 🔗 Related

- [[Notes/Ideas/CookHomie|CookHomie Idea]]
- [[Notes/Ideas/HomieOS|HomieOS]]
- [[01-Architecture]]
- [[02-DataModel]]
- [[03-RepoStructure]]
- [[04-Roadmap]]
