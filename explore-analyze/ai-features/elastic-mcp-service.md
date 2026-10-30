---
navigation_title: "{{mcp-service}}"
description: Connect Claude, Cursor, VS Code, ChatGPT, or another MCP host to Elastic serverless projects at mcp.elastic.co.
type: overview
# Lifecycle is unconfirmed. preview is a placeholder until product names preview, beta, or ga.
applies_to:
  stack: unavailable
  serverless: preview
products:
  - id: cloud-serverless
  - id: elasticsearch
  - id: kibana
  - id: observability
  - id: security
---

# {{mcp-service}} [elastic-mcp-service]

If you work in Claude, Cursor, Visual Studio Code, ChatGPT, Codex, or Gemini, you can use these MCP hosts to connect to the {{mcp-service}} at `https://mcp.elastic.co`.

After you authorize it, the agent discovers the {{serverless-full}} projects you can access and calls Elastic with your permissions.
<!-- TBD: What is "the agent" in this paragraph? -->

## Use cases [use-cases]

- **Start in the client you already use.** Connect the host, sign in, and ask it to create a {{serverless-short}} project and work with the data you add to it.
<!-- TO-DO: This bullet uses both "client" and "host" terminology seeming for the same thing, which confuses me -->
- **Use every project you can access from one connection.** You don't add a new endpoint each time you create or switch projects.
- **Let an agent do that work as you.** After you authorize the connection, an agent can take the same actions you can take in those projects.

Choose a different option when:

- You already have a project or deployment and you want {{agent-builder}} tools, including tools you created. For example, you can run an {{esql}} query from the client without opening {{kib}}. Use the [{{agent-builder}} MCP server](agent-builder/mcp-server.md). That server also works on {{stack}} deployments.
- The project uses [private connectivity](/deploy-manage/security/private-connectivity.md). The  {{mcp-service}} doesn't support those projects.
- You use {{ech}} or a self-managed cluster. This service is for {{serverless-short}} projects only. Use the [Elastic CLI](cli://index.md) or the {{agent-builder}} MCP server when those apply.

## How it works [how-it-works]

You connect the MCP host to `https://mcp.elastic.co` and authorize it with OAuth 2.1. The service finds the {{serverless-short}} projects you can access and routes each call to the project. Your data stays in the project. The connection acts as you.

The host doesn't load a tool for every Elastic API. It looks up the operation it needs, reads that operation's schema, and runs the call. That keeps the conversation from filling with the full API surface, and Elastic can change an API without breaking the client.
<!-- TO-DO: This is another paragraph where it's unclear what the "client" and "host" overlap is -->

Read, write, and destructive calls are separate so a destructive call can be confirmed. Hosts that support confirmation ask you before that call runs.
<!-- TBD: Unnecessary disclaimer?
Not every host shows that prompt.
-->

You can create and update projects through this service. You can't delete a project through it.

:::{important}
The connection can do what you can do. You can't limit it to selected projects or to read-only actions.
:::

## How it relates to other Elastic features [how-it-relates]

- [{{agent-builder}} MCP server](agent-builder/mcp-server.md): Use this when you already have a project or deployment and you want {{agent-builder}} tools, including custom tools. The {{agent-builder}} MCP server is one endpoint per project. The [Elastic Cloud Serverless connector for Claude](claude-connector.md) uses this server.
   <!-- TBD: It isn't the starting point if you don't have a project yet? -->
- [{{agent-builder}}](elastic-agent-builder.md): Use this when you want to chat in {{kib}} and build agents, tools, and skills in the product.
- [Docs MCP server](/get-started/machine-readable-docs.md): Use this when you want Elastic documentation only. It doesn't need an Elastic account and it doesn't connect to your data.
- [AI agent skills](agent-skills.md): Use these when you want Elastic procedures in a coding agent.
  <!-- TBD: How is a "coding agent" related to the "agent", "client" and "host" in earlier sections  -->
  Skills don't connect the agent to a project. You can install them with this service but hey aren't required.
- [Elastic CLI](cli://index.md): Use this when you want a terminal instead of an MCP host, including for self-managed clusters this service doesn't cover.

## Next steps [next-steps]

- Connect a client at [`https://mcp.elastic.co`](https://mcp.elastic.co).
- If you don't have a project yet, [create an {{serverless-full}} project](/deploy-manage/deploy/elastic-cloud/create-serverless-project.md). You can also ask the connected client to create one.

## Related pages [related-pages]

- [Create an {{serverless-full}} project](/deploy-manage/deploy/elastic-cloud/create-serverless-project.md)
- [Ingest for search use cases](/solutions/search/ingest-for-search.md)
- [Elastic fundamentals](/get-started/index.md)
