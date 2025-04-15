---
title: "GitHub Actions Az Cli Login Failed"
date: 2022-03-17T15:50:00Z
draft: true
---


{{< hint danger >}}
Error: Az CLI Login failed. Please check the credentials and make sure az is installed on the runner
{{< /hint>}}


Turns out I forgot to assign SP a RBAC role and scope!

```powershell
$sub=Get-AzSubscription -SubscriptionName 'data.prod.subscription'
new-azroleassignment -ObjectId $sp.Id -RoleDefinitionName 'Contributor' -Scope "/subscriptions/$($sub.SubscriptionId)"
```