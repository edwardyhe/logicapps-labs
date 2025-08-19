---
title: Connect tools to external services - Module 03
description: Learn how to connect a conversational agent workflow in Azure Logic Apps to external services by using connectors and how to expose connector actions as tools for agents.
ms.service: azure-logic-apps
author: edwardyhe
ms.author: edwardyhe
ms.topic: tutorial
ms.date: 08/13/2025
#Customer intent: As an integration solution sevelo
---

# Connect your tools to external services (Module 03)

In this module, you learn how to connect a conversational agent workflow in Azure Logic Apps to external systems by using connectors and how to expose connector actions as tools that the agent can call during a conversation.

When you finish this module, you'll achieve the goals and complete the tasks in the following list:

- Understand key connector concepts such as actions, connections, authentication, and limits.
- Add one read-only connector action as a tool that your agent can autonomously and directly call without needing authentication.
- ptionally add an authenticated connector action as a tool that handles inputs and errors.
- Apply best practices for safe, reliable tool use in conversational scenarios.

This module keeps the primary flow simple without using [agent parameters](https://learn.microsoft.com/azure/logic-apps/agent-workflows-concepts#key-concepts) or [on-behalf-of (OBO) authorization](https://learn.microsoft.com/entra/identity-platform/v2-oauth2-on-behalf-of-flow). In Module 4, you learn how to parameterize inputs. In Module 5, you learn how to add OBO authorization.

For more information, see [Connectors overview for Azure Logic Apps connectors](https://learn.microsoft.com/azure/connectors/introduction)

## Connectors and their relationship to tools

In Azure Logic Apps, *connectors* offer prebuilt operations that provide a fast, secure, and reliable way for an agent to take an action in the real world without needing you to write integration code. The following table describes other benefits that connectors offer when you build integration workflows:

| Benefit | Description |
|---------|-------------|
| Breadth of integrations | Choose operations offered by more than 1,400 connectors across Microsoft 365, Azure services, SaaS apps, developer tools, and data platforms. |
| Faster time to value | Use declarative actions so you don't have to build and maintain custom API clients. Reduce boilerplate, errors, and maintenance overhead. |
| Secure connections | Standardize authentication with connections that can use Open Authorization (OAuth 2.0), API keys, or managed identity. Store secrets securely and reuse connections across tools. |
| Reliability | Take advantage of built-in retries, pagination, and robust error surfaces exposed in run history and monitoring. |
| Consistent patterns | Simplify how an agent calls operations and how you compose workflows around them by using a unified model. |
| Observability | Trace tool calls, diagnose failures, and tune prompts by using workflow run history, monitoring, and telemetry. |
| Enterprise readiness | Combine connectors with network isolation and other platform controls to meet organizational requirements. |

In conversational agent scenarios, connectors turn operations that work with external services and systems into tools that agents can use. When you provide a clear tool name and description, the agent can better decide when to call connector operations as tools and how to summarize results back to the user. To benefit from built-in authentication, throttling guidance, schemas, and monitoring, use connectors when available, rather than raw HTTP.

> [!NOTE]
>
> Conversational agent workflows don't support connector triggers. Conversational agents start when a 
> chat session starts, from the chat client integrated with Azure Logic Apps. No separate workflow
> trigger exists in the agent because the agent directly calls tool actions during the conversation.

The following table helps map the relationship between connector operations and the tools that conversational agents use:

| Component | Description |
| --- | --- |
| Connectors | - Provide operations that you use to create reusable integrations where Azure Logic Apps can call external services, such as Microsoft 365, GitHub, Azure services, SaaS apps, and more. <br><br>- Expose connector operations as triggers (event-based starts) and actions (steps you can call). <br><br>- Use a connection that holds authentication and environment-specific settings. <br><br>- Managed connectors vs. built-in actions: managed connectors often require connections; built-in actions (HTTP, Compose, Variables) do not. |
| Tools | - A single action or action sequence that agent can invoke to satisfy a user's request (for example, "get weather," "look up a ticket").
	- In Azure Logic Apps agents, a tool can be backed by a connector action, a built-in action, or another workflow.
	- You provide the tool’s name and description so the agent knows when to call it. Good descriptions drive better tool selection.

## Prerequisites

- An Azure subscription with permissions to create logic apps and connections.
- A conversational logic app agent set up in the previous module(s) with model/provider configured.
- For the exercises:
	- Primary example: RSS connector (no authentication required).
	- Optional alternative: MSN Weather connector (no authentication required).
	- Optional authenticated example: a GitHub account and either OAuth 2.0 or a personal access token (PAT) with minimal read scopes.

> [!NOTE]
> This module focuses on connector-backed tools. Agent parameters and OBO patterns come later.

---

## Connector concepts you will use

- Actions vs. triggers: in conversational agents, connector triggers are not supported. Use actions as tools. Connector triggers are supported for autonomous agents.
- Connections and authentication: OAuth 2.0, API key, managed identity, or none depending on the connector.
- Inputs and outputs: actions define parameters and return JSON; your agent summarizes outputs for the user.
- Limits and reliability: expect throttling (HTTP 429), timeouts, retries, and pagination; design prompts and tools to handle gracefully.
- Connection references: your workflow binds to a specific connection at runtime.

---

## Part 1 (primary): Expose a simple read-only connector as a tool (RSS)

We will start with a no-auth example to see the pattern end to end without parameters.

### Step 1 — Open your agent
- Open your logic app (Standard) and navigate to your conversational agent workflow (Agent Loop).

### Step 2 — Add a tool from a connector action
- Add a new Tool.
- Choose to create from a Connector action and search for "RSS".
- Select "RSS — List items in feed". This adds the action to the tool.
- Name and describe the tool clearly, for example:
	- Tool name: GetLatestRss
	- Description: "Retrieves the latest posts from a specific RSS feed and returns titles and links for summarization."

> [!TIP]
> The LLM uses the tool description to decide when to call the tool. Keep it short, specific, and outcome-focused.

### Step 3 — Configure the connector action
- Feed URL: set a fixed URL (no parameters yet), for example:
	- https://techcommunity.microsoft.com/gxcuf89792/rss/board?board.id=AzureLogicApps
- Number of items: 5–10 (keeps responses concise).
- Save.

### Step 4 — Map outputs for the agent
- Keep outputs compact. The RSS action returns items with title, primaryLink, publishDate, etc.
- If your designer supports selecting outputs, include only fields you need (title, link).
- Save the workflow.

### Step 5 — Test in chat
- Ask: "What is new from the Azure Logic Apps blog?"
- Verify the tool is invoked and the agent summarizes the items with links.
- Use Monitoring/Run history to confirm tool calls and action success.

> [!NOTE]
> Inputs are hardcoded to avoid parameters in this module. You will parameterize them in Module 04. You can substitute any no-auth connector using the same pattern.

---

## Optional alternative: Read-only connector tool (MSN Weather)

Prefer weather instead of RSS? Use the MSN Weather connector.

### Steps
- Add a Tool -> Connector action -> search "MSN Weather".
- Choose "Get current weather".
- Connection: no authentication required; create with defaults if prompted.
- Configure with fixed inputs (no parameters yet), for example, Location = "Seattle, US", Units = "imperial".
- Tool name: GetCurrentWeather
- Description: "Gets current weather for a fixed location and returns temperature and conditions for summarization."
- Test by asking: "What is the weather right now?"

> [!TIP]
> In Module 04, convert Location and Units into agent parameters so the agent can pass values dynamically.

---

## Part 2 (optional): Expose an authenticated connector as a tool (GitHub)

Add a tool that requires authentication. Keep inputs static for now; parameterize in Module 04.

### Step 1 — Add the GitHub connector action
- Add a Tool -> From Connector -> search "GitHub".
- Pick a read operation such as "Get issue" (owner, repo, issue number) or "Get repository".

### Step 2 — Create the GitHub connection
- When prompted, create a new GitHub connection.
- Authenticate with OAuth 2.0 using least-privilege scopes, or with a PAT with minimal read access.
- Prefer secure secret storage (for example, Key Vault-backed references) where available.

### Step 3 — Configure the tool (static inputs for this module)
- Tool name: GetGitHubIssue
- Description: "Looks up a GitHub issue by owner/repo and number; returns title, state, labels, and URL."
- Map static example inputs directly on the action (no agent parameters yet), for example, owner="octocat", repo="Hello-World", number="1".
- Save.

### Step 4 — Test in chat
- Ask: "Show me the details of the demo GitHub issue."
- Verify the tool is called and the result summarized.

> [!TIP]
> For private repositories, ensure the connection identity has access. In Module 04, replace static values with agent parameters so the user can specify owner/repo/number.

---

## Best practices for tools and connectors in agents

- Favor read-first: start with read-only tools; add writes later with confirmations.
- Clear descriptions: be specific about when to use (and not use) a tool.
- Provide examples and hints: improve parameter extraction with formats (for example, owner/repo) and sample utterances.
- Validate inputs: constrain enums and set defaults. Let the agent ask clarifying questions when required.
- Handle errors gracefully: plan for 401/403, 404, 429, timeouts. Keep responses concise and helpful.
- Respect limits: avoid fetching huge pages; summarize instead of retrieving everything.
- Least privilege: use managed identity or scoped tokens; store secrets securely.
- Require confirmation for writes: for create/update/delete, require explicit user confirmation.
- Observability: use run history/telemetry to verify tool calls and tune prompts.

---

## Troubleshooting

- Connection shows "Not authorized"
	- Re-authenticate with required scopes and verify target resource permissions.
- The agent does not call the tool
	- Strengthen the tool description and reduce overlap with other tools; verify input mappings.
- Wrong or missing parameters
	- Provide stronger hints and examples; make key inputs required in Module 04.
- 429/Throttling
	- Limit frequency, cache, or ask the user to narrow the query.
- Not found (404)
	- Verify owner/repo/number or location format; suggest alternatives.

---

## Adapting this to other connectors

- ServiceNow: list groups or create incidents (see Agent-in-a-Day samples in this repo for screenshots and patterns).
- Microsoft 365: get profile or list recent mail (requires OAuth and tenant permissions).
- Any REST (Representational State Transfer) API: use HTTP built-in, or managed connector if available.

Pattern recap: add the action, create/select the connection, set safe defaults, map compact outputs, test in chat, and observe runs. In Module 04 you will parameterize inputs; in Module 05 you will add user context (OBO) so the agent can act as the signed-in user where supported.

---

## Cleanup

- Remove demo tools if not needed.
- If you created new connections (for example, GitHub), remove or rotate credentials per your organization's policy.

---

## Next steps

- Module 04 — Add parameters to your tools (dynamic inputs from the agent).
- Module 05 — Add user context to your tools (OBO patterns).
=======
# Module 03 - Connect your tools to external services 

Introduce connectors as a way to enhance tool capabilities.

Brief description about connectors in Azure Logic Apps with links to existing docs.
Prerequisites (define which service to use)
Create an action
Create a connection
Configure connector actions
No agent parameters and no OBO yet - that is later

