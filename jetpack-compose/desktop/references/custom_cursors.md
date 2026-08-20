# Custom Cursors in Jetpack Compose Desktop

## Description
Use this document when you need to add, redesign, audit, or verify mouse cursor and pointer hover icon behavior in a Jetpack Compose Desktop or Compose Multiplatform desktop application.

Expected output:

- A short analysis of the app structure, target source sets, interaction surfaces, and pointer devices.
- A recommendation for built-in `PointerIcon` values, desktop platform cursors, or custom image cursors.
- Clean implementation that preserves click, drag, text selection, accessibility, and existing design-system behavior.
- Verification notes for hover regions, nested cursor precedence, enabled/disabled state, custom cursor image quality, and platform behavior.

## Prerequisites

- Jetpack Compose Desktop or Compose Multiplatform desktop target enabled.
- Access to the module that contains desktop UI source sets, usually `commonMain`, `desktopMain`, `jvmMain`, or a desktop app module.
- Compose UI dependency that provides `androidx.compose.ui.input.pointer.pointerHoverIcon` and `PointerIcon`.
- AWT availability when using native desktop cursors or custom cursor images. Keep `java.awt.*` imports out of source sets that compile for Android, iOS, or web.

## Current documentation notes

- `Modifier.pointerHoverIcon(icon, overrideDescendants = false)` sets the pointer icon while the cursor is hovered over that modifier's input region.
- `overrideDescendants = false` is the default, so child composables can set their own pointer icons.
- `overrideDescendants = true` makes the parent's icon win for all descendants under that parent.
- Compose built-ins cover the most portable cases: `PointerIcon.Default`, `PointerIcon.Hand`, `PointerIcon.Text`, and `PointerIcon.Crosshair`.
- Desktop-only code can wrap a `java.awt.Cursor` in `PointerIcon` for platform cursors not exposed as Compose built-ins, such as resize, wait, or move cursors.
- Custom image cursors on desktop should be created with `Toolkit.createCustomCursor(image, hotSpot, name)` and cached. Do not allocate cursor images or AWT cursor instances during routine recomposition.
- AWT cursor creation is platform-sensitive: hotspot values must be inside the cursor bounds, supported image sizes vary, unsupported custom cursor sizes may be resized with poor quality, invalid images can result in a transparent cursor, and headless environments cannot create cursors.
- Pointer cursor changes are affordances only. They must not be the only indication that an action is available, disabled, draggable, selected, or destructive.

## Workflow

### Step 1: Discover the cursor surface

Before editing, inspect the relevant UI:

1. Locate the target composables, toolbar buttons, links, text editors, canvas surfaces, draggable items, splitters, resize handles, timeline controls, maps, charts, and disabled controls.
2. Find active source sets: `commonMain`, `desktopMain`, `jvmMain`, Android-specific source sets, and any desktop-only app module.
3. Search for existing cursor helpers or wrappers: `pointerHoverIcon`, `PointerIcon`, `Cursor`, `createCustomCursor`, `hoverable`, `collectIsHoveredAsState`, `onPointerEvent`, `pointerInput`, `clickable`, `combinedClickable`, `draggable`, and `TextField`.
4. Identify whether the app already has design-system wrappers for buttons, links, drag handles, toolbars, text fields, or canvas tools. Prefer adding cursor behavior there instead of scattering raw modifiers.
5. Identify nested interactive regions. Parent canvas modes, child buttons, text fields, popup menus, and resize handles often need different cursor precedence.
6. Confirm the desired behavior for disabled, read-only, loading, dragging, selection, and modal states.

Do not add custom cursor code until you know the pointer region, the owning source set, and how the cursor should interact with descendants.

### Step 2: Choose the cursor type

Use the simplest cursor that communicates the action:

