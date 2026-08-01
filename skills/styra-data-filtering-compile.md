---
name: Push authorization down to the database with OPA data filtering
description: Use the OPA Compile API (partial evaluation) to turn a Rego policy into UCAST or SQL conditions you apply to a query, so users only ever see rows they are allowed to see.
api: openapi/styra-enterprise-opa-openapi.yaml
operations: [compileQueryWithPartialEvaluation, health]
---

# Data filtering via partial evaluation (Compile API)

Instead of fetching rows and filtering in the app, ask OPA to compile the policy into conditions you fold into your database query. This is Enterprise OPA's data-filtering capability.

## Auth
- Same as decision evaluation: `Authorization: Bearer <token>` when token auth is enabled (`authentication/styra-authentication.yml`).

## Steps
1. (Optional) Confirm the server is up with **health** — `GET /health` (add `?bundles=true` to require bundle activation).
2. Call **compileQueryWithPartialEvaluation** — `POST /v1/compile/{path}` with `{ options, unknowns, input }`. `unknowns` names the data that is not yet known (e.g. `data.tickets`).
3. Select the output target with the `Accept` header:
   - `application/vnd.styra.ucast.{all,minimal,linq,prisma}+json` → UCAST conditions for an ORM/query builder.
   - `application/vnd.styra.sql.{postgresql,mysql,sqlserver,sqlite}+json` → a SQL WHERE fragment for that dialect.
   - `application/vnd.styra.multitarget+json` → several targets at once.
   - `application/json` → raw residual query (`CompileResultJSON`).
4. Apply the returned condition to your query and run it.

## Rules
- Honor any `MaskingRule` entries in the compile result — mask those fields in the output.
- Pick the SQL dialect that matches your database exactly; there is no auto-detection.
- A `400 BadRequest` usually means the policy could not be partially evaluated for the given unknowns — inspect `errors[].location`.
- See `conventions/styra-conventions.yml` for the content-negotiation matrix.
