---
title: "Npm E401 and CERT_NOT_YET_VALID"
date: 2022-01-11T11:47:54Z
tags: [github actions, npm, nodejs, e401, cert_not_yet_valid, docker, acr, npmrc, GH Secrets]
---

Today a PR Merge resulted in a GHA failure.  Sadly, this is not the only CICD pipeline to fail this year!  This particular pipeline builds a NodeJS Image, pushes the image to ACR and deploys the service to a production Docker Swarm (on merge to main).

This was the error:

{{< hint danger >}}
---
[3/7] RUN npm install:

#7 1.469 npm ERR! code E401

#7 1.470 npm ERR! Unable to authenticate, need: Bearer authorization_uri=https://login.windows.net/736f9f**-0**9-49**-86**-b******31f407, Basic realm="https://pkgsprodsu3weu.app.pkgs.visualstudio.com/", TFS-Federated

#7 1.475

#7 1.475 npm ERR! A complete log of this run can be found in:

#7 1.475 npm ERR!     /root/.npm/_logs/2021-12-27T09_38_24_060Z-debug.log

{{< /hint >}}

Mmmm, E401? 🤔

I've not seen this error before but as it was auth related, assumed the PAT token had expired <sup>1</sup>.  Google...

I did not have the actual .npmrc file that was in a GH Secret so I used my own local .npmrc file to confirm I can pull from our organisation's npm feed.

```powershell
cd <project-root>
rm node-modules
npm cache clean --force
npm install
```

It worked.  So, I had my first fallback option - regenerate PAT, register centrally for a reminder of when the PAT is to expire and update GH secret.  However, I was not yet done.  I had not yet reproduced verbatim the pipeline, egro a `docker build`.

So, I ran this:

```powershell
docker build -t <image:tag> .
```

Oh no!  The error has returned!

This is the Dockerfile:

```dockerfile
FROM node:14.4.0
WORKDIR /
COPY . .
RUN npm install
RUN echo 'module.exports = ' | cat - node_modules/@<redacted>/<redacted>/dist/libs/<redacted>-lib/index.js > temp && mv temp helpers/index.js
RUN cd helpers && ls -la
RUN head -10 helpers/index.js
ENV PORT=80
EXPOSE 80
CMD ["npm", "run", "<redacted>"]
```

Apart from it not being the LTS, there was nothing obviously wrong with it plus it had been building ok leading up to this.

This a the credentials part of the .npmrc:

```
registry=https://pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry
always-auth=true
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry/:username=<any-value-not-empty>
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry/:_password=<Base64-encoded-PAT>
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry/:email=<email-is-not-used>
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry/:always-auth=true
```

According to this Microsoft documentation post ➡ [npm scopes](https://docs.microsoft.com/en-us/azure/devops/artifacts/npm/scopes?view=azure-devops#credentials-setup), the token structure was incomplete. I corrected the structure:

```
registry=https://pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry
always-auth=true
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry/:username=<any-value-not-empty>
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry/:_password=<Base64-encoded-PAT>
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry/:email=<email-is-not-used>
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/registry/:always-auth=true
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/:username=<any-value-not-empty>
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/:_password=<Base64-encoded-PAT>
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/:email=<email-is-not-used>
//pkgs.dev.azure.com/<org-name>/_packaging/<feed-name>/npm/:always-auth=true
```

I re-ran the `docker build -t <image:tag> .` This time I got a different error but the original `E401` error had gone away!

The new error:

{{< hint danger >}}

#7 43.61 npm ERR! code CERT_NOT_YET_VALID

#7 43.61 npm ERR! errno CERT_NOT_YET_VALID

#7 43.61 npm ERR! request to https://xuavsblobprodsu6weus12.blob.core.windows.net/b-7a3f75bdbbf3432bbe2621e93c98932a/86CDB768B8C395B14974*********1963D831555B55E3458*********************.blob?sv=2019-07-07&sr=b&si=1&sig=LUCuO42mrmOAx5N*************zds9RWS0v2qL%2FwbB86c%3D&spr=https&se=2022-01-12T11%3A42%3A24Z&rscl=x-e2eid-59793e65-**********-a727df6e-***********-session-59793e65-**********-a727df6e-8c628cc8&rscd=attachment%3B%20filename%3D%22string-width-4.2.3.tgz%22 failed, reason: certificate is not yet valid

{{< /hint>}}

Mmmm, `CERT_NOT_YET_VALID`.  I'd not seen this error before.  Google...

After a short period of online research 👀 I found a suggestion about being explicit in setting the default npm registry.  This meant I had to insert `RUN npm config set registry http://registry.npmjs.org` before the `RUN npm install` command.  The resulting Dockerfile then looked like this:


```dockerfile
FROM node:14.4.0
WORKDIR /
COPY . .
RUN npm config set registry http://registry.npmjs.org
RUN npm install 
RUN echo 'module.exports = ' | cat - node_modules/@<redacted>/<redacted>/dist/libs/<redacted>-lib/index.js > temp && mv temp helpers/index.js
RUN cd helpers && ls -la
RUN head -10 helpers/index.js
ENV PORT=80
EXPOSE 80
CMD ["npm", "run", "prod"]
```

I re-ran the `docker build -t <image:tag> .` and success!  🥳

I updated the appropriate GH Secret with the modified .npmrc file and asked the author of the PR that had reported this issue originally to make the 1 line change to the Dockerfile.  He made the change, prompted for another PR review and merged to main on approval.  The GHA ran successfully!  The usual monitoring post deploy and feature/fix was confirmed and I set about updating internal documentation providing instructions on what to do to remediate for others and notifying all via slack that of this.


# References

- [Npm scopes](https://docs.microsoft.com/en-us/azure/devops/artifacts/npm/scopes?view=azure-devops)

--- 

<sup>1</sup> - secrets that expire need to be registered centrally on a system that can notify you in advance, giving you ample time to remediate.  For our Azure AAD SP (Service Principals) client secrets, we run an Automation Runbook each day that traverses the Azure Resource Graph and alerts me via email of those SP that will be expiring within 30 days.