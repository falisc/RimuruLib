# Rimuru UI Documentation

This document describes the current `RimuruUIInternal/UI.lua` API and the example usage in `Example.lua`.

## Loading

```lua
local UI = loadstring(game:HttpGet("https://fal.lol/raw/5WLgR7ThiV"))()
```

`UI.lua` returns the library table. The library writes its bundled icon and theme assets through the executor asset functions when a window is created. A runtime with `writefile`, `makefolder`, and either `getcustomasset` or `getsynasset` is required for local image assets.

## Short Creation Example

```lua
local window = UI:CreateWindow({
    Title = "Rimuru.temp",
    Theme = "Rimuru Hub",
    Width = 680,
    Height = 460,
})

local tab = window:CreateTab("Main", "target")
local section = tab:CreateSection("General", {Column = "left"})
section:CreateCheckbox({Name = "Enabled", Default = true})
```

## Library API

### `UI:CreateWindow(config)`

Creates and returns a `Window` object. Existing `RimuruUI_Internal` UI is removed before the new window is created.

Supported configuration fields:

| Field | Type | Default | Description |
| --- | --- | --- | --- |
| `Title` | string | `"Rimuru.temp"` | Window title and watermark title. |
| `Width` | number | `680` | Normal window width in pixels. |
| `Height` | number | `460` | Normal window height in pixels. |
| `RailWidth` | number | 27% of width | Width of the left navigation rail. |
| `Theme` | string | `"Rimuru Dark"` | Initial theme name. |
| `ConfigFolder` | string | `"RimuruConfigs"` | Folder used for JSON configuration files. |
| `ConfigName` | string | empty | Initial config name and autoload target. |
| `AutoSave` | boolean | `false` | Saves changed widget state automatically. |
| `UIKeybind` | string | `"RightAlt"` | Keyboard key used to show or hide the UI. |
| `FloatingIsland` | boolean | `true` | Enables the watermark/minimized floating island. |
| `Screensaver` | boolean | `false` | Enables the opaque full-screen theme wallpaper mode. |
| `IconData` | table | bundled icons | Optional path-to-base64 asset map written before UI creation. |

The default window size is `680 × 460`.

### `UI:ShowLoader(config)`

Shows the themed loading panel and removes it after the requested duration.

```lua
UI:ShowLoader({
    Title = "Rimuru.temp",
    Status = "Loading interface...",
    Duration = 1.5,
    Theme = "Rimuru Hub",
})
```

Fields are `Title`, `Status`, `Duration`, `Theme`, and optional `IconData`. The loader uses the selected theme palette, including the matching outline and accent colors.

### `UI:Notify(title, message, duration)`

Creates a notification for the most recently created window. Notifications use the floating island surface when the island is enabled and visible; otherwise they appear as regular overlay cards.

### `UI:SetTheme(values)`

Applies direct palette overrides to every existing window. Supported palette keys are `Background`, `Surface`, `Card`, `Hover`, `Border`, `Text`, `Muted`, `Disabled`, `Divider`, `Accent`, `Selected`, `Sidebar`, `SidebarTransparency`, `WindowTransparency`, `SurfaceTransparency`, `CardTransparency`, `SectionTransparency`, `BackgroundImage`, and `BackgroundImageTransparency`.

For selecting one of the built-in themes by name, use `window:SetTheme(name)`.

## Current Themes

| Theme | Style | Background asset |
| --- | --- | --- |
| `Rimuru Dark` | Opaque dark UI with no wallpaper. | None |
| `Rimuru Hub` | Light blue frosted UI. | `RimuruAssets/ThemeAssets/rimurubackground.jpg` |
| `Rimuru Forest` | Dark green forest palette using the original drawing UI colors. | `RimuruAssets/ThemeAssets/Forest.jpg` |
| `Rimuru Crimson` | Dark red/crimson palette using the original drawing UI colors. | `RimuruAssets/ThemeAssets/Crimson.jpg` |

Theme wallpapers use 50% image transparency inside the UI. Screensaver mode uses the selected theme wallpaper at 0% image transparency and covers the full screen. Theme changes also update the main outline, loader, watermark, keybind overlay, controls, and widget surfaces.

## Window API

### `window:CreateTab(name, icon, config)`

Creates and returns a `Tab` object.

```lua
local tab = window:CreateTab("Visuals", "target", {
    Group = "Utility",
})
```

Fields:

- `name`: visible tab label.
- `icon`: one of the registered icon names below.
- `Group`: optional uppercase navigation group label.
- `Bottom`: optional boolean that places the tab in the bottom navigation area.

