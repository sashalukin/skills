# Keyboard Shortcuts in Android Compose Large Screens

## Description
Use this document when you need to add, redesign, expose, or verify keyboard shortcuts in a Jetpack Compose application targeting Android large screens, tablets, or Android desktop environments.

Expected output:

- A short analysis of the app structure, navigation model, primary screens, and major user actions.
- A recommended shortcut scheme limited to common desktop conventions and high-value domain actions.
- A clean implementation with predictable scope, dispatch, labels, and tests or manual verification notes.
- User-visible shortcut discovery through an existing menu, help surface, command palette, or a minimal shortcut help dialog.

## Prerequisites

- Jetpack Compose dependencies setup.
- Access to the relevant Android screen or component where shortcuts should apply.
- Existing app actions or state holders identified before wiring keyboard input.

## Current documentation notes

- Compose supports keyboard handlers in focused elements and parent containers.
- Any `focusable` container or component can receive `onPreviewKeyEvent` and `onKeyEvent` callbacks.
- `onPreviewKeyEvent` runs before child handlers and is preferred when a shortcut must intercept an event.
- `onKeyEvent` runs after child handlers and is preferred when focused children, especially text inputs, should handle the key first.
- Return `true` only for events that your code actually consumes.
- Android `TextField` and `BasicTextField` already provide standard text editing shortcuts for hardware keyboards. Do not replace them.

## Workflow

### Step 1: Discover structure and actions

Before choosing shortcuts, inspect the application:

1. Locate the screen root or relevant parent containers where shortcuts should apply.
2. Locate the navigation model: search for navigation state, routes, tabs, panes, command palette, drawers, dialogs, and screen-level state holders.
3. List primary screens and central workflows. Record the actions users perform repeatedly, such as save, search, open, close, create, delete, play/pause, run, refresh, switch panes, or submit.
4. Search for existing menus, toolbar buttons, command palettes, help dialogs, shortcut registries, action IDs, analytics events, or string resources. Reuse these labels and actions where possible.
5. Check for editable regions: search for `TextField`, `OutlinedTextField`, `BasicTextField`, rich text editors, code editors, tables, and embedded browser/Swing/AWT components.

Do not implement shortcuts until this pass identifies both the action and the correct owner for that action.

### Step 2: Recommend the shortcut scheme

Use this policy when proposing shortcuts:

- Prefer standard OS conventions first: Save, Find, New, Open, Close, Undo, Redo, Copy, Cut, Paste, Select All, Delete, Refresh, and Help should match users' platform expectations.
- Use Ctrl as the primary modifier for app command shortcuts on Android. Display labels as `Ctrl+S`.
- Add domain shortcuts only for frequent, central actions that define the app, such as Space for play/pause in a media app or `R` for reply in mail. Keep these easy to remember.
- Avoid obscure shortcuts, overloaded multi-key chords, and shortcuts for rare settings or one-time setup.
- Do not steal standard text editing shortcuts from editable controls. Avoid app-level `Ctrl+A`, `Ctrl+C`, `Ctrl+X`, `Ctrl+V`, `Ctrl+Z`, or `Ctrl+Shift+Z` unless the active component is known not to be editable.
- Avoid OS, window-manager, and browser-reserved shortcuts such as Alt+Tab, Alt+Tab, Search+Space, Ctrl+Alt+Delete, common screenshot shortcuts, and function keys already used by the environment.
- Handle Escape deliberately: close the topmost dialog or popup first, then navigate back if that is the app's established behavior. Do not make Escape exit the whole app unless the product already does that and the user asks for it.
- Keep every shortcut discoverable in menus, help, a command palette, or a shortcut dialog.

Common starting points:

| Action | Shortcut | Notes |
| --- | --- | --- |
| Save | Ctrl+S | Use only when the app has an explicit save action. |
| Find/Search | Ctrl+F | Focus the existing search field or open search UI. |
| New item | Ctrl+N | Use for the primary creation action. |
| Open | Ctrl+O | Use for open/import flows. |
| Close active document/dialog | Ctrl+W or Escape | Match existing close/back behavior. |
| Refresh | Ctrl+R or F5 | Avoid browser-like reload if embedded web content has focus. |
| Help / shortcuts | F1 and optionally `?` | Map to visible shortcut help. |
| Delete selected item | Delete or Backspace | Never trigger while text is being edited. |
| Play/pause | Space | Only when media/canvas focus makes this expected. |

#### Reference category patterns

For app-specific shortcut design, search current official docs or help pages for 2-4 comparable apps in the same category before finalizing the scheme. Cite or note those sources in the implementation summary. Use the patterns as evidence, but do not blindly copy shortcuts that conflict with OS/editor behavior or do not match the current app's workflows.

