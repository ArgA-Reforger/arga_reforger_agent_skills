# ArgA Reforger agent skills

A family of 44 developer skills for writing Arma Reforger Enforce Script with an LLM — covering the full scripting surface from language basics to multiplayer replication, damage systems, and Workbench plugins.

---

## How it works — Hub and Spoke

The family uses a **Hub-and-Spoke** architecture:

```
*.c file / Enforce / Enfusion context
        │
        ▼
┌─────────────────────┐
│   reforger-skill    │  ← Hub — always loads on .c files
│   (naming rules)    │       enforces ARGA_ prefix, file conventions
└──────────┬──────────┘
           │  delegates domain questions to ↓
    ┌──────┴───────────────────────────────────┐
    │  43 wiki spoke skills                    │
    │  each loads when its keywords appear     │
    │                                          │
    │  reforger-wiki-arc        (ARC, Managed) │
    │  reforger-wiki-multiplayer (RplRpc, ...) │
    │  reforger-wiki-component  (EOnPostInit)  │
    │  ...                                     │
    └──────────────────────────────────────────┘
```

The hub loads automatically on any `.c` file. Spoke skills load on top of it when domain-specific keywords appear in context. You never activate them manually — the LLM picks the right combination based on what you're working on.

---

## Skill file locations

```
skills/
  reforger-skill/SKILL.md              ← Hub router
  reforger-wiki-oop-basics/SKILL.md
  reforger-wiki-arc/SKILL.md
  reforger-wiki-multiplayer/SKILL.md
  ... (43 spoke SKILL.md files)
```

---

## Spoke skills by domain

### Language Fundamentals

| Skill | Loads when you mention… |
|---|---|
| `reforger-wiki-oop-basics` | `class`, `extends`, `modded`, `sealed`, constructors |
| `reforger-wiki-scripting-first-steps` | `Print`, `PrintFormat`, Workbench setup, Remote Console |
| `reforger-wiki-values` | `int`, `float`, `bool`, `string`, `array`, `map`, `ref`, `typename` |
| `reforger-wiki-operators` | `&&`, `\|\|`, `<<`, `>>`, `%`, bitwise operators |
| `reforger-wiki-keywords` | `override`, `out`, `inout`, `notnull`, `owned`, `thread`, `super` |
| `reforger-wiki-conventions` | naming conventions, `m_`, `s_`, `g_`, `SCR_`, Allman style |
| `reforger-wiki-sqf-to-enforce` | SQF migration, `forEach`, `hint`, Arma 3 patterns |

### Advanced OOP & Memory

| Skill | Loads when you mention… |
|---|---|
| `reforger-wiki-oop-advanced` | `Cast`, upcasting, downcasting, class hierarchy |
| `reforger-wiki-arc` | `ARC`, `Managed`, `weak reference`, `memory leak`, `autoptr` |
| `reforger-wiki-class-template` | `class template`, `<Class T>`, `Cast helper` |
| `reforger-wiki-preprocessor-directives` | `#define`, `#ifdef`, `#include` |
| `reforger-wiki-preprocessor-macros` | `__FILE__`, `__LINE__`, debug macros |
| `reforger-wiki-script-invoker` | `ScriptInvoker`, `ScriptInvokerVoid`, event subscription |

### Configs & Resources

| Skill | Loads when you mention… |
|---|---|
| `reforger-wiki-base-container` | `BaseContainer`, `GetOwner`, `BaseContainerTools` |
| `reforger-wiki-config-object` | `ConfigObject`, `[BaseContainerProps]`, `[Attribute]`, `uiwidget` |
| `reforger-wiki-config-class` | config asset creation, Workbench config, `.conf` root class |
| `reforger-wiki-scripting-conf` | `UserConfig`, `ResourceName approach`, `SCR_ConfigHelper` |
| `reforger-wiki-prefab-data` | `EntityPrefabData`, `GetPrefabData`, `ComponentData` |
| `reforger-wiki-resource-usage` | `ResourceName`, `Resource.Load`, resource lifetime |
| `reforger-wiki-serialisation` | `JsonSaveContext`, `BinarySaveContext`, `SerializationSave`, `WriteValue` |

### Entity Lifecycle

