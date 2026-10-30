---
navigation_title: Elastic MCP service
description: Learn what the hosted Elastic MCP service does, who it's for, and how it differs from the Agent Builder MCP server.
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

# Elastic MCP service [elastic-mcp-service]

The Elastic MCP service is a hosted [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) server at `https://mcp.elastic.co`.
Connect the AI client you already use, such as Claude, Cursor, VS Code, or ChatGPT, to your {{serverless-short}} projects.
Each call runs in the project, with your permissions.

## Use cases [use-cases]

Use the Elastic MCP service when you want Elastic from the client where you already work.

<!-- TBD: Is it intended for autonomous use by agents? Or in human-AI interactions? Neither or both? -->

- **Work across the projects you can access.** You connect once, then use the serverless projects available to you, instead of configuring a separate endpoint for each project.
<!-- TBD: Which tool requires you to "configure a separate endpoint for each project? It's unclear what this is compared to -->
- **Let an agent act as you.** After you authorize the connection, the agent can take the actions you can take in those projects.
- **Approve what an AI client can reach.** If you administer access for your organization, you review a connection that acts as a person, in that person's projects.
<!-- TBD: This is worded very unclearly and seems like a variation of the previous bullet. This page should be very high-level and not duplicate details. -->

This service does not apply in these cases:
<!-- TBD: Some places it's referred to as a server, others as a "service". Which is right? -->

- You need {{agent-builder}} tools, including tools you created. Use the [{{agent-builder}} MCP server](agent-builder/mcp-server.md). That server also works on {{stack}} deployments.
<!-- TBD: What's an example of a tool so I know why I'd be interested in following this link? -->
- The project uses [private connectivity](/deploy-manage/security/private-connectivity.md). This service does not support those projects.
<!-- TBD: Are there reasons why you'd use other options like Elastic CLI or Elastic Claude Plugin instead? -->

## How it works [how-it-works]

You connect your MCP client to `https://mcp.elastic.co` and authorize it with OAuth 2.1.
The service accepts the connection, finds the {{serverless-short}} projects you can access, and routes each call to the project.
It does not copy your data out of the project.

The client does not receive a separate tool for every Elastic API.
<!-- TBD: Why do users care about lack of separate tools? -->
It looks up the operation it needs, reads that operation's schema, and runs the call.
Read, write, and destructive calls are handled separately.
<!-- TBD: Why does the separate handling matter to users? -->
When the client can ask you to confirm an action, a destructive call waits for that confirmation.
<!-- TBD: This is unclear. What affects "when the client can ask"? Does it differ in each environment? -->

:::{important}
At launch, the connection can do what you can do. You can't yet limit it to selected projects or to read-only actions.
:::

Most MCP clients connect without a client that you register yourself.
Some clients require your organization to register the client.
That registration is not the [{{agent-builder}} OAuth client](/deploy-manage/app-connections/create-oauth-client.md).
<!-- TBD: I don't understand what the point of this paragraph is. Is it trying to tell me about the OAuth requirements? Seems useless as-is -->

## How it relates to other Elastic features [how-it-relates]

These are separate features.
<!-- For a fuller comparison, refer to [AI-powered features](/explore-analyze/ai-features.md#elastic-mcp-service). 

TBD: I think it's better to do that here after all, instead of in the parent page.
-->

- [{{agent-builder}} MCP server](agent-builder/mcp-server.md): {{agent-builder}} tools for one project or deployment, on {{serverless-short}} and on {{stack}}. The [Elastic Cloud Serverless connector for Claude](claude-connector.md) uses this server.
<!-- TBD: Per above, if we assume that new users have no context on what Agent Builder tools are, what's a brief summary of the value? -->
- [{{agent-builder}}](elastic-agent-builder.md): Chat in {{kib}} and build agents, tools, and skills in the product.
- [Docs MCP server](/get-started/machine-readable-docs.md): Elastic documentation only. It does not connect to your data.
- [AI agent skills](agent-skills.md): Elastic procedures for a coding agent. Skills don't connect the agent to a project. You can use them with this service, but it doesn't require them.
- [{{context-engine}}](context-engine.md): Reusable context for agents. It is a separate product from this connection.
<!-- TBD: I think context engine is still hidden from the navigation tree and likely has applies-to tags. Verify -->
- [Elastic CLI](https://www.elastic.co/docs/reference/elastic-cli/cli): Command-line access to Elastic.
<!-- TBD: This sounds like there's no overlap with the MCP Server. Verify and omit from this list if so -->

## Next steps [next-steps]

- Connect a client at [`https://mcp.elastic.co`](https://mcp.elastic.co).
- Compare this service with the {{agent-builder}} MCP server on the [AI-powered features](/explore-analyze/ai-features.md#elastic-mcp-service) page.
<!-- TBD: I don't think either of these agent builder links are reasonable next steps. Rather, I think we should assume they want to start setting up projects, ingesting data, or query it -->
- If you need {{agent-builder}} tools, start with the [{{agent-builder}} MCP server](agent-builder/mcp-server.md).

## Related pages [related-pages]

<!-- TBD: These pages are all related to Agent builder not the MCP Server so I don't see the value of any of these links. Maybe it's more useful to link to overview pages about serverless projects, index basics, or other introductory content -->

- [Connect an MCP host to {{agent-builder}}](/deploy-manage/app-connections/connect-mcp-host.md)
- [Create or edit an OAuth client in {{agent-builder}}](/deploy-manage/app-connections/create-oauth-client.md)
- [{{agent-builder}} permissions](agent-builder/permissions.md)
