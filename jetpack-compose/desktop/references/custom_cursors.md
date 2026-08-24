# Custom Cursors in Jetpack Compose for desktops

## Description
Use this document when adding or modifying mouse cursor behavior in Jetpack Compose for Android desktop environment.

Expected output:

- A short analysis of the app structure, interaction surfaces, and input methods.
- A recommendation for standard cursor types.
- Clean implementation preserving click, drag, text selection, and accessibility.

## Workflow

### Step 1: Discover the cursor surface

Inspect the relevant UI:

1. Locate the target composables: toolbar buttons, links, text editors, canvas surfaces, draggable items, splitters, resize handles, and disabled controls.
2. Search for existing cursor helpers or wrappers: `pointerHoverIcon`, `PointerIcon`, `hoverable`, `pointerInput`, `clickable`, `combinedClickable`, `draggable`, and `TextField`.
3. Identify existing design-system wrappers (buttons, links, drag handles, etc.) and add cursor behavior there instead of scattering raw modifiers.
4. Identify nested interactive regions which might need different cursor precedence.
5. Confirm behavior for disabled, read-only, loading, dragging, selection, and modal states.

Understand the pointer region and descendant interaction before adding cursor code.

### Step 2: Choose the cursor type

Use the simplest cursor that communicates the action:

- **Default:** Normal content, disabled controls, non-interactive regions, and standard buttons.
- **Hand:** Links, clickable text, and icon-only clickable controls. Do not use for every hoverable surface.
- **Text:** Editable or selectable text regions and custom text editors.
- **Crosshair:** Precision canvas targeting, drawing, or region selection.
- **Resize cursors:** Split panes, column edges, and resizable shapes. Use Android system cursors when needed.
- **Wait cursor:** Short blocking operations. Prefer visible progress indicators.

Avoid novelty cursors. Stick to standard OS cursor conventions.

### Step 3: Implement built-in pointer icons

Apply `pointerHoverIcon` to the interactive region.

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
            .pointerHoverIcon(if (enabled) PointerIcon.Hand else PointerIcon.Default),
    )
}
```

Implementation notes:

- Keep cursor state aligned with enabled state. Disabled controls normally keep `Default`.
- Prefer `clickable` or `combinedClickable` since they include semantics and accessibility.
- Use wrappers for repeated components.
- Verify modifier ordering. The hover region should match the visual affordance.

### Step 4: Handle nested cursor precedence

Choose precedence carefully:

- Default to `overrideDescendants = false` so children keep their own cursors.
- Set `overrideDescendants = true` only for modal/mode-owned regions (e.g. drawing mode, disabled overlay) to force a single cursor across children.
- Avoid `overrideDescendants = true` on broad app roots.
- Audit popups, menus, and overlays after changing precedence.

Example parent-owned drawing cursor:

```kotlin
@Composable
fun DrawingSurface(
    drawingMode: Boolean,
    content: @Composable () -> Unit,
) {
    Box(
        modifier = Modifier.pointerHoverIcon(
            icon = if (drawingMode) PointerIcon.Crosshair else PointerIcon.Default,
            overrideDescendants = drawingMode,
        ),
    ) {
        content()
    }
}
```

### Step 5: Use Android system cursors

Wrap `android.view.PointerIcon` in a Compose `PointerIcon` for system cursors not in built-ins.

```kotlin
fun Modifier.horizontalResizeCursor(): Modifier = composed {
    val context = LocalContext.current
    val icon = remember(context) {
        PointerIcon(android.view.PointerIcon.getSystemIcon(context, android.view.PointerIcon.TYPE_HORIZONTAL_DOUBLE_ARROW))
    }
    pointerHoverIcon(icon)
}
```

Common Android system predefined cursors (`android.view.PointerIcon`):

- `TYPE_ARROW`
- `TYPE_CROSSHAIR`
- `TYPE_TEXT`
- `TYPE_WAIT`
- `TYPE_HAND`
- `TYPE_HORIZONTAL_DOUBLE_ARROW`
- `TYPE_VERTICAL_DOUBLE_ARROW`
- `TYPE_TOP_RIGHT_DIAGONAL_DOUBLE_ARROW`
- `TYPE_TOP_LEFT_DIAGONAL_DOUBLE_ARROW`

Use `android.view.PointerIcon.getSystemIcon(context, type)` to fetch the correct Android system cursor.

### Step 6: Coordinate with interactions

Match real behavior:

- `clickable`: Use `Hand` only when enabled. Preserve semantics.
- `hoverable`: Use for visual hover styling, not just cursor changes.
- `draggable`: Show resize/move cursors on handles. Consider `overrideDescendants = true` during drags.
- Text fields: Preserve `Text` cursor over editable text. Avoid overriding it from parents.
- Selection regions: Use `Text` for text selection and `Crosshair` for spatial selection.
- Split panes: Use directional resize cursors on the handle, not the whole pane.
- Disabled controls: Keep `Default`. Do not use `Wait` to mean disabled.
- Loading states: Prefer progress indicators. Use `Wait` only for short blocked areas.
- Popups and menus: Verify cursor resets when entering/leaving popups.

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