- **Default:** Normal content, disabled controls, non-interactive regions, and buttons whose existing component already supplies the expected cursor behavior.
- **Hand:** Links, clickable text, icon-only clickable controls, and canvas objects that act like buttons. Do not use `Hand` for every hoverable surface.
- **Text:** Editable or selectable text regions, custom text editors, code editors, text selection overlays, and text insertion handles.
- **Crosshair:** Precision canvas targeting, drawing, crop, measurement, color picking, or region selection.
- **Resize cursors:** Split panes, column edges, crop handles, resizable shapes, or window-like handles. Use platform/AWT cursors in desktop source sets when Compose built-ins are not enough.
- **Move cursor:** Dragging or repositioning an object when movement is the primary operation.
- **Wait cursor:** Short blocking operations only when the UI cannot respond. Prefer visible progress indicators and keep the cursor state scoped to the blocked region.
- **Custom image cursor:** Specialized creative tools, games, maps, CAD-like editors, whiteboards, and branded tools where a platform cursor is not expressive enough.

Avoid novelty cursors for standard app controls. Users rely on OS cursor conventions to predict click, text, drag, and resize behavior.

### Step 3: Place code in the right source set

Source-set placement matters:

- Put built-in `PointerIcon` usage in `commonMain` only when the same composable legitimately needs pointer-hover behavior on every target that compiles that code.
- Put desktop-only cursor behavior in `desktopMain`, `jvmMain`, or a desktop app module when it depends on `java.awt.Cursor`, `java.awt.Toolkit`, desktop pointer event APIs, desktop resources, or desktop-only interactions.
- If shared UI needs desktop-specific cursors, use one of these patterns:
  - A small common wrapper such as `Modifier.platformPointerHoverIcon(...)` with desktop `actual` behavior and no-op or built-in behavior elsewhere.
  - Inject a `Modifier` or cursor policy from the desktop module into shared UI.
  - Keep the shared composable cursor-free and wrap it at desktop call sites.
- Do not import `java.awt.*` from `commonMain` in multiplatform code.
- Do not make Android touch UI depend on desktop hover behavior. Android can have mouse/stylus pointers, but custom AWT cursors are desktop-only.

Example `expect`/`actual` wrapper for a shared app:

```kotlin
// commonMain
import androidx.compose.ui.Modifier

expect fun Modifier.handCursor(enabled: Boolean = true): Modifier
```

```kotlin
// desktopMain or jvmMain
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon

actual fun Modifier.handCursor(enabled: Boolean): Modifier =
    if (enabled) this.pointerHoverIcon(PointerIcon.Hand) else this
```

```kotlin
// androidMain or other non-desktop target
import androidx.compose.ui.Modifier

actual fun Modifier.handCursor(enabled: Boolean): Modifier = this
```

### Step 4: Implement built-in pointer icons

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

### Step 5: Implement conditional cursor state

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

Use pointer events only when the app already needs pointer position, buttons, or raw enter/exit behavior. `onPointerEvent` is desktop-oriented and may require experimental opt-in; `pointerInput` is the stable lower-level option for common code.

### Step 6: Handle nested cursor precedence

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

### Step 7: Use desktop platform cursors

When the desired cursor exists in AWT but not in Compose built-ins, create the platform cursor in desktop-only code and wrap it in `PointerIcon`.

```kotlin
// desktopMain or jvmMain
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon
import java.awt.Cursor

@Composable
fun Modifier.horizontalResizeCursor(): Modifier {
    val icon = remember {
        PointerIcon(Cursor.getPredefinedCursor(Cursor.E_RESIZE_CURSOR))
    }
    return this.pointerHoverIcon(icon)
}
```

Common AWT predefined cursors:

- `Cursor.DEFAULT_CURSOR`
- `Cursor.CROSSHAIR_CURSOR`
- `Cursor.TEXT_CURSOR`
- `Cursor.WAIT_CURSOR`
- `Cursor.HAND_CURSOR`
- `Cursor.MOVE_CURSOR`
- `Cursor.N_RESIZE_CURSOR`, `S_RESIZE_CURSOR`, `E_RESIZE_CURSOR`, `W_RESIZE_CURSOR`
- `Cursor.NE_RESIZE_CURSOR`, `NW_RESIZE_CURSOR`, `SE_RESIZE_CURSOR`, `SW_RESIZE_CURSOR`

Use `Cursor.getPredefinedCursor(type)` instead of constructing `Cursor(type)` when you want an AWT predefined cursor.

### Step 8: Create and cache custom image cursors

Use custom image cursors only in desktop source sets. Load or generate the image once, scale it to a supported size, validate the hotspot, and cache the resulting `PointerIcon`.

