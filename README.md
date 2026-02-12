# SwainOS Documentation

## Overview
This repository contains the living project documentation for SwainOS, including product goals, implementation plans, backend/frontend mapping notes, and Supabase migration history.

## Repository Structure
- `docs/`: product and technical documentation
- `action-plan/`: milestone plans and execution log
- `supabase/migrations/`: ordered SQL migrations for schema and analytics rollups

## Key Documents
- `docs/purpose.md`
- `docs/goals.md`
- `docs/objectives.md`
- `docs/scope-and-modules.md`
- `docs/success-criteria-and-phases.md`
- `docs/swainos-code-documentation-backend.md`
- `docs/swainos-code-documentation-frontend.md`
- `docs/itinerary-data-mapping.md`

## Action Plans
Use `action-plan/` for delivery tracking:
- plan templates and completed plans by milestone
- `action-plan/action-log` as the timestamped implementation log

## Supabase Migrations
- Migrations are versioned with zero-padded prefixes (for example `0001_...sql`).
- New migrations should be additive, clearly named, and committed in execution order.
- Current history includes foundational schema setup through itinerary lead-flow and revenue rollup migrations.

## Related Repositories
- Frontend: `https://github.com/ianswain2/swainos_frontend`
- Backend: `https://github.com/ianswain2/swainos_backend`
- Documentation: `https://github.com/ianswain2/swainos_documentation`
