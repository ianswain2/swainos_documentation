# SwainOS Documentation

## Overview
Project documentation repository for product scope, technical contracts, data mappings, action plans, and references to backend-owned Supabase migration history.

## Structure
- `docs/`: product and engineering reference documents
- `action-plan/`: execution plans and action log
- backend-owned migrations live in `SwainOS_BackEnd/supabase/migrations/`

## Core documents

| Document | Role |
|---|---|
| `docs/swainos-code-documentation-backend.md` | FastAPI layout, endpoint families, data/rollup model |
| `docs/swainos-code-documentation-frontend.md` | Next.js structure, routes, SSR patterns, UX notes |
| `docs/frontend-data-queries.md` | **Who calls what** — web app ↔ `/api/v1/*` map |
| `docs/sample-payloads.md` | Example JSON contracts |
| `docs/swainos-terminology-glossary.md` | Canonical UI terms and field naming |

Cross-links: **Related documentation** in each file points to the other core docs.

## Migration Standards
- Versioned, additive SQL files with ordered numeric prefixes
- Historical migrations remain immutable
- New behavior is introduced through new migration files

## Related Repositories
- Frontend: `https://github.com/ianswain2/swainos_frontend`
- Backend: `https://github.com/ianswain2/swainos_backend`
- Documentation: `https://github.com/ianswain2/swainos_documentation`
