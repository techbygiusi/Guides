# VSCode Custom Theme Tutorial

In this tutorial, you'll learn how to create your own custom theme for Visual Studio Code using the `settings.json` file. This file allows you to personalize the colors and appearance of various components in VSCode.

## Step 1: Open VSCode Settings

1. Open Visual Studio Code.
2. Go to the menu and select **File** > **Preferences** > **Settings** (or use `Ctrl+,`).
3. On the top-right corner, click on the **{} Open Settings (JSON)** icon.

This will open the `settings.json` file, where you'll add your custom theme settings.

## Step 2: Add Your Custom Theme to `settings.json`

Copy the following settings and paste them into your `settings.json` file:

```json
{
    "files.autoSave": "onFocusChange",
    "editor.fontFamily": "Dank Mono",
    "editor.fontSize": 20,
    "editor.cursorBlinking": "phase",
    "workbench.sideBar.location": "right",
    "workbench.tips.enabled": false,
    "editor.cursorSmoothCaretAnimation": "on",

    "workbench.colorCustomizations": {
        // Editor colors
        "editor.background": "#F4FCE3", // Light green editor background
        "editor.foreground": "#2C6E49", // Dark green text
        "editorLineNumber.foreground": "#A3B18A", // Line numbers
        "editorCursor.foreground": "#52734D", // Cursor color
        "editor.selectionBackground": "#C3E6CB", // Selection highlight
        "editor.lineHighlightBackground": "#E9F5DB", // Change to your desired background color
        "editor.lineHighlightBorder": "#2C6E49",     // Optional: Add a border around the highlighted line

        // Sidebar
        "sideBar.background": "#E9F5DB", // Sidebar background
        "sideBar.foreground": "#2C6E49", // Sidebar text
        
        // Activity bar
        "activityBar.background": "#D9E6CF", // Activity bar background
        "activityBar.foreground": "#52734D", // Icons and text

        // Tabs
        "tab.activeBackground": "#F4FCE3", // Light green for active tab
        "tab.activeForeground": "#2C6E49", // Dark green text for active tab
        "tab.inactiveBackground": "#E9F5DB", // Lighter green for inactive tabs
        "tab.inactiveForeground": "#A3B18A", // Muted green for inactive tab text
        "tab.hoverBackground": "#C3E6CB", // Color when hovering over a tab
        "tab.hoverForeground": "#2C6E49", // Text color when hovering over a tab
        "tab.border": "#2C6E49", // Ensure borders are consistent
        
        // Title bar
        "titleBar.activeBackground": "#D9E6CF", // Title bar background
        "titleBar.activeForeground": "#2C6E49", // Title bar text
        "titleBar.inactiveBackground": "#E9F5DB", // Inactive title bar background
        "titleBar.inactiveForeground": "#A3B18A", // Inactive title bar text
        
        // Status bar (bottom bar)
        "statusBar.background": "#D9E6CF", // Status bar background
        "statusBar.foreground": "#2C6E49", // Status bar text
        "statusBar.debuggingBackground": "#A3B18A", // Debugging mode background
        "statusBar.debuggingForeground": "#2C6E49", // Debugging mode text
        "statusBar.noFolderBackground": "#E9F5DB", // Background when no folder is open
        "statusBar.noFolderForeground": "#2C6E49", // Text color when no folder is open

        // Panel (e.g., terminal, output)
        "panel.background": "#E9F5DB", // Panel background
        "panel.border": "#A3B18A", // Panel border
        "panelTitle.activeForeground": "#2C6E49", // Active panel title
        "panelTitle.inactiveForeground": "#A3B18A", // Inactive panel title
        
        // General borders and highlights
        "contrastBorder": "#C3E6CB", // Borders for better contrast
        "focusBorder": "#52734D", // Focus indicator

        // Hover and button customizations
        "button.background": "#C3E6CB", // Button background
        "button.foreground": "#2C6E49", // Button text
        "button.hoverBackground": "#A3B18A", // Button hover background
        "dropdown.background": "#E9F5DB", // Dropdown menu background
        "dropdown.foreground": "#2C6E49", // Dropdown menu text
        "dropdown.border": "#A3B18A", // Dropdown border
        "menu.background": "#E9F5DB", // Menu background
        "menu.foreground": "#2C6E49", // Menu text
        "menu.selectionBackground": "#C3E6CB", // Menu hover background
        "menu.selectionForeground": "#2C6E49", // Menu hover text
        "menu.selectionBorder": "#52734D", // Border for menu hover

        // Editor Tabs and Editor Group colors
        "editorGroup.background": "#F4FCE3", // Background of the area where open editors are shown
        "editorGroup.border": "#A3B18A", // Border of the editor group
        "editorGroupHeader.tabsBackground": "#F4FCE3", // Background of the header where tabs are listed
        "editorGroupHeader.border": "#A3B18A", // Border of the header

        // Input field background and text
        "input.background": "#F4FCE3", // Input field background (matches editor background)
        "input.foreground": "#2C6E49", // Input field text color (dark green)
        "input.placeholderForeground": "#A3B18A", // Placeholder text color
        "input.border": "#A3B18A", // Border color for input fields
        "input.focusBorder": "#52734D", // Border color when input field is focused
        "input.activeBackground": "#C3E6CB", // Background color of active input fields
        "inputOption.activeBackground": "#C3E6CB", // Background for active options (e.g., checkboxes)
        "inputValidation.errorBackground": "#FFEEEE", // Error background color
        "inputValidation.errorBorder": "#FF0000", // Error border color
        "inputValidation.infoBackground": "#C3E6CB", // Info background color
        "inputValidation.infoBorder": "#2C6E49", // Info border color
        "inputValidation.warningBackground": "#FFF0A1", // Warning background color
        "inputValidation.warningBorder": "#FFA500" // Warning border color
    },
    "workbench.colorTheme": "Default Light+",
    "workbench.startupEditor": "none",

    "editor.tokenColorCustomizations": {
        "textMateRules": [
            {
                "scope": "comment",
                "settings": {
                    "foreground": "#B4C6A6", // Pastel green for comments
                    "fontStyle": "italic"   // Makes comments italic (optional)
                }
            },
            {
                "scope": "keyword",
                "settings": {
                    "foreground": "#D89BBA", // Pastel pink for keywords
                    "fontStyle": "bold"     // Makes keywords bold (optional)
                }
            },
            {
                "scope": "string",
                "settings": {
                    "foreground": "#A3D8D1" // Pastel teal for strings
                }
            },
            {
                "scope": "variable",
                "settings": {
                    "foreground": "#C8D5B9" // Light pastel green for variables
                }
            },
            {
                "scope": "function",
                "settings": {
                    "foreground": "#A8C3D9", // Pastel blue for function names
                    "fontStyle": "bold"     // Makes functions bold (optional)
                }
            },
            {
                "scope": "number",
                "settings": {
                    "foreground": "#E5C3C6" // Soft pink for numbers
                }
            },
            {
                "scope": "type",
                "settings": {
                    "foreground": "#E7D3A6" // Pastel yellow for data types
                }
            },
            {
                "scope": "operator",
                "settings": {
                    "foreground": "#C8A2C8" // Soft lavender for operators
                }
            },
            {
                "scope": "constant",
                "settings": {
                    "foreground": "#B8CEDA" // Light blue for constants
                }
            }
        ]
    },
    "workbench.editor.empty.hint": "hidden"
}
```

## Step 3: Save Your Settings

After pasting the code, save your `settings.json`. Your new theme will take effect immediately.

## Conclusion

By following these steps, you've created a custom theme for Visual Studio Code. Feel free to modify the colors to suit your preferences, and enjoy a personalized coding experience!

## Images with my example `settings.json`
![Screenshot 2024-12-13 200348](https://github.com/user-attachments/assets/0c3516ed-c91f-4553-b861-b81e7088c9c0)

![image](https://github.com/user-attachments/assets/d500d884-779a-4523-a3ca-9a60a06a3bd4)

