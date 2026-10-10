# VS-Code Keybindings

## File navigation (local)

`Ctrl + G` go to line number

`Alt + PgUp/PgDn` Scroll page up/down

`Ctrl + ↑/↓` Scroll line up/down

`Home/End` Go to beginning/end of **line**

`Ctrl + Home/End` Go to beginning/end of **file**

`Ctrl + Shift + \` Jump to matching bracket

`Alt + Enter` Select all occurrences of Find match

`Alt + ←` Go Back to last edit (custom, see overrides below; by default `Alt + ←` is Go Back to previous cursor position)

`Alt + →` Go Forward to next edit (custom, see overrides below)


`Alt + ,` Go Back to previous cursor position (custom)

`Alt + .` Go Forward to next cursor position (if available) (custom)

## File manipulation

`Alt + ↑/↓` Move line up/down

`Ctrl + ]/[` Indent/outdent line

## Formatting

`Ctrl + Shift + I` - format with current formatter (Linux default; on Windows it's `Shift + Alt + F`)

## Suggestions / Completion / Hints

`Ctrl + Space`, `Ctrl + I` Trigger suggestion

`Ctrl + Shift + Space` Trigger parameter hints

## All Files Definitions (global)

`F2` Rename Symbols

`F12` Go to Symbol Definition

`Ctrl + F12` Go to Symbol Implementation

`Ctrl + T` Search/Show all Symbols

## Override Keybindings

```json
// keybindings.json

// Place your key bindings in this file to override the defaults:
// Ctrl+P then search Keyboard Shortcuts (JSON) and select it
// copy paste then save this json

[
    {
        "key": "alt+left",
        "command": "workbench.action.navigateBackInEditLocations",
        "when": "canNavigateBackInEditLocations"
    },
    {
        "key": "alt+right",
        "command": "workbench.action.navigateForwardInEditLocations",
        "when": "canNavigateForwardInEditLocations"
    },
    {
        "key": "alt+,",
        "command": "workbench.action.navigateBack",
        "when": "canNavigateBack"
    },
    {
        "key": "alt+.",
        "command": "workbench.action.navigateForward",
        "when": "canNavigateForward"
    }
]
```
