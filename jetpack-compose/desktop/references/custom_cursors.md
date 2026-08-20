# Custom Cursors in Jetpack Compose Desktop

## Description
Use this document when you need to add, redesign, audit, or verify mouse cursor and pointer hover icon behavior in a Jetpack Compose application targeting Android large screens, tablets with keyboards/mice, or Android desktop environments (e.g. ChromeOS, Samsung DeX).

Expected output:

- A short analysis of the app structure, interaction surfaces, and input methods.
- A recommendation for built-in `PointerIcon` values or native Android `android.view.PointerIcon` values.
- Clean implementation that preserves click, drag, text selection, accessibility, and existing design-system behavior.
- Verification notes for hover regions, nested cursor precedence, and enabled/disabled state.

## Prerequisites

- Jetpack Compose dependencies setup.
- Compose UI dependency that provides `androidx.compose.ui.input.pointer.pointerHoverIcon` and `PointerIcon`.

## Current documentation notes

- `Modifier.pointerHoverIcon(icon, overrideDescendants = false)` sets the pointer icon while the cursor is hovered over that modifier's input region.
- `overrideDescendants = false` is the default, so child composables can set their own pointer icons.
- `overrideDescendants = true` makes the parent's icon win for all descendants under that parent.
- Compose built-ins cover the most common cases: `PointerIcon.Default`, `PointerIcon.Hand`, `PointerIcon.Text`, and `PointerIcon.Crosshair`.
- For system cursors not exposed as Compose built-ins (such as resize or wait cursors), you can wrap an `android.view.PointerIcon` in a Compose `PointerIcon`.
- Pointer cursor changes are affordances only. They must not be the only indication that an action is available, disabled, draggable, selected, or destructive.

## Workflow

### Step 1: Discover the cursor surface

Before editing, inspect the relevant UI:

1. Locate the target composables, toolbar buttons, links, text editors, canvas surfaces, draggable items, splitters, resize handles, timeline controls, maps, charts, and disabled controls.
2. Search for existing cursor helpers or wrappers: `pointerHoverIcon`, `PointerIcon`, `hoverable`, `collectIsHoveredAsState`, `pointerInput`, `clickable`, `combinedClickable`, `draggable`, and `TextField`.
3. Identify whether the app already has design-system wrappers for buttons, links, drag handles, toolbars, text fields, or canvas tools. Prefer adding cursor behavior there instead of scattering raw modifiers.
4. Identify nested interactive regions. Parent canvas modes, child buttons, text fields, popup menus, and resize handles often need different cursor precedence.
5. Confirm the desired behavior for disabled, read-only, loading, dragging, selection, and modal states.

Do not add custom cursor code until you know the pointer region and how the cursor should interact with descendants.

### Step 2: Choose the cursor type

Use the simplest cursor that communicates the action:

- **Default:** Normal content, disabled controls, non-interactive regions, and buttons whose existing component already supplies the expected cursor behavior.
- **Hand:** Links, clickable text, icon-only clickable controls, and canvas objects that act like buttons. Do not use `Hand` for every hoverable surface.
- **Text:** Editable or selectable text regions, custom text editors, code editors, text selection overlays, and text insertion handles.
- **Crosshair:** Precision canvas targeting, drawing, crop, measurement, color picking, or region selection.
- **Resize cursors:** Split panes, column edges, crop handles, resizable shapes, or window-like handles. Use Android system cursors when Compose built-ins are not enough.
- **Wait cursor:** Short blocking operations only when the UI cannot respond. Prefer visible progress indicators and keep the cursor state scoped to the blocked region.

Avoid novelty cursors for standard app controls. Users rely on standard cursor conventions to predict click, text, drag, and resize behavior.

### Step 3: Implement built-in pointer icons

Apply `pointerHoverIcon` to the same interactive region the user can click, drag, resize, or select.

```kotlin
import androidx.compose.foundation.clickable
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon
import androidx.compose.ui.semantics.Role

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
            .then(
                if (enabled) {
                    Modifier.pointerHoverIcon(PointerIcon.Hand)
                } else {
                    Modifier
                }
            ),
    )
}
```

Implementation notes:

- Keep cursor state aligned with enabled state. Disabled controls normally keep `Default`.
- Use `clickable` or `combinedClickable` for click behavior when possible because they include semantics, focus, and keyboard activation. Cursor modifiers do not add accessibility behavior.
- Prefer a wrapper when a repeated component always needs the same cursor.
- Verify modifier ordering if layout modifiers change the pointer region. The hover region should match the visual and clickable affordance, not accidental padding outside it.

### Step 4: Implement conditional cursor state

Use state to choose the cursor when the active tool, mode, enabled state, hover target, or drag state changes. Keep the icon calculation cheap and stable.

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon

enum class CanvasTool {
    Select,
    Draw,
    Pan,
    Disabled,
}

@Composable
fun Modifier.canvasToolCursor(
    tool: CanvasTool,
    canInteract: Boolean,
): Modifier {
    val icon = remember(tool, canInteract) {
        when {
            !canInteract -> PointerIcon.Default
            tool == CanvasTool.Draw -> PointerIcon.Crosshair
            tool == CanvasTool.Pan -> PointerIcon.Hand
            else -> PointerIcon.Default
        }
    }

    return this.pointerHoverIcon(icon)
}
```

When hover state itself is needed for non-cursor UI, prefer existing interaction state:

```kotlin
import androidx.compose.foundation.hoverable
import androidx.compose.foundation.interaction.MutableInteractionSource
import androidx.compose.foundation.interaction.collectIsHoveredAsState
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon

