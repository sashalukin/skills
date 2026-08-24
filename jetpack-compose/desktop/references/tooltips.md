# Add tooltips in Jetpack Compose for desktops

## Description
Use this document when you need to add or redesign tooltips in a Jetpack Compose application targeting Android large screens and desktop environments.

Expected output:

- A short analysis of the tooltip anchors, trigger mechanisms, and relevant input modes.
- Clean implementation that preserves accessibility, focus behavior, and design-system conventions.

## Workflow

### Step 1: Identify tooltip targets and text

Inspect the relevant UI:

1. Identify elements that benefit from a tooltip, such as icon-only buttons, compact toolbar actions, disabled actions, and controls with keyboard shortcuts.
2. Exclude elements where a tooltip should not carry essential information, such as navigation labels, labeled buttons, form labels, and validation errors.

For each tooltip, write brief, sentence-case copy:
- Prefer verbs for actions: `Add column`, `Export CSV`, `Mute track`.
- Keep plain tooltips brief (1-5 words).
- Do not repeat visible button text unless the tooltip adds a shortcut or consequence.
- Add shortcut hints only when accurate, e.g., `Save (Ctrl+S)`.
- For disabled controls, explain the requirement: `Select a row first`.
- Check whether long-press already starts dragging, selection, or a context menu. Do not add tooltip behavior that competes with an existing gesture.

### Step 2: Implement the tooltips

Iterate through the identified targets and implement tooltips for each according to the following patterns.

- **Code:** Use an existing design-system component when possible. Use a shared wrapper for repeated tooltips and inline `TooltipBox` for one-off tooltips.
- **Type:** Use `PlainTooltip` for a short label. Use `RichTooltip` for extra details or one action.
- **Trigger:** Use the default hover, long-press, and keyboard-focus behavior unless the tooltip should only be shown from code.

Use the current tooltip API:

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
- Keep the real button or control inside `TooltipBox`. Do not move its click, focus, enabled state, or accessibility behavior to the wrapper.
- Keep the wrapper the same size as the control.
- For a rich tooltip with an action, use `rememberTooltipState(isPersistent = true)`, set `hasAction = true`, and dismiss it after the action.
- For a tooltip shown only from code, set `enableUserInput = false`. Call `show()` from a coroutine and call `dismiss()` directly.

#### Show tooltips from code (optional)

Use this only for brief, nonessential help, such as feature education.

- Call `tooltipState.show()` from a coroutine.
- Call `tooltipState.dismiss()` directly when the tooltip is no longer needed.
- Set `enableUserInput = false` if hover, long press, and keyboard focus should not show it.
- Trigger `show()` from a user event or a one-time effect, not directly during composition.
- Show only one tooltip at a time.

## Common pitfalls

- **Missing tooltip state:** Reusing one `TooltipState` for multiple tooltips.
- **Wrong platform API:** Attempting to use Compose Desktop's `TooltipArea` on Android.
- **Stuck tooltips:** Failing to dismiss persistent tooltips after their action completes.
- **Accessibility regression:** Removing `contentDescription` from an icon just because it has a tooltip. Tooltips do not replace semantic accessibility.
- **Clipping:** Failing to check tooltip positioning near window edges.

## Reference links

Consult these references before proceeding with the implementation.

- Android Developers Tooltip guide: https://developer.android.com/develop/ui/compose/components/tooltip
- Material Design 3 tooltip guidelines: https://m3.material.io/components/tooltips
