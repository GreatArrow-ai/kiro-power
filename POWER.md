---
name: "great-arrow"
displayName: "GreatArrow.ai"
description: "Connect to your GreatArrow.ai workspace — semantic memory, AI agents, integrations, calendar, email, and 560 MCP tools in one server."
keywords: ["great-arrow", "gad", "memory", "workspace", "rag", "agents"]
author: "Manito AI"
---

# GreatArrow.ai

## Overview

GreatArrow.ai (GAD) is an AI workspace platform that gives every connected AI assistant access to a shared semantic memory, 22 agents (1 orchestrator + 21 specialists), and deep integrations with Google, Microsoft, Slack, Notion, GitHub, and more — all through a single MCP server.

Once connected, your AI assistant can:

- **Search and create memories** across four semantic layers (working, episodic, semantic, procedural)
- **Invoke specialist agents** (code assistant, document analyst, security analyst, meeting coordinator, etc.)
- **Run the orchestrator** to decompose complex goals across multiple specialists
- **Read and write to integrations** — calendar events, emails, GitHub issues/PRs, Slack messages, Notion pages, cloud storage
- **Generate images**, fetch web content, manage todos, and more

All data is workspace-scoped with Row Level Security. Every tool call is authenticated, rate-limited, and audit-logged.

## Onboarding

### Prerequisites

- A GreatArrow.ai account at https://www.greatarrow.ai
- At least one workspace (created automatically on first sign-in)

### Step 1: Mint a Personal Access Token

1. Sign in at https://www.greatarrow.ai
2. Navigate to **Install** → https://www.greatarrow.ai/install
3. Choose any install method — the universal installer or the Kiro-specific card
4. A token prefixed with `gad_` will be minted for you
5. Copy the token — it's shown only once

Alternatively, visit https://www.greatarrow.ai/install/kiro for Kiro-specific instructions.

### Step 2: Add your token

