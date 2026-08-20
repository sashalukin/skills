# Tooltips in Jetpack Compose Desktop

## Description
Use this document when you need to add, redesign, audit, or verify tooltips in a Jetpack Compose Desktop or Compose Multiplatform desktop application.

Expected output:

- A short analysis of the app structure, target source sets, UI anchors, and pointer/keyboard use cases.
- A recommendation for Material 3 `TooltipBox` or desktop-only foundation `TooltipArea`.
- Clean implementation that preserves accessibility, focus, pointer behavior, and existing design-system style.
- Manual or automated verification notes for hover, focus/keyboard, dismissal, clipping, source-set correctness, and copy quality.

## Prerequisites

- Jetpack Compose Desktop or Compose Multiplatform desktop target enabled.
- Access to the module that contains desktop UI source sets, usually `commonMain`, `desktopMain`, or `jvmMain`.
- Material 3 Compose dependency when using `androidx.compose.material3.TooltipBox`, `PlainTooltip`, or `RichTooltip`.
- Compose Foundation dependency when using JetBrains desktop-only `androidx.compose.foundation.TooltipArea`.
- Opt in to experimental APIs at the narrowest practical scope:
  - `@OptIn(ExperimentalMaterial3Api::class)` for Material 3 tooltip APIs.
  - `@OptIn(ExperimentalFoundationApi::class)` for foundation `TooltipArea`.

## Current documentation notes

- Prefer Material 3 `TooltipBox` for shared Compose UI and Material-styled apps. It works with `PlainTooltip`, `RichTooltip`, `TooltipState`, `rememberTooltipState()`, and `TooltipDefaults` position providers.
- Use `PlainTooltip` for short labels that describe icon-only buttons, toolbar controls, and compact actions.
- Use `RichTooltip` for extra context, a title, multiline explanation, links, or an action such as Dismiss or Learn more.
- Use `rememberTooltipState()` unless state must be constructed outside composition. Each `TooltipBox` should have its own `TooltipState`.
- Use `TooltipState.show()` and `TooltipState.dismiss()` from a coroutine when a tooltip must be shown or hidden programmatically.
- Use persistent tooltips for actionable rich tooltips. Non-persistent tooltips dismiss after the default short duration; persistent tooltips dismiss on outside click or explicit `dismiss()`.
- Material 3 `TooltipBox` can respond to pointer hover and long press through `enableUserInput`. On desktop, verify hover behavior with a mouse or trackpad.
- Material 3 position-provider APIs have evolved. In older/current projects you may see `TooltipDefaults.rememberPlainTooltipPositionProvider()` and `rememberRichTooltipPositionProvider()`. Newer Compose Multiplatform Material 3 also exposes `rememberTooltipPositionProvider(TooltipAnchorPosition, spacingBetweenTooltipAndAnchor)`. Match the project's dependency version.
- Use JetBrains foundation `TooltipArea` only for desktop-specific UI in `desktopMain` or `jvmMain`, or when a project already uses it consistently. It gives desktop-oriented placement such as cursor-relative or component-relative tooltips and a configurable hover delay.

## Workflow

### Step 1: Discover the tooltip surface

Before editing, inspect the relevant UI:

1. Locate the target composables, toolbar rows, icon buttons, menus, data tables, canvas controls, inspector panels, and disabled controls.
2. Find the active source sets: `commonMain`, `desktopMain`, `jvmMain`, Android-specific source sets, and any shared UI modules.
3. Search for existing tooltip helpers or wrappers: `TooltipBox`, `PlainTooltip`, `RichTooltip`, `TooltipArea`, `TooltipPlacement`, `rememberTooltipState`, `BasicTooltipBox`, `LocalTooltip`, `WithTooltip`, and design-system components.
4. Check whether the app uses Material 3 (`androidx.compose.material3`) or Material 2/foundation styling.
5. Identify anchors that are unclear without text: icon-only buttons, compact toolbar actions, color swatches, timeline controls, disabled actions, status badges, table headers, graph/canvas controls, destructive actions, and hidden shortcut affordances.
6. Identify where a tooltip is not appropriate: main navigation labels, visible text buttons, form labels, critical validation errors, instructions the user needs before acting, and anything required to complete a task.

Do not add tooltips until you know the source set, anchor component, copy, and expected trigger behavior.

