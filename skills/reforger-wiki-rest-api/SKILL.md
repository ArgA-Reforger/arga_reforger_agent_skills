---
name: reforger-wiki-rest-api
description: "Trigger: RestCallback, RestContext, GET request, POST request, HTTP header. RestApi scripting — context creation, GET/POST requests, callback handling, and lifetime rules."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "RestCallback"
    - "RestContext"
    - "GET request"
    - "POST request"
    - "HTTP header"
---

## Activation Contract

Load this skill when writing Enforce Script code that communicates with an external web API via `RestApi`, `RestContext`, or `RestCallback`.

## Hard Rules

**Scope and limitations**
- `RestApi` supports only GET and POST HTTP methods. Other HTTP verbs (PUT, DELETE, PATCH) are NOT available.
- Accepted data size is below 1 MB. Larger responses will fail.
- Custom HTTP headers are NOT supported.
- Print output from REST callbacks is limited.
- REST requests are asynchronous — they do NOT block the main game thread.

**Context**
- A `RestContext` represents a base URL. Create it once and reuse it for multiple requests to the same host.
- Obtain a context via `GetGame().GetRestApi().GetContext("https://example.com/")`.
- The URL passed to `GetContext` is the root; append the endpoint path in the `GET`/`POST` call.

**Callback**
- Create a custom callback class inheriting from `RestCallback`.
- Override `OnSuccess(string data, int dataSize)`, `OnError(int errorCode)`, and `OnTimeout()`.
- The scripter is responsible for callback object lifetime. Hold the callback in a `ref` member variable for the duration of the request — if it gets garbage-collected the callback will fire on a null reference.
- `OnSuccess` data string may be large; `Print()` does not output strings exceeding its buffer limit — use the `dataSize` parameter to guard.

**Making requests**
- `context.GET(callback, "endpoint")` — sends HTTP GET to `<base>/<endpoint>`.
- `context.GET(callback, "endpoint?param=value")` — query string is appended directly to the endpoint string.
- POST requests are available via `context.POST(callback, "endpoint", body)`.

**Error handling**
- Error codes are processed asynchronously. Never assume a request completed synchronously.
- Common error codes (from `EJsonApiError`): network send failure, receive failure, timeout, parse error.

## Key APIs / Patterns

```enforce
// Callback class
class MyRestCallback : RestCallback
{
    override void OnSuccess(string data, int dataSize)
    {
        PrintFormat("Success: %1 bytes", dataSize);
        if (dataSize > 0)
            Print(data);
    }

    override void OnError(int errorCode)
    {
        PrintFormat("Error: %1", errorCode);
    }

    override void OnTimeout()
    {
        Print("Request timed out.");
    }
}

// Holder class — keeps callback alive
class ApiClient
{
    protected ref MyRestCallback m_Callback;

    void SendRequest()
    {
        if (!m_Callback)
            m_Callback = new MyRestCallback();

        RestContext ctx = GetGame().GetRestApi().GetContext("https://httpbin.org/");

        // Simple GET
        ctx.GET(m_Callback, "get");

        // GET with query string
        ctx.GET(m_Callback, "get?x=10&y=5");
    }
}
```

**Calling from game code**
```enforce
class MyComponent : ScriptComponent
{
    protected ref ApiClient m_ApiClient;

    override void EOnPostInit(IEntity owner)
    {
        m_ApiClient = new ApiClient();
        m_ApiClient.SendRequest();
    }
}
```

## References

- PDF: `REST API Usage – Arma Reforger - Bohemia Interactive Community.pdf`
- Wiki: `https://community.bistudio.com/wiki/Arma_Reforger:REST_API_Usage`
- See also: `reforger-wiki-json-api-struct` (JsonApiStruct as REST callback payload handler)
