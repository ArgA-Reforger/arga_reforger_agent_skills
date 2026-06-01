---
name: reforger-wiki-damage-effects
description: "Trigger: SCR_DamageEffectComponent, damage particle, damage sound, OnDamageStateChanged, DamageEffect. Visual and audio damage effects tied to entity damage states."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "SCR_DamageEffectComponent"
    - "damage particle"
    - "damage sound"
    - "OnDamageStateChanged"
    - "DamageEffect"
---

## Activation Contract

Load this skill when implementing or reviewing visual and audio feedback for entity damage states: particle effects on damage, sound triggers on state change, `SCR_DamageEffectComponent` configuration, or any `DamageEffect` subclass. Do not activate for gameplay damage logic — use `reforger-wiki-damage-system` for that.

## Hard Rules

**Separation of concerns**
- `SCR_DamageEffectComponent` handles VISUAL and AUDIO feedback only — it does NOT apply damage.
- Damage logic lives in `SCR_DamageManagerComponent` (see `reforger-wiki-damage-system`). Never mix the two.
- Effects are data-driven: define them in prefab attributes, not in code.

**DamageEffect class hierarchy**
```
DamageEffect                        // base class — do not subclass directly without reading docs
  SCR_DamageEffect                  // script-side base — extend this for custom effects
    SCR_ParticleDamageEffect        // particle-based visual effect on damage
    SCR_SoundDamageEffect           // sound played on damage state change
    SCR_LightDamageEffect           // dynamic light on damaged entities (e.g. fire glow)
```

**State-driven activation**
- Effects are bound to `EDamageState` transitions: `UNDAMAGED`, `DAMAGED`, `DESTROYED`.
- Each `DamageEffect` entry declares which `EDamageState` activates it.
- `OnDamageStateChanged` on the component fires when the state transitions — effects react to this callback, NOT to individual hit events.
- Do NOT trigger particles or sounds directly in `OnDamage`; use the state machine.

**Particle effect rules**
- `SCR_ParticleDamageEffect` requires a valid particle `ResourceName` (`m_ParticleEffect`).
- The particle is spawned at the entity origin by default; override `m_vOffset` to shift it relative to the entity.
- Particle lifetime is managed by the effect — do NOT manually delete the particle object.
- Use `m_bLoop` for continuous effects (smoke, fire); leave it false for one-shot hit flashes.

**Sound effect rules**
- `SCR_SoundDamageEffect` plays a sound signal via `SoundComponent` on the entity.
- `m_sSoundEvent` must match an exact FMOD event path configured for the entity.
- Sounds respect the entity's audio context — no direct `SoundSystem` calls inside `SCR_SoundDamageEffect`.

**Replication**
- `SCR_DamageEffectComponent` listens for `OnDamageStateChanged` which fires both on authority and on proxies after replication.
- Effects spawned client-side (particles, sounds) are NOT replicated — each machine spawns its own local effect.
- Never rely on an effect being present on the authority machine to infer client state.

**Prefab workflow**
1. Add `SCR_DamageEffectComponent` to the entity prefab in World Editor.
2. In the component attributes, configure the `Effects` array.
3. Each array element is a `DamageEffect` subtype with its specific fields.
4. The engine calls `OnDamageStateChanged` and the component iterates its effects array automatically.

## Key APIs / Patterns

```enforce
// Listening to damage state changes from outside the component
SCR_DamageEffectComponent effectComp =
    SCR_DamageEffectComponent.Cast(entity.FindComponent(SCR_DamageEffectComponent));
if (!effectComp) return;

// The component self-registers with the damage manager; no manual subscription needed.
// To add a runtime effect:
SCR_ParticleDamageEffect fx = new SCR_ParticleDamageEffect();
fx.m_ParticleEffect = particleResourceName;
fx.m_eActivationState = EDamageState.DESTROYED;
effectComp.AddEffect(fx);

// Checking current damage state (from damage manager, not effect component)
SCR_DamageManagerComponent dmgMgr =
    SCR_DamageManagerComponent.Cast(entity.FindComponent(SCR_DamageManagerComponent));
if (!dmgMgr) return;
EDamageState state = dmgMgr.GetState();

// Override example in a custom SCR_DamageEffect subclass
class ARGA_MyCustomDamageEffect : SCR_DamageEffect
{
    override void OnActivate(IEntity owner, EDamageState state)
    {
        super.OnActivate(owner, state);
        // Custom effect startup
    }

    override void OnDeactivate(IEntity owner)
    {
        super.OnDeactivate(owner);
        // Cleanup custom effect
    }
}
```

## References

- PDF: `Damage Effects – Arma Reforger - Bohemia Interactive Community.pdf`
- Wiki: `https://community.bistudio.com/wiki/Arma_Reforger:Damage_Effects`
- Related spoke: `reforger-wiki-damage-system` (damage logic, class hierarchy, SetHealth caveats)
