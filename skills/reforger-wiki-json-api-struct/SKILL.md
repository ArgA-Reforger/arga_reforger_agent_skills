---
name: reforger-wiki-json-api-struct
description: "Trigger: JsonApiStruct, SCR_JsonApiStruct, SerializeToJson, DeserializeFromJson, json struct. JsonApiStruct-based JSON encode/decode, file I/O, error handling, and validation patterns."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "JsonApiStruct"
    - "SCR_JsonApiStruct"
    - "SerializeToJson"
    - "DeserializeFromJson"
    - "json struct"
---

## Activation Contract

Load this skill when using `JsonApiStruct` (or classes inheriting from it) to encode Enforce Script objects to JSON, decode JSON onto objects, read/write JSON files, or handle REST API callback payloads.

## Hard Rules

**Class setup**
- Inherit from `JsonApiStruct` to create a serialisable struct.
- In the constructor, register every variable that should participate in JSON encode/decode via `RegV("variableName")`.
- Registered variable names are CASE-SENSITIVE. A JSON property `"Health"` does NOT match a variable named `"health"`.
- Non-registered variables are silently ignored during pack/expand.
- Multi-type arrays are NOT supported by Enforce Script — all array elements must be the same type.

**Supported types for auto registration**
`float`, `int`, `bool`, `string`, `array<T>`, nested `JsonApiStruct` objects, arrays of objects.

**Encoding (Pack)**
- Call `Pack()` to serialise the object to an internal buffer.
- Call `AsString()` to retrieve the packed JSON as a string.
- Call `PackToFile("path.json")` to pack and write to file in one call.
- Override `OnPack()` to add variables not auto-registered (use `StoreFloat`, `StoreInt`, `StoreObject`, `StartArray`/`ItemString`/`EndArray`, etc.).

**Decoding (Expand)**
- Call `ExpandFromRAW(string data)` to decode JSON text onto the object.
- Call `LoadFromFile("path.json")` to load and expand from a file — `Expand` is called automatically.
- Override `OnExpand()` to handle custom pre-processing before expand starts.
- Override `OnBufferReady()` to access the packed string after a successful `Pack`.

**Error handling**
- Override `OnSuccess(int errorCode)` — called when a pending store operation (typically from REST) completes with `ETJSON_OK`.
- Override `OnError(int errorCode)` — called on failure. Check `EJsonApiError` enum for specific codes:
  - `ETJSON_PARSERERROR` — malformed JSON input
  - `ETJSON_FAILFILELOAD` — file could not be loaded
  - `ETJSON_FAILFILESAVE` — file could not be saved
  - `ETJSON_TIMEOUT` — REST transmission timeout
  - `ETJSON_NOBUFFERS` — too many objects processing at once
- Error codes are processed asynchronously for REST requests to avoid blocking the main thread.

**Object lifetime**
- The scripter is responsible for keeping the `JsonApiStruct` object alive for the duration of an async operation. If it is garbage-collected before the callback fires, the callback will be lost.
- Use `ref` or a persistent member variable to hold the object.

**Hierarchy**
- Nested objects: declare child `JsonApiStruct` members, create them with `new` in the constructor, and `RegV("childName")`.
- `OnPack()` is called hierarchically on all registered child objects.

## Key APIs / Patterns

```enforce
// Simple struct
class MyData : JsonApiStruct
{
    string name;
    int score;

    void MyData()
    {
        RegV("name");
        RegV("score");
    }
}

// Pack to string
MyData data = new MyData();
data.name = "Player";
data.score = 42;
data.Pack();
string json = data.AsString(); // {"name":"Player","score":42}

// Pack to file
data.PackToFile("$profile:save.json");

// Load from file
MyData loaded = new MyData();
loaded.LoadFromFile("$profile:save.json");
Print(loaded.name); // "Player"

// Load from raw string
MyData fromStr = new MyData();
fromStr.ExpandFromRAW(json);

// Hierarchy example
class Avatar : JsonApiStruct
{
    string id;
    void Avatar() { RegV("id"); }
}

class PlayerProfile : JsonApiStruct
{
    string playerName;
    ref Avatar avatar;

    void PlayerProfile()
    {
        avatar = new Avatar();
        RegV("playerName");
        RegV("avatar");
    }
}

// Custom OnPack with array
class ItemList : JsonApiStruct
{
    protected ref array<string> m_aItems = {};

    void ItemList() { /* no RegV needed if using OnPack */ }

    override void OnPack()
    {
        StartArray("items");
        foreach (string item : m_aItems)
            ItemString(item);
        EndArray();
    }
}
```

**JSON Validation pattern**
```enforce
// Round-trip validation
MyData original = new MyData();
original.Pack();
string data1 = original.AsString();
original.ExpandFromRAW(data1);
original.Pack();
string data2 = original.AsString();
// data1 and data2 should be identical if the struct is consistent
```

## References

- PDF: `JsonApiStruct Usage – Arma Reforger - Bohemia Interactive Community.pdf`
- Wiki: `https://community.bistudio.com/wiki/Arma_Reforger:JsonApiStruct_Usage`
- See also: `reforger-wiki-json` (raw JSON format rules), `reforger-wiki-rest-api` (REST callback usage), `reforger-wiki-serialisation` (SCR_JsonSaveContext)
