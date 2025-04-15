---
title: "Nodejs Install E401"
date: 2022-01-22T16:54:48Z
tags: [nodejs, e401, azure functions, npm feed, upstream feed, typescript]
---

Today I created a simple nodeJS Azure Functions Applicaiton to start building out a PoC and when I tried to install it's dependencies like so:

```powershell
npm install
```

I got this little cherub back instead:

{{< hint danger >}}

npm ERR! code E401

npm ERR! Incorrect or missing password.

npm ERR! If you were trying to login, change your password, create an

npm ERR! authentication token or enable two-factor authentication then

npm ERR! that means you likely typed your password in incorrectly.

npm ERR! Please try again, or recover your password at:

npm ERR!     https://www.npmjs.com/forgot

npm ERR!

npm ERR! If you were doing some other operation then your saved credentials are

npm ERR! probably out of date. To correct this please try logging in again with:
npm ERR!     npm login

npm ERR! A complete log of this run can be found in:

npm ERR!     C:\Users\garrard.kitchen\AppData\Local\npm-cache\_logs\2022-01-22T16_53_11_848Z-debug-0.log

{{</ hint >}}

Ok then, I've _obviously_ set my default registry to something other than npmjs!

This will not have been an issue or noticeable if I had have been connected to my works VPN as I know for a fact, the upstream sources of our npm Azure DevOps npm feed is in fact `https://registry.npmjs.org`.

Anyhow, to work through this slight annoyance, I typed:

```powershell
nodejs install --registry https://registry.npmjs.org
...
npm install --registry https://registry.npmjs.org

added 78 packages, and audited 79 packages in 12s

30 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

One `npm run prestart` and one `npm run start` later, I was seeing this:


```
[17:09:00] Starting compilation in watch mode...


Azure Functions Core Tools
Core Tools Version:       4.0.3971 Commit hash: d0775d487c93ebd49e9c1166d5c3c01f3c76eaaf  (64-bit)
Function Runtime Version: 4.0.1.16815

[17:09:01] Found 0 errors. Watching for file changes.


Functions:

        svrless: [GET,POST] http://localhost:7071/api/svrless

For detailed output, run func with --verbose flag.
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST http://127.0.0.1:54476/AzureFunctionsRpcMessages.FunctionRpc/EventStream application/grpc -
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /AzureFunctionsRpcMessages.FunctionRpc/EventStream'
[2022-01-22T17:09:02.473Z] Worker process started and initialized.
```

All good, but now I need to change my defaults ...