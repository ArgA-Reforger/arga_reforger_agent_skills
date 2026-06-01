---
name: reforger-skill
description: "Trigger: *.c, Enforce, Enfusion. High-performance, memory-safe, network-synchronized Enforce scripting for Arma Reforger."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "*.c"
    - "Enforce"
    - "Enfusion"
---

## Activation Contract

Load this skill when editing or generating `.c` files inside an Enforce/Reforger project. Do not apply to `.json`, `.txt`, or non-Enforce files.

## Hard Rules

**ARC Memory Safety**
- Back-references (child → parent) MUST be weak: never use `ref` on them.
- Local variables are already strong; NEVER add `ref` to locals.
- Null-check ALL weak references before use — they can become `null` at any time.

**Class & File Naming**
- Classes: `ARGA_` prefix, PascalCase — e.g. `ARGA_MyComponent`.
- Files: `arga_` prefix, snake_case, `.c` extension — e.g. `arga_my_component.c`.
- Class body closing brace MUST end with `;`.

**RPC & Replication**
- Client → Server: method prefix `RpcAsk_`, decorator `[RplRpc(RplChannel.Reliable, RplRcver.Server)]`.
- Server → Clients: method prefix `RpcDo_`, decorator `[RplRpc(RplChannel.Reliable, RplRcver.Broadcast)]`.
- Invoke via `Rpc(Rpc_MethodName, args...)`.
- Replicated properties: `[RplProp()]` attribute + call `Replication.BumpMe()` after every mutation.

**Dedicated Server Safety**
- NEVER use `#ifdef SERVER` compile-time guards — Enforce does not support server-only compilation.
- Use runtime role checks: `Replication.IsServer()`, `IsMaster()`, `IsProxy()`.
- Client-only components (`BaseSoundComponent`, `CameraHandlerComponent`) are not instantiated on dedicated server; always null-check before use.

**Loop Performance**
- Declare loop pointers OUTSIDE the loop scope to avoid ARC release overhead per iteration.
- Pre-cache collection size: `for (int i, count = list.Count(); i < count; i++)`.
- Use indexed foreach when index is needed: `foreach (int i, ARGA_Obj obj : list)`.

## Decision Gates

| Need | Base Class |
|------|------------|
| Logic bound to a World Entity; event callbacks (`EOnInit`, `EOnFrame`); frame ticks | `ScriptComponent` |
| Native network synchronization with Replication engine integration | `GameComponent` |
| Read-only config, prefab attributes, no ticking, Workbench-exposed data | `BaseContainer` |

## Execution Steps

1. Confirm the file is `.c` and the project targets Enforce/Reforger.
2. Apply naming: `ARGA_` class prefix, `arga_` file prefix; close class body with `;`.
3. Select base class from the Decision Gate above.
4. Enforce ARC: weak back-references (no `ref`), no `ref` on locals.
5. Register entity events in constructor: `SetEventMask(EntityEvent.INIT | EntityEvent.FRAME)`.
6. For networked state: `[RplProp()]` + `Replication.BumpMe()` on every write.
7. For RPCs: apply `RpcAsk_` / `RpcDo_` prefixes and matching `[RplRpc()]` decorators.
8. Guard all client-only component accesses with null checks.

## References

- `scripts\` — Reforger scripts source of truth
- `https://community.bistudio.com/wiki/Category:Arma_Reforger/Modding` — Bohemia Modding
- `https://community.bistudio.com/wikidata/external-data/arma-reforger/ArmaReforgerScriptAPIPublic/` — Arma Reforger Script API
- `https://community.bistudio.com/wikidata/external-data/arma-reforger/EnfusionScriptAPIPublic/` — 
Enfusion Script API

