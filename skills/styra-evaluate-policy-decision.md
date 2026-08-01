---
name: Evaluate an authorization policy decision with OPA
description: Ask a self-hosted OPA / Enterprise OPA instance for an allow/deny (or richer) decision by POSTing an input document to a named policy path.
api: openapi/styra-enterprise-opa-openapi.yaml
operations: [executePolicyWithInput, executeDefaultPolicyWithInput, executeBatchPolicyWithInput]
---

# Evaluate a policy decision

Open Policy Agent (OPA) / Enterprise OPA evaluates Rego policies and returns a decision. The API is self-hosted (default `http://localhost:8181`).

## Auth
- OPA started with `--authentication=token` requires `Authorization: Bearer <token>` on every request. Otherwise the API is open on the bound interface. See `authentication/styra-authentication.yml`.

## Steps
1. Decide the decision path. A policy `package authz` with rule `allow` is queried at `/v1/data/authz/allow`.
2. Call **executePolicyWithInput** — `POST /v1/data/{path}` with body `{"input": { ... }}`. The `input` is your request context (user, action, resource). The decision comes back as `{"result": <value>, "decision_id": "..."}`.
3. To query the server's configured default decision instead, call **executeDefaultPolicyWithInput** — `POST /` with the raw input document as the body.
4. To evaluate many inputs in one round trip, call **executeBatchPolicyWithInput** — `POST /v1/batch/data/{path}` with a map of id → input; read per-entry results (mixed success/failure is a 207-style `BatchMixedResults`).

## Rules
- Capture `result.decision_id` and log it — it correlates to the decision-log event (`conventions/styra-conventions.yml`).
- A `404 PolicyNotFound` means the path is undefined (policy/bundle not loaded), not a deny. Treat undefined vs. `false` distinctly.
- Errors use `{code, message, errors[]}` (not RFC 9457) — see `errors/styra-problem-types.yml`.
- Requests/responses may be gzip-compressed.
