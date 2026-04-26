---
title: CookHomie Repo Structure
tags: [project, cookhomie, structure]
---

> **Navigation:** [[00-CookHomie|Overview]] → [[01-Architecture]] → [[02-DataModel]] → 03 RepoStructure → [[04-Roadmap]]

## Full Monorepo Layout

```
CookHomie/
├── src/
│   ├── CookHomie.Api/                  # C# solution (Clean Architecture)
│   │   ├── CookHomie.Domain/
│   │   │   ├── Entities/               # InventoryItem, Recipe, RecipeIngredient, ShoppingItem
│   │   │   ├── Enums/                  # Location, Priority
│   │   │   └── Interfaces/             # IInventoryRepository, IRecipeRepository, IShoppingRepository
│   │   ├── CookHomie.Application/
│   │   │   ├── UseCases/               # GetInventory, AddItem, GetRecipeMatches, BuildShoppingList…
│   │   │   ├── DTOs/                   # Request/Response objects
│   │   │   └── Interfaces/             # IInventoryService, IRecipeService, IShoppingService
│   │   ├── CookHomie.Infrastructure/
│   │   │   ├── Persistence/            # AppDbContext (EF Core)
│   │   │   ├── Repositories/           # InventoryRepository, RecipeRepository, ShoppingRepository
│   │   │   └── Migrations/
│   │   └── CookHomie.WebApi/
│   │       ├── Controllers/            # InventoryController, RecipesController, ShoppingController
│   │       ├── Program.cs
│   │       └── appsettings.json
│   │
│   ├── CookHomie.Web/                  # Nuxt 3 frontend
│   │   ├── pages/
│   │   │   ├── index.vue               # Dashboard
│   │   │   ├── inventory.vue           # Inventory list + AddItemModal
│   │   │   ├── recipes/
│   │   │   │   ├── index.vue           # Recipe browser + AddRecipeModal
│   │   │   │   └── [id].vue            # Recipe detail
│   │   │   └── shopping.vue            # Shopping list
│   │   ├── components/
│   │   │   ├── inventory/
│   │   │   │   ├── InventoryCard.vue
│   │   │   │   ├── InventoryFilters.vue
│   │   │   │   └── AddItemModal.vue
│   │   │   ├── recipes/
│   │   │   │   ├── RecipeCard.vue
│   │   │   │   ├── IngredientList.vue
│   │   │   │   └── AddRecipeModal.vue
│   │   │   ├── shopping/
│   │   │   │   └── ShoppingItem.vue
│   │   │   └── ui/                     # Shared: Modal.vue, Badge.vue, ExpiryWarning.vue
│   │   ├── composables/
│   │   │   ├── useInventory.ts
│   │   │   ├── useRecipes.ts
│   │   │   └── useShopping.ts
│   │   ├── layouts/
│   │   │   └── default.vue             # Sidebar nav
│   │   ├── server/api/                 # Nuxt proxy to C# API
│   │   └── nuxt.config.ts
│   │
│   └── CookHomie.MCP/                  # Python MCP server (AI gateway)
│       ├── server.py                   # FastMCP app entry point
│       ├── api_client.py               # HTTP client for C# API
│       ├── tools/
│       │   ├── inventory.py            # get_inventory, get_expiring_items
│       │   ├── recipes.py              # suggest_recipes, get_missing_ingredients
│       │   └── shopping.py             # build_shopping_list, add_to_shopping_list
│       ├── prompts/
│       │   └── system.md               # CookHomie AI personality & context
│       └── pyproject.toml
│
├── docker/
│   ├── Dockerfile.api
│   ├── Dockerfile.web
│   └── Dockerfile.mcp
├── docker-compose.yml                  # Production-style local run
├── docker-compose.dev.yml              # Hot reload for dev
├── .env.example
└── README.md
```

## API Endpoints Reference

### /api/inventory
| Method | Route | Description |
|--------|-------|-------------|
| GET | /api/inventory | List all · ?location=Fridge&expiring=true |
| GET | /api/inventory/{id} | Single item |
| POST | /api/inventory | Add item |
| PUT | /api/inventory/{id} | Update item |
| DELETE | /api/inventory/{id} | Remove item |
| GET | /api/inventory/expiring | Items expiring within N days |
| GET | /api/inventory/low | Items below minimum quantity |

### /api/recipes
| Method | Route | Description |
|--------|-------|-------------|
| GET | /api/recipes | List all · ?tag=quick&maxMinutes=30 |
| GET | /api/recipes/{id} | Single recipe with ingredients |
| POST | /api/recipes | Add recipe |
| PUT | /api/recipes/{id} | Update recipe |
| DELETE | /api/recipes/{id} | Remove recipe |
| GET | /api/recipes/matches | Cookable from current inventory (with match %) |
| GET | /api/recipes/{id}/missing | Missing ingredients for recipe |

### /api/shopping
| Method | Route | Description |
|--------|-------|-------------|
| GET | /api/shopping | Current shopping list |
| POST | /api/shopping | Add item |
| PUT | /api/shopping/{id} | Update item (bought, qty) |
| DELETE | /api/shopping/{id} | Remove item |
| POST | /api/shopping/clear-bought | Remove all bought items |
| POST | /api/shopping/from-recipe/{id} | Auto-add missing ingredients |

## MCP Tools Reference

| Tool | Description |
|------|-------------|
| get_inventory(location?) | Current kitchen inventory, optional location filter |
| get_expiring_items(days?) | Items expiring within N days (default 3) |
| suggest_recipes(preferences?) | AI-ranked recipe matches from current inventory |
| get_missing_ingredients(recipe_id) | What's missing for a specific recipe |
| build_shopping_list(goal?) | AI-generated smart shopping list |
| add_to_shopping_list(items[]) | Write specific items to shopping list |

## 🔗 Related

- [[00-CookHomie|Overview]]
- [[01-Architecture]]
- [[02-DataModel]]
- [[04-Roadmap]]
