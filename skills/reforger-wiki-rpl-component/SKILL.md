---
name: reforger-wiki-rpl-component
description: "Trigger: RplComponent, RplProp, Replication.BumpMe, replication component, RplIdentity. RplComponent / BaseRplComponent Doxygen API reference — role queries, ownership transfer, streaming control."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "RplComponent"
    - "RplProp"
    - "Replication.BumpMe"
    - "replication component"
    - "RplIdentity"
---

## Activation Contract

Load this skill when working directly with `RplComponent` or `BaseRplComponent` APIs: querying entity role, transferring ownership, controlling streaming/relevancy, or obtaining `RplId`/`RplIdentity` values. Complements `reforger-wiki-multiplayer` (which covers RPC patterns and `Replication.*` statics).

## Hard Rules

**Class hierarchy**
- `RplComponentClass : BaseRplComponentClass : GenericComponentClass`
- `RplComponent : BaseRplComponent : GenericComponent`
- `RplComponent` is the concrete class placed on entities in the World Editor.
- All meaningful API lives in `BaseRplComponent` — `RplComponent` itself adds nothing.

**Role queries — use these to guard authority-only or owner-only code**
- `IsMaster()` — returns `true` if this machine hosts the authority entity (server for most objects).
- `IsProxy()` — returns `true` if this machine hosts a proxy (not the authoritative instance).
- `IsOwner()` — returns `true` if this machine is the owner (elevated proxy or authority that is also owner).
- `IsOwnerProxy()` — returns `true` if this is an owner proxy specifically (not the authority).
- `IsRemoteProxy()` — returns `true` if this is a proxy that is NOT the owner.

**Identity and IDs**
- `Id()` — returns the `RplId` for this component's entity. Cache the result; `Replication.FindItemId()` uses a table lookup.
- `ChildId(item)` — returns `RplId` for a child item managed within this component.
- `RplIdentity` — represents a connection (machine). Used for ownership transfer.

**Ownership transfer**
- `Give(RplIdentity newOwner)` — transfers ownership of this entity to the specified machine. Call only on the authority.
- Use `Replication.FindOwner(rplId)` to get the current owner's `RplIdentity`.

**Streaming control**
- `EnableSpatialRelevancy(bool)` — enable or disable distance-based relevancy for this node.
- `EnableStreaming(bool)` — enable or disable streaming for this node entirely.
- Spatial relevancy is NOT propagated through entity hierarchy — if a root node is not in the spatial map, child spatial nodes are also excluded.

**Insertion**
- `InsertToReplication(ctx)` — manually insert this component into the replication system.
- `InsertItem(instance)` — insert a managed instance as a child item within this component's replication node.
- `ReleaseFromRpl()` — remove this component from replication. Check `IsReleasedFromRpl()` before using replication API on potentially released components.

**GetNode**
- `GetNode()` — returns the raw `RplNode`. Use for advanced replication-graph operations; prefer the role-query methods for everyday guards.

## Key APIs / Patterns

```enforce
// Get the RplComponent from an entity
RplComponent rpl = RplComponent.Cast(entity.FindComponent(RplComponent));
if (!rpl)
    return; // entity has no replication component

// Guard authority-only code
if (rpl.IsMaster())
{
    // Only runs on the authoritative machine
    m_iValue = newValue;
    Replication.BumpMe();
}

// Guard owner-only code
if (rpl.IsOwner())
{
    Rpc(RpcAsk_RequestChange, newValue);
}

// Transfer ownership to a player's machine
RplIdentity playerIdentity = playerController.GetRplIdentity();
rpl.Give(playerIdentity);

// Get this entity's RplId (cache it — avoid calling in loops)
RplId myId = rpl.Id();
```

## References

- Doxygen: `class_rpl_component_class.html`, `_base_rpl_component_8c_source.html`, `_rpl_identity_8c_source.html`
- Related: `reforger-wiki-multiplayer` (RPC patterns, Replication statics)
