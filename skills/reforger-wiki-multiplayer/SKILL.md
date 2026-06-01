---
name: reforger-wiki-multiplayer
description: "Trigger: RplRpc, RplChannel, RpcAsk_, RpcDo_, BumpMe, JIP, Replication.IsServer, IsMaster, IsProxy. Multiplayer replication architecture, RPC patterns, and authority/proxy/owner model."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "RplRpc"
    - "RplChannel"
    - "RpcAsk_"
    - "RpcDo_"
    - "BumpMe"
    - "JIP"
    - "Replication.IsServer"
    - "IsMaster"
    - "IsProxy"
---

## Activation Contract

Load this skill when writing or reviewing multiplayer replication code: RPC methods, RplProp synchronisation, authority/proxy/owner logic, BumpMe, JIP, or any `Replication.*` call. Do not activate for single-player-only scripts that never reference replication APIs.

## Hard Rules

**Architecture**
- There is exactly ONE server per session. Clients never communicate directly with each other — all traffic passes through the server.
- In single player the local machine IS the server. Correct multiplayer code must work flawlessly in single player.
- An entity's role (authority / proxy) is set at creation and NEVER changes.
- Only call RPCs from the authority or owner — never attempt to call server-directed RPCs from a plain proxy.
- NEVER call an RPC from `EOnInit`; the RPC system is not ready during initialisation.

**RPC Prefixes (by convention)**
- `RpcAsk_*` — owner asks authority to perform an action (sender = owner, receiver = authority, use `RplRcver.Server`).
- `RpcDo_*` — authority tells proxies to perform an action (sender = authority, receiver = owner or broadcast, use `RplRcver.Owner` / `RplRcver.Broadcast`).

**Channel choice**
- `RplChannel.Reliable` — packet is guaranteed. Use for state-changing events. More expensive.
- `RplChannel.Unreliable` — packet may be lost or superseded. Use for frequent, non-critical updates (e.g. position hints).

**RplProp**
- Decorate with `[RplProp]` only properties that the authority owns and must broadcast.
- After changing a `[RplProp]` property on the authority, call `Replication.BumpMe()` — without it the change is NOT broadcast.
- `onRplName` callback runs on proxies (including JIP proxies) but NEVER on the authority.
- Do NOT call `Replication.BumpMe()` speculatively — only call it when a property was actually modified.

**RplSave / RplLoad**
- Override `RplSave(ScriptBitWriter)` on the authority to write custom streaming data.
- Override `RplLoad(ScriptBitReader)` on the proxy to read it.
- Read order in `RplLoad` MUST exactly match write order in `RplSave` with the same bit counts, otherwise data is corrupted.
- `[RplProp]` properties are automatically handled by streaming — do NOT duplicate them in `RplSave`/`RplLoad`.

**JIP (Join In Progress)**
- `[RplProp]` properties are automatically synchronised to JIP clients via the streaming system.
- Manual `RplSave`/`RplLoad` data is also streamed to JIP clients when the entity becomes relevant.
- Proxy relevancy (stream in / stream out) is engine-controlled; a broadcast RPC sent while the proxy is not streamed will NOT reach that client.

**Streaming out / relevancy**
- Streaming out while the authority exists is CURRENTLY UNDEFINED BEHAVIOUR — avoid triggering it.
- An entity 5 km away from a player may have its proxy deleted automatically. Broadcast RPCs will not reach that machine.

**Replication static helpers**
- `Replication.IsServer()` — returns `true` on the server machine (authority for most entities).
- `Replication.IsClient()` — returns `true` on client machines.
- `Replication.BumpMe()` — signals that one or more `[RplProp]` properties changed and should be broadcast.
- `Replication.FindItemId(item)` — returns `RplId` for a managed item (avoid in tight loops; use a cached variable).
- `Replication.FindItem(rplId)` — returns the managed item for an `RplId`.

## Key APIs / Patterns

```enforce
// RplRpc attribute signature
[RplRpc(RplChannel.Reliable, RplRcver.Server)]
void RpcAsk_DoSomething(int param)
{
    // Runs on authority (server)
    m_iValue = param;
    Replication.BumpMe();
}

[RplRpc(RplChannel.Reliable, RplRcver.Broadcast)]
void RpcDo_NotifyProxies(int param)
{
    // Runs on all proxies (not on authority)
}

// Triggering an RPC
Rpc(RpcAsk_DoSomething, 42);    // called owner-side → runs on server
Rpc(RpcDo_NotifyProxies, 42);   // called authority-side → runs on all proxies

// RplProp with callback
[RplProp(onRplName: "OnValueUpdated")]
protected int m_iValue;

protected void OnValueUpdated()
{
    // Called on proxies (including JIP) when m_iValue is synchronised
    // NOT called on authority
}

// RplSave / RplLoad (custom streaming)
override bool RplSave(ScriptBitWriter writer)
{
    writer.Write(m_iSoldierId, 32);
    writer.Write(m_iHealth, 7);   // 7 bits sufficient for 0..100
    writer.WriteBool(m_bHadLunch);
    return true;
}

override bool RplLoad(ScriptBitReader reader)
{
    if (!reader.Read(m_iSoldierId, 32)) return false;
    if (!reader.Read(m_iHealth, 7))    return false;
    if (!reader.ReadBool(m_bHadLunch)) return false;
    return true;
}
```

**RplRcver permission table**

| Action | Authority | Proxy (non-owner) | Owner proxy |
|---|---|---|---|
| `RplRcver.Server` | runs locally | dropped | runs locally |
| `RplRcver.Owner` | runs locally if authority is owner | dropped | runs locally |
| `RplRcver.Broadcast` | runs on all proxies, NOT locally | dropped (no proxy on other machines) | runs locally |

## References

- PDF: `Multiplayer Scripting – Arma Reforger - Bohemia Interactive Community.pdf`
- Doxygen: `_replication_8c_source.html`, `_rpl_rcver_8c.html`, `_rpl_channel_8c.html`
- Wiki: `https://community.bistudio.com/wiki/Arma_Reforger:Multiplayer_Scripting`
