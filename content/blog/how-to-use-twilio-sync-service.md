---
title: "Using Twilio Sync Without Trusting the Browser"
date: 2020-04-17T09:14:47+01:00
draft: true
tags: [cloud, twilio, sync, javascript, security]
---

In this article, you'll learn how Twilio Sync models shared state, how a browser obtains access, and what must be secured before clients can read or mutate it. That matters because real-time state is useful only when its access policy is as deliberate as its data model.

Twilio Sync provides cloud-hosted primitives such as Documents, Lists, Maps, and Streams. Client SDKs subscribe to changes and keep local views updated. The client should not hold your Twilio Account SID and Auth Token; a trusted server creates a short-lived Access Token scoped to the required Sync Service.

## Issue a token on the server

The exact web framework is incidental. The important boundary is that the API key secret stays server-side and the authenticated application user becomes the Sync identity.

~~~javascript
import twilio from "twilio";

const AccessToken = twilio.jwt.AccessToken;
const SyncGrant = AccessToken.SyncGrant;

export function createSyncToken(identity) {
  const token = new AccessToken(
    process.env.TWILIO_ACCOUNT_SID,
    process.env.TWILIO_API_KEY,
    process.env.TWILIO_API_SECRET,
    { identity, ttl: 3600 }
  );

  token.addGrant(new SyncGrant({
    serviceSid: process.env.TWILIO_SYNC_SERVICE_SID
  }));

  return token.toJwt();
}
~~~

Authenticate the caller before returning this token. Use a restricted API key, rotate it, and keep the token lifetime short enough for the risk of the application.

## Connect from the client

~~~javascript
import { SyncClient } from "twilio-sync";

const response = await fetch("/api/sync-token", {
  credentials: "include"
});
const { token } = await response.json();

const sync = new SyncClient(token);
const document = await sync.document("shared-status");

document.on("updated", event => {
  renderStatus(event.value.data);
});
~~~

Refresh the Access Token when the SDK reports that it is expiring. Treat object names as identifiers, not as an authorisation mechanism.

## Choose the smallest primitive

Use a Document for one JSON object, a Map for keyed records, a List for ordered items, and a Stream for transient messages. Sync is not a relational database, a durable event log, or a substitute for server-side validation. Validate important transitions on a trusted service and consider a server-authoritative write path for financial, identity, or entitlement data.

Plan for concurrent updates, reconnects, and duplicate UI events. Make handlers idempotent, expose connection state, and decide what the interface should do while offline.

## References
- [Twilio Sync documentation](https://www.twilio.com/docs/sync)
- [Twilio Sync access tokens](https://www.twilio.com/docs/sync/identity-and-access-tokens)
- [Twilio API key guidance](https://www.twilio.com/docs/iam/api-keys)

## Closing thought

Real-time updates make shared state move quickly; a server-side authority boundary is what stops mistakes and impersonation moving at the same speed.
