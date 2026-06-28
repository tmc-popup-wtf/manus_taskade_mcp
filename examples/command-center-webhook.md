# Command Center Webhook

Use this pattern when you want a single webhook entry point for your command center and a clear handoff target of **Manus**.

## Recommended Webhook URL

Create an n8n Webhook trigger with the path:

```text
/webhook/taskade-command-center
```

Your final URL will be:

```text
https://YOUR_N8N_HOST/webhook/taskade-command-center
```

If you are testing in n8n first, use the temporary test URL that n8n generates in the editor.

## What the Webhook Should Do

1. Receive a command-center payload.
2. Pass the request to your Taskade MCP-enabled agent.
3. Let the agent update Taskade agents, tasks, projects, or knowledge as needed.
4. Relay the resulting summary and action payload to **Manus**.

## Suggested Payload

```json
{
  "command": "Update our support and market-intel agents for the current sprint",
  "workspace": "PopUpEmpire.wtf HQ",
  "folder": "04-Hack_wtf",
  "sendTo": "Manus",
  "context": {
    "priority": "high",
    "requestedBy": "command-center",
    "followUp": "Return a summary of all agent changes"
  }
}
```

## Agent Instructions

Use an agent prompt like this in your n8n AI Agent node:

> You are the command-center operator for Taskade. Interpret incoming requests, use Taskade MCP tools to update or create agents when needed, summarize what changed, and prepare a clean downstream handoff whenever `sendTo` is set to `Manus`.

## Manus Handoff

If Manus has its own webhook receiver, forward the post-agent payload to a second endpoint such as:

```text
https://YOUR_MANUS_HOST/webhook/taskade-command-center
```

Suggested relay body:

```json
{
  "sendTo": "Manus",
  "command": "Update our support and market-intel agents for the current sprint",
  "taskadeSummary": "Agent configs updated, knowledge attached, and publication status reviewed.",
  "nextAction": "Execute the implementation checklist and report completion."
}
```

## Important Limitation

This repository does **not** provision a live public webhook URL for you. The webhook URL is created by your automation host (for example, n8n), and the Manus destination URL must be one you control.

## Best Fit in This Repo

- Use [`n8n-taskade-mcp-workflow.json`](./n8n-taskade-mcp-workflow.json) as the Taskade MCP agent layer.
- Put the webhook trigger in front of that workflow in your command center.
- Route all Manus-specific delivery through your own downstream HTTP/webhook endpoint.
