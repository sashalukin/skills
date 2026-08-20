# Keyboard Shortcuts in Jetpack Compose Desktop

## Description
Use this document when you need to add, redesign, expose, or verify keyboard shortcuts in a Jetpack Compose Desktop or Compose Multiplatform desktop application.

Expected output:

- A short analysis of the app structure, navigation model, primary screens, and major user actions.
- A recommended shortcut scheme limited to common desktop conventions and high-value domain actions.
- A clean implementation with predictable scope, dispatch, labels, and tests or manual verification notes.
- User-visible shortcut discovery through an existing menu, help surface, command palette, or a minimal shortcut help dialog.

## Prerequisites

- Jetpack Compose Desktop or Compose Multiplatform desktop target enabled.
- Access to the desktop entry point, usually in `desktopMain`, `jvmMain`, or a desktop app module.
- Existing app actions or state holders identified before wiring keyboard input.

## Current documentation notes

- Compose Multiplatform desktop supports keyboard handlers in two scopes: focused elements and the current window.
- `Window`, `singleWindowApplication`, and `Dialog` can receive `onPreviewKeyEvent` and `onKeyEvent` callbacks for window-scope handling.
- `onPreviewKeyEvent` runs before child handlers and is preferred when a shortcut must intercept an event.
- `onKeyEvent` runs after child handlers and is preferred when focused children, especially text inputs, should handle the key first.
- Return `true` only for events that your code actually consumes.
- Android `TextField` and `BasicTextField` already provide standard text editing shortcuts for hardware keyboards. Do not replace them from desktop shared code unless the user explicitly requested shared Android behavior.

## Workflow

### Step 1: Discover structure and actions

Before choosing shortcuts, inspect the application:

1. Locate the desktop entry point: search for `fun main`, `application`, `singleWindowApplication`, `Window`, `DialogWindow`, and `MenuBar`.
2. Locate the navigation model: search for navigation state, routes, tabs, panes, command palette, drawers, dialogs, and screen-level state holders.
3. List primary screens and central workflows. Record the actions users perform repeatedly, such as save, search, open, close, create, delete, play/pause, run, refresh, switch panes, or submit.
4. Search for existing menus, toolbar buttons, command palettes, help dialogs, shortcut registries, action IDs, analytics events, or string resources. Reuse these labels and actions where possible.
5. Check for editable regions: search for `TextField`, `OutlinedTextField`, `BasicTextField`, rich text editors, code editors, tables, and embedded browser/Swing/AWT components.

Do not implement shortcuts until this pass identifies both the action and the correct owner for that action.

### Step 2: Recommend the shortcut scheme

Use this policy when proposing shortcuts:

- Prefer standard OS conventions first: Save, Find, New, Open, Close, Undo, Redo, Copy, Cut, Paste, Select All, Delete, Refresh, and Help should match users' platform expectations.
- Use Command/Meta on macOS and Ctrl on Windows/Linux for app command shortcuts. Display labels as `Cmd+S` on macOS and `Ctrl+S` elsewhere.
- Add domain shortcuts only for frequent, central actions that define the app, such as Space for play/pause in a media app or `R` for reply in mail. Keep these easy to remember.
- Avoid obscure shortcuts, overloaded multi-key chords, and shortcuts for rare settings or one-time setup.
- Do not steal standard text editing shortcuts from editable controls. Avoid app-level `Ctrl+A`, `Ctrl+C`, `Ctrl+X`, `Ctrl+V`, `Ctrl+Z`, or `Ctrl+Shift+Z` unless the active component is known not to be editable.
- Avoid OS, window-manager, and browser-reserved shortcuts such as Alt+Tab, Cmd+Tab, Cmd+Space, Ctrl+Alt+Delete, common screenshot shortcuts, and function keys already used by the environment.
- Handle Escape deliberately: close the topmost dialog or popup first, then navigate back if that is the app's established behavior. Do not make Escape exit the whole app unless the product already does that and the user asks for it.
- Keep every shortcut discoverable in menus, help, a command palette, or a shortcut dialog.

