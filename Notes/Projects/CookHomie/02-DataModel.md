---
title: CookHomie Data Model
tags: [project, cookhomie, database]
---

> **Navigation:** [[00-CookHomie|Overview]] → [[01-Architecture]] → 02 DataModel → [[03-RepoStructure]] → [[04-Roadmap]]

## Entities (v1)

### InventoryItem
| Field | Type | Notes |
|-------|------|-------|
| Id | uuid (PK) | |
| Name | text | required |
| Category | text | e.g. "dairy", "vegetable" |
| Location | enum | Fridge / Freezer / Pantry / Spices |
| Quantity | decimal | |
| Unit | text | g, ml, units, etc. |
| ExpiresAt | date? | nullable |
| IsOpened | bool | |
| Notes | text? | nullable |
| CreatedAt | timestamp | |
| UpdatedAt | timestamp | |

### Recipe
| Field | Type | Notes |
|-------|------|-------|
| Id | uuid (PK) | |
| Name | text | required |
| Instructions | text | |
| PrepMinutes | int | |
| CookMinutes | int | |
| Tags | text[] | PostgreSQL native array: "quick", "high-protein"… |
| Source | text? | URL or "manual" |
| CreatedAt | timestamp | |

### RecipeIngredient
| Field | Type | Notes |
|-------|------|-------|
| Id | uuid (PK) | |
| RecipeId | uuid (FK → Recipe) | |
| IngredientName | text | name-based, not FK to inventory |
| Quantity | decimal | |
| Unit | text | |
| IsOptional | bool | |

### ShoppingItem
| Field | Type | Notes |
|-------|------|-------|
| Id | uuid (PK) | |
| Name | text | required |
| Quantity | decimal? | |
| Unit | text? | |
| Priority | enum | Low / Normal / High |
| IsBought | bool | |
| LinkedRecipeId | uuid? | nullable FK → Recipe |
| CreatedAt | timestamp | |

## Key Design Decisions

**Name-based ingredient matching:** RecipeIngredient stores `IngredientName` as text rather than a FK to InventoryItem. Ingredient names are naturally fuzzy ("chicken breast" ≈ "chicken"). The C# API uses case-insensitive contains matching; the MCP layer can use AI for smarter fuzzy matching.

**Tags as text[]:** PostgreSQL native array avoids a join table for MVP. Filterable with `ANY()` queries.

**All IDs are UUIDs:** Safe for future multi-user or sync scenarios without conflicts.

**MealPlan is v2:** Kept out of the MVP data model to stay focused.

## 🔗 Related

- [[00-CookHomie|Overview]]
- [[01-Architecture]]
- [[03-RepoStructure]]
