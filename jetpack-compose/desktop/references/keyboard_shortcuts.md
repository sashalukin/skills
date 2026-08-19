# Keyboard Shortcuts in Jetpack Compose Desktop

## Description
Use this document when you need to add keyboard shortcuts or keyboard event handling to a Jetpack Compose Desktop application.

## Prerequisites
- Jetpack Compose Desktop dependencies setup.
- Desktop target enabled.

## Common Problems and Solutions
- **Unnecessary shortcuts:** You must only generate highly relevant shortcuts that support the core workflows of the application (e.g., standard OS shortcuts and domain-specific actions like 'C' for Cut in a video editor). Do not generate shortcuts for obscure or infrequent actions.
- **Incorrect shortcut scope:** You must determine the exact UI level for the shortcut. Apply global shortcuts to the root `App` composable. Apply screen shortcuts to the screen's root composable. Apply component shortcuts directly to the component.
- **Unreceived key events:** You must apply `Modifier.focusable()` to the element. You must also request focus using a `FocusRequester`.
- **Event propagation issues:** You must use `Modifier.onPreviewKeyEvent` to consume events before children. You must use `Modifier.onKeyEvent` to process events after children.
- **Incorrect event consumption:** You must return `true` if the event is consumed. You must return `false` if the event should continue propagating.
- **Modifier keys:** You must use `isCtrlPressed` for standard modifier combinations on Android hardware keyboards (e.g., Ctrl+S, Ctrl+C).

## Instructions

### Step 1: Evaluate required shortcuts
Determine the necessary shortcuts for the current UI context based on the application's domain.
- **Domain-specific workflows:** You must identify and implement shortcuts for core actions that users perform frequently in this specific type of app (e.g., 'C' for Cut in a video editor, 'Space' for Play/Pause in media, 'R' for Reply in email).
- **Standard OS conventions:** You must also support expected standard shortcuts where applicable:
  - **Confirmation:** Enter
  - **Cancellation / Close:** Escape
  - **Save / Submit:** Ctrl+S
  - **Delete:** Backspace or Delete
  - **Search:** Ctrl+F
  - **Help:** F1 or `?`

### Step 2: Determine scope and focus
Decide where the shortcut must be intercepted.
- **Global:** Attach to the top-level container or `Window`.
- **Screen-level:** Attach to the main container of the specific screen.
- **Component-level:** Attach directly to the specific interactive composable.

You must add `Modifier.focusable()` to the target composable. You must request focus using a `FocusRequester` when the composable enters the screen.

### Step 3: Implement keyboard shortcuts
Apply `Modifier.onPreviewKeyEvent` to intercept events. You must only respond to `KeyEventType.KeyDown`.

```kotlin
import androidx.compose.foundation.focusable
import androidx.compose.foundation.layout.Box
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.focus.FocusRequester
import androidx.compose.ui.focus.focusRequester
import androidx.compose.ui.input.key.*

@Composable
fun ScreenWithShortcuts(onSave: () -> Unit, onClose: () -> Unit) {
    val focusRequester = remember { FocusRequester() }

    Box(
        modifier = Modifier
            .focusRequester(focusRequester)
            .focusable()
            .onPreviewKeyEvent { keyEvent ->
                if (keyEvent.type != KeyEventType.KeyDown) return@onPreviewKeyEvent false
                
                when {
                    // Save: Ctrl+S
                    keyEvent.isCtrlPressed && keyEvent.key == Key.S -> {
                        onSave()
                        true
                    }
                    // Close: Escape
                    keyEvent.key == Key.Escape -> {
                        onClose()
                        true
                    }
                    else -> false
                }
            }
    ) {
        // Screen Content
    }

    LaunchedEffect(Unit) {
        focusRequester.requestFocus()
    }
}
```

### Step 4: Add to Shortcuts Helper
You must expose the implemented shortcuts to the user. 
- Search the codebase for an existing shortcuts helper, help dialog, or menu.
- If a shortcuts helper exists, you must add the new shortcuts to it.
- If no shortcuts helper exists, you must create a simple help dialog listing the active keyboard shortcuts. You must map this dialog to the F1 key.
