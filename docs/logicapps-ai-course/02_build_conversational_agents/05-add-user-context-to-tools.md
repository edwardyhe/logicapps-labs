---
title: Add user context to tools - Module 05
description: Learn how to set up on-behalf-of (OBO) authorization for your tools. Learn how to run connector actions with a signed-in user identity by using OBO authorization in conversational agent workflows for Azure Logic Apps.
ms.service: azure-logic-apps
author: edwardyhe
ms.author: edwardyhe
ms.topic: tutorial
ms.date: 08/27/2025
# Customer intent for this module:
# - Describe when to use and how to set up OBO. // Only if OBO works from portal chat client. Otherwise defer until after deployment section.
# - Provide some client code if useful, but link to later A2A section so customers have context.
---

# Add user context to tools by running connector actions with a signed-in user identity (Module 05)

In this module, you learn how to set up an agent that acts *on behalf of* (OBO) the signed-in user, meaning that the agent can run connector actions by using that user's identity. This module describes scenarios for where to add OBO authorization, known also as *user context*, how to configure the appropriate connections, test with different users, and learn the limitations.

When you finish this module, you'll achieve the goals and complete the tasks in the following list:

- Understand user-delegated versus app-only identities for tools.
- Configure a connector-backed tool to run with user context.
- Test behavior changes across users with different permissions.
- Apply best practices for consent, scopes, and error handling.

