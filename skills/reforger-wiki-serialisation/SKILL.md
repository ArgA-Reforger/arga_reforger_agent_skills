---
name: reforger-wiki-serialisation
description: "Trigger: SCR_JsonSaveContext, SCR_JsonLoadContext, SCR_BinSaveContext, SCR_BinLoadContext, WriteValue, ReadValue, SerializationSave, SerializationLoad, [NonSerialized], ExportToString, ImportFromString, SaveToFile, LoadFromFile. JSON and binary serialisation of objects in Enforce Script."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "SCR_JsonSaveContext"
    - "SCR_JsonLoadContext"
    - "SCR_BinSaveContext"
    - "SCR_BinLoadContext"
    - "WriteValue"
    - "ReadValue"
    - "SerializationSave"
    - "SerializationLoad"
    - "ExportToString"
    - "ImportFromString"
    - "SaveToFile"
    - "LoadFromFile"
---

## Activation Contract

Load this skill when the user asks about:
- Serialising/deserialising data to/from JSON strings or binary files
- `SCR_JsonSaveContext` / `SCR_JsonLoadContext` (JSON, key-based, order-independent)
- `SCR_BinSaveContext` / `SCR_BinLoadContext` (binary, order-dependent)
- Automatic object serialisation (all properties serialised by default) vs. custom `SerializationSave`/`SerializationLoad` methods
- `[NonSerialized]` decorator to exclude a field from automatic serialisation
- Passing an empty string as the name parameter for top-level complex struct read/write

Do NOT load for: config class decoration (→ reforger-wiki-config-object), BaseContainer property read/write (→ reforger-wiki-base-container), `[JsonEntry]` / `JsonApiStruct` (→ separate JSON API docs).

## Hard Rules

- **JSON** uses key names → `ReadValue` order does NOT matter.
- **Binary** ignores key names → `ReadValue` order MUST exactly match `WriteValue` order.
- Objects that will be deserialised must NOT have constructor parameters — the deserialiser cannot pass arguments; the object is instantiated via default constructor.
- `[NonSerialized]` only skips the field during **automatic** (simple) object serialisation. If the class defines `SerializationSave`/`SerializationLoad`, those methods take full control and `[NonSerialized]` is ignored.
- If `SerializationSave` is defined, the save context calls it exclusively — automatic property processing does NOT happen.
- If `SerializationLoad` is defined, the load context calls it exclusively — automatic property processing does NOT happen.
- Always call `context.IsValid()` at the start of `SerializationSave`/`SerializationLoad`.
- Passing an empty string `""` as the name parameter to `WriteValue`/`ReadValue` allows writing/reading a complex top-level struct without a named wrapper key.

## Key APIs / Patterns

```enforce
// JSON save
SCR_JsonSaveContext saveCtx = new SCR_JsonSaveContext();
saveCtx.WriteValue("key1", someString);
saveCtx.WriteValue("key2", someInt);
string json = saveCtx.ExportToString();   // export to string for sending/storing

// JSON load (order-independent)
SCR_JsonLoadContext loadCtx = new SCR_JsonLoadContext();
loadCtx.ImportFromString(json);
string s;
int i;
loadCtx.ReadValue("key2", i);   // order does not matter for JSON
loadCtx.ReadValue("key1", s);

// Binary save
SCR_BinSaveContext binSave = new SCR_BinSaveContext();
binSave.WriteValue("k1", someString);
binSave.WriteValue("k2", someInt);
binSave.SaveToFile("data.bin");

// Binary load (MUST match WriteValue order)
SCR_BinLoadContext binLoad = new SCR_BinLoadContext();
binLoad.LoadFromFile("data.bin");
binLoad.ReadValue("k1", s);   // same order as WriteValue
binLoad.ReadValue("k2", i);

// Simple automatic object serialisation (all members serialised)
class MyData : Managed
{
    protected int m_iValue = 42;
    protected string m_sLabel;

    [NonSerialized]
    protected float m_fIgnored = 3.14;   // excluded from auto-serialisation
}

// Custom serialisation — full control, NonSerialized ignored here
class MyData : Managed
{
    protected int m_iValue = 42;
    protected string m_sLabel = "Hello";
    protected float m_fExtra = 3.14;

    bool SerializationSave(BaseSerializationSaveContext context)
    {
        if (!context.IsValid())
            return false;
        context.WriteValue("val", m_iValue);
        context.WriteValue("label", m_sLabel);
        // m_fExtra intentionally omitted
        return true;
    }

    bool SerializationLoad(BaseSerializationLoadContext context)
    {
        if (!context.IsValid())
            return false;
        context.ReadValue("val", m_iValue);
        context.ReadValue("label", m_sLabel);
        return true;
    }
}
```

## References

- PDF: `Serialisation – Arma Reforger - Bohemia Interactive Community.pdf`
- See also: `reforger-wiki-config-object` (config class decoration), `reforger-wiki-base-container` (BaseContainer property access)