The active tab is a label state rather than an interactive button while selected. Use `window:SelectTab(tab)` to select a tab in code.

### `window:SelectTab(tab)`

Selects a tab and displays its sections.

### `window:SetTheme(name)`

Selects one of the four built-in themes by exact name. Unknown names fall back to `Rimuru Dark`.

### `window:SetVisible(value)`

Shows or hides the normal UI with the spring animation. The UI keybind calls this method.

### `window:SetMinimized(value)`

Minimizes or restores the window. When `FloatingIsland` is enabled, minimizing transfers the visible controls to the floating island and hides the full window.

### `window:SetWatermarkVisible(value)`

Enables or disables the floating island/watermark. Existing minimized state and notifications are relaid out automatically.

### `window:SetKeybindOverlayVisible(value)`

Enables or disables the keybind overlay.

### `window:SetDarkeningEnabled(value)`

Enables or disables the darkening layer behind the UI.

### `window:SetRainEnabled(value)`

Enables or disables the rain effect.

### `window:SetScreensaverEnabled(value)`

Enables or disables full-screen opaque wallpaper mode. It is shown only for themes that have a background asset.

### `window:SetUIToggleKeybind(key)`

Changes the UI visibility key. Keys use Roblox `Enum.KeyCode` names such as `RightAlt`, `F`, or `Insert`. `Backspace`, `None`, and `Unknown` become `Unbound`.

### `window:Filter(query)`

Filters widgets by widget name. Empty sections are hidden while filtering and restored when the query is cleared.

### `window:SaveConfig(name)`

Writes the current theme, Auto Save state, and widget values to a JSON file. The optional name overrides the current config name.

### `window:LoadConfig(name)`

Loads a JSON config by name and applies its theme, Auto Save state, widget values, keybinds, and color transparency.

### `window:CreateConfig(name)`

Sets an optional config name and saves the current state.

### `window:RefreshConfigs()`

Refreshes the Auto Load dropdown from the config index and available JSON files.

### `window:DeleteConfig(name)`

Deletes a named config and updates the Auto Load list.

### `window:ClosePopup()`

Closes the currently open dropdown, keybind mode menu, or color picker popup.

### `window:Destroy()`

Disconnects input, animation, rain, and overlay connections, cancels active tweens, and removes the entire UI.

## Tabs and Sections

### `tab:CreateSection(title, config)`

Creates and returns a `Section` object. Sections are frosted cards in image themes and are placed in two columns.

```lua
local section = tab:CreateSection("Targeting", {
    Column = "left",
})
```

`Column` accepts `"left"` or `"right"`. `Mode` is accepted as an alias for `Column`.

The returned section exposes `section.SetCollapsed(true or false)` for scripted collapse/restore behavior.

## Widgets

Every widget returns a handle containing at least `Name`, `Root`, and `Label`. Most value widgets also expose `Value` and `Set`.

### `section:CreateLabel(name)`

Creates muted informational text. It has no callback and no editable value.

### `section:CreateButton(config)`

Creates an action button.

```lua
section:CreateButton({
    Name = "Run action",
    Text = "Click",
    Icon = "lightning",
    Callback = function()
        print("clicked")
    end,
})
```

Fields:

- `Name`: widget label.
- `Text`: button text; defaults to `"Click"`.
- `Icon`: optional icon name. When present, the icon is displayed on the button.
- `Callback`: called with no arguments.

A string can be passed instead of a table; it becomes the widget name.

### `section:CreateCheckbox(config)`

Creates a boolean checkbox.

```lua
local enabled = section:CreateCheckbox({
    Name = "Enabled",
    Default = false,
    Keybind = {
        Default = "F",
        Mode = "TOGGLE",
    },
    Callback = function(value)
        print(value)
    end,
})
```

Fields:

- `Name`: widget label.
- `Default`: initial boolean value.
- `Keybind`: optional string or table. Table fields are `Default` or `Key`, and `Mode`.
- `Callback`: receives the new boolean value.

The handle exposes `Set(value, silent)`, `Value`, `Box`, `Row`, and optional `Keybind`. `Section:CreateToggle` is an alias for `CreateCheckbox`.

### `section:CreateSlider(config)`

Creates a numeric slider with an editable value field.

```lua
local distance = section:CreateSlider({
    Name = "Distance",
    Min = 0,
    Max = 500,
    Default = 200,
    Step = 1,
    Callback = function(value)
        print(value)
    end,
})
```

Fields:

