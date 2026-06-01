---
name: reforger-wiki-event-handlers
description: "Trigger: GetOnPlayerSpawned, SCR_BaseGameMode, event mask, AddEventMask, GetGame. Event handler patterns: ScriptInvoker getters, EventHandlerManagerComponent, and IEntity event list."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "GetOnPlayerSpawned"
    - "SCR_BaseGameMode"
    - "event mask"
    - "AddEventMask"
    - "GetGame"
---

## Activation Contract

Load this skill when code subscribes to or fires game events via `ScriptInvoker` getters (e.g., `GetOnPlayerSpawned`), uses `EventHandlerManagerComponent`, or queries the full list of `IEntity` event methods. Do NOT load for low-level `SetEventMask` usage on a single entity (use `reforger-wiki-entity-lifecycle`); load this skill specifically for handler subscription and game-level events.

## Hard Rules

**ScriptInvoker Event Getters**
- NEVER instantiate a `ScriptInvoker` directly as a member. Always use the getter pattern: the class lazily creates the invoker on first call.
- Using `ScriptInvoker` directly (not `ScriptInvokerBase`-derived typed variants) is BAD PRACTICE — use typed wrappers from `SCR_ScriptInvokerHelper.c`.
- Subscribe with `GetOnXXX().Insert(HandlerMethod)` and unsubscribe with `GetOnXXX().Remove(HandlerMethod)` — always remove when the subscriber is deleted.

**EventHandlerManagerComponent Events**
- The `EventHandlerManagerComponent` groups high-level game events. Access it via:
  `EventHandlerManagerComponent ehmc = EventHandlerManagerComponent.Cast(entity.FindComponent(EventHandlerManagerComponent));`
- Subscribe through the component's ScriptInvoker getters (not `IEntity.SetEventMask`).
- Key events exposed:
  - `OnADSChanged` — character switches to/from ADS
  - `OnInspectionModeChanged` — character enters/exits inspection mode
  - `OnDestroyed` (DamageManagerComponent) — entity destroyed (default hit zone)
  - `OnCompartmentEntering / Entered / Left / Leaving` — compartment (vehicle) events
  - `OnLightStateChanged` — light state toggle
  - `OnTurretReload` — reload started/finished (`isFinished` flag)
  - `OnWeaponChanged / OnMuzzleChanged / OnAmmoCountChanged / OnMagazineChanged` — weapon system
  - `OnProjectileShot / OnFiremodeChanged / OnWeaponAttachmentChanged / OnZeroingChanged` — weapon state
  - `OnDayStart / OnNightStart` — time-of-day events (from `TimeAndWeatherManagerEntity`)
  - `OnGrenadeThrown` — grenade throw event

**IEntity Event Methods (full list)**
- `EOnInit` — entity allocated and initialised
- `EOnVisible` — entity becomes visible
- `EOnFrame` — every simulation frame
- `EOnPostFrame` — after all frame events this frame
- `EOnFixedFrame` — every ~33ms (30 fps fixed)
- `EOnFixedPostFrame` — after fixed frame pass
- `EOnAnimEvent` — animation system event
- `EOnSimulate` — physics sub-step (may run multiple times/frame)
- `EOnPostSimulate` — after simulate pass
- `EOnPhysicsActive` — RigidBody physics activated/deactivated
- `EOnPhysicsMove` — physics engine moved this entity (may run on non-main thread — AVOID cross-entity mutation here)
- `EOnJointBreak` — joint on RigidBody broken
- `EOnTouch` — touched by another entity (requires `TouchComponent`)
- `EOnContact` — contact with another RigidBody registered
- `EOnDiag` — diagnostic frame event (only when Entity Diag is enabled in Diag Menu)
- `EOnUser0-4` — user-defined events (`EntityEvent.EV_USER+0` through `+4`)

**GetGame() Access Pattern**
- `GetGame()` returns the global `BaseGame` instance. Always null-check in early-init code.
- Use `GetGame().GetGameMode()` to obtain `SCR_BaseGameMode` and access its ScriptInvoker getters.

## Key APIs / Patterns

```c
// Subscribing to a game-level ScriptInvoker
class ARGA_MyManager : Managed
{
    void Init()
    {
        // Get game mode and subscribe
        SCR_BaseGameMode gm = SCR_BaseGameMode.Cast(GetGame().GetGameMode());
        if (!gm)
            return;
        gm.GetOnPlayerSpawned().Insert(OnPlayerSpawned);
    }

    void Cleanup()
    {
        SCR_BaseGameMode gm = SCR_BaseGameMode.Cast(GetGame().GetGameMode());
        if (!gm)
            return;
        gm.GetOnPlayerSpawned().Remove(OnPlayerSpawned);
    }

    protected void OnPlayerSpawned(SCR_SpawnRequestComponent requestComponent, IEntity player)
    {
        // Handler logic
    }
}

// EventHandlerManagerComponent subscription
EventHandlerManagerComponent ehmc = EventHandlerManagerComponent.Cast(
    entity.FindComponent(EventHandlerManagerComponent));
if (ehmc)
    ehmc.GetOnDestroyed().Insert(OnEntityDestroyed);

// ScriptInvoker getter lazy-init pattern (inside a class)
protected ref ScriptInvokerVoid m_OnSomethingHappened;

ScriptInvokerVoid GetOnSomethingHappened()
{
    if (!m_OnSomethingHappened)
        m_OnSomethingHappened = new ScriptInvokerVoid();
    return m_OnSomethingHappened;
}

// Firing the event
void TriggerSomething()
{
    if (m_OnSomethingHappened)
        m_OnSomethingHappened.Invoke();
}
```

## References

- PDF: `Event Handlers – Arma Reforger - Bohemia Interactive Community.pdf`
- Wiki: `https://community.bistudio.com/wiki/Arma_Reforger:Event_Handlers`
