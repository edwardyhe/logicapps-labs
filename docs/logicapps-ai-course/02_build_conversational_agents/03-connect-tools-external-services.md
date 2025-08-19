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
> Conversational agent workflows must always start with the default **A2A Trigger**, and not any other trigger. 
> Conversational agents start running when a chat session starts from the integrated chat client in Azure Logic Apps.
> No separate workflow trigger exists in the agent because the agent directly calls tool actions during the conversation.

The following table helps map the relationship between connector operations and the tools that conversational agents use:

| Component | Description |
|-----------|-------------|
| Connectors | - Provide operations that you use to create reusable integrations where Azure Logic Apps can call external services, such as Microsoft 365, GitHub, Azure services, SaaS apps, and more. <br><br>- Expose connector operations as triggers, which are events that start the workflow, and actions, which are tasks that the agent can call. <br><br>- Create a connection that specifies authentication and environment-specific settings. <br><br>- Managed connectors versus built-in operations: Managed connector operations run in global, multitenant Azure and usually require connections. Built-in operations, such as Request, Recurrence, HTTP, Compose, Until, and so on, directly run with the Azure Logic Apps runtime. |
| Tools | - Provides a single action or an action sequence that an agent can call to satisfy a user's request, such as "Get weather" or "Look up a ticket". <br><br>- For agents in Azure Logic Apps, a tool can use a connector action, a built-in action, or even another workflow. <br><br>- Provide a tool name and description so that the agent can better determine when to call the tool. Good descriptions drive better tool selection. |

## Prerequisites

- An Azure account and subscription. If you don't have a subscription, [sign up for a free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- A conversational logic app agent set up in the previous module(s) with model/provider configured.

- A Standard logic app resource and conversational agent workflow with the model set up from previous modules.

  For the examples, this workflow include

  - Primary example: RSS connector, which doesn't need authentication.
  - Optional alternative: MSN Weather connector, which doesn't need authentication.
  - Optional example with authentication: A GitHub account with either OAuth 2.0 or a personal access token (PAT) with minimal read scopes.

> [!NOTE]
>
> This module focuses on connector-backed tools. Later modules cover agent parameters and on-behalf-of (OB) authorization patterns.

## Connector concepts for this module

| Concept | Description |
|---------|-------------|
| Triggers versus actions | Conversational agent workflows must use the default **A2A Trigger**. Autonomous agent workflows can use any available trigger. Agents in both workflows use actions as tools. |
| Connections and authentication | OAuth 2.0, API key, managed identity, or none, based on the connector. |
| Inputs and outputs | Actions define parameters and return JSON. The agent summarizes outputs for the end user. |
| Limits and reliability | Expect throttling errors (HTTP 429), timeouts, retries, and pagination. Design your prompts and tools to gracefully handle errors and exceptions. |
| Connection references | Your workflow binds to a specific connection at runtime. |

## Part 1 - Expose a basic read-only RSS connector as a tool (primary example)

To more easily demonstrate the conversational agent pattern from end to end, this example doesn't use authentication or agent parameters.

### Step 1 - Add a tool for your agent

1. In the [Azure portal](https://portal.azure.com), open your Standard logic app resource.

1. Open the conversational agent workflow in the designer.

1. On the designer, inside the agent and under **Add tool**, select the plus sign (+) to open the pane where you can browse available actions.

1. On the **Add an action** pane, follow these [general steps](https://learn.microsoft.com/azure/logic-apps/create-workflow-with-trigger-or-action?tabs=standard#add-action) to add the **RSS** action named **List items in feed** to the empty tool.

1. Provide a clear and brief name and desription for the tool, for example:

   - Name: **Get latest RSS**
   - Description: **Retrieves latest posts from a specific RSS feed and returns the titles and links for a summary.**

     The large language model (LLM) uses the tool description to help the agent decide when to call the tool. Keep the description short, specific, and outcome-focused.

### Step 2 - Set up the connector action

1. In the **RSS** action that you added, provide the following information:

   | Parameters | Description |
   |------------|-------------|
   | **Feed URL** | A fixed URL without any parameters, for example: **https://techcommunity.microsoft.com/gxcuf89792/rss/board?board.id=AzureLogicApps** |
   | **Number of items** | Specify 5-10 times to keep the agent responses concise. |

1. Save your workflow.

### Step 3 — Map outputs for the agent

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

