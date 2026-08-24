# Keyboard Shortcuts in Jetpack Compose for desktops

## Description
Use this document when you need to add, redesign, or expose keyboard shortcuts in a Jetpack Compose application targeting Android large screens and desktop environments.

Expected output:

- A short analysis of the app structure, navigation model, primary screens, and major user actions.
- A recommended shortcut scheme limited to common desktop conventions and high-value domain actions.
- A clean implementation with predictable scope.
- User-visible shortcut discovery via Keyboard Shortcut Helper.

## Workflow

### Step 1: Discover structure and actions

Inspect the application:

1. Locate the screen root or relevant parent containers.
2. Locate the navigation model.
3. List primary workflows to identify high-value shortcut candidates (e.g., save, search, create, delete, play/pause, submit).
4. Search for existing menus, actions, or string resources to reuse.
5. Check for editable regions (`TextField`, embedded components).

Identify the action and its owner before implementing shortcuts.

### Step 2: Recommend the shortcut scheme

- Prefer standard OS conventions (Save, Find, Undo, etc.).
- Use Ctrl as the primary modifier on Android.
- Add domain shortcuts only for frequent, central actions.
- Avoid obscure shortcuts or overloaded chords.
- Do not override standard text editing shortcuts.
- Avoid OS-reserved shortcuts (e.g., Alt+Tab).
- Handle Escape deliberately (close dialogs before navigating back).

Common starting points:

| Action | Shortcut | Notes |
| --- | --- | --- |
| Save | Ctrl+S | Use when the app has an explicit save action. |
| Find/Search | Ctrl+F | Focus search field. |
| New item | Ctrl+N | Primary creation action. |
| Open | Ctrl+O | Use for open/import flows. |
| Close active document/dialog | Ctrl+W or Escape | Match close/back behavior. |
| Refresh | Ctrl+R or F5 | Avoid browser reload if web content is focused. |
| Delete selected item | Delete or Backspace | Never trigger during text edit. |
| Play/pause | Space | Only when expected by focus. |

#### Reference category patterns

First, identify the category of your application. Then, research official documentation or help pages for 2-4 comparable industry-leading apps in that same category to see what shortcuts they implement. Cite those sources in the implementation summary.

| Category | Useful shortcut patterns to research | Adaptation guidance |
| --- | --- | --- |
| Productivity & Office | Quick switcher or search (`Ctrl+K` or `Ctrl+P`), in-page find (`Ctrl+F`), new item (`Ctrl+N`), standard rich-text formatting (`Ctrl+B/I/U`), Escape for selection or back behavior. | Keep editing shortcuts native inside text fields and editors. Prefer a command palette for broad workspace actions and reserve unmodified keys for focused editor modes only. |
| Creativity & Design | Actions menu (`Ctrl+K`), tool or mode keys, comments, zoom-to-fit or zoom-to-selection, playback (`Space`), record (`R`), panel switching, ratings or flags. | Scope tool keys to canvas, timeline, mixer, or design surfaces. Avoid overriding text entry, OS zoom, or browser shortcuts. |
| Entertainment & Media | Play/pause (`Space`), seek with arrow keys, volume with up/down arrows, mute (`M`), fullscreen (`F`), search (`Ctrl+K` or `Ctrl+F`), shuffle/repeat/queue actions. | Use plain media keys only when media focus is clear. Avoid triggering playback while forms, comments, or search fields are focused. |
| Education & Learning | Escape to close overlays, media controls (`Space`, arrows, `F`, `M`), answer submission or next-step navigation when focus is explicit. | Prioritize accessibility and predictable focus traversal. Avoid shortcuts that can interfere with quizzes, assignments, or text answers. |
| Communication & Social | Quick switcher (`Ctrl+K`), channel/chat search (`Ctrl+F`), global search (`Ctrl+Shift+F`), emoji picker, mute/deafen, unread/read actions, Alt arrow navigation, Escape to cancel or mark read. | Do not intercept composition, markdown, or text-editing shortcuts while the message box is active. Keep navigation and moderation shortcuts scoped to list or conversation focus. |