@Composable
fun Modifier.hoverAwareCursor(enabled: Boolean): Modifier {
    val interactionSource = remember { MutableInteractionSource() }
    val hovered by interactionSource.collectIsHoveredAsState()

    return this
        .hoverable(interactionSource)
        .pointerHoverIcon(
            if (enabled && hovered) PointerIcon.Hand else PointerIcon.Default,
        )
}
```

Use `pointerInput` only when the app already needs raw pointer position or buttons.

### Step 5: Handle nested cursor precedence

Choose parent and child behavior deliberately:

- Default to `overrideDescendants = false` so text fields, child buttons, resize handles, and embedded controls keep their own cursor.
- Set `overrideDescendants = true` only for modal or mode-owned regions where a parent must force one cursor across everything below it, such as disabled overlay, drawing mode, crop mode, drag-marquee mode, or a temporary wait state.
- Do not use `overrideDescendants = true` on broad app roots unless every child really should lose its cursor.
- Audit popups, menus, tooltips, and overlays after changing cursor precedence. They may be separate windows or layers with independent pointer behavior.

Example parent-owned drawing cursor:

```kotlin
import androidx.compose.foundation.layout.Box
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon

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

### Step 6: Use Android system cursors

When the desired cursor exists in the Android system but not in Compose built-ins, create the platform cursor using `android.view.PointerIcon` and wrap it in a Compose `PointerIcon`.

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon
import androidx.compose.ui.platform.LocalContext

@Composable
fun Modifier.horizontalResizeCursor(): Modifier {
    val context = LocalContext.current
    val icon = remember(context) {
        PointerIcon(android.view.PointerIcon.getSystemIcon(context, android.view.PointerIcon.TYPE_HORIZONTAL_DOUBLE_ARROW))
    }
    return this.pointerHoverIcon(icon)
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

### Step 7: Coordinate with interactions

Cursors should match real behavior:

- `clickable` / `combinedClickable`: Use `Hand` only when the element is enabled and click activation is expected. Keep keyboard activation and semantics intact.
- `hoverable`: Use when hover state drives visual styling, labels, previews, or analytics. Cursor-only changes do not require tracking hover state.
- `draggable` / drag handles: Show resize or move cursors on the handle before drag starts. During drag, consider `overrideDescendants = true` on the drag surface if descendants should not change the cursor mid-drag.
- Text fields and editors: Preserve `Text` cursor over editable/selectable text. Avoid parent `overrideDescendants = true` around text input unless the parent mode intentionally disables editing.
- Selection regions: Use `Text` for text selection and `Crosshair` for spatial selection. Do not mix them in the same region.
- Split panes and resizable columns: Use directional resize cursors on the narrow handle, not the whole pane.
- Disabled or read-only controls: Keep `Default`, or use visible disabled styling and explanatory tooltips. Do not use `Wait` to mean disabled.
- Loading states: Prefer progress indicators. Use `Wait` only for short blocked areas, and reset it immediately when the operation ends.
- Popups and menus: Verify cursor resets correctly when entering or leaving popup content. Avoid parent overrides that make menus look like canvas tools.

### Step 8: Accessibility and UX checklist

Cursor work is not complete until it remains usable without a mouse:

- Every cursor affordance has an equivalent visual state such as hover color, focus ring, handle shape, selected state, disabled style, or tooltip.
- Every clickable cursor region remains keyboard reachable when appropriate.
- Cursor changes do not replace `contentDescription`, labels, roles, or semantics.
- Users can still identify text insertion, text selection, resize handles, dragging, and blocked states.
- The cursor never flickers rapidly when hovering over nested children or while recomposition updates state.

### Step 9: Verification checklist

Run the app locally on a device or emulator with a connected mouse and verify:

- Hovering each target shows the expected cursor.
- Leaving the target returns to the expected parent or default cursor.
- Enabled and disabled states update the cursor without stale state.
- Nested children can override the parent when `overrideDescendants = false`.
- Parent override wins only where intended when `overrideDescendants = true`.
- The clickable, draggable, resize, and text-selection hit regions match the cursor region.
- Text fields, selectable text, and custom editors still show an I-beam cursor and allow selection.
- Drag start, drag movement, drag cancellation, and drag end do not leave the wrong cursor behind.
- Popups, dropdowns, tooltips, dialogs, and menus display and dismiss normally.

Suggested commands, adjusted for the project:

```bash
./gradlew :app:assembleDebug
./gradlew :app:test
```

If the repo has UI or screenshot tests for desktop interactions, add or update a focused test for the wrapper or cursor policy. Full cursor shape assertions are usually manual; automated tests should still cover state mapping.

## Common pitfalls

- Using `pointerHoverIcon` as a substitute for `clickable`, `hoverable`, semantics, focus, or visual states.
- Showing `Hand` over disabled controls.
- Setting `overrideDescendants = true` on a broad parent and accidentally suppressing `Text` cursors in text fields.
- Calling `android.view.PointerIcon.getSystemIcon` directly inside recomposition callbacks without `remember`.
- Letting cursor state diverge from actual interaction state during drag, modal, loading, or disabled transitions.

## Official reference links

- Compose `pointerHoverIcon` API reference: <https://developer.android.com/reference/kotlin/androidx/compose/ui/input/pointer/package-summary#pointerHoverIcon(androidx.compose.ui.Modifier,androidx.compose.ui.input.pointer.PointerIcon,kotlin.Boolean)>
- Compose `PointerIcon` API reference: <https://developer.android.com/reference/kotlin/androidx/compose/ui/input/pointer/PointerIcon>
- Android pointer input overview: <https://developer.android.com/develop/ui/compose/touch-input/pointer-input>
- Android `PointerIcon` API reference: <https://developer.android.com/reference/android/view/PointerIcon>