```kotlin
// desktopMain or jvmMain
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon
import java.awt.Dimension
import java.awt.Graphics2D
import java.awt.GraphicsEnvironment
import java.awt.Image
import java.awt.Point
import java.awt.RenderingHints
import java.awt.Toolkit
import java.awt.image.BufferedImage
import javax.imageio.ImageIO

@Composable
fun Modifier.brushCursor(): Modifier {
    val icon = remember { loadCustomCursorIcon("/cursors/brush-32.png", Point(3, 29), "Brush") }
    return this.pointerHoverIcon(icon)
}

private fun loadCustomCursorIcon(
    resourcePath: String,
    preferredHotSpot: Point,
    accessibleName: String,
): PointerIcon {
    if (GraphicsEnvironment.isHeadless()) return PointerIcon.Default

    val toolkit = Toolkit.getDefaultToolkit()
    val preferredSize = 32
    val bestSize = toolkit.getBestCursorSize(preferredSize, preferredSize)
    if (bestSize.width <= 0 || bestSize.height <= 0) return PointerIcon.Default

    val source = requireNotNull(object {}.javaClass.getResource(resourcePath)) {
        "Missing cursor resource: $resourcePath"
    }
    val image = ImageIO.read(source)
    val cursorImage = image.scaleToCursorSize(bestSize)
    val hotSpot = preferredHotSpot.clampInside(bestSize)

    return PointerIcon(
        toolkit.createCustomCursor(cursorImage, hotSpot, accessibleName),
    )
}

private fun BufferedImage.scaleToCursorSize(size: Dimension): BufferedImage {
    if (width == size.width && height == size.height) return this

    val scaled = BufferedImage(size.width, size.height, BufferedImage.TYPE_INT_ARGB)
    val graphics = scaled.createGraphics()
    try {
        graphics.setRenderingHint(
            RenderingHints.KEY_INTERPOLATION,
            RenderingHints.VALUE_INTERPOLATION_BICUBIC,
        )
        graphics.drawImage(
            getScaledInstance(size.width, size.height, Image.SCALE_SMOOTH),
            0,
            0,
            null,
        )
    } finally {
        graphics.dispose()
    }
    return scaled
}

private fun Point.clampInside(size: Dimension): Point =
    Point(
        x.coerceIn(0, size.width - 1),
        y.coerceIn(0, size.height - 1),
    )
```

Custom cursor guidance:

- Prefer PNG with transparency for cursor art.
- Provide a 1x logical cursor image with enough contrast for light and dark backgrounds. Also test on HiDPI displays.
- Keep the hotspot on the precise active pixel: arrow tip, brush tip, crosshair center, resize edge, or grab point.
- Use `Toolkit.getBestCursorSize(width, height)` before creating the cursor. Some platforms support only one size; some return `0 x 0` when custom cursors are unsupported.
- Use a single-frame image. Multi-frame images are invalid for AWT custom cursors.
- Pass a meaningful `name`; AWT documents it as a localized description for Java Accessibility.
- Cache by semantic cursor key, theme, scale, and source resource if those values can vary.
- Fall back to a built-in cursor if the resource is missing, image decoding fails, the environment is headless, or the platform does not support custom cursors.

Example small cache outside composition:

```kotlin
// desktopMain or jvmMain
import androidx.compose.ui.input.pointer.PointerIcon
import java.awt.Point
import java.util.concurrent.ConcurrentHashMap

object DesktopCursorCache {
    private val cache = ConcurrentHashMap<String, PointerIcon>()

    fun custom(
        key: String,
        resourcePath: String,
        hotSpot: Point,
        accessibleName: String,
    ): PointerIcon = cache.getOrPut(key) {
        loadCustomCursorIcon(resourcePath, hotSpot, accessibleName)
    }
}
```

### Step 9: Coordinate with interactions

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
- Embedded Swing/AWT: Check whether the embedded component sets its own cursor. Compose cursor modifiers may not control child Swing components.

### Step 10: Accessibility and UX checklist

Cursor work is not complete until it remains usable without a mouse:

- Every cursor affordance has an equivalent visual state such as hover color, focus ring, handle shape, selected state, disabled style, or tooltip.
- Every clickable cursor region remains keyboard reachable when appropriate.
- Cursor changes do not replace `contentDescription`, labels, roles, or semantics.
- Custom cursor art has enough contrast on light, dark, image, and canvas backgrounds.
- Custom cursor art is not so large that it hides the target or nearby content.
- Hotspots feel precise at normal and HiDPI scale.
- Users can still identify text insertion, text selection, resize handles, dragging, and blocked states.
- Cursor behavior follows OS conventions on macOS, Windows, and Linux.
- The cursor never flickers rapidly when hovering over nested children or while recomposition updates state.

### Step 11: Verification checklist

Run the app locally on desktop and verify:

- Hovering each target shows the expected cursor.
- Leaving the target returns to the expected parent or default cursor.
- Enabled and disabled states update the cursor without stale state.
- Nested children can override the parent when `overrideDescendants = false`.
- Parent override wins only where intended when `overrideDescendants = true`.
- The clickable, draggable, resize, and text-selection hit regions match the cursor region.
- Text fields, selectable text, and custom editors still show an I-beam cursor and allow selection.
- Drag start, drag movement, drag cancellation, and drag end do not leave the wrong cursor behind.
- Popups, dropdowns, tooltips, dialogs, and menus display and dismiss normally.
- Custom cursor images load from packaged resources in a production build, not only from IDE working directories.
- Custom cursor hotspots are correct at 1x and HiDPI scale.
- Custom cursor fallback is acceptable when the platform reports `0 x 0` custom cursor support or the app runs in headless tests.
- No cursor objects are repeatedly allocated during recomposition or pointer movement.
- The implementation compiles for every target source set touched by the change.

Suggested commands, adjusted for the project:

```bash
./gradlew :desktopApp:compileKotlinDesktop
./gradlew :desktopApp:run
./gradlew :desktopApp:test
```

If the repo has UI or screenshot tests for desktop interactions, add or update a focused test for the wrapper or cursor policy. Full cursor shape assertions are usually manual or platform-specific; automated tests should still cover state mapping, source-set compilation, and custom cursor cache behavior.

## Common pitfalls

- Adding `java.awt.Cursor` imports to `commonMain` and breaking non-desktop targets.
- Using `pointerHoverIcon` as a substitute for `clickable`, `hoverable`, semantics, focus, or visual states.
- Showing `Hand` over disabled controls.
- Setting `overrideDescendants = true` on a broad parent and accidentally suppressing `Text` cursors in text fields.
- Creating `Toolkit`, `BufferedImage`, or `Cursor` objects directly in recomposition or pointer move callbacks.
- Using a custom cursor image size without calling `getBestCursorSize`.
- Putting the hotspot outside the final scaled image.
- Loading cursor images from a filesystem path that fails after packaging.
- Using low-contrast or oversized custom art that hides the target.
- Forgetting that custom cursor creation can fail or be unsupported in headless/test environments.
- Letting cursor state diverge from actual interaction state during drag, modal, loading, or disabled transitions.
- Adding desktop-only cursor code without running compilation for the desktop source set.

## Official reference links

- Compose `pointerHoverIcon` API reference: <https://developer.android.com/reference/kotlin/androidx/compose/ui/input/pointer/package-summary#pointerHoverIcon(androidx.compose.ui.Modifier,androidx.compose.ui.input.pointer.PointerIcon,kotlin.Boolean)>
- Compose `PointerIcon` API reference: <https://developer.android.com/reference/kotlin/androidx/compose/ui/input/pointer/PointerIcon>
- Android pointer input overview: <https://developer.android.com/develop/ui/compose/touch-input/pointer-input>
- Compose Multiplatform desktop mouse event listeners: <https://kotlinlang.org/docs/multiplatform/compose-desktop-mouse-events.html>
- Java AWT `Toolkit.createCustomCursor` and `getBestCursorSize`: <https://docs.oracle.com/javase/8/docs/api/java/awt/Toolkit.html#createCustomCursor-java.awt.Image-java.awt.Point-java.lang.String->
- Java AWT `Cursor` predefined and system cursors: <https://docs.oracle.com/javase/8/docs/api/java/awt/Cursor.html>