### Step 2: Choose the API

Use this decision rule:

- **Material 3 `TooltipBox`:** Default choice for Material 3 apps, shared `commonMain` UI, Android plus desktop code, and tooltips that should follow Material semantics and styling.
- **Material 3 `PlainTooltip`:** Default for icon-only actions and brief command labels.
- **Material 3 `RichTooltip`:** Use for supplementary details, feature education, irreversible/destructive context, or a short action inside the tooltip.
- **Foundation `TooltipArea`:** Use only in desktop-specific source sets when the project is not using Material 3 tooltips, when the existing desktop UI already uses `TooltipArea`, or when cursor-relative placement and desktop hover delay are specifically needed.
- **Custom popup:** Use only when the requested behavior is not a tooltip anymore, such as onboarding coach marks, validation panels, contextual menus, hover previews, or persistent inspectors.

Avoid mixing `TooltipArea` and Material 3 tooltip styling in the same screen unless the codebase already has a wrapper that normalizes appearance.

### Step 3: Place code in the right source set

Source-set placement matters:

- Put Material 3 `TooltipBox` wrappers in `commonMain` when the same composable is used on Android and desktop.
- Put desktop-only tooltip behavior in `desktopMain` or `jvmMain` if it depends on foundation `TooltipArea`, AWT/Swing, cursor-specific behavior, or desktop-only delay/placement.
- If shared UI needs different tooltip implementations per target, use one of these patterns:
  - `expect`/`actual` wrapper such as `PlatformTooltip(...)`.
  - A common wrapper whose desktop implementation is injected from the desktop module.
  - A no-op or alternate implementation for unsupported targets if product requirements allow it.
- Do not import `androidx.compose.foundation.TooltipArea` from `commonMain` unless the project's Compose Multiplatform version and target matrix actually support that usage.
- Keep tooltip text in the same localization/string-resource system used by the app. Do not hard-code user-visible copy in a codebase that localizes strings.

Example shared wrapper using Material 3:

```kotlin
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.PlainTooltip
import androidx.compose.material3.Text
import androidx.compose.material3.TooltipBox
import androidx.compose.material3.TooltipDefaults
import androidx.compose.material3.rememberTooltipState
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ActionTooltip(
    text: String,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit,
) {
    TooltipBox(
        modifier = modifier,
        positionProvider = TooltipDefaults.rememberPlainTooltipPositionProvider(),
        tooltip = { PlainTooltip { Text(text) } },
        state = rememberTooltipState(),
        content = content,
    )
}
```

Example desktop-only wrapper using foundation `TooltipArea`:

```kotlin
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.TooltipArea
import androidx.compose.foundation.TooltipPlacement
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.shadow
import androidx.compose.ui.unit.DpOffset
import androidx.compose.ui.unit.dp

@OptIn(ExperimentalFoundationApi::class)
@Composable
fun DesktopTooltipArea(
    text: String,
    modifier: Modifier = Modifier,
    delayMillis: Int = 500,
    content: @Composable () -> Unit,
) {
    TooltipArea(
        modifier = modifier,
        delayMillis = delayMillis,
        tooltipPlacement = TooltipPlacement.CursorPoint(
            alignment = Alignment.BottomEnd,
            offset = DpOffset(12.dp, 12.dp),
        ),
        tooltip = {
            Surface(
                modifier = Modifier.shadow(4.dp),
                shape = RoundedCornerShape(4.dp),
                color = MaterialTheme.colorScheme.inverseSurface,
                contentColor = MaterialTheme.colorScheme.inverseOnSurface,
            ) {
                Text(text = text, modifier = Modifier.padding(horizontal = 8.dp, vertical = 6.dp))
            }
        },
        content = content,
    )
}
```

### Step 4: Write tooltip copy

Tooltip copy should be useful, short, and consistent:

- Prefer verbs for actions: `Add column`, `Export CSV`, `Mute track`, `Open settings`.
- Use sentence case unless the app's design system says otherwise.
- Keep plain tooltips brief, usually 1-5 words and rarely more than one short sentence.
- Do not repeat visible button text unless the tooltip adds shortcut, state, or consequence information.
- Add shortcut hints only when they are accurate for the platform, such as `Save (Cmd+S)` on macOS and `Save (Ctrl+S)` on Windows/Linux.
- For disabled controls, explain the requirement: `Select a row first`, `Connect a device to record`, `Finish sync before exporting`.
- Do not put critical warnings only in a tooltip. Surface critical information inline, in a dialog, or in supporting text.
- Do not use tooltips as documentation. Rich tooltips can teach one concept, but long help belongs in a help panel or docs.

