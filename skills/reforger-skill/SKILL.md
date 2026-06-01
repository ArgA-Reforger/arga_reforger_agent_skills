---
name: reforger-skill
description: "Trigger: *.c, Enforce, Enfusion. Hub router for Arma Reforger Enforce scripting — applies ARGA naming conventions and delegates to spoke skills for specific APIs."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "2.0.0"
  triggers:
    - "*.c"
    - "Enforce"
    - "Enfusion"
---

## Activation Contract

Load this skill when editing or generating `.c` files inside an Enforce/Reforger project. Do not apply to `.json`, `.txt`, or non-Enforce files. This hub applies only global naming and file conventions; all specific API, language, and domain rules are handled by spoke skills loaded alongside this one.

## Hard Rules

**Class & File Naming**
- Classes: `ARGA_` prefix, PascalCase — e.g. `ARGA_MyComponent`.
- Files: `ARGA_` prefix, PascalCase, `.c` extension — matches class name exactly — e.g. `ARGA_MyComponent.c`.
- Class body closing brace MUST end with `;`.
- Enum names: `ARGA_E` prefix, PascalCase — e.g. `ARGA_EMyState`.
- File must be in `scripts\Game\` or the appropriate subdirectory matching its class type.

## Spoke Skills

The following spoke skills cover specific domains. They are loaded alongside this hub when their exclusive trigger keywords appear in context:

| Spoke skill | Exclusive triggers | Domain |
|---|---|---|
| `reforger-wiki-oop-basics` | `class`, `extends`, `modded`, `sealed`, inheritance | OOP class model, constructors, visibility |
| `reforger-wiki-scripting-first-steps` | `Print`, `PrintFormat`, `Remote Console`, Workbench setup | Scripting environment and debug output |
| `reforger-wiki-values` | `int`, `float`, `bool`, `string`, `vector`, `array`, `map`, `set`, `typename`, `enum`, `const`, `ref` | Types, values, scope, casting |
| `reforger-wiki-operators` | `&&`, `\|\|`, `<<`, `>>`, `%`, `==`, `!=`, `+=`, bitwise | Operators and precedence |
| `reforger-wiki-keywords` | `override`, `out`, `inout`, `notnull`, `owned`, `auto`, `new`, `delete`, `thread`, `super`, `vanilla`, `proto`, `native`, `volatile`, `event`, `typedef` | Language keywords |
| `reforger-wiki-conventions` | naming conventions, `m_`, `s_`, `g_`, `SCR_`, `TAG_`, Allman style, `camelCase`, `PascalCase` | Code style and order conventions |
| `reforger-wiki-sqf-to-enforce` | SQF, `forEach`, `while`, `switch`, `hint`, migration, Arma 3 | SQF → Enforce Script migration |
| `reforger-wiki-oop-advanced` | `Cast`, `upcasting`, `downcasting`, `class hierarchy` | Advanced OOP: type casting, class hierarchies, mod load order |
| `reforger-wiki-arc` | `ARC`, `Managed`, `weak reference`, `memory leak`, `cyclic reference`, `reference counting`, `autoptr` | Automatic Reference Counting memory model |
| `reforger-wiki-class-template` | `class template`, `generic class`, `<Class T>`, `Cast helper` | Generic / template class syntax |
| `reforger-wiki-preprocessor-directives` | `#define`, `#ifdef`, `#ifndef`, `#endif`, `#include`, preprocessor | Preprocessor directives |
| `reforger-wiki-preprocessor-macros` | `__FILE__`, `__LINE__`, debug context macros | Built-in preprocessor macros |
| `reforger-wiki-script-invoker` | `ScriptInvoker`, `ScriptInvokerVoid`, `ScriptInvokerBase`, `event handler`, `event subscription` | ScriptInvoker event subscription system |
| `reforger-wiki-base-container` | `BaseContainer`, `GetOwner`, `BaseContainerList`, `BaseContainerTools`, `SCR_BaseContainerTools`, `BaseContainerProps` | BaseContainer data model (Prefab/Config/IEntitySource) |
| `reforger-wiki-config-object` | `ConfigObject`, `[BaseContainerProps]`, `[Attribute]`, `configRoot`, `NamingConvention`, `uiwidget`, `ParamEnum` | Config Editor decorators and widget reference |
| `reforger-wiki-config-class` | `config asset`, `config file creation`, `Workbench config`, `editable property`, `SCR_BaseContainerHolder` | Creating a .conf root class and Workbench asset |
| `reforger-wiki-scripting-conf` | `.conf file`, `UserConfig`, `ResourceName approach`, `Object approach`, `ParamString`, `ParamFloat`, `SCR_ConfigHelperT` | Loading .conf files from script at runtime |
| `reforger-wiki-prefab-data` | `EntityPrefabData`, `GetPrefabData`, `SCR_EntityPrefabData`, `prefab data`, `ComponentData`, `GetComponentData` | Shared prefab data pattern (Class class) |
| `reforger-wiki-resource-usage` | `ResourceName`, `Resource.Load`, `resource lifetime`, `keep resource reference`, `BaseResourceObject` | Safe resource loading and reference lifetime |
| `reforger-wiki-serialisation` | `SCR_JsonSaveContext`, `SCR_JsonLoadContext`, `SCR_BinSaveContext`, `SerializationSave`, `SerializationLoad`, `WriteValue`, `ReadValue` | JSON and binary serialisation |

## References

- `scripts\` — Reforger scripts source of truth
- `https://community.bistudio.com/wiki/Category:Arma_Reforger/Modding` — Bohemia Modding
- `https://community.bistudio.com/wikidata/external-data/arma-reforger/ArmaReforgerScriptAPIPublic/` — Arma Reforger Script API
- `https://community.bistudio.com/wikidata/external-data/arma-reforger/EnfusionScriptAPIPublic/` — Enfusion Script API

