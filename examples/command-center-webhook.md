# Command Center Webhook

Use this pattern when you want a single webhook entry point for your command center and a clear handoff target of **Claude**.

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

This `taskade-command-center` webhook is the inbound entry point for your Taskade-driven workflow.

## What the Webhook Should Do

1. Receive a command-center payload.
2. Pass the request to your Taskade MCP-enabled agent.
3. Let the agent update Taskade agents, tasks, projects, or knowledge as needed.
4. Relay the resulting summary and action payload to your **Claude** handoff endpoint.

## Suggested Payload

```json
{
  "command": "Update our support and market-intelligence agents for the current sprint",
  "workspace": "PopUpEmpire.wtf HQ",
  "folder": "04-Hack_wtf",
  "sendTo": "Claude",
  "context": {
    "priority": "high",
    "requestedBy": "command-center",
    "followUp": "Return a summary of all agent changes"
  }
}
```

## Agent Instructions

Use an agent prompt like this in your n8n AI Agent node:

> You are the command-center operator for Taskade. Interpret incoming requests, use Taskade MCP tools to update or create agents when needed, summarize what changed, and prepare a clean downstream handoff to the configured Claude endpoint whenever `sendTo` is set to `Claude`.

The bundled [`n8n-taskade-mcp-workflow.json`](./n8n-taskade-mcp-workflow.json) uses an OpenAI chat-model node as a sample. Swap that node for an Anthropic/Claude-compatible chat-model node in n8n if you want Claude to run the orchestration step too. To actually deliver the handoff payload to Claude, add your own downstream HTTP Request node or external proxy step after the Taskade agent completes.

## Claude Handoff

If you run Claude through your own proxy, automation layer, or webhook receiver, forward the post-agent payload to that downstream endpoint, for example:

```text
https://your-claude-proxy.example.com/webhook/claude-receiver
```

This `claude-receiver` URL is a separate downstream target from the inbound `taskade-command-center` webhook above.

Suggested relay body:

```json
{
  "sendTo": "Claude",
  "command": "Update our support and market-intelligence agents for the current sprint",
  "taskadeSummary": "Agent configs updated, knowledge attached, and publication status reviewed.",
  "nextAction": "Execute the implementation checklist and report completion."
}
```

## Important Limitation

This repository does **not** provision a live public webhook URL for you. The webhook URL is created by your automation host (for example, n8n), and the Claude destination URL should be a proxy or integration endpoint that you control.

## Best Fit in This Repo

- Use [`n8n-taskade-mcp-workflow.json`](./n8n-taskade-mcp-workflow.json) as the Taskade MCP agent layer.
- Put the webhook trigger in front of that workflow in your command center.
- Route all Claude-specific delivery through your own downstream HTTP/webhook endpoint.
