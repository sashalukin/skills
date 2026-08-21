# Tooltips in Android Compose Large Screens

## Description
Use this document when you need to add or redesign tooltips in a Jetpack Compose application targeting Android large screens and desktop environments.

Expected output:

- A short analysis of the tooltip anchors, trigger mechanisms, and relevant input modes.
- Clean implementation that preserves accessibility, focus behavior, and design-system conventions.

## Workflow

### Step 1: Identify targets and write copy

Inspect the relevant UI:

1. Identify elements that benefit from a tooltip, such as icon-only buttons, compact toolbar actions, disabled actions, and controls with keyboard shortcuts.
2. Exclude elements where a tooltip should not carry essential information, such as navigation labels, labeled buttons, form labels, and validation errors.

For each tooltip, write brief, sentence-case copy:
- Prefer verbs for actions: `Add column`, `Export CSV`, `Mute track`.
- Keep plain tooltips brief (1-5 words).
- Do not repeat visible button text unless the tooltip adds a shortcut or consequence.
- Add shortcut hints only when accurate, e.g., `Save (Ctrl+S)`.
- For disabled controls, explain the requirement: `Select a row first`.

### Step 2: Implement the tooltips

Iterate through the identified targets and choose the most appropriate implementation pattern for each:

#### Pattern A: Reusable Wrapper (Recommended)
Use a wrapper when a repeated component always needs the same tooltip styling or positioning logic.

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ActionTooltip(
    text: String,
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit,
) {
    TooltipBox(
        positionProvider = TooltipDefaults.rememberPlainTooltipPositionProvider(),
        tooltip = { PlainTooltip { Text(text) } },
        state = rememberTooltipState(),
        modifier = modifier,
    ) {
        content()
    }
}
```

#### Pattern B: Inline Plain Tooltip
Use `TooltipBox` directly around the anchor for one-off tooltips. The anchor must remain the actual interactive control.

```kotlin
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
- Give each inline `TooltipBox` its own `rememberTooltipState()`. Do not share state across multiple controls.
- Prefer `TooltipDefaults.rememberTooltipPositionProvider(TooltipAnchorPosition.Above)` if available in the project's Compose version.

#### Pattern C: Rich Tooltip
Use a rich tooltip when it needs title/body structure or an action.

```kotlin
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
- Use `isPersistent = true` when `RichTooltip` contains actions or links.
- Keep actions inside rich tooltips minimal. 

### Step 3: Programmatic control (Optional)

When tooltips are used for onboarding or validation hints, control them programmatically:
- Call `tooltipState.show()` from a coroutine; it is a suspend function.
- Call `tooltipState.dismiss()` when the related UI disappears or an action completes.
- Do not show multiple tooltips at once.

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
