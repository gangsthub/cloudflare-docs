---
type: example
summary: Send debugging information in an errored response to a logging service.
tags:
  - Debugging
languages:
  - JavaScript
  - TypeScript
  - Python
pcx_content_type: configuration
title: Debugging logs
weight: 1001
layout: example
---

{{<tabs labels="js | ts | py">}}
{{<tab label="js" default="true">}}

```js
export default {
  async fetch(request, env, ctx) {
    // Service configured to receive logs
    const LOG_URL = "https://log-service.example.com/";

    async function postLog(data) {
      return await fetch(LOG_URL, {
        method: "POST",
        body: data,
      });
    }

    let response;

    try {
      response = await fetch(request);
      if (!response.ok && !response.redirected) {
        const body = await response.text();
        throw new Error(
          "Bad response at origin. Status: " +
            response.status +
            " Body: " +
            // Ensure the string is small enough to be a header
            body.trim().substring(0, 10)
        );
      }
    } catch (err) {
      // Without ctx.waitUntil(), your fetch() to Cloudflare's
      // logging service may or may not complete
      ctx.waitUntil(postLog(err.toString()));
      const stack = JSON.stringify(err.stack) || err;
      // Copy the response and initialize body to the stack trace
      response = new Response(stack, response);
      // Add the error stack into a header to find out what happened
      response.headers.set("X-Debug-stack", stack);
      response.headers.set("X-Debug-err", err);
    }
    return response;
  },
};
```

{{</tab>}}
{{<tab label="ts">}}

```ts
interface Env {}
export default {
  async fetch(request, env, ctx): Promise<Response> {
    // Service configured to receive logs
    const LOG_URL = "https://log-service.example.com/";

    async function postLog(data) {
      return await fetch(LOG_URL, {
        method: "POST",
        body: data,
      });
    }

    let response;

    try {
      response = await fetch(request);
      if (!response.ok && !response.redirected) {
        const body = await response.text();
        throw new Error(
          "Bad response at origin. Status: " +
            response.status +
            " Body: " +
            // Ensure the string is small enough to be a header
            body.trim().substring(0, 10)
        );
      }
    } catch (err) {
      // Without ctx.waitUntil(), your fetch() to Cloudflare's
      // logging service may or may not complete
      ctx.waitUntil(postLog(err.toString()));
      const stack = JSON.stringify(err.stack) || err;
      // Copy the response and initialize body to the stack trace
      response = new Response(stack, response);
      // Add the error stack into a header to find out what happened
      response.headers.set("X-Debug-stack", stack);
      response.headers.set("X-Debug-err", err);
    }
    return response;
  },
} satisfies ExportedHandler<Env>;
```

{{</tab>}}
{{<tab label="py">}}

```py
import json
import traceback
from pyodide.ffi import create_once_callable
from js import Response, fetch, Headers

async def on_fetch(request, _env, ctx):
    # Service configured to receive logs
    log_url = "https://log-service.example.com/"

    async def post_log(data):
        return await fetch(log_url, method="POST", body=data)

    response = await fetch(request)

    try:
        if not response.ok and not response.redirected:
            body = await response.text()
        # Simulating an error. Ensure the string is small enough to be a header
        raise Exception(f'Bad response at origin. Status:{response.status} Body:{body.strip()[:10]}')
    except Exception as e:
        # Without ctx.waitUntil(), your fetch() to Cloudflare's
        # logging service may or may not complete
        ctx.waitUntil(create_once_callable(post_log(e)))
        stack = json.dumps(traceback.format_exc()) or e
        # Copy the response and add to header
        response = Response.new(stack, response)
        response.headers["X-Debug-stack"] = stack
        response.headers["X-Debug-err"] = e

    return response
```

{{</tab>}}
{{</tabs>}}