Use `Ctrl` notation in recommendations when the shortcut should use Ctrl.

| Category | Representative apps | Useful shortcut patterns to research | Adaptation guidance |
| --- | --- | --- |
| Productivity & Office | Notion, Notability, Grammarly | Quick switcher or search (`Ctrl+K` or `Ctrl+P`), in-page find (`Ctrl+F`), new item (`Ctrl+N`), standard rich-text formatting (`Ctrl+B/I/U`), Escape for selection or back behavior. | Keep editing shortcuts native inside text fields and editors. Prefer a command palette for broad workspace actions and reserve unmodified keys for focused editor modes only. |
| Creativity & Design | Lightroom, CapCut, Canva, Figma, FL Studio | Actions menu (`Ctrl+K`), shortcut help (`Ctrl+/` or `?`), tool or mode keys, comments, zoom-to-fit or zoom-to-selection, playback (`Space`), record (`R`), panel switching, ratings or flags. | Scope tool keys to canvas, timeline, mixer, or design surfaces. Avoid overriding text entry, OS zoom, or browser shortcuts; expose context-dependent shortcuts in visible help. |
| Entertainment & Media | Netflix, Hulu, Spotify | Play/pause (`Space` or Enter), seek with arrow keys, volume with up/down arrows, mute (`M`), fullscreen (`F`), search (`Cmd/Ctrl+K` or `Cmd/Ctrl+F`), shuffle/repeat/queue actions. | Use plain media keys only when media focus is clear. Avoid triggering playback while forms, comments, or search fields are focused. |
| Education & Learning | Canvas, Nearpod, Duolingo | Shortcut popup, Escape to close overlays, rich-content editor help, media controls (`Space`, arrows, `F`, `M`), answer submission or next-step navigation when focus is explicit. | Prioritize accessibility and predictable focus traversal. Make shortcuts discoverable and avoid shortcuts that can interfere with quizzes, assignments, or text answers. |
| Communication & Social | WhatsApp, Discord, LINE, Instagram, Reddit | Quick switcher (`Ctrl+K`), channel/chat search (`Ctrl+F`), global search (`Ctrl+Shift+F`), emoji picker, mute/deafen, unread/read actions, Alt/Option arrow navigation, Escape to cancel or mark read. | Do not intercept composition, markdown, or text-editing shortcuts while the message box is active. Keep navigation and moderation shortcuts scoped to list or conversation focus. |

### Step 3: Decide shortcut scope

Choose the narrowest scope that still matches user expectations:

- **Screen command:** Attach a modifier to the screen root when the shortcut is meaningful for the entire screen. Ensure the root is focusable.


- **Dialog command:** Put Escape, Enter, and dialog-specific shortcuts on the dialog content so they do not leak to the underlying screen.

Prefer screen-scope handlers for screen commands such as Save and Help. Prefer local modifiers for selection, canvas movement, table navigation, or domain tools whose meaning changes by component.

### Step 4: Choose propagation and focus behavior

- Use `onPreviewKeyEvent` when the shortcut must be intercepted before a child consumes it, such as Save, global search, or closing a modal dialog.
- Use `onKeyEvent` when children should get the first chance, such as shortcuts that might conflict with text editing, list navigation, or embedded components.
- Handle `KeyEventType.KeyDown` for command shortcuts so the action fires when the key is pressed. Guard against repeat events if repeated execution would be harmful.
- Return `true` only after invoking the action. Return `false` for unknown keys, unsupported states, disabled commands, and events a child or parent should handle.
- Use `Modifier.focusable()` and `FocusRequester` only for local component or screen handlers that need to receive focus-based events. Do not add focus modifiers just because a window-level handler exists.
- Keep normal typing, Tab traversal, Space/Enter click behavior, and text selection working unless the focused component intentionally owns a custom interaction.

### Step 5: Implement a central shortcut registry

For anything beyond one or two shortcuts, create a small registry and dispatcher instead of scattering key checks across composables.

Recommended structure:

- Define stable command IDs or enum values.
- Define one shortcut list used by dispatch, menus, and help.
- Keep labels stable and user-facing.
- Keep matcher functions pure where possible so they can be unit-tested.
- 

Example registry and dispatcher:

```kotlin
enum class AppCommand {
    Save,
    Find,
    ShowShortcuts,
}

@Immutable
data class AppShortcut(
    val command: AppCommand,
    val label: String,
    val key: Key,
    val keyLabel: String,
    val usesPrimaryModifier: Boolean = false,
    val shift: Boolean = false,
)

fun AppShortcut.displayText(): String {
    val parts = buildList {
        if (usesPrimaryModifier) add("Ctrl")
        if (shift) add("Shift")
        add(keyLabel)
    }
    return parts.joinToString("+")
}

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
    AppShortcut(AppCommand.Save, "Save", Key.S, "S", usesPrimaryModifier = true),
    AppShortcut(AppCommand.Find, "Find", Key.F, "F", usesPrimaryModifier = true),
    AppShortcut(AppCommand.ShowShortcuts, "Keyboard Shortcuts", Key.F1, "F1"),
)
```

