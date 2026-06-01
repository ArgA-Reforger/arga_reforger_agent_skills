---
name: reforger-wiki-preprocessor-macros
description: "Trigger: __FILE__, __LINE__, preprocessor macro, debug context. Built-in preprocessor macros for file and line context in Enforce Script debug output."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "__FILE__"
    - "__LINE__"
    - "preprocessor macro"
    - "debug context"
    - "GetAbsolutePath"
---

## Activation Contract

Load this skill when the context involves the built-in preprocessor macros `__FILE__` or `__LINE__` for debug output, logging, or path resolution in Enforce Script. Do NOT load for code that does not reference these macros.

## Hard Rules

**Available Macros**

| Macro | Replaced by | Notes |
|---|---|---|
| `__FILE__` | A string containing the current file's relative script path. | Resolved at compile time. |
| `__LINE__` | A string containing the current file's line number. | Resolved at compile time — value is the line where `__LINE__` appears. |

**__FILE__**
- Expands to the relative path string, e.g., `"scripts/WorkbenchGame/ScriptEditor/TAG_MyTestPlugin.c"`.
- Can be passed to `Workbench.GetAbsolutePath(__FILE__, absPath, true)` to resolve the full filesystem path for tooling.
- Particularly useful in `Print()` calls and log helpers to identify which file triggered a message.

**__LINE__**
- Expands to the line number as a string, e.g., `"4"`.
- Since it is a string, arithmetic with it results in string concatenation NOT integer addition: `__LINE__ + 2` → `"42"` (not `6`).
- Use sparingly — the line number shifts if code above is inserted.

**Use Cases**
- Debug logging: `Print(__FILE__ + ":" + __LINE__ + " — unexpected null");`
- Assert helpers that report location without a debugger.
- Workbench plugin path resolution.

**Note on Preprocessor Macros vs. Directives**
- These are READ-ONLY built-in macros — you cannot define `__FILE__` or `__LINE__` yourself.
- For user-defined flags and conditional compilation see the `reforger-wiki-preprocessor-directives` skill.

## Key APIs / Patterns

```c
// Log with file context
void TAG_LogError(string message)
{
    Print("[ERROR] " + __FILE__ + " — " + message, LogLevel.ERROR);
}

// Log with file + line
Print(__FILE__ + ":" + __LINE__, LogLevel.NORMAL);
// Output: "scripts/Game/TAG_MyClass.c:42"

// Resolve absolute path (Workbench only)
string absPath;
Workbench.GetAbsolutePath(__FILE__, absPath, true);
Print(absPath);  // e.g. "D:/MyMods/TAG_MyMod/scripts/Game/TAG_MyClass.c"

// __LINE__ arithmetic is string concatenation — be explicit
int lineNum = __LINE__.ToInt();  // convert first if arithmetic is needed
```

## References

- PDF: `Scripting_ Preprocessor Macros – Arma Reforger - Bohemia Interactive Community.pdf`
- Wiki: `https://community.bistudio.com/wiki/Arma_Reforger:Scripting:_Preprocessor_Macros`