| Skill | Loads when you mention… |
|---|---|
| `reforger-wiki-entity` | `GenericEntity`, `CreateEntity`, `SpawnEntityPrefab` |
| `reforger-wiki-i-entity` | `IEntity`, `QueryEntitiesBySphere`, `FindEntityByName` |
| `reforger-wiki-entity-lifecycle` | `EOnInit`, `EOnFrame`, `EOnDelete`, `SetEventMask` |
| `reforger-wiki-entity-activeness` | `EOnActivate`, `EOnDeactivate`, `ActiveState` |
| `reforger-wiki-component` | `ScriptComponent`, `GameComponent`, `FindComponent`, `EOnPostInit` |
| `reforger-wiki-event-handlers` | `GetOnPlayerSpawned`, `SCR_BaseGameMode`, `AddEventMask` |
| `reforger-wiki-input-manager` | `InputManager`, `AddActionListener`, `EActionTrigger` |
| `reforger-wiki-action-manager` | `ActionManager`, `AddAction`, `SCR_ActionsManagerComponent` |

### Multiplayer & Networking

| Skill | Loads when you mention… |
|---|---|
| `reforger-wiki-multiplayer` | `RplRpc`, `RpcAsk_`, `RpcDo_`, `BumpMe`, `JIP`, `IsMaster` |
| `reforger-wiki-rpl-component` | `RplComponent`, `RplProp`, `RplIdentity`, ownership transfer |
| `reforger-wiki-json` | `JsonSerializer`, `JsonLoadFile`, `JsonSaveFile` |
| `reforger-wiki-json-api-struct` | `JsonApiStruct`, `SerializeToJson`, `DeserializeFromJson` |
| `reforger-wiki-rest-api` | `RestCallback`, `RestContext`, GET/POST requests |

### Modding & Diagnostics

| Skill | Loads when you mention… |
|---|---|
| `reforger-wiki-damage-system` | `SCR_DamageManagerComponent`, `EDamageType`, `OnDamage` |
| `reforger-wiki-damage-effects` | `DamageEffect`, `SCR_DamageEffect`, `OnEffectAdded`, `HandleConsequences` |
| `reforger-wiki-scripting-modding` | `modded` keyword, mod priority, addon dependency graph |
| `reforger-wiki-scripting-example` | scripting example, `SCR_TW_`, end-to-end implementation |
| `reforger-wiki-temporary-feature` | feature flag, temporary override |
| `reforger-wiki-best-practices` | best practices, null safety, SRP, mod-friendliness |
| `reforger-wiki-dos-donts` | anti-patterns, forbidden patterns, scripting pitfalls |
| `reforger-wiki-performance` | `CallLater`, `GetTickCount`, GC pressure, profiling |
| `reforger-wiki-doxygen` | `@param`, `@return`, `@code`, API documentation |
| `reforger-wiki-workbench-plugin` | `WorkbenchPlugin`, `WorldEditor`, editor menus, undo/redo |

---

## What the hub always enforces

Regardless of which spokes load, the hub applies these rules on every `.c` file:

- **Class naming** — `ARGA_` prefix, PascalCase — e.g. `ARGA_MyComponent`
- **File naming** — `ARGA_MyComponent.c` matches the class name exactly
- **Class body** — closing brace ends with `;`
- **Enum naming** — `ARGA_E` prefix — e.g. `ARGA_EMyState`
- **File location** — `scripts\Game\` or the appropriate subdirectory

---

## Audit status

All 43 spoke skills have been cross-verified against real engine/game source — the local Doxygen dumps (`Doxgen/html_1.7.0.49/`, `Doxgen/html_1.7.0.54/`, which only cover the generic engine layer: `Core`/`GameLib`/`WorkbenchCommon`) and [arexplorer.zeroy.com](https://arexplorer.zeroy.com/) (a live, Doxygen-generated site that additionally covers the Arma Reforger game-specific layer, `scripts/Game`/`GameCode`, which the local dumps do not index). Skills with a `version` above `1.0.0` in their frontmatter received at least one correction from this process.

Full findings and reasoning — including every corrected class name, signature, and callback — are archived in this project's Engram memory under topic keys `reforger/skills-audit-batch1`, `batch2`, `batch3`, and the `reforger/skills-reverify-arexplorer*` / `reforger/skills-audit-final-16` follow-ups.

---

## License

AGPL-3.0
