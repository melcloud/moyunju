---
title: "Azure Function Proxy Setting"
date: 2019-11-21T08:48:55+11:00
draft: false
toc: false
images:
tags:
  - Azure
  - Azure Function
---

When configuring Azure Function to Proxy and a catch all route, a request to http://function.azurewebsites.com/foo/bar will be translated into http://some.domain/foo%2Fbar. Setting `AZURE_FUNCTION_PROXY_BACKEND_URL_DECODE_SLASHES` needs to be changed to true if destination is not hosted in IIS. This allows URL to be translated back to http://some.domain/foo/bar before dispatching.

```json
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "all-api": {
            "matchCondition": {
                "route": "/api/{*restOfPath}"
            },
            "backendUri": "https://api.example.com/{restOfPath}",
            "debug": true
        }
    }
}
```
