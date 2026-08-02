---
name: Triage Bigeye data quality issues
description: Find open data quality issues in a Bigeye workspace, read one in full, and update its status, priority or timeline as you work it.
api: openapi/bigeye-observability-openapi.json
operations:
  - IssueService_GetIssues
  - IssueService_GetIssue
  - IssueService_UpdateIssue
  - IssueService_BatchUpdateIssues
  - IssueService_GetTableIssues
---

# Triage Bigeye data quality issues

Use this when someone asks "what is broken in our data right now" or hands you a
Bigeye issue to work.

## Before you start

- Base URL is `https://app.bigeye.com` unless the customer is on a dedicated
  stack, in which case it is `https://<company-prefix>.bigeye.com`.
- Send `Authorization: apikey <API_KEY>`. The literal string `apikey ` is the
  prefix — it is **not** `Bearer`.
- Send `X-Bigeye-Workspace-Id: <workspaceId>` on every call. Without it,
  workspace-scoped operations return `400 {"code":400,"message":"A workspace ID
  must be supplied"}`.
- Your Bigeye role gates what you can do. A view-only key can read issues but
  will fail the update calls.

## Steps

1. **List the open issues.** `IssueService_GetIssues` — `POST /api/v1/issues/fetch`.
   The filter goes in the request body, not the query string. Filter by status
   to get the live queue; `pageCursor` in the body pages through large results.
2. **Narrow to a table when you have one.** `IssueService_GetTableIssues` —
   `POST /api/v1/issues/tables/fetch` returns an issues summary grouped by table,
   which is the cheaper call when the question is "is this table healthy".
3. **Read the issue in full.** `IssueService_GetIssue` —
   `GET /api/v1/issues/{issueId}`. Use the internal integer id from step 1, not
   the display name shown in the UI.
4. **Work it.** `IssueService_UpdateIssue` — `PUT /api/v1/issues/{issueId}` to
   set status or priority, or add a timeline message. Use
   `IssueService_BatchUpdateIssues` (`PUT /api/v1/issues/batch`) when you are
   closing a group of related issues in one pass.

## Rules

- **There is no idempotency contract.** No `Idempotency-Key` exists anywhere in
  the Bigeye API. Never blind-retry a `PUT` or `POST` after a timeout — re-read
  the issue with `IssueService_GetIssue` and check whether your change landed.
- **Errors are grpc-gateway shaped, not RFC 9457.** The body is
  `{code, message, error, details[]}` on `application/json`. The specs only
  declare `200` and a catch-all `default`, so branch on the HTTP status and the
  `code` field, not on a documented problem type. See
  `errors/bigeye-problem-types.yml`.
- Escalate to lineage rather than guessing at cause — see
  `bigeye-trace-lineage-impact.md`.
