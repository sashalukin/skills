---
name: add-compose-tooltips
description: Add or redesign Material 3 tooltips in Android Jetpack Compose apps for large screens and desktop windowing. Use for tooltip text, triggers, positioning, accessibility, rich actions, and programmatic display. Do not use Compose Multiplatform Desktop TooltipArea.
---

# Add tooltips in Jetpack Compose

Expected output:

- Brief analysis of targets and input methods, followed by an implementation that preserves accessibility, focus, and design-system rules.

## Workflow

### Step 1: Identify tooltip targets and text

Inspect the relevant UI:

1. Identify controls that need extra context, such as icon-only buttons, unclear toolbar actions, and controls with hidden keyboard shortcuts.
2. Skip controls that are already clear from visible text. Never use a tooltip as the only source of a label, instruction, validation error, or disabled-state requirement.
3. Check whether long press already starts dragging, selection, or a context menu. Skip the tooltip if it conflicts.

For each tooltip, use a localized string and brief, sentence-case text:
- Prefer verbs for actions: `Add column`, `Export CSV`, `Mute track`.
- Keep plain tooltips short and clear.
- Do not repeat visible button text unless the tooltip adds a shortcut or consequence.
- Add shortcut hints only when accurate, e.g., `Save (Ctrl+S)`.
- For disabled controls, explain the requirement: `Select a row first`.

### Step 2: Implement the tooltips

Iterate through the identified targets and implement tooltips for each according to the following patterns:

- **Reuse existing components:** Use a shared wrapper for repeated tooltips and inline `TooltipBox` for one-off tooltips.
- **Tooltip type:** Use `PlainTooltip` for a short label. Use `RichTooltip` for extra details or one action.
- **Tooltip trigger:** Use the default hover, long-press, and keyboard-focus behavior unless the tooltip should only be shown from code.

With Material 3 1.4.0 or newer, use:

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ActionTooltip(
    text: String,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit,
) {
    TooltipBox(
        positionProvider = TooltipDefaults.rememberTooltipPositionProvider(
            positioning = TooltipAnchorPosition.Above,
        ),
        tooltip = { PlainTooltip { Text(text) } },
        state = rememberTooltipState(),
        modifier = modifier,
    ) {
        content()
    }
}
```

Follow these rules:

- Give each `TooltipBox` its own `TooltipState`.
- Wrap exactly one control. Keep its click, focus, enabled state, and accessibility on the control, and keep the wrapper bounds tight.
- For a rich tooltip with an action, use `rememberTooltipState(isPersistent = true)`, set `hasAction = true`, and dismiss it after the action.

#### Show tooltips from code (optional)

Use this only for brief, nonessential help, such as feature education.

- Call `tooltipState.show()` from a coroutine.
- Call `tooltipState.dismiss()` directly when the tooltip is no longer needed.
- Set `enableUserInput = false` if hover, long press, and keyboard focus should not show it.
- Trigger `show()` from a user event or a one-time effect, not directly during composition.

## Reference links

Consult these references before proceeding with the implementation.

- AndroidX Material 3 tooltip samples: <https://android.googlesource.com/platform/frameworks/support/+/androidx-main/compose/material3/material3/samples/src/main/java/androidx/compose/material3/samples/TooltipSamples.kt>
- Material Design 3 tooltip guidelines: <https://m3.material.io/components/tooltips/guidelines>