Examples:

- Good plain tooltip: `Export CSV`
- Good disabled tooltip: `Select at least one message`
- Good rich title: `Shared filters`
- Good rich body: `Changes apply to everyone using this dashboard.`
- Poor tooltip: `Click this button to perform the export action using the configured comma-separated-values export pipeline.`

### Step 5: Implement Material 3 plain tooltips

Use `TooltipBox` around the anchor. The tooltip anchor must remain the actual interactive control.

```kotlin
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Download
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.PlainTooltip
import androidx.compose.material3.Text
import androidx.compose.material3.TooltipBox
import androidx.compose.material3.TooltipDefaults
import androidx.compose.material3.rememberTooltipState
import androidx.compose.runtime.Composable

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ExportButton(onExport: () -> Unit) {
    TooltipBox(
        positionProvider = TooltipDefaults.rememberPlainTooltipPositionProvider(),
        tooltip = { PlainTooltip { Text("Export CSV") } },
        state = rememberTooltipState(),
    ) {
        IconButton(onClick = onExport) {
            Icon(
                imageVector = Icons.Filled.Download,
                contentDescription = "Export CSV",
            )
        }
    }
}
```

Implementation notes:

- Keep `rememberTooltipState()` inside the composable that owns the tooltip.
- Give each `TooltipBox` its own state; do not share one state across a row of controls.
- Use `TooltipDefaults.rememberPlainTooltipPositionProvider()` when that is the project's available API.
- On newer Material 3 versions, prefer the non-deprecated position provider if the project uses it:

```kotlin
TooltipDefaults.rememberTooltipPositionProvider(
    positioning = TooltipAnchorPosition.Above,
)
```

- If the anchor already has visible text, only add a tooltip when it provides extra information such as a shortcut or disabled reason.

### Step 6: Implement Material 3 rich tooltips

Use a rich tooltip when it needs title/body structure or an action. Persistent state is usually right when the tooltip contains interactive content.

```kotlin
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Info
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.Icon
import androidx.compose.material3.IconButton
import androidx.compose.material3.RichTooltip
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.material3.TooltipBox
import androidx.compose.material3.TooltipDefaults
import androidx.compose.material3.rememberTooltipState
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.rememberCoroutineScope
import kotlinx.coroutines.launch

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun SharedFiltersInfoButton() {
    val tooltipState = rememberTooltipState(isPersistent = true)
    val scope = rememberCoroutineScope()

    TooltipBox(
        positionProvider = TooltipDefaults.rememberRichTooltipPositionProvider(),
        tooltip = {
            RichTooltip(
                title = { Text("Shared filters") },
                action = {
                    TextButton(
                        onClick = { scope.launch { tooltipState.dismiss() } },
                    ) {
                        Text("Dismiss")
                    }
                },
            ) {
                Text("Changes apply to everyone using this dashboard.")
            }
        },
        state = tooltipState,
    ) {
        IconButton(onClick = { scope.launch { tooltipState.show() } }) {
            Icon(
                imageVector = Icons.Filled.Info,
                contentDescription = "Shared filters information",
            )
        }
    }
}
```

Implementation notes:

- Use `isPersistent = true` when `RichTooltip` contains actions or links.
- Use a regular click action on the anchor only if click behavior is useful. Do not make an info icon clickable only to duplicate hover behavior unless the product needs explicit manual reveal.
- Keep actions inside rich tooltips minimal. If the action changes application state or starts a workflow, consider a popover or dialog instead.
- If a tooltip action dismisses the tooltip, call `tooltipState.dismiss()` from a coroutine or direct callback as required by the API in use.

### Step 7: Control show and dismiss behavior

Programmatic control is useful for help modes, onboarding, validation hints after failed commands, and explicit info buttons.

