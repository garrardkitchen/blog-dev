---
title: "Diagnosing Azure Login and RBAC Failures in GitHub Actions"
date: 2022-03-17T15:50:00Z
draft: true
tags: [cloud, github-actions, azure-cli, rbac, workload-identity]
---

In this article, you'll learn how to separate Azure authentication from authorisation, verify the identity used by a GitHub Actions job, and assign the narrowest role at the correct scope. That matters because a successful token request does not imply permission to change a subscription.

The original workflow reported:

{{< callout type="error" >}}
Error: Az CLI Login failed. Please check the credentials and make sure az is installed on the runner
{{< /callout >}}

In this case, the service principal existed but did not have an Azure role assignment at the scope the workflow needed. The message blurred two different stages.

## Authentication or authorisation?

Authentication establishes the identity and obtains a token. Authorisation evaluates that identity against Azure RBAC for a particular action and resource scope.

After the login step, inspect the active context without printing tokens:

~~~bash
az account show --query "{tenant:tenantId, subscription:id, user:user.name}" -o json
az role assignment list --assignee-object-id "$AZURE_CLIENT_OBJECT_ID" --all --include-inherited -o table
~~~

Also inspect the actual failing command and HTTP status. An invalid secret, issuer, audience, tenant, or federated credential usually prevents login. A 403 after login points towards role, scope, deny assignments, policy, or a data-plane permission model.

## Prefer workload identity federation

For GitHub Actions, prefer OpenID Connect federation with azure/login over a long-lived client secret. Grant the job id-token: write, configure a federated credential whose repository, branch or environment subject is appropriately narrow, and store tenant, subscription, and client identifiers as configuration rather than secrets.

The Azure identity still needs an RBAC role. Assign the least-privilege role at the smallest scope that contains the resources it must manage. Contributor at subscription scope is broad and cannot create role assignments; use a resource-group or resource scope and a narrower built-in or custom role where possible.

The historical PowerShell shape was:

~~~powershell
$subscription = Get-AzSubscription -SubscriptionName 'data.prod.subscription'
$scope = "/subscriptions/$($subscription.SubscriptionId)"
New-AzRoleAssignment -ObjectId $servicePrincipalObjectId -RoleDefinitionName 'Contributor' -Scope $scope
~~~

Run role-assignment commands from a trusted provisioning identity, not from the deployment identity unless delegation is intentional. Use the service principal's object ID, not its application/client ID, where the command expects an object ID.

## Allow for propagation, but diagnose first

New role assignments can take time to propagate and cached tokens may not immediately reflect every change. Retry with bounded backoff after confirming the assignment exists at the intended scope. Repeatedly creating broader roles is not a propagation strategy.

If the workflow accesses a service data plane—Key Vault secrets, Storage blobs, or a database—confirm whether that service uses Azure RBAC, access policies, or its own permissions. Management-plane Contributor does not automatically grant access to protected data.

## References

- [Azure Login action with OpenID Connect](https://github.com/Azure/login#login-with-openid-connect-oidc)
- [Azure RBAC scope](https://learn.microsoft.com/azure/role-based-access-control/scope-overview)
- [Troubleshoot Azure RBAC](https://learn.microsoft.com/azure/role-based-access-control/troubleshooting)

## Closing thought

An Azure login proves who the workflow is; the role assignment and scope decide whether that identity should be allowed to do the work.
