# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Factory Inventory Management System — Full-stack app with Vue 3 frontend, Python FastAPI backend, and in-memory mock data (no database).

> ⚠️ **This repository is PUBLIC.** Do not commit credentials, internal hostnames, or private registry URLs. `client/.npmrc` pins the public npm registry and `client/package-lock.json` is gitignored to prevent registry leaks — leave both in place.

## Commands

```bash
# Start both servers (background)
./scripts/start.sh
./scripts/stop.sh

# Backend (port 8001)
cd server && uv run python main.py

# Frontend (port 3000)
cd client && npm install && npm run dev

# Build frontend
cd client && npm run build

# Run all tests (55 tests)
cd tests && uv run pytest backend/ -v

# Run single test file
cd tests && uv run pytest backend/test_inventory.py -v

# Run single test
cd tests && uv run pytest -k test_get_all_inventory -v

# Test coverage
cd tests && uv run pytest --cov=../server --cov-report=html
```

## Stack

- **Frontend**: Vue 3 + Composition API + Vite (port 3000)
- **Backend**: Python FastAPI (port 8001)
- **Data**: JSON files in `server/data/` → loaded at startup via `server/mock_data.py` → filtered in-memory
- **i18n**: English/Japanese with automatic USD/JPY currency switching via `useI18n` composable

## Architecture

**Data flow**: Vue filters → `client/src/api.js` (axios) → FastAPI endpoints → `apply_filters()` / `filter_by_month()` on in-memory data → Pydantic validation → JSON response → Vue refs → computed properties for display

**Filter system**: 4 global filters managed by `useFilters` composable (singleton pattern):
- Time Period (month/quarter) — NOT supported by `/api/inventory` (no time dimension)
- Warehouse, Category, Order Status — passed as query params (`?warehouse=...&category=...&status=...&month=...`)
- `"all"` value bypasses that filter dimension

**Composables** (`client/src/composables/`):
- `useFilters.js` — singleton; exports refs + `getCurrentFilters()` returning `{warehouse, category, status, month}`
- `useI18n.js` — locale switching (en/ja), currency auto-selection, product/customer/warehouse name translations
- `useAuth.js` — mock user data, `getInitials()`, `logout()`

**API endpoints**:
- `GET /api/inventory` — warehouse, category filters
- `GET /api/inventory/{item_id}` — single item (404 if missing)
- `GET /api/orders` — warehouse, category, status, month filters
- `GET /api/orders/{order_id}` — single order (404 if missing)
- `GET /api/dashboard/summary` — all 4 filters
- `GET /api/demand`, `/api/backlog` — no filters
- `GET /api/spending/*` — summary, monthly, categories, transactions
- `GET /api/reports/quarterly`, `/api/reports/monthly-trends`

## Critical Tool Usage Rules

### Subagents
- **vue-expert**: **MANDATORY for creating or significantly modifying any `.vue` file**
- **code-reviewer**: Use after writing significant code
- **Explore**: For codebase understanding and pattern searching

### Skills
- **backend-api-test**: Use when writing/modifying tests in `tests/backend/`

### MCP Tools
- **GitHub MCP** (`mcp__github__*`) for ALL GitHub operations (exception: local branches use `git checkout -b`)
- **Playwright MCP** (`mcp__playwright__*`) for browser testing against `localhost:3000` / `localhost:8001`

## Common Issues

1. Use unique keys in `v-for` (`sku`, `month`, etc.) — never array index
2. Validate dates before `.getMonth()` calls
3. Update Pydantic models in `server/main.py` when changing JSON data structure
4. Revenue goals: $800K/month single, $9.6M YTD all months
5. Mock data spans 12 months of 2025; warehouses: San Francisco, London, Tokyo; categories: Circuit Boards, Sensors, Actuators, Controllers, Power Supplies

## Design System

- Colors: Slate/gray (#0f172a, #64748b, #e2e8f0)
- Status: green (delivered), blue (shipped), yellow (processing), red (backordered)
- Charts: Custom SVG, CSS Grid layouts
- No emojis in UI
