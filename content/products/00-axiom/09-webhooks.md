---
title: "Webhooks"
metaTitle: "Hypi Platform CRUD Documentation"
metaDescription: "Hypi platform documentation for adding Webhooks into your app"
---

## Overview

Webhooks provide a way for external APIs to notify you when some event has occured in their system.
That is a typical usecase for this API.

These APIs are generally not secure. Different providers have different mechanisms for proving to you that the request is genuinely from them.
Hypi provides you access to all the data sent in the request, this enables you to perform verification as you see fit.

On this page, the following schema is used:

```graphql
type Query {
  myInlineFn(payload: WebhookPayload): WebhookResponse @tan(type:Groovy, inline: """
    return [
      "status": 200,
      "headers": payload.headers,
      "body": payload.url.host + ":" + payload.url.port + payload.url.path + "?" + payload.url.queryParams + "<>" + payload.body
    ]
  """)
}
```

## Defined types

A function must be defined which will process the Webhook request.
Hypi defines both the argument and response type that this function must use.

> Required
>
> The function signature MUST be `(payload: WebhookPayload): WebhookResponse` for the webhook to work correctly.


## Example

A simple webhook example looks like this:

<div className={"code-container"}>

<div className={"code-column"}>

```
#GraphQL query
mutation Upsert($values: HypiUpsertInputUnion!) {
  upsert(values: $values) {
    id
  }
}
# Data
{
  "values": {
    "Webhook": [
      {
  "hypi": {
    "id": "wid"
  },
  "name": "wh1",
  "query": {
    "type": "Query",
    "field": "myInlineFn"
  }
}
    ]
  }
}
```
</div>
<div className={"code-column"}>

```json
{
  "data": {
    "upsert": [
      {
        "id": "wid"
      }
    ]
  }
}
```
</div>

</div>

Once a Webhook is created, it can be called by making a request to:

* https://api.hypi.app/webhook/{domain}/{webhook name}

where `{domain}` is the instance domain for you app and `{webhook name}` is the name of the webhook to be executed.
In the example on this page, the name is `wh1`.