For more information, see [Microsoft identity platform and OAuth 2.0 On-Behalf-Of flow](https://learn.microsoft.com/entra/identity-platform/v2-oauth2-on-behalf-of-flow).

## Identities for tools

The following table describes different identities that a tool can use to run actions:

| Identity | Description |
|----------|-------------|
| Per-user (OBO) | A connector action runs by using a delegated token for the signed-in user. The result depends on the user's permissions and licenses. |
| App-only (application identity) | A connector action runs by using a managed identity or an app or service principal. The result depends on app permissions and configuration. <br><br>**Important**: Use an app-only identity when the tool performs shared operations that aren't tied to a user like posting to a shared channel, running a back office job, or using a service account. |
| Connection reference | Your workflow binds each connector action to a specific connection that determines how to perform authentication. |

## Scenarios for on-behalf-of (OBO) authorization

When you need a tool to respect the signed-in user's permissions, licenses, or personal data boundaries, set up OBO authorization when available. The following list provides a few common example scenarios:

- Microsoft 365: Read a person's mail, calendar, files, or profile.
- ServiceDesk or ITSM: Attribute actions to the requester.
- Enterprise APIs with per-user authorization or auditing requirements.

Many solutions use mixed authorization methods. For example, a solution might read user data with OBO for personalization, and then perform a write operation by using an app-only identity after explicit confirmation.

## Limitations

In conversational agent workflows, support for OBO authorization applies only to *shared* managed connectors that use delegated, per-user connections to work with Microsoft services or systems.

## Prerequisites

- An Azure account and subscription. If you don't have a subscription, [sign up for a free Azure account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- A Standard logic app resource and conversational agent workflow from previous modules.

  - To use OBO authorization, the logic app resource requires that you set up Easy Auth, previously known as App Service Authentication, on your Standard logic app resource.

    For more information, see the following articles:

    - [Authentication and authorization in Azure App Service and Azure Functions](https://learn.microsoft.com/azure/app-service/overview-authentication-authorization)
    - [Configure your App Service or Azure Functions app to use Microsoft Entra sign-in](https://learn.microsoft.com/azure/app-service/configure-authentication-provider-aad)

  - The conversational agent workflow requires a connector operation that works with a Microsoft service or system. The connection must also support delegated, per-user connections, for example, Microsoft 365, SharePoint, or Microsoft Graph. If required, the connection might also need tenant administrator consent.

    > [!TIP]
    > 
    > Make sure to keep tool descriptions concise and include guidance about the data that they access,
    > for example, such a description might say "Gets the signed-in user's next five calendar events."

- The chat client interface integrated with conversational agent workflows.

  This chat client handles token delegation for per-user connections. If you want to use a custom client, the client must pass the user token to the agent.

  > [!NOTE]
  >
  > If you are building a custom client that can't pass a user token yet, complete the concepts with the app-only connections.
  > In a later module, you integrate a custom client that supplies the user's access token to the agent.

- At least two test users for conversing through chat with different permissions in your tenant.

  For example, one user might have access to a mailbox or website, while the other user doesn't have access.

## Part 1 - Set up Easy Auth on your logic app

Your Standard logic app resource requires that you set up Easy Auth so that the delegated identity can flow. At runtime, the chat client integrated with Azure Logic Apps prompts each first-time user to authenticate and uses their identity afterwards.

1. In the [Azure portal](https://portal.azure.com), open your Standard logic app resource.

1. On the resource sidebar, under **Settings**, select **Authentication**.

1. On the **Authentication** page, select **Add identity provider**. From the **Identity provider** list, select **Microsoft** for Microsoft Entra ID.

1. Create or select an app registration by using the options for conversational agents.

1. Set up additional checks for the sign-in process, based on your scenario.

1. Require authentication for requests as appropriate for your environment.

1. When you're done, select **Add** to save your selections.

The following example shows a sample Easy Auth setup:

![Screenshot shows Azure portal, Standard logic app resource and Easy Auth Auth setup.](./media/05-add-user-context-to-tools/easy-auth-setup.png)

For more information, see the following articles:

- [Authentication and authorization in Azure App Service and Azure Functions](https://learn.microsoft.com/azure/app-service/overview-authentication-authorization)
- [Configure your App Service or Azure Functions app to use Microsoft Entra sign-in](https://learn.microsoft.com/azure/app-service/configure-authentication-provider-aad)

## Part 2 - Choose the identity model for each tool action

1. Determine the authorization to use for each tool action:

   - For "my data" or user-personalized operations, such as "Get my upcoming meetings", use OBO.
   - For shared resources or automations, such as "Post today's health status to the operations channel", Use app-only authorization.

1. Document the required scopes or permissions for each tool so you can consent them later.

## Part 3 - Create a per-user, delegated connection

To support delegated user access, create the connection so your tool can use a connector operation:

1. In the [Azure portal](https://portal.azure.com), open the conversational agent workflow in the designer.

1. In the designer, add or select the connector action that you want your workflow to run with OBO, for example, a Microsoft 365 action.

1. On the **Create connection** pane, select **Create as per-user connection?**, which is required and available only for Microsoft service or system connectors, and then select **Sign in**.

   > [!IMPORTANT]
   >
   > You must create per-user connections with the **Create as per-user connection?** option. 
   > You can't convert an existing app-only connection to a per-user connection, so you must
   > create a new per-user connection. If you don't see the per-user connection option,
   > you might be editing an app-only connection. In this case, create a new connection.

1. Complete the sign-in and consent flow, which authorizes the workflow to use your credentials.

   At this point, any sign-in exists only for validation or consent. At runtime, each chat user authenticates at first use, and their identity is used for subsequent calls.

### Expectations for chat first use and reuse

When a tool first uses a connector action with per-user authorization in the chat client, an authentication prompt appears for the user to sign in. After the user signs in, subsequent calls made with same per-user connection don't require reauthentication.

> [!NOTE]
>
> The connection uses credentials that belong to the user in the chat session, not the connection creator.
> This behavior makes sure that the tool runs with the signed-in user's permissions.

## Example: List unread Outlook emails for a signed-in user

The following example shows how to add a tool that lists the unread emails for a signed-in user by using an Outlook connector action with OBO.

### Part 1 - Add the Outlook connector action

1. In the [Azure portal](https://portal.azure.com), open your Standard logic app resource.

1. Find and open your conversational agent workflow in the designer.

1. On the designer, inside the agent and next to any existing tool, select the plus sign (+) for **Add an action** to open the pane where you can browse available actions.

1. On the **Add an action** pane, follow these [general steps](https://learn.microsoft.com/azure/logic-apps/create-workflow-with-trigger-or-action?tabs=standard#add-action) to add an **Office 365 Outlook** action, for example, **Get emails (V3)**, as a tool.

### Part 2 - Set up the per-user, delegated connection

1. On the designer, select the Outlook action that you added as a tool.

1. On the **Create connection** pane, select **Create as per-user connection?**, and then select **Sign in**.

1. Complete the sign-in and consent flow, which authorizes the workflow to use your credentials.

The following example shows how the action appears at this point:

![Screenshot shows Outlook action with selected per-user delegated connection option.](media/05-add-user-context-to-tools/create-obo-connection.png)

### Part 3 - Set up the action

1. On the designer, select the Outlook action to open the information pane for the action.

1. On the **Parameters** tab, provide the following information:

   | Parameter | Value |
   |-----------|-------|
   | **Folder** | **Inbox** |
   | **Fetch only unread messages** | **Yes** |
   | **Top** | **10** |

### Part 4 - Name and describe the tool

1. On the designer, select the tool action to open the information pane for the tool.

1. On the **Details** tab, provide the following information:

   - Name: **List unread emails**
   - Description: **Lists 10 most recent unread emails from the Inbox for the signed-in user.**

   The following example shows how the tool appears at this point:

   ![Screenshot shows tool action with description.](./media/05-add-user-context-to-tools/list-unread-emails-tool.png)

1. Save your workflow.

### Part 5 - Test in chat

When your logic app has Easy Auth set up, the chat client interface is no longer available through the Azure portal. Instead, follow these steps:

1. On the designer toolbar, select **Chat**. In the information message, select **Configure Easy Auth and use the chat client for production purposes**.

   If the link doesn't work for you, in a new browser tab, access the chat client outside the portal at the following URL:

   **`https://<logic-app-name>.azurewebsites.net/api/agentsChat/<workflow-name>/IFrame`**

   ![Screenshot shows link to chat client outside the portal when Easy Auth is set up.](media/05-add-user-context-to-tools/get-integrated-iframe.png)

1. In the chat client interface, ask the following question: **What unread emails do I have?**

   If you're using the tool for the first time, the agent prompts you to sign in for authentication, for example:

   ![Screenshot shows chat client interface and prompt to authenicate.](media//05-add-user-context-to-tools/auth-required.png)

   After you authenticate, the chat client interface that authentication successfully completed, for example:

   ![Screenshot shows chat client interface with successful authentication.](media//05-add-user-context-to-tools/auth-completed.png)

   The chat client interface now returns a summary with unread emails, specifically the subject, sender, and received time.

You can follow a similar pattern when you use other Microsoft 365 connectors, such as OneDrive with **List my recent files** or Teams with with **List my joined teams or recent messages**.

> [!IMPORTANT]
> 
> For OBO scenarios, make sure to select the per-user connection option.

### Part 3 - Test with users who have different access

1. In the chat client outside the Azure portal, start a session as User A.

1. Ask the agent to perform an operation that requires OBO authorization, for example, **Show my upcoming events today**.

1. Confirm that the tool successfully runs and that the results reflect the data and permissions for User A.

1. Repeat these steps as User B. Confirm that the tool successfully runs and that the results reflect the data and permissions for User B.

   Based on access, results for User B might differ from User A.

## Part 5 - Plan for client integration (token pass-through)

If your production experience uses a custom chat client that's web-based, mobile, or another service, plan to provide the user's access token to the agent and implement the OBO flow, based on the following process:

1. Capture the user's sign-in through your app. Get an access token for the target resource, for example, Microsoft Graph, with the required scopes.

1. Pass the token to your agent call, based on your integration model (Module 10).

1. Set up your tool to use the delegated token or a connection that recognizes the user's context.

For more information and sample client code, see [Module 10 - Connect your agents using A2A protocol](10-connect-agents-a2a-protocol.md).

## Review best practices

The following table describes best practices to consider for OBO authorization scenarios:

| Concept | Description |
|---------|-------------|
| Least privilege | Avoid broad permissions and request the minimal delegated scopes needed for each tool. |
| Administrator consent | Reduce consent prompts during chat by getting tenant administrator consent where required. |
| Sensitive data | Avoid showing personally identifiable information (PII) or secrets. Redact or summarize as needed. |
| Mixed identity patterns | Set up OBO authorization for read operations and use app-only authorization for write operations after explicit confirmation. |
| Clear feedback | Instruct the agent to briefly summarize permission errors and suggest remediation, for example, "You might not have access to this mailbox". |
| Auditing and logging | Track and analyze the tools that run and the identity (user or app) they use by reviewing the workflow run history and telemetry. |

## Troubleshoot problems

The following table describes some common problems and troubleshooting suggestions:

| Problem | Suggestion |
|---------|------------|
| 401/403 Unauthorized | Confirm that the connection uses delegated permissions and the user has access to the resource. Check conditional access policies. |
| Consent or scope mismatch | Confirm that the requested scopes match the operation. Recreate the connection if scopes changed. |
| Token audience (aud) errors | Confirm that the access token is issued for the correct resource when using a custom client. |
| Rate limits (429 errors) | Apply backoff or ask the user to narrow the request. |
| Mixed identity confusion | Check the connection that a tool uses. Make sure to clearly label app-only and user-delegated connections. |

## Related content

- [Module 06 - Extend tool functionality with patterns](06-extend-tools-with-patterns.md)
- [Module 10 - Connect your agents using A2A protocol](10-connect-agents-a2a-protocol.md)
