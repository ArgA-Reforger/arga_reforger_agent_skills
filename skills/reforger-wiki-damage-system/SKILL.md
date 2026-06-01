---
name: reforger-wiki-damage-system
description: "Trigger: SCR_DamageManagerComponent, DamageType, HitZone, EDamageType, OnDamage. Damage system class hierarchy, logic flow, DamageEffects API, SetHealth caveats, and mod-compatibility patterns."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "SCR_DamageManagerComponent"
    - "DamageType"
    - "HitZone"
    - "EDamageType"
    - "OnDamage"
---

## Activation Contract

Load this skill when writing or reviewing damage handling code: overriding `OnDamage`, working with `HitZone`, applying `EDamageType`, using `SetHealth`, or reasoning about the damage system's class hierarchy and logic flow.

## Hard Rules

**Class hierarchy (engine → script)**
```
HitZoneContainerComponent         // engine-side (extends GameComponent)
DamageManagerComponent            // engine-side
  SCR_DamageManagerComponent      // script-side — PRIMARY extension point
ExtendedDamageManagerComponent    // engine-side
  SCR_ExtendedDamageManagerComponent  // script-side — read its comments and warning before use
    SCR_CharacterDamageManagerComponent  // script-side — character-specific
```
- `SCR_DamageManagerComponent` is the main script-side class. Extend it for custom damage managers.
- `SCR_ExtendedDamageManagerComponent` introduces `DamageEffects` (replaces `damageOverTime`). Read its inline comments carefully before subclassing — there is a documented warning.
- Some API from `DamageManagerComponent` is NOT useful for managers inheriting from `ExtendedDamageManagerComponent`.

**Logic flow (per damage event)**
1. `DamageManagerComponent.OnDamage()` checks: handling enabled, not hijacked, counts as a hit, entity not dead, damage should be dealt.
2. Damage is sent to the `HitZone` → calculates effective damage → replicates hit to clients → deals damage.
3. Hit zone passes damage to parent hit zones.
4. If `DamageState` did not change → exit. If it changed → hit zone fires `OnDamageStateChanged`, then manager fires its own `OnDamageStateChanged`.

**SetHealth — use sparingly**
- `HitZone.SetHealth(value)` changes HP directly without going through the damage logic flow. No instigator, no `OnDamage` path.
- Only `OnHealthSet` and `OnDamageStateChanged` are called when using `SetHealth`.
- Use it only when you need to force a health state change that bypasses normal damage (e.g., force-kill with damage handling disabled).
- `SetHealth` is NOT appropriate as a general-purpose damage applier.

**Preferred way to force-destroy an entity**
- Call `HandleDamage` with a "realistic" but sufficient amount of damage rather than max HP true damage.
- Do NOT use incorrect parameter order — the real signature is `HandleDamage(EDamageType type, float damage, ...)`. Passing `(maxHealth, true)` is wrong and a mod-compatibility anti-pattern: a modded entity (e.g. high-health Terminator mod) will be killed even if it should survive.
- Always think: "could a valid modded entity survive this damage amount?" If no, reconsider the approach.

**DamageEffects (SCR_ExtendedDamageManagerComponent only)**
- `DamageEffects` replaces the legacy `damageOverTime` API.
- Legacy API (`OnDamageOverTimeAdded`, `OnDamageOverTimeRemoved`, `IsDamagedOverTime`, `GetDamageOverTime`, `RemoveDamageOverTime`) maps to new equivalents (`OnDamageEffectAdded`, etc.).
- `SCR_CharacterDamageManagerComponent.IsBleeding()` wraps the persistent effect check for characters.

**EDamageType**
- The damage type enum (`EDamageType`) classifies the source of damage (ballistic, explosion, fire, etc.).
- Always pass the correct `EDamageType` when calling damage-related API — incorrect types may cause wrong gameplay behaviour (suppression, bleeding, armour penetration, etc.).

**Replication**
- The damage system replicates hits to clients so they can calculate damage locally and display effects. Do not duplicate replication for damage events handled by the system.

## Key APIs / Patterns

```enforce
// Override OnDamage in SCR_DamageManagerComponent subclass
override void OnDamage(
    EDamageType type,
    float damage,
    HitZone pHitZone,
    notnull Instigator instigator,
    vector hitPos,
    float speed)
{
    // Always call super to preserve base behaviour
    super.OnDamage(type, damage, pHitZone, instigator, hitPos, speed);

    // Custom logic after base damage is applied
}

// Override OnDamageStateChanged
override void OnDamageStateChanged(EDamageState previousState, EDamageState newState)
{
    super.OnDamageStateChanged(previousState, newState);
    // React to state transition (e.g. play death animation)
}

// Bad: direct SetHealth to kill — bypasses OnDamage, no instigator
// pHitZone.SetHealth(0);  // AVOID unless you specifically need to bypass the flow

// Better: HandleDamage with a realistic amount
// Ensure the amount is "survivable by a modded entity" in principle
GetDamageManager().HandleDamage(EDamageType.TRUE, sufficientDamage, hitZone, instigator);

// Get the damage manager from an entity
SCR_DamageManagerComponent dmgMgr =
    SCR_DamageManagerComponent.Cast(entity.FindComponent(SCR_DamageManagerComponent));
if (!dmgMgr) return;

// DamageEffects API (SCR_ExtendedDamageManagerComponent)
// OnDamageEffectAdded replaces OnDamageOverTimeAdded
// OnDamageEffectRemoved replaces OnDamageOverTimeRemoved
```

## References

- PDF: `Damage System – Arma Reforger - Bohemia Interactive Community.pdf`
- Wiki: `https://community.bistudio.com/wiki/Arma_Reforger:Damage_System`
- Targets Reforger version 1.2.0 (as stated in the official document)
