# Custom Cursors in Jetpack Compose

## Description
Use this document when you need to implement custom mouse cursors (pointer icons) for specific hover states or interactions in a Jetpack Compose Desktop application.

## Prerequisites
- Jetpack Compose Desktop dependencies setup.
- Desktop target enabled.

## Common Problems and Solutions
- **Changing the cursor on hover:** You must use `Modifier.pointerHoverIcon(icon: PointerIcon)`.
- **Dynamic cursors based on state:** You must pass a conditional `PointerIcon` to `Modifier.pointerHoverIcon()`.
- **Using custom images as cursors:** You must create a `java.awt.Cursor` using `java.awt.Toolkit` and wrap it in `PointerIcon`.
- **Performance degradation:** You must instantiate `java.awt.Cursor` objects outside of recomposition loops, or use `remember` to prevent unnecessary memory allocations.
- **Overriding child cursors:** You must set `overrideDescendants = true` if the parent cursor must override child cursors.

## Instructions

### Step 1: Evaluate cursor requirements
Determine the required cursor type for the UI element.
- Standard cursors: Hand, Text, Crosshair, Default.
- Custom image cursors: Loaded from an image asset or file.

### Step 2: Implement standard cursors
Apply the `pointerHoverIcon` modifier to the Compose element. You must use the `androidx.compose.ui.input.pointer.PointerIcon` class.

```kotlin
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon

// Apply a standard hand cursor
Modifier.pointerHoverIcon(PointerIcon.Hand)
```

### Step 3: Implement conditional cursors
When the cursor depends on state, you must use a conditional expression inside the modifier.

```kotlin
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon

val isClickable = false 
Modifier.pointerHoverIcon(
    if (isClickable) PointerIcon.Hand else PointerIcon.Default
)
```

### Step 4: Implement custom image cursors
When the project requires a custom image as a cursor, you must create a `java.awt.Cursor` and convert it to a `PointerIcon`.
You must cache or `remember` the cursor instance to avoid performance issues during recomposition.

```kotlin
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon
import java.awt.Point
import java.awt.Toolkit
import java.awt.image.BufferedImage

// Inside a @Composable function:
val customPointerIcon = remember {
    val image = BufferedImage(16, 16, BufferedImage.TYPE_INT_ARGB)
    // Load or draw image pixels here
    val awtCursor = Toolkit.getDefaultToolkit().createCustomCursor(image, Point(0, 0), "CustomCursor")
    PointerIcon(awtCursor)
}

Modifier.pointerHoverIcon(customPointerIcon)
```

### Step 5: Override descendant cursors
When a parent component must force its cursor on all children, you must set `overrideDescendants` to true.

```kotlin
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerIcon
import androidx.compose.ui.input.pointer.pointerHoverIcon

Modifier.pointerHoverIcon(
    icon = PointerIcon.Hand,
    overrideDescendants = true
)
```
