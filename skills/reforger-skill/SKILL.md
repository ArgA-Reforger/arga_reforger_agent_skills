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
| `reforger-wiki-conventions` | naming conventions, `m_`, `s_`, `g_`, `SCR_`, `TAG_`, Allman style, `[Attribute]` | Code style and order conventions |
| `reforger-wiki-sqf-to-enforce` | SQF, `forEach`, `while`, `switch`, `hint`, migration, Arma 3 | SQF → Enforce Script migration |

## References

- `scripts\` — Reforger scripts source of truth
- `https://community.bistudio.com/wiki/Category:Arma_Reforger/Modding` — Bohemia Modding
- `https://community.bistudio.com/wikidata/external-data/arma-reforger/ArmaReforgerScriptAPIPublic/` — Arma Reforger Script API
- `https://community.bistudio.com/wikidata/external-data/arma-reforger/EnfusionScriptAPIPublic/` — Enfusion Script API

