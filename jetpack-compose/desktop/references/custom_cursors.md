# Add pointer cursors in Jetpack Compose

## Description
Use this document when adding or modifying pointer cursor behavior in a Jetpack Compose application targeting Android large screens and desktop environments.

Expected output:

- A short analysis of the app structure, interaction surfaces, and input methods.
- A recommended system cursor for each target.
- Clean implementation preserving click, drag, text selection, and accessibility.

## Workflow

### Step 1: Find cursor targets

Inspect the relevant UI:

1. Find links, clickable text, icon-only controls, selectable text, custom editors, canvas tools, draggable items, splitters, and resize handles.
2. Search for `pointerHoverIcon`, `PointerIcon`, `clickable`, `combinedClickable`, `hoverable`, `draggable`, `pointerInput`, and `SelectionContainer`.
3. Find shared design-system components that own repeated interaction patterns.
4. Note nested regions and states that may change the cursor: disabled, read-only, loading, dragging, selection modes, and modal modes.

### Step 2: Process each target

For each target:

1. Leave it unchanged if Compose or the system already shows the correct cursor.
2. Choose the simplest cursor that communicates the action:
   - **Default:** Normal content, standard buttons, disabled controls, and non-interactive regions.
   - **Hand:** Links, clickable text, and icon-only clickable controls. Do not use it for every clickable surface.
   - **Text:** Custom editable or selectable text regions. Do not override the correct cursor of standard text fields.
   - **Crosshair:** Precision drawing, targeting, and spatial selection.
   - **Resize:** Splitters, resizable edges, and resize handles. Match the cursor direction to the handle.
   - **Grab or grabbing:** Use `Grab` for movable surfaces and `Grabbing` only during an active drag.
   - **Wait:** A short, temporarily blocked region. Also show visible progress.
3. Keep the real interaction in the appropriate component or modifier, such as `clickable`, `combinedClickable`, or `draggable`. A cursor modifier only changes the pointer icon.
4. Apply `pointerHoverIcon` to the same bounds as the interaction.
5. Use `PointerIcon.Default`, `Hand`, `Text`, or `Crosshair` when possible.
6. Use `PointerIcon(android.view.PointerIcon.TYPE_...)` for resize, grab, grabbing, wait, and other Android system cursors.
7. Leave `overrideDescendants` at its default `false` so nested controls can set their own cursors. Use `true` only when the parent owns the entire mode or region.
8. Keep the cursor aligned with enabled, read-only, loading, dragging, selection, and modal state.
9. Put repeated cursor behavior in a shared component or modifier.

Built-in cursor example:

```kotlin
@Composable
fun TextAction(
    text: String,
    enabled: Boolean,
    onClick: () -> Unit,
) {
    Text(
        text = text,
        modifier = Modifier
            .clickable(
                enabled = enabled,
                role = Role.Button,
                onClick = onClick,
            )
            .pointerHoverIcon(
                if (enabled) PointerIcon.Hand else PointerIcon.Default,
            ),
    )
}
```

## Common pitfalls

- Showing `Hand` over disabled controls.
- Setting `overrideDescendants = true` on a broad parent and accidentally suppressing `Text` cursors in text fields.
- Letting cursor state diverge from actual interaction state during drag, modal, loading, or disabled transitions.
- Applying the cursor to an area that does not match the interaction bounds.


## Reference links

Consult these references before proceeding with the implementation.

- Compose `pointerHoverIcon` API reference: <https://developer.android.com/reference/kotlin/androidx/compose/ui/input/pointer/package-summary#pointerHoverIcon(androidx.compose.ui.Modifier,androidx.compose.ui.input.pointer.PointerIcon,kotlin.Boolean)>
- Compose `PointerIcon` API reference: <https://developer.android.com/reference/kotlin/androidx/compose/ui/input/pointer/PointerIcon>
- Android pointer input overview: <https://developer.android.com/develop/ui/compose/touch-input/pointer-input>