- `Name`: widget label.
- `Min`: lower bound; defaults to `0`.
- `Max`: upper bound; defaults to `100`.
- `Default`: initial value; defaults to `Min`.
- `Step`: rounding increment; defaults to `1`.
- `Increment`: alias for `Step`.
- `Callback`: receives the numeric value.

The handle exposes `Set(value, silent)` and `Value`.

### `section:CreateDropdown(config)`

Creates a searchable dropdown. The popup supports mouse selection and, for multi-select lists, Select All and Unselect All.

```lua
local mode = section:CreateDropdown({
    Name = "Mode",
    Default = "Nearest",
    Options = {"Nearest", "Visible", "Random"},
    Callback = function(value)
        print(value)
    end,
})
```

Fields:

- `Name`: widget label.
- `Default`: initial option, or an array when `Multi = true`.
- `Options`: array of selectable values.
- `Multi`: boolean; enables multiple selection.
- `Callback`: receives the selected option for single-select, or an array for multi-select.

The handle exposes `Set(value)`, `SetOptions(options)`, and `Value`. `SetOptions` removes selections that no longer exist in the option list.

### `section:CreateKeybind(config)`

Creates a key assignment widget and registers it with the keybind overlay.

```lua
local bind = section:CreateKeybind({
    Name = "Target Key",
    Default = "G",
    Mode = "TOGGLE",
    ShowInOverlay = true,
    Callback = function(key)
        print(key)
    end,
})
```

Fields:

- `Name`: widget label.
- `Default`: Roblox key name; `Backspace` becomes `Unbound`.
- `Mode`: `TOGGLE`, `HOLD`, `ALWAYS`, or `CLICK`. Matching is case-insensitive.
- `ShowInOverlay`: defaults to `true`.
- `Callback`: called with the newly assigned `Enum.KeyCode`.

Right-clicking the key field opens the mode menu. The handle exposes `Set`-style button data through `Button`, `Icon`, and `Keybind`; the key entry exposes `KeyName`, `Mode`, `Active`, and `SetActive(value)`.

### `section:CreateColorPicker(config)`

Creates a color picker with hue, saturation, brightness, hexadecimal input, copy support, and transparency control.

```lua
local color = section:CreateColorPicker({
    Name = "Accent Color",
    Default = Color3.fromRGB(83, 178, 224),
    Transparency = 0,
    Callback = function(value, transparency)
        print(value, transparency)
    end,
})
```

Fields:

- `Name`: widget label.
- `Default`: initial `Color3`.
- `Transparency`: initial transparency from `0` to `1`.
- `Callback`: receives `(Color3, transparency)`.

The handle exposes `Set(color, silent)`, `SetTransparency(value, silent)`, `Value`, `Swatch`, `Stroke`, and `Transparency`.

### `section:CreateInput(config)`

Creates a single-line text input.

```lua
local label = section:CreateInput({
    Name = "Label Text",
    Placeholder = "Enter text...",
    Default = "Ready",
    Callback = function(value)
        print(value)
    end,
})
```

Fields:

- `Name`: widget label.
- `Placeholder`: hint text; defaults to `"Type here..."`.
- `Default`: initial string.
- `Callback`: called when focus leaves the input and receives the string value.

The handle exposes `Set(value, silent)`, `Value`, and `Input`.

## Icon Names

Icon names are case-insensitive. Pass them as the second argument to `CreateTab` or as `Icon` in `CreateButton`.

### Bundled raster icons

`logo`, `home`, `settings`, `visual`, `eye`, `eyeoff`, `search`, `minimize`, `close`, `combat`, `assist`, `keyboard`, `star`, `click`, `player`, `location`, `skull`, `target`, and `lightning`.

The four added logos are stored at:

- `RimuruAssets/Logos/location.png`
- `RimuruAssets/Logos/skull.png`
- `RimuruAssets/Logos/target.png`
- `RimuruAssets/Logos/lightning.png`

### Procedural fallback icons

`info`, `terminal`, and `dot` are generated by the library when no raster asset is registered. Unknown names return an empty icon frame.

## Example Layout

```lua
local window = UI:CreateWindow({
    Title = "Rimuru.temp",
    Width = 680,
    Height = 460,
    Theme = "Rimuru Hub",
    ConfigFolder = "RimuruConfigs",
    ConfigName = "ViolenceDistrict",
})

local tab = window:CreateTab("Visuals", "target", {Group = "Utility"})
local section = tab:CreateSection("Targeting", {Column = "left"})
section:CreateCheckbox({Name = "Enabled", Default = true})
section:CreateButton({Name = "Mark target", Icon = "lightning"})
```

## Credits

All credits: catscure on Discord.