```kotlin
import androidx.compose.material3.ExperimentalMaterial3Api
import androidx.compose.material3.PlainTooltip
import androidx.compose.material3.Text
import androidx.compose.material3.TooltipBox
import androidx.compose.material3.TooltipDefaults
import androidx.compose.material3.rememberTooltipState
import androidx.compose.runtime.Composable
import androidx.compose.runtime.rememberCoroutineScope
import kotlinx.coroutines.launch

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ManuallyShownTooltip(
    showHelp: Boolean,
    anchor: @Composable (show: () -> Unit, dismiss: () -> Unit) -> Unit,
) {
    val tooltipState = rememberTooltipState(isPersistent = true)
    val scope = rememberCoroutineScope()

    TooltipBox(
        positionProvider = TooltipDefaults.rememberPlainTooltipPositionProvider(),
        tooltip = { PlainTooltip { Text("Create a new project") } },
        state = tooltipState,
    ) {
        anchor(
            show = { scope.launch { tooltipState.show() } },
            dismiss = { tooltipState.dismiss() },
        )
    }

    LaunchedEffect(showHelp) {
        if (!showHelp && tooltipState.isVisible) {
            tooltipState.dismiss()
        }
    }
}
```

Rules for manual control:

- Call `show()` from a coroutine; it is a suspend function.
- Call `dismiss()` when the related UI disappears, the user changes mode, or a rich tooltip action completes.
- Avoid `initialIsVisible = true` except in previews, focused onboarding steps, or explicitly requested help modes.
- Do not show several tooltips at once. The default Material 3 mutator mutex helps coordinate this; avoid replacing it unless there is a strong reason.
- If you disable `enableUserInput`, provide a clear alternate trigger and test it.

### Step 8: Tune positioning

Use defaults first:

- `TooltipDefaults.rememberPlainTooltipPositionProvider()` for plain tooltips when available.
- `TooltipDefaults.rememberRichTooltipPositionProvider()` for rich tooltips when available.
- `TooltipDefaults.rememberTooltipPositionProvider(TooltipAnchorPosition.Above)` or another anchor position when the project uses newer Material 3 APIs.
- `TooltipPlacement.CursorPoint(...)` for desktop foundation `TooltipArea` when the tooltip should follow the cursor.
- `TooltipPlacement.ComponentRect(...)` for desktop foundation `TooltipArea` when the tooltip should align to the component instead of the pointer.

Positioning checks:

- Prefer above the anchor for toolbar controls unless it collides with window edges.
- For controls near top edges, verify fallback below the anchor.
- For right-aligned toolbars and side panels, verify the tooltip remains inside the window.
- For dense tables and canvases, cursor-relative `TooltipArea` can be useful, but it should not cover the value being inspected.
- Avoid custom `PopupPositionProvider` unless defaults cannot satisfy a real layout requirement.
- Test with resized windows, high DPI scaling, long localized strings, RTL layout if the app supports it, and multiple monitors when possible.

### Step 9: Preserve accessibility

Tooltips help pointer users, but they are not a substitute for accessibility semantics.

- Every icon-only action must have an accessible name, usually through `Icon(contentDescription = "...")` or semantics on the anchor.
- Keep the tooltip and content description aligned but not necessarily identical. The content description names the control; the tooltip can include a shortcut or disabled reason.
- If an icon is decorative inside a text button, use `contentDescription = null` for the icon and put the accessible name on the button text.
- Do not put essential instructions only in a hover tooltip. Keyboard, touch, and assistive-tech users may not discover it.
- Ensure the anchor is focusable when it is interactive. Do not add focusability to purely decorative elements just to show a tooltip.
- For rich tooltips with interactive actions, verify keyboard focus can reach the action and Escape/outside click behavior does not trap focus.
- If `focusable = false` is used on `TooltipBox`, verify assistive-tech access and pointer behavior still match the product requirement.
- For disabled controls, remember that disabled composables may not receive hover/focus events. If the product requires a disabled reason tooltip, wrap an enabled parent container or use a semantic wrapper while keeping the actual action disabled.

### Step 10: Audit existing tooltips

When auditing, report findings by anchor and severity:

- Missing tooltip on icon-only controls where the icon is ambiguous.
- Tooltip text duplicates visible text without adding value.
- Tooltip contains required instructions or errors that should be visible inline.
- `TooltipState` is shared across unrelated anchors.
- `show()` is called outside a coroutine or never dismissed.
- Rich tooltip contains actions but uses non-persistent state.
- `TooltipArea` leaks into common source sets or non-desktop targets.
- Tooltip wrappers alter the anchor size, padding, semantics, or click target.
- Tooltip placement clips at window edges, overlays the anchor, or covers the data being inspected.
- Tooltip copy is too long, jargon-heavy, inconsistently capitalized, or unlocalized.
- Icon `contentDescription` is missing, stale, or inconsistent with the action.

## Common Problems and Solutions

- **Missing tooltip state:** Use `rememberTooltipState()` for each `TooltipBox`. Do not reuse one state for a whole toolbar.
- **Wrong API for source set:** Use Material 3 in shared UI; keep foundation `TooltipArea` in desktop-specific source sets unless the project explicitly supports it elsewhere.
- **Deprecated position provider warnings:** Match the Compose version. If `rememberPlainTooltipPositionProvider()` or `rememberRichTooltipPositionProvider()` is deprecated, migrate to `rememberTooltipPositionProvider(TooltipAnchorPosition...)`.
- **Tooltip does not appear on desktop:** Verify the anchor is inside `TooltipBox` content, `enableUserInput` is true, the desktop window is focused, and no overlay is consuming pointer events.
- **Tooltip appears but cannot be dismissed:** Use non-persistent state for passive plain tooltips, or call `dismiss()` from rich tooltip actions and mode changes.
- **Tooltip action is hard to use:** Use `isPersistent = true`, verify keyboard focus, and consider replacing the tooltip with a popover/dialog.
- **Disabled control has no tooltip:** Wrap the disabled control in an enabled parent tooltip anchor, but keep semantics honest and do not make the disabled action clickable.
- **Tooltip changes layout:** Ensure the tooltip content is in the popup/tooltip slot, not measured as part of the anchor's normal layout. Be careful with custom wrappers around `TooltipArea`.
- **Tooltip covers important UI:** Change anchor position, cursor offset, alignment, or use a richer help surface.
- **Accessibility regression:** Add or repair accessible names on anchors and do not rely on hover-only disclosure for required information.

## Verification Checklist

Run the app and verify:

- Hovering a pointer over each anchor shows the tooltip after the expected delay.
- Moving the pointer away or clicking outside dismisses the tooltip.
- Programmatic `show()` and `dismiss()` paths work without stuck state.
- Rich tooltip actions are reachable with keyboard and pointer, and dismiss correctly.
- Window resizing does not clip or misplace the tooltip.
- Tooltips near every window edge remain readable.
- High DPI scaling and long localized strings do not overflow awkwardly.
- Tooltip copy matches command labels, visible shortcuts, and localization conventions.
- Icon-only anchors have accessible names.
- Keyboard-only users can still complete the workflow without relying on tooltip-only information.
- Touch/long-press behavior still works if the same shared UI is used on Android.
- Automated tests or previews still compile for every affected source set.

Suggested technical checks:

```bash
./gradlew :desktopApp:compileKotlin
./gradlew :desktopApp:desktopTest
./gradlew :composeApp:compileKotlinDesktop
./gradlew :composeApp:compileDebugKotlinAndroid
```

Adjust task names to the repository. At minimum, run the compile task for every source set touched.

## Official References

- Android Developers: Tooltip in Jetpack Compose: https://developer.android.com/develop/ui/compose/components/tooltip
- Compose Multiplatform desktop tooltips with `TooltipArea`: https://kotlinlang.org/docs/multiplatform/compose-desktop-tooltips.html
- Compose Multiplatform Material 3 `TooltipDefaults`: https://kotlinlang.org/api/compose-multiplatform/material3/androidx.compose.material3/-tooltip-defaults/
- Compose Multiplatform Material 3 `TooltipState`: https://kotlinlang.org/api/compose-multiplatform/material3/androidx.compose.material3/-tooltip-state.html
- Compose Multiplatform Material 3 `RichTooltip`: https://kotlinlang.org/api/compose-multiplatform/material3/androidx.compose.material3/-rich-tooltip.html
- AndroidX Material 3 tooltip source and KDoc: https://android.googlesource.com/platform/frameworks/support/+/androidx-main/compose/material3/material3/src/commonMain/kotlin/androidx/compose/material3/Tooltip.kt
- Material Design 3 tooltip guidelines: https://m3.material.io/components/tooltips