### Step 3: Decide shortcut scope

Attach your shortcut dispatcher to the correct level of the UI hierarchy:

- **Global commands:** Attach the dispatcher to a focusable screen root so shortcuts work anywhere in the app.
- **Dialog commands:** Attach a dialog-specific dispatcher to the dialog content so background shortcuts don't leak, and dialog shortcuts (like Escape) don't trigger when the dialog is closed.

Prefer root-level handlers for all commands registered in your central registry.

### Step 4: Implement a central shortcut registry

All standard application shortcuts must be defined in a central registry so they can be dispatched consistently and published to the system.

- Define stable command IDs or enum values.
- Define one shortcut list for dispatch and the system helper.
- Keep matcher functions pure for unit testing.

Example registry and dispatcher:

```kotlin
enum class AppCommand {
    Save,
    Find,
}

@Immutable
data class AppShortcut(
    val command: AppCommand,
    val label: String,
    val key: Key,
    val usesPrimaryModifier: Boolean = false,
    val shift: Boolean = false,
)

fun KeyEvent.matches(shortcut: AppShortcut): Boolean {
    if (type != KeyEventType.KeyDown || key != shortcut.key) return false
    if (isAltPressed) return false
    if (isShiftPressed != shortcut.shift) return false

    return if (shortcut.usesPrimaryModifier) {
        isCtrlPressed && !isMetaPressed
    } else {
        !isCtrlPressed && !isMetaPressed
    }
}

val appShortcuts = listOf(
    AppShortcut(AppCommand.Save, "Save", Key.S, usesPrimaryModifier = true),
    AppShortcut(AppCommand.Find, "Find", Key.F, usesPrimaryModifier = true),
)
```

Dispatch from the screen root:

```kotlin
class AppActions(
    val save: () -> Unit,
    val find: () -> Unit,
)

fun dispatchAppShortcut(event: KeyEvent, actions: AppActions): Boolean {
    val command = appShortcuts.firstOrNull { event.matches(it) }?.command ?: return false

    when (command) {
        AppCommand.Save -> actions.save()
        AppCommand.Find -> actions.find()
    }
    return true
}

@Composable
fun MainScreen(
    actions: AppActions,
    modifier: Modifier = Modifier
) {
    val focusRequester = remember { FocusRequester() }
    
    Box(
        modifier = modifier
            .fillMaxSize()
            .focusRequester(focusRequester)
            .focusable()
            // Use onKeyEvent to let children handle events first, or onPreviewKeyEvent to intercept
            .onKeyEvent { event ->
                dispatchAppShortcut(event, actions)
            }
    ) {
        /* App content */
    }
    
    LaunchedEffect(Unit) {
        focusRequester.requestFocus()
    }
}
```

### Step 5: Publish to Keyboard Shortcut Helper

Every implemented shortcut must be visible to users via the system Keyboard Shortcut Helper.

- Override `onProvideKeyboardShortcuts` in the hosting `Activity`.
- Map your internal `AppShortcut` list to Android's `KeyboardShortcutGroup` and `KeyboardShortcutInfo` (Hint: use `shortcut.key.nativeKeyCode` to get the integer keycode).

## Common pitfalls

- Too many shortcuts for rare actions.
- Wrong scope (e.g. app-wide commands on local components).
- Missing `focusable()` and `FocusRequester` on the capturing container.
- Intercepting standard text editing shortcuts (e.g. improperly using `onPreviewKeyEvent` instead of `onKeyEvent`).
- Propagation bugs (returning true when event should continue).
- Undiscoverable shortcuts.

## Reference links

Consult these references before proceeding with the implementation.

- Compose keyboard input guide: https://developer.android.com/develop/ui/compose/touch-input/keyboard-input/commands
- Keyboard Shortcuts Helper guide: https://developer.android.com/develop/ui/compose/touch-input/keyboard-input/keyboard-shortcuts-helper
- Android desktop keyboard interaction guidance: https://developer.android.com/design/ui/desktop/guides/interaction/keyboard