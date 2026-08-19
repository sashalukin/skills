# Tooltips in Jetpack Compose Desktop

## Description
Use this document when you need to add hover tooltips to UI elements in a Jetpack Compose Desktop application to enhance the user experience.

## Prerequisites
- Jetpack Compose Desktop dependencies setup.
- Material 3 Compose dependency (`androidx.compose.material3:material3`).
- Desktop target enabled.

## Common Problems and Solutions
- **Missing or incorrect state:** You must maintain a `TooltipState` using `rememberTooltipState()` to control the tooltip's visibility.
- **Incorrect positioning:** You must provide a valid `positionProvider`. Use `TooltipDefaults.rememberPlainTooltipPositionProvider()` for plain tooltips. Use `TooltipDefaults.rememberRichTooltipPositionProvider()` for rich tooltips.
- **Choosing the wrong tooltip type:** You must use `PlainTooltip` for brief, single-line text descriptions. You must use `RichTooltip` for complex information containing titles, multiline text, or interactive actions.
- **Accessibility failures:** You must ensure the anchor component (e.g., an `Icon` or `Button`) has an appropriate `contentDescription`.
- **Opt-in requirements:** You must opt-in to `@ExperimentalMaterial3Api` as the Material 3 Tooltip APIs are experimental.

## Instructions

### Step 1: Evaluate tooltip requirements
Determine the type of tooltip needed for the UI element based on the content complexity.
- **Plain Tooltip:** Short, descriptive text.
- **Rich Tooltip:** Descriptive text with a title, subtext, or interactive actions.

### Step 2: Implement a Plain Tooltip
Use `TooltipBox` with a `PlainTooltip`. You must supply a `TooltipState` and a `positionProvider`.

```kotlin
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Info
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
fun PlainTooltipExample() {
    val tooltipState = rememberTooltipState()
    
    TooltipBox(
        positionProvider = TooltipDefaults.rememberPlainTooltipPositionProvider(),
        tooltip = {
            PlainTooltip {
                Text("This is a simple plain tooltip.")
            }
        },
        state = tooltipState
    ) {
        IconButton(onClick = { /* Action */ }) {
            Icon(
                imageVector = Icons.Filled.Info,
                contentDescription = "Information"
            )
        }
    }
}
```

### Step 3: Implement a Rich Tooltip
When the UI requires structured information, use `TooltipBox` with a `RichTooltip`. You must define the `title` and `text` slots. You can optionally include an `action` slot.

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
import androidx.compose.runtime.rememberCoroutineScope
import kotlinx.coroutines.launch

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun RichTooltipExample() {
    val tooltipState = rememberTooltipState(isPersistent = true)
    val scope = rememberCoroutineScope()
    
    TooltipBox(
        positionProvider = TooltipDefaults.rememberRichTooltipPositionProvider(),
        tooltip = {
            RichTooltip(
                title = { Text("Rich Tooltip Title") },
                action = {
                    TextButton(
                        onClick = { scope.launch { tooltipState.dismiss() } }
                    ) {
                        Text("Acknowledge")
                    }
                }
            ) {
                Text("This is a rich tooltip containing detailed information and an action.")
            }
        },
        state = tooltipState
    ) {
        IconButton(onClick = { /* Action */ }) {
            Icon(
                imageVector = Icons.Filled.Info,
                contentDescription = "Detailed Information"
            )
        }
    }
}
```
