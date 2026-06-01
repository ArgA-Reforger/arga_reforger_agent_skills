---
name: reforger-wiki-script-invoker
description: "Trigger: ScriptInvoker, ScriptInvokerVoid, ScriptInvokerBase, event handler, event subscription. Event/callback system using ScriptInvoker in Enforce Script."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "ScriptInvoker"
    - "ScriptInvokerVoid"
    - "ScriptInvokerBase"
    - "event handler"
    - "event subscription"
    - "GetOnBark"
---

## Activation Contract

Load this skill when the context involves event systems, callbacks, or observer patterns using `ScriptInvoker`, `ScriptInvokerVoid`, or `ScriptInvokerBase<func>` in Enforce Script. Do NOT load for generic `.c` edits without event/invoker keywords.

## Hard Rules

**Core Rule — Never use ScriptInvoker directly**
- Using the raw `ScriptInvoker` class is considered BAD PRACTICE — always use the typed variants (`ScriptInvokerVoid`, `ScriptInvokerBase<func>`) or a `typedef`-ed type alias.

**Ownership and Lazy Initialization**
- The event owner declares the invoker as a `protected ref` member and initializes it lazily inside a getter method.
- The getter returns the invoker, creating it if null: `if (!m_OnBark) m_OnBark = new ScriptInvokerVoid();`
- Subscribers NEVER access the invoker field directly — always through the owner's getter.

**Invoker Types**
- `ScriptInvokerVoid` — for events with no arguments; subscribed methods must have signature `void MyMethod()`.
- `ScriptInvokerBase<func>` — for events with arguments. Declare the function signature via `typedef` first.
- Custom typed invokers using `typedef` are the preferred pattern for non-void events.

**Subscribe / Unsubscribe**
- Subscribe: `owner.GetOnEvent().Insert(OnEvent);` — registers a method reference.
- Unsubscribe: `owner.GetOnEvent().Remove(OnEvent);` — MUST be called in the subscriber's destructor to prevent dangling callbacks.
- Failing to unsubscribe causes the invoker to call a method on a destroyed object → VM crash.

**Firing an Event**
- Check if the invoker exists before calling `Invoke()`: `if (m_OnBark) m_OnBark.Invoke();`
- The lazy-init getter pattern means an owner with zero subscribers never allocates the invoker object.

**Signature Matching**
- Subscriber methods MUST match the invoker's declared signature exactly (same parameter types, same count).
- Mismatched signatures are a runtime error — the Script Editor may not catch them at compile time.

**Back-References from Subscriber**
- Subscriber holds a back-reference to the event source as a WEAK reference (no `ref`) to avoid cyclic ARC leaks.

## Key APIs / Patterns

```c
// --- Event Owner (void event, no args) ---
class TAG_Dog
{
    protected ref ScriptInvokerVoid m_OnBark;  // lazy-initialized

    ScriptInvokerVoid GetOnBark()
    {
        if (!m_OnBark)
            m_OnBark = new ScriptInvokerVoid();
        return m_OnBark;
    }

    void Bark()
    {
        if (m_OnBark)          // only invoke if invoker exists (subscribers registered)
            m_OnBark.Invoke();
        Print("Dog barked", LogLevel.DEBUG);
    }
}

// --- Subscriber ---
class TAG_Neighbour
{
    protected TAG_Dog m_Dog;  // WEAK reference — no ref keyword

    void TAG_Neighbour(notnull TAG_Dog dog)
    {
        m_Dog = dog;
        m_Dog.GetOnBark().Insert(OnDogBark);  // subscribe
    }

    void ~TAG_Neighbour()
    {
        if (m_Dog)
            m_Dog.GetOnBark().Remove(OnDogBark);  // MUST unsubscribe in destructor
    }

    protected void OnDogBark()
    {
        Print("The dog barked!", LogLevel.NORMAL);
    }
}

// --- Event with typed argument (int) ---
void ScriptInvokerIntMethod(int i);
typedef func ScriptInvokerIntMethod;
typedef ScriptInvokerBase<ScriptInvokerIntMethod> ScriptInvokerInt;

class TAG_EventHolder
{
    protected ref ScriptInvokerInt m_OnScoreChanged;

    ScriptInvokerInt GetOnScoreChanged()
    {
        if (!m_OnScoreChanged)
            m_OnScoreChanged = new ScriptInvokerInt();
        return m_OnScoreChanged;
    }

    void AddScore(int points)
    {
        m_iScore += points;
        if (m_OnScoreChanged)
            m_OnScoreChanged.Invoke(m_iScore);  // pass arg to all subscribers
    }

    protected int m_iScore;
}

// Subscriber method MUST match signature: void WarnOfScoreChange(int newScore)
```

## References

- PDF: `ScriptInvoker Usage – Arma Reforger - Bohemia Interactive Community.pdf`
- Doxygen: `scripts/GameLib/tools.c` (ScriptInvokerBase at line 117, ScriptInvokerVoid at line 131)
- Wiki: `https://community.bistudio.com/wiki/Arma_Reforger:ScriptInvoker_Usage`
