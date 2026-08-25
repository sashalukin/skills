# Add keyboard shortcuts in Jetpack Compose

## Description
Use this document when you need to add, redesign, or expose keyboard shortcuts in a Jetpack Compose application targeting Android large screens and desktop environments.

Expected output:

- A short analysis of the app structure, navigation model, primary screens, and major user actions.
- A recommended shortcut scheme limited to common desktop conventions and high-value domain actions.
- A clean implementation with predictable scope.
- User-visible shortcut discovery via Keyboard Shortcut Helper.

## Workflow

### Step 1: Discover app structure and actions

Inspect the application:

1. Find the screens and containers where shortcuts may be needed, then list the actions users perform often, such as save, search, create, delete, play/pause, or submit.
2. Find existing action handlers, menus, shortcuts, and localized strings to reuse.
3. Find `TextField` and other components that already handle keyboard input.

Identify the action and its owner before implementing shortcuts.

### Step 2: Choose the shortcuts

- Prefer common OS conventions and use Ctrl as the primary modifier on Android.
- Add domain shortcuts only for frequent, central actions.
- Do not override standard text editing shortcuts.
- Use Escape to cancel or dismiss the current temporary UI. Do not use it as general Back navigation.

Common starting points:

| Action | Shortcut | Notes |
| --- | --- | --- |
| Save | Ctrl+S | Use when the app has an explicit save action. |
| Find/Search | Ctrl+F | Focus search field. |
| New item | Ctrl+N | Primary creation action. |
| Open | Ctrl+O | Use for open/import flows. |
| Refresh | Ctrl+R or F5 | Avoid browser reload if web content is focused. |
| Delete selected item | Delete or Backspace | Never trigger during text edit. |
| Play/pause | Space | Only when expected by focus. |

### Step 3: Decide shortcut scope

Attach your shortcut dispatcher to the correct level of the UI hierarchy:

- **Screen-wide commands:** Attach the handler to a common ancestor of the focused screen content.
- **Modal commands:** Attach the handler inside the active dialog, menu, or sheet.
- **Contextual commands:** Attach the handler to the focused editor, canvas, list, or media surface.

### Step 4: Implement a central shortcut registry

Define custom app-level shortcuts in one registry so dispatch and Keyboard Shortcut Helper use the same source of truth.

- Use stable command IDs.
- Store localized labels, keys, and Android modifier masks.
- Keep matching logic pure and reusable.
- Consume a shortcut only when its action is available and executes.

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

- Override `onProvideKeyboardShortcuts` in the hosting `Activity`. Keyboard Shortcut Helper is available on API level 24 and higher.
- Map the registry to `KeyboardShortcutGroup` and `KeyboardShortcutInfo`.
- Resolve shortcut and group labels from string resources.
- Use `shortcut.key.nativeKeyCode` for the key code and `shortcut.modifiers` for the modifier mask. Import `androidx.compose.ui.input.key.nativeKeyCode`.
- Group shortcuts by screen or use case when helpful.

## Common pitfalls

- Adding shortcuts for rare actions.
- Attaching handlers at the wrong scope or outside the active focus path.
- Stealing focus with an invisible root container.
- Intercepting focused-child behavior unintentionally with `onPreviewKeyEvent`.
- Repeating one-time commands on repeated `KeyDown` events.
- Consuming disabled or unhandled commands.
- Publishing Keyboard Shortcut Helper entries that do not match actual dispatch.

## Reference links

Consult these references before proceeding with the implementation.

- Compose keyboard input guide: https://developer.android.com/develop/ui/compose/touch-input/keyboard-input/commands
- Keyboard Shortcuts Helper guide: https://developer.android.com/develop/ui/compose/touch-input/keyboard-input/keyboard-shortcuts-helper
- Android desktop keyboard interaction guidance: https://developer.android.com/design/ui/desktop/guides/interaction/keyboard