Dispatch from the screen root:

```kotlin
class AppActions(
    val save: () -> Unit,
    val find: () -> Unit,
    val showShortcuts: () -> Unit,
)

fun dispatchAppShortcut(event: KeyEvent, actions: AppActions): Boolean {
    val command = appShortcuts.firstOrNull { event.matches(it) }?.command ?: return false

    when (command) {
        AppCommand.Save -> actions.save()
        AppCommand.Find -> actions.find()
        AppCommand.ShowShortcuts -> actions.showShortcuts()
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
            .onPreviewKeyEvent { event ->
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

Use `onKeyEvent` instead when the shortcut should defer to focused children:

```kotlin
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

### Step 6: Add screen or component shortcuts

Use local handlers when the shortcut only makes sense in one screen or component. Add focus only when needed.

```kotlin
@Composable
fun MediaScreen(
    onTogglePlayback: () -> Unit,
    modifier: Modifier = Modifier,
) {
    val focusRequester = remember { FocusRequester() }

    Box(
        modifier = modifier
            .fillMaxSize()
            .focusRequester(focusRequester)
            .focusable()
            .onPreviewKeyEvent { event ->
                val plainSpace =
                    event.type == KeyEventType.KeyDown &&
                        event.key == Key.Spacebar &&
                        !event.isCtrlPressed &&
                        !event.isMetaPressed &&
                        !event.isAltPressed &&
                        !event.isShiftPressed

                if (plainSpace) {
                    onTogglePlayback()
                    true
                } else {
                    false
                }
            }
    ) {
        /* Media UI */
    }

    LaunchedEffect(Unit) {
        focusRequester.requestFocus()
    }
}
```

### Step 7: Update menus and help

Every implemented shortcut must be visible to users.

1. If the app already has a command palette, shortcuts panel, or help dialog, add the shortcut there.
2. If the app has no discovery surface, create a minimal keyboard shortcut dialog and map it to F1. Add `?` as an alternate only when it does not interfere with typing in the active UI.
3. Reuse the same `AppShortcut` list for the handler and the visible list.
4. Keep menu item labels and shortcut help labels identical.

Minimal help dialog example:

```kotlin
@Composable
fun ShortcutHelpDialog(
    shortcuts: List<AppShortcut>,
    onDismiss: () -> Unit,
) {
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Keyboard Shortcuts") },
        text = {
            Column {
                shortcuts.forEach { shortcut ->
                    Row(Modifier.padding(vertical = 4.dp)) {
                        Text(shortcut.label, Modifier.weight(1f))
                        Text(shortcut.displayText())
                    }
                }
            }
        },
        confirmButton = {
            TextButton(onClick = onDismiss) {
                Text("Close")
            }
        },
    )
}
```

### Step 8: Verify behavior

Verification checklist:

- Run the existing build or test command for the changed module when practical.
- Unit-test pure matcher and dispatcher functions where possible: primary modifier behavior, wrong modifier rejection, disabled commands, and event consumption.
- Add Compose UI or integration tests for key events.
- Manually verify each major shortcut on Android.
- Verify normal typing in `TextField`, `BasicTextField`, search boxes, and editors.
- Verify standard text editing shortcuts still work inside editable components.
- Verify Tab and Shift+Tab focus traversal still work.
- Verify Escape closes only the topmost dialog/popup or performs the documented back action.
- Verify shortcut labels in menus/help match the actual behavior on Android.
- Run `git diff --check` before finishing.

## Common problems and solutions

- **Too many shortcuts:** Remove shortcuts for rare actions. Keep only expected OS conventions and central domain actions.
- **Wrong scope:** Move app-wide commands to the screen root; move mode-specific commands to the screen or component that owns the mode.
- **Focus hacks on window shortcuts:** Always add `focusable()` and `FocusRequester` on the container capturing the events.
- **Unreceived local key events:** For local modifiers, make the target focusable and request focus only when that target should own keyboard input.
- **Stolen text editing:** Do not intercept standard editing shortcuts with preview handlers above editable controls. Use narrower scope, `onKeyEvent`, or explicit focus-state guards.
- **Propagation bugs:** Return `false` when a command is not available or the event should continue to children or parents.
- **Platform mismatch:** Do not hard-code Ctrl for desktop app commands. Use Meta/Ctrl.
- **Invisible shortcuts:** Add every shortcut to menus, shortcut help, or the command palette in the same change as the handler.
