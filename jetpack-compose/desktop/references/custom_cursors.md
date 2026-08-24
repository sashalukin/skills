# Add pointer cursors in Jetpack Compose

## Description
Use this document when adding or modifying pointer cursor behavior in a Jetpack Compose application targeting Android large screens and desktop environments.

Expected output:

- A short analysis of the app structure, interaction surfaces, and input methods.
- A recommendation for standard cursor types.
- Clean implementation preserving click, drag, text selection, and accessibility.

## Workflow

### Step 1: Find cursor targets

Inspect the relevant UI:

1. Find links, clickable text, icon-only controls, custom editors, canvas tools, draggable items, splitters, and resize handles.
2. Search for `pointerHoverIcon`, `PointerIcon`, `clickable`, `combinedClickable`, `hoverable`, `draggable`, `pointerInput`, and text fields.
3. Find shared design-system components. Change them when the same cursor rule applies to every instance.
4. Note nested regions and states that may change the cursor: disabled, read-only, loading, dragging, selection, and modal modes.

### Step 2: Process each target

For each target:

1. Leave it unchanged if Compose or the system already shows the correct cursor.
2. Choose the simplest cursor that communicates the action:
   - **Default:** Normal content, standard buttons, disabled controls, and non-interactive regions.
   - **Hand:** Links, clickable text, and icon-only clickable controls. Do not use it for every clickable surface.
   - **Text:** Custom editable or selectable text regions. Do not override the correct cursor of standard text fields.
   - **Crosshair:** Drawing, precision targeting, and spatial selection.
   - **Resize:** Splitters, resizable edges, and resize handles. Match the cursor direction to the handle.
   - **Grab or grabbing:** Draggable items when moving is the primary action.
   - **Wait:** A short, temporarily blocked region. Also show visible progress.
3. Keep the real interaction in the appropriate component or modifier, such as `clickable`, `combinedClickable`, or `draggable`. A cursor modifier only changes the pointer icon.
4. Apply `pointerHoverIcon` to the same bounds as the interaction.
5. Use `PointerIcon.Default`, `Hand`, `Text`, or `Crosshair` when possible.
6. Use `android.view.PointerIcon` for resize, grab, grabbing, wait, and other system cursors. Create the icon once with `remember`.
7. Keep `overrideDescendants = false` so nested controls can use their own cursors. Set it to `true` only when the parent owns the entire mode or region.
8. Keep the cursor aligned with enabled, read-only, loading, dragging, selection, and modal state.
9. Put repeated cursor behavior in a shared component or modifier.

Built-in cursor example:

```kotlin
@Composable
fun LinkText(
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

- Using `pointerHoverIcon` as a substitute for `clickable`, `hoverable`, semantics, focus, or visual states.
- Showing `Hand` over disabled controls.
- Setting `overrideDescendants = true` on a broad parent and accidentally suppressing `Text` cursors in text fields.
- Calling `android.view.PointerIcon.getSystemIcon` directly inside recomposition callbacks without `remember`.
- Letting cursor state diverge from actual interaction state during drag, modal, loading, or disabled transitions.

## Reference links

Consult these references before proceeding with the implementation.

- Compose `pointerHoverIcon` API reference: <https://developer.android.com/reference/kotlin/androidx/compose/ui/input/pointer/package-summary#pointerHoverIcon(androidx.compose.ui.Modifier,androidx.compose.ui.input.pointer.PointerIcon,kotlin.Boolean)>
- Compose `PointerIcon` API reference: <https://developer.android.com/reference/kotlin/androidx/compose/ui/input/pointer/PointerIcon>
- Android pointer input overview: <https://developer.android.com/develop/ui/compose/touch-input/pointer-input>
