# arga_reforger_skill
Help for writing scripts with LLMs

---

## reforger-skill — Enforce Scripting Developer Skill

A developer skill that guides LLMs to generate high-performance, memory-safe, and network-synchronized [Enforce script](https://community.bistudio.com/wiki/Arma_Reforger:Scripting_First_Steps) code for Arma Reforger, preventing common C# and Unity hallucinations.

### How to use

This skill loads automatically when your agent edits or reviews `.c` files inside an Enforce/Reforger project. No manual activation is needed.

**Triggers:** `*.c` files · `Enforce` · `Enfusion`

**What it enforces:**
- ARC memory safety — weak back-references, no `ref` on locals
- Class and file naming — `ARGA_` prefix (PascalCase) / `arga_` prefix (snake_case `.c`)
- RPC conventions — `RpcAsk_` for client→server, `RpcDo_` for server→clients
- Dedicated server safety — runtime role checks only, null-checks on client-only components
- Component selection — decision table for `ScriptComponent`, `GameComponent`, `BaseContainer`
- Loop performance — pre-declared pointers, cached collection sizes

**Skill file:** `skills/reforger-skill/SKILL.md`

---

## reforger-skill — Skill de scripting Enforce

Un developer skill que guía a los LLMs para generar código [Enforce script](https://community.bistudio.com/wiki/Arma_Reforger:Scripting_First_Steps) de alto rendimiento, seguro en memoria y sincronizado en red para Arma Reforger, previniendo alucinaciones típicas de C# y Unity.

### Cómo usarlo

El skill se carga automáticamente cuando el agente edita o revisa archivos `.c` dentro de un proyecto Enforce/Reforger. No requiere activación manual.

**Disparadores:** archivos `*.c` · `Enforce` · `Enfusion`

**Qué aplica:**
- Seguridad de memoria ARC — referencias débiles para back-references, sin `ref` en variables locales
- Convenciones de nombre — prefijo `ARGA_` (PascalCase) / prefijo `arga_` (snake_case `.c`)
- Convenciones de RPC — `RpcAsk_` para cliente→servidor, `RpcDo_` para servidor→clientes
- Seguridad en servidor dedicado — solo checks de rol en runtime, null-checks en componentes client-only
- Selección de componentes — tabla de decisión para `ScriptComponent`, `GameComponent`, `BaseContainer`
- Rendimiento en loops — punteros pre-declarados, tamaños de colección cacheados

**Archivo del skill:** `skills/reforger-skill/SKILL.md`
