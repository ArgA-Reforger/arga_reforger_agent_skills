---
name: reforger-wiki-workbench-plugin
description: "Trigger: WorkbenchPlugin, WorldEditor, editor script, OnMenuOpened, EditorMenu. Creating Workbench editor plugins: menus, tools, and editor-side script utilities."
disable-model-invocation: true
user-invocable: false
license: MIT
metadata:
  author: arga-reforger-team
  version: "1.0.0"
  triggers:
    - "WorkbenchPlugin"
    - "WorldEditor"
    - "editor script"
    - "OnMenuOpened"
    - "EditorMenu"
---

## Activation Contract

Load this skill when writing or reviewing Workbench editor plugin scripts: custom menus, editor-side utilities, `WorkbenchPlugin` subclasses, `WorldEditor` API usage, or any script that runs inside the editor (not the game). Do NOT apply to game-runtime scripts.

## Hard Rules

**Plugin vs game script**
- Workbench plugins run ONLY inside the editor — never at game runtime.
- Plugin scripts live in `scripts/WorkbenchGame/` (game-side editor scripts) or `scripts/Workbench/` (pure editor tools).
- NEVER import or depend on gameplay classes (like `SCR_BaseGameMode`) inside a plugin class — they may not exist in the editor context.
- Use `[WorkbenchPluginAttribute]` to register a class as a Workbench plugin; without it the class is invisible to the editor.

**`WorkbenchPlugin` anatomy**
- Subclass `WorkbenchPlugin` and annotate with `[WorkbenchPluginAttribute(name: "...", shortcut: "...", wbModules: { "WorldEditor" })]`.
- `Run()` is the entry point — called when the user triggers the plugin via menu or shortcut.
- `CanRun()` returns a bool; return `false` to disable the menu item when prerequisites are not met (e.g. no selection, wrong context).
- `GetAttributes()` returns an array of `WorkbenchPluginAttribute` instances that describe each menu entry or shortcut.

**Menu registration**
- Plugins appear in Workbench under the module they declare in `wbModules`.
- `WorldEditor` module: appears in World Editor menus.
- `ResourceManager` module: appears in Resource Manager menus.
- Use `OnMenuOpened(string name)` to initialise plugin state when the menu containing it is opened.

**Editor API access**
- Inside `Run()`, access the World Editor API via `WorldEditor editor = Workbench.GetModule(WorldEditor)`.
- Null-check the result — the module may not be active.
- `editor.GetSelection()` returns the currently selected entities.
- `editor.BeginAction("MyAction")` / `editor.EndAction()` wraps modifications for undo support — ALWAYS wrap world modifications in Begin/EndAction.

**Undo / redo**
- Any modification to world entities via plugin MUST be wrapped in `editor.BeginAction()` / `editor.EndAction()`.
- Skipping Begin/EndAction makes the change non-undoable — a UX bug that affects every user of your plugin.
- For property changes: call `editor.ModifyProperty(entity, propertyName, value)` inside the action.

**Error handling in plugins**
- Use `Print(message, LogLevel.ERROR)` for plugin errors — they appear in the Workbench Output tab.
- Never throw unhandled exceptions in `Run()` — it may crash the editor session.
- Always validate preconditions in `CanRun()` before allowing `Run()` to execute.

**Prefab and resource operations**
- Use `ResourceManager.GetModule(ResourceManager)` to access the resource manager in plugin context.
- `editor.Save()` flushes changes to disk — call explicitly after modifications if autosave is not sufficient.
- Batch modifications: open one `BeginAction`, make all changes, then one `EndAction`. Do NOT nest actions.

## Key APIs / Patterns

```enforce
// Plugin class with attribute registration
[WorkbenchPluginAttribute(
    name: "ARGA_SetupTool",
    shortcut: "",
    description: "ARGA setup utility for batch entity configuration.",
    wbModules: { "WorldEditor" },
    awesomeFontCode: 0xf013  // gear icon
)]
class ARGA_SetupPlugin : WorkbenchPlugin
{
    // Called when the user clicks the menu item or presses the shortcut
    override void Run()
    {
        WorldEditor editor = Workbench.GetModule(WorldEditor);
        if (!editor)
        {
            Print("ARGA_SetupPlugin: WorldEditor not available", LogLevel.ERROR);
            return;
        }

        array<IEntitySource> selection = {};
        editor.GetSelection(selection);
        if (selection.IsEmpty())
        {
            Print("ARGA_SetupPlugin: No entities selected", LogLevel.WARNING);
            return;
        }

        // Wrap all modifications in a single undoable action
        editor.BeginAction("ARGA_SetupPlugin_Configure");
        foreach (IEntitySource src : selection)
        {
            ConfigureEntity(editor, src);
        }
        editor.EndAction();
    }

    // Return false to grey out the menu item when nothing is selected
    override bool CanRun()
    {
        WorldEditor editor = Workbench.GetModule(WorldEditor);
        if (!editor) return false;
        array<IEntitySource> sel = {};
        editor.GetSelection(sel);
        return !sel.IsEmpty();
    }

    protected void ConfigureEntity(WorldEditor editor, IEntitySource src)
    {
        // Modify a property inside the open action
        // editor.ModifyProperty(src, "propertyName", newValue);
    }
}
```

**Access World Editor from script**
```enforce
WorldEditor editor = Workbench.GetModule(WorldEditor);
if (!editor) return;

// Get selected entities
array<IEntitySource> selected = {};
editor.GetSelection(selected);

// Save world changes
editor.Save();
```

## References

- PDF: `Workbench Plugin – Arma Reforger - Bohemia Interactive Community.pdf`
- Wiki: `https://community.bistudio.com/wiki/Arma_Reforger:Workbench_Plugin`
- Related spokes: `reforger-wiki-scripting-first-steps` (Workbench setup), `reforger-wiki-entity` (entity API)