Common starting points:

| Action | macOS | Windows/Linux | Notes |
| --- | --- | --- | --- |
| Save | Cmd+S | Ctrl+S | Use only when the app has an explicit save action. |
| Find/Search | Cmd+F | Ctrl+F | Focus the existing search field or open search UI. |
| New item | Cmd+N | Ctrl+N | Use for the primary creation action. |
| Open | Cmd+O | Ctrl+O | Use for open/import flows. |
| Close active document/dialog | Cmd+W or Escape | Ctrl+W or Escape | Match existing close/back behavior. |
| Refresh | Cmd+R | Ctrl+R or F5 | Avoid browser-like reload if embedded web content has focus. |
| Help / shortcuts | F1 and optionally `?` | F1 and optionally `?` | Map to visible shortcut help. |
| Delete selected item | Delete or Backspace | Delete or Backspace | Never trigger while text is being edited. |
| Play/pause | Space | Space | Only when media/canvas focus makes this expected. |

#### Reference category patterns

For app-specific shortcut design, search current official docs or help pages for 2-4 comparable apps in the same category before finalizing the scheme. Cite or note those sources in the implementation summary. Use the patterns as evidence, but do not blindly copy shortcuts that conflict with OS/editor behavior or do not match the current app's workflows.

Use `Cmd/Ctrl` notation in recommendations when the shortcut should use Command on macOS and Ctrl on Windows/Linux.

| Category | Representative apps | Useful shortcut patterns to research | Adaptation guidance |
| --- | --- | --- | --- |
| Productivity & Office | Notion, Notability, Grammarly | Quick switcher or search (`Cmd/Ctrl+K` or `Cmd/Ctrl+P`), in-page find (`Cmd/Ctrl+F`), new item (`Cmd/Ctrl+N`), standard rich-text formatting (`Cmd/Ctrl+B/I/U`), Escape for selection or back behavior. | Keep editing shortcuts native inside text fields and editors. Prefer a command palette for broad workspace actions and reserve unmodified keys for focused editor modes only. |
| Creativity & Design | Lightroom, CapCut, Canva, Figma, FL Studio | Actions menu (`Cmd/Ctrl+K`), shortcut help (`Cmd/Ctrl+/` or `?`), tool or mode keys, comments, zoom-to-fit or zoom-to-selection, playback (`Space`), record (`R`), panel switching, ratings or flags. | Scope tool keys to canvas, timeline, mixer, or design surfaces. Avoid overriding text entry, OS zoom, or browser shortcuts; expose context-dependent shortcuts in visible help. |
| Entertainment & Media | Netflix, Hulu, Spotify | Play/pause (`Space` or Enter), seek with arrow keys, volume with up/down arrows, mute (`M`), fullscreen (`F`), search (`Cmd/Ctrl+K` or `Cmd/Ctrl+F`), shuffle/repeat/queue actions. | Use plain media keys only when media focus is clear. Avoid triggering playback while forms, comments, or search fields are focused. |
| Education & Learning | Canvas, Nearpod, Duolingo | Shortcut popup, Escape to close overlays, rich-content editor help, media controls (`Space`, arrows, `F`, `M`), answer submission or next-step navigation when focus is explicit. | Prioritize accessibility and predictable focus traversal. Make shortcuts discoverable and avoid shortcuts that can interfere with quizzes, assignments, or text answers. |
| Communication & Social | WhatsApp, Discord, LINE, Instagram, Reddit | Quick switcher (`Cmd/Ctrl+K`), channel/chat search (`Cmd/Ctrl+F`), global search (`Cmd/Ctrl+Shift+F`), emoji picker, mute/deafen, unread/read actions, Alt/Option arrow navigation, Escape to cancel or mark read. | Do not intercept composition, markdown, or text-editing shortcuts while the message box is active. Keep navigation and moderation shortcuts scoped to list or conversation focus. |

### Step 3: Decide shortcut scope

Choose the narrowest scope that still matches user expectations:

- **Window/app command:** Use `onPreviewKeyEvent` or `onKeyEvent` parameters on `Window`, `singleWindowApplication`, or `Dialog` for commands that should work anywhere in that window. This does not require `Modifier.focusable()` or `FocusRequester`.
- **Screen command:** Attach a modifier to the screen root when the shortcut is meaningful only on that route, tab, pane, or mode.
- **Component command:** Attach a modifier to the focused component for editor, canvas, list, table, or custom control behavior.
- **Dialog command:** Put Escape, Enter, and dialog-specific shortcuts on the `DialogWindow` or dialog content so they do not leak to the underlying window.

Prefer window-scope handlers for app commands such as Save and Help. Prefer local modifiers for selection, canvas movement, table navigation, or domain tools whose meaning changes by screen.

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
- Keep platform-specific modifier decisions in the desktop source set. If shared code needs to know the primary modifier, pass it in or use `expect`/`actual`.

Example registry and dispatcher:

```kotlin
import androidx.compose.runtime.Immutable
import androidx.compose.ui.input.key.Key
import androidx.compose.ui.input.key.KeyEvent
import androidx.compose.ui.input.key.KeyEventType
import androidx.compose.ui.input.key.isAltPressed
import androidx.compose.ui.input.key.isCtrlPressed
import androidx.compose.ui.input.key.isMetaPressed
import androidx.compose.ui.input.key.isShiftPressed
import androidx.compose.ui.input.key.key
import androidx.compose.ui.input.key.type

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

fun isMacOs(): Boolean =
    System.getProperty("os.name").contains("Mac", ignoreCase = true)

fun AppShortcut.displayText(macOS: Boolean = isMacOs()): String {
    val parts = buildList {
        if (usesPrimaryModifier) add(if (macOS) "Cmd" else "Ctrl")
        if (shift) add("Shift")
        add(keyLabel)
    }
    return parts.joinToString("+")
}

fun KeyEvent.matches(shortcut: AppShortcut, macOS: Boolean = isMacOs()): Boolean {
    if (type != KeyEventType.KeyDown || key != shortcut.key) return false
    if (isAltPressed) return false
    if (isShiftPressed != shortcut.shift) return false

    return if (shortcut.usesPrimaryModifier) {
        if (macOS) isMetaPressed && !isCtrlPressed else isCtrlPressed && !isMetaPressed
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

Dispatch from the desktop window:

```kotlin
import androidx.compose.ui.input.key.KeyEvent
import androidx.compose.ui.window.singleWindowApplication

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

fun main() = singleWindowApplication(
    title = "Example",
    onPreviewKeyEvent = { event ->
        dispatchAppShortcut(
            event = event,
            actions = AppActions(
                save = { /* save document */ },
                find = { /* focus or open search */ },
                showShortcuts = { /* show shortcut help */ },
            )
        )
    }
) {
    /* App content */
}
```

Use `onKeyEvent` instead when the shortcut should defer to focused children:

```kotlin
fun main() = singleWindowApplication(
    title = "Example",
    onKeyEvent = { event ->
        dispatchAppShortcut(
            event = event,
            actions = AppActions(
                save = { /* save document */ },
                find = { /* focus or open search */ },
                showShortcuts = { /* show shortcut help */ },
            )
        )
    }
) {
    /* App content */
}
```

### Step 6: Add screen or component shortcuts

Use local handlers when the shortcut only makes sense in one screen or component. Add focus only when needed.

```kotlin
import androidx.compose.foundation.focusable
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.focus.FocusRequester
import androidx.compose.ui.focus.focusRequester
import androidx.compose.ui.input.key.Key
import androidx.compose.ui.input.key.KeyEventType
import androidx.compose.ui.input.key.isAltPressed
import androidx.compose.ui.input.key.isCtrlPressed
import androidx.compose.ui.input.key.isMetaPressed
import androidx.compose.ui.input.key.isShiftPressed
import androidx.compose.ui.input.key.key
import androidx.compose.ui.input.key.onPreviewKeyEvent
import androidx.compose.ui.input.key.type

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