The fastest path: on the [install page](https://www.greatarrow.ai/install), pick **Kiro (Power)** and download your personalized Power — the `gad_` token is already baked into its `mcp.json`. Unzip it, then in Kiro open **Powers → Add Custom Power → Local Directory** and point it at the unzipped folder.

If you installed this Power from a shared source instead, open its `mcp.json` and replace the placeholder:

```json
{
  "mcpServers": {
    "great-arrow": {
      "type": "http",
      "url": "https://www.greatarrow.ai/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_GAD_TOKEN",
        "X-MCP-Client": "kiro-power"
      }
    }
  }
}
```

Replace `YOUR_GAD_TOKEN` with the `gad_...` token you minted in Step 1. Leave `X-MCP-Client` set to `kiro-power` — that tells the server to send Kiro a focused starter set of tools that fits its request-size limit (see Dynamic Tool Discovery below). Use the canonical `www.` host: the bare domain 307-redirects and drops the `Authorization` header.

### Step 3: Verify

After saving, the MCP server should connect automatically. You can verify by asking Kiro to run `workspace_introspect` — it returns your active workspace, granted scopes, connected integrations, and available agents.

### Security Notes

- **Never commit your token** to version control. The power's mcp.json is local to your Kiro installation.
- Tokens can be **paused** (reversible) or **revoked** (permanent) at https://www.greatarrow.ai/account/connections
- The server enforces **rate limiting**, **circuit breakers**, and **scope-based access control** on every call.
- If you see a 401, your token may have been revoked or expired — mint a fresh one at `/install`.

## Common Workflows

### Workflow 1: Search Before Answering (Proactive RAG)

The most important pattern. Before answering any question about prior work, projects, decisions, or people — search memory first.

**Steps:**

1. Call `memory_search` with a natural-language query
2. If results are relevant, cite them by ID in your response
3. If no results, try one rephrase before falling back to general knowledge

**Example:**

```
memory_search({ query: "deployment checklist for production releases" })
```

### Workflow 2: Remember Important Decisions

After a decision, fact, or "remember this" moment — persist it.

**Steps:**

1. Call `memory_search` first with the same content to check for duplicates
2. If no duplicate exists, call `memory_create` with content, title, and tags
3. The classifier auto-picks the layer (WORKING/EPISODIC/SEMANTIC/PROCEDURAL) and importance

**Example:**

```
memory_create({
  content: "We decided to use Stripe metered billing for overage charges at $0.01/unit",
  title: "Billing: Stripe overage model decision",
  tags: ["billing", "stripe", "architecture-decision"]
})
```

### Workflow 3: Orient in a New Session

When starting a fresh conversation, call these two tools to understand the workspace state:

1. `workspace_introspect` — returns workspace info, scopes, integrations, agents, top tags
2. `mcp_capabilities` — structured manifest of all tools grouped by scope

### Workflow 4: Delegate Complex Tasks to Agents

For multi-step goals, use the orchestrator:

```
orchestrator_run({ query: "Research competitor pricing for AI workspace tools and summarize findings" })
```

For targeted tasks, invoke a specialist directly:

```
agent_invoke({ agentId: "code-assistant", task: "Review this function for security issues", context: "..." })
```

### Workflow 5: Work with Integrations

Check what's connected, then act:

```
integrations_list()                              // See all connections
calendar_upcoming_events({ days_ahead: 7 })      // Next week's meetings
email_send({ to: ["team@example.com"], subject: "Summary", text: "..." })
github_issue_create({ owner: "org", repo: "app", title: "Bug: ..." })
```

## Dynamic Tool Discovery

This Power connects as `kiro-power`, so a new session starts with a focused core set of 47 tools. That keeps context lean, improves tool-selection accuracy, and — importantly — keeps the `tools/list` response small enough for Kiro to accept (the full 560-tool surface exceeds Kiro's request-size limit and fails with "Improperly formed request"). Two meta-tools let you expand the surface on demand:

### `tools_discover` — Browse the Full Catalog

Query the complete tool catalog (560 tools) grouped by domain. Supports natural-language search and returns ML-powered recommendations based on your usage patterns.

**Parameters:**

| Param    | Type              | Description                                                         |
| -------- | ----------------- | ------------------------------------------------------------------- |
| `domain` | string (optional) | Filter by scope: `read`, `write`, `agents`, `integrations`, `admin` |
| `query`  | string (optional) | Natural-language description of what you want to do                 |

**Returns:** Tools grouped by scope, each annotated with `active` (whether it's in your current session) and a `recommendations` array of up to 5 suggested tools with scores and reasons.

**Example:**

```
tools_discover({ query: "send an email" })
// → finds email_send, shows it's not active, recommends enabling it
```

### `tools_enable` — Activate Tools On Demand

Add tools to your active session. After calling this, the newly enabled tools appear in your tool list immediately (via `notifications/tools/list_changed`).

**Parameters:**

| Param   | Type                  | Description                                                                 |
| ------- | --------------------- | --------------------------------------------------------------------------- |
| `tools` | string[] (1–50 items) | Tool names to activate. Use `["*"]` to enable all tools your scope permits. |

**Returns:** `{ enabled: [...], active_count: N, total_available: 560 }`

**Example:**

```
tools_enable({ tools: ["email_send", "github_issue_create"] })
// → both tools now appear in tools/list
```

### Dynamic Filtering Behavior

- **Session init**: Only the curated core set (47 tools) is exposed in `tools/list` for the `kiro-power` client
- **Expand on demand**: Use `tools_discover` to find what you need, then `tools_enable` to activate
- **Recommendations**: The `initialize` response includes a `recommended_tools` field with personalized suggestions based on your history with this client
- **Persistent within a session**: Tools you enable stay visible for the rest of the session; re-enable after a reconnect
- **Enable in small batches**: Add the handful of tools you need for the task. Avoid `tools_enable({ tools: ["*"] })` on Kiro — enabling all 560 puts every definition back into `tools/list` and re-trips Kiro's request-size limit. The wildcard is meant for clients without that cap.

### Recommended Workflow

1. Start a session — you get the core tools plus recommendations in the init response
2. If you need a capability you don't have, call `tools_discover({ query: "what I want to do" })`
3. Review the results and recommendations
4. Call `tools_enable({ tools: ["tool_name"] })` to activate what you need
5. The tool is now available for the rest of your session

## Tool Categories

The server exposes 560 tools across 5 scopes. The tool names below are a
representative sample per scope, not the full list — browse the rest with
`tools_discover`:

| Scope              | Tools                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Purpose                              |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| `mcp:read`         | memory_search, memory_think, memory_graph_query, memory_list, memory_get, memory_tags, memory_history, memory_search_history, agent_list, agent_thread_list, agent_thread_get, agent_thread_history, integrations_list, integration_sync_status, integration_sync_logs, calendar_upcoming_events, workspace_introspect, mcp_capabilities, chat_history, chat_session_get, document_get, document_search, cloudstorage_list_connections, cloudstorage_list_files, job_status, dlq_list, todo_list, slack_search_messages, ghl_contact_list | Search, introspect, fetch            |
| `mcp:write`        | memory_create, memory_update, memory_delete, memory_revert, calendar_create_event, calendar_update_event, calendar_delete_event, email_send, github_issue_create, github_issue_comment, github_pr_review, github_pr_merge, github_actions_dispatch, notion_create_page, confluence_create_page, sheets_update_range, slack_post_thread_reply, cloudstorage_upload, cloudstorage_download, task_create, image_generate, web_fetch, document_delete, todo_create, todo_update                                                               | Create and mutate                    |
| `mcp:agents`       | agent_invoke, orchestrator_run, agent_thread_rollback, todo_assign                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | Invoke specialists and orchestrators |
| `mcp:integrations` | integration_sync, integration_link, integration_unlink, integration_disconnect, integration_connect_url, cloudstorage_import                                                                                                                                                                                                                                                                                                                                                                                                              | Manage external connections          |
| `mcp:admin`        | workspace_create_application, workspace_attach_repo, job_run, dlq_retry, billing_me                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Workspace management and job control |

## Best Practices

- **Always search before creating** — call `memory_search` before `memory_create` to avoid duplicates
- **Cite memory hits** — when grounding an answer in memory, reference the hit IDs
- **Use specific tags** — short, specific tags on memories improve future retrieval
- **Check integrations before assuming** — call `integrations_list` before assuming what's connected
- **Don't echo tokens** — never repeat the `gad_` token in chat output
- **Use workspace_introspect on first turn** — orient yourself before diving into tasks

## Troubleshooting

### Error: 401 Unauthorized

**Cause:** Token is invalid, revoked, or expired.
**Solution:**

1. Check token status at https://www.greatarrow.ai/account/connections
2. If revoked, mint a new token at `/install`
3. Update the `Authorization` header in your mcp.json

### Error: 403 Forbidden

**Cause:** Token doesn't have the required scope for the tool you're calling.
**Solution:** The error response includes a `reauth_url` — share it with the user to upgrade permissions.

### Error: Connection refused / timeout

**Cause:** The server may be temporarily unavailable.
**Solution:**

1. Check https://www.greatarrow.ai/api/health
2. If health returns 503, a critical subsystem is down — wait and retry
3. If health returns 200 with "degraded", optional services are down but core works

### Error: Rate limited (429)

**Cause:** Too many requests in a short window.
**Solution:** Back off and retry after the `Retry-After` header value.

### Error: "Improperly formed request" / no tools appear

**Cause:** Kiro rejected an oversized `tools/list` response. This happens when the server sends the full 560-tool surface instead of the curated set.
**Solution:**

1. Confirm `X-MCP-Client` is set to `kiro-power` (not `kiro`) in your `mcp.json` headers — `kiro` receives the full surface (it's meant for the stdio proxy).
2. Confirm the `url` is the canonical `https://www.greatarrow.ai/api/mcp` (the bare domain redirects and can drop headers).
3. If you previously ran `tools_enable({ tools: ["*"] })`, start a fresh session — the wildcard re-inflates the tool list past Kiro's limit. Enable specific tools instead.

## MCP Config Placeholders

**IMPORTANT:** Before using this power, replace the following placeholder in `mcp.json`:

- **`YOUR_GAD_TOKEN`**: Your personal access token for GreatArrow.ai.
  - **How to get it:**
    1. Sign in at https://www.greatarrow.ai
    2. Go to https://www.greatarrow.ai/install
    3. Choose **Kiro (Power)** — download the personalized Power (token pre-filled), or copy the `gad_...` token shown on screen (shown only once)
    4. If you copied the token, paste it in place of `YOUR_GAD_TOKEN` in mcp.json

**After replacing the placeholder, your mcp.json should look like:**

```json
{
  "mcpServers": {
    "great-arrow": {
      "type": "http",
      "url": "https://www.greatarrow.ai/api/mcp",
      "headers": {
        "Authorization": "Bearer gad_abc123your_actual_token_here",
        "X-MCP-Client": "kiro-power"
      }
    }
  }
}
```

---

**MCP Server:** great-arrow
**Transport:** HTTP (Streamable)
**Client id:** `kiro-power` (curated starter tool set + on-demand expansion)
**Endpoint:** https://www.greatarrow.ai/api/mcp
