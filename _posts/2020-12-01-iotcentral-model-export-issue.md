---
layout: post
title:  "IoT Central model export/import issue"
date:   2020-12-01 12:00:00 +0700
categories: custommayd azure iot iotcentral
---
I ran into an issue on IoT Central today where I exported a model, deleted it, tried to re-import and it failed. I discovered the issue after some manual manipulation of the exported JSON and it was related to the generated @id property for the request object of a command capability.

This is the displayed error:

![Encountered error while parsing model document.]({{ site.url }}/assets/iotcentral-error.png)

This is a fragment of what was exported and where I found the issue:

```json
{
    "@id": "dtmi:refrigeratedTrucksDm12012020:RefrigeratedTruck7fe:GoToCustomer;1",
    "@type": "Command",
    "commandType": "synchronous",
    "displayName": {
        "en": "Go to customer"
    },
    "name": "GoToCustomer",
    "request": {
        "@id": "dtmi:refrigeratedTrucksDm12012020:RefrigeratedTruck7fe:GoToCustomer:__request:CustomerID;1",
        "@type": "CommandPayload",
        "displayName": {
        "en": "Customer ID"
        },
        "name": "CustomerID",
        "schema": "integer"
    }
}
```

This is the edited JSON that worked (I removed the `__` characters from the `__request` part of the @id):

```json
{
    "@id": "dtmi:refrigeratedTrucksDm12012020:RefrigeratedTruck7fe:GoToCustomer;1",
    "@type": "Command",
    "commandType": "synchronous",
    "displayName": {
        "en": "Go to customer"
    },
    "name": "GoToCustomer",
    "request": {
        "@id": "dtmi:refrigeratedTrucksDm12012020:RefrigeratedTruck7fe:GoToCustomer:request:CustomerID;1",
        "@type": "CommandPayload",
        "displayName": {
        "en": "Customer ID"
        },
        "name": "CustomerID",
        "schema": "integer"
    }
}
```

I hope this helps someone.