1. If the app already has a menu bar, command palette, shortcuts panel, or help dialog, add the shortcut there.
2. If the app has no discovery surface, create a minimal keyboard shortcut dialog and map it to F1. Add `?` as an alternate only when it does not interfere with typing in the active UI.
3. Reuse the same `AppShortcut` list for the handler and the visible list.
4. Keep menu item labels and shortcut help labels identical.

Menu example:

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.input.key.Key
import androidx.compose.ui.input.key.KeyShortcut
import androidx.compose.ui.window.FrameWindowScope
import androidx.compose.ui.window.MenuBar

@OptIn(ExperimentalComposeUiApi::class)
@Composable
fun FrameWindowScope.AppMenu(
    macOS: Boolean,
    onSave: () -> Unit,
    onFind: () -> Unit,
    onShowShortcuts: () -> Unit,
) {
    MenuBar {
        Menu("File", mnemonic = 'F') {
            Item(
                text = "Save",
                shortcut = KeyShortcut(Key.S, ctrl = !macOS, meta = macOS),
                onClick = onSave,
            )
        }
        Menu("Edit", mnemonic = 'E') {
            Item(
                text = "Find",
                shortcut = KeyShortcut(Key.F, ctrl = !macOS, meta = macOS),
                onClick = onFind,
            )
        }
        Menu("Help", mnemonic = 'H') {
            Item(
                text = "Keyboard Shortcuts",
                shortcut = KeyShortcut(Key.F1),
                onClick = onShowShortcuts,
            )
        }
    }
}
```

Minimal help dialog example:

```kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.AlertDialog
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp

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

### Step 8: Handle Android shared-code cases

If the implementation touches shared Android/Desktop code:

- Keep the desktop primary modifier policy: Command/Meta on macOS and Ctrl on Windows/Linux.
- Keep Android hardware keyboard guidance separate: use Android's Ctrl-based shortcuts and publish Android shortcuts to the platform Keyboard Shortcuts Helper when appropriate.
- Do not place JVM-only APIs such as `System.getProperty("os.name")` in `commonMain`.
- Prefer `expect`/`actual`, injected platform info, or desktop-only wrappers for platform differences.

### Step 9: Verify behavior

Verification checklist:

- Run the existing build or test command for the changed module when practical.
- Unit-test pure matcher and dispatcher functions where possible: primary modifier behavior, wrong modifier rejection, disabled commands, and event consumption.
- Add Compose UI or integration tests for key events when the project already has Compose desktop tests.
- Manually verify each major shortcut on the target desktop OS.
- Verify normal typing in `TextField`, `BasicTextField`, search boxes, and editors.
- Verify standard text editing shortcuts still work inside editable components.
- Verify Tab and Shift+Tab focus traversal still work.
- Verify Escape closes only the topmost dialog/popup or performs the documented back action.
- Verify shortcut labels in menus/help match the actual behavior on macOS and Windows/Linux.
- Run `git diff --check` before finishing.

## Common problems and solutions

- **Too many shortcuts:** Remove shortcuts for rare actions. Keep only expected OS conventions and central domain actions.
- **Wrong scope:** Move app-wide commands to `Window` or `singleWindowApplication`; move mode-specific commands to the screen or component that owns the mode.
- **Focus hacks on window shortcuts:** Do not add `focusable()` or `FocusRequester` for `Window` or `singleWindowApplication` handlers.
- **Unreceived local key events:** For local modifiers, make the target focusable and request focus only when that target should own keyboard input.
- **Stolen text editing:** Do not intercept standard editing shortcuts with preview handlers above editable controls. Use narrower scope, `onKeyEvent`, or explicit focus-state guards.
- **Propagation bugs:** Return `false` when a command is not available or the event should continue to children or parents.
- **Platform mismatch:** Do not hard-code Ctrl for desktop app commands. Use Meta/Command on macOS and Ctrl on Windows/Linux.
- **Invisible shortcuts:** Add every shortcut to menus, shortcut help, or the command palette in the same change as the handler.
