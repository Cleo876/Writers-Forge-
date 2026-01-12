# Writer's Forge Extension Developer Manual (v1.5.5)

**Welcome to the Forge.**

This guide is the definitive manual for building extensions for **Writer's Forge v1.5.5**. Whether you are a seasoned developer or writing your first script, this document will teach you how to inject custom tools, analytics, and UI elements directly into the editor.

## 1. How Extensions Work

Writer's Forge is a lightweight, single-file application. Extensions are JavaScript modules injected at runtime.

When you load an extension, the Forge does two things:
1.  **Registers it:** Saves your code to the user's browser database (IndexedDB).
2.  **Executes it:** Runs your code immediately.

**Key Concept:** Your script communicates with the application through the global `forge` object.

---

## 2. The Blueprint: `forge.register()`

Every extension **must** start by calling `forge.register()`. This function takes a "Configuration Object" that tells the Forge what your tool is and what it does.

### The Configuration Object

| Property | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| **`name`** | String | **Yes** | The title of your tool (e.g., "Pacing Tracker"). |
| **`version`** | String | **Yes** | The version number (e.g., "1.0.0"). |
| **`developer`** | String | **Yes** | **Your Name.** This gives you credit in the UI. |
| **`description`** | String | No | A short summary of what the tool does. |
| **`onEnable`** | Function | **Yes** | **The Start Button.** Runs when the extension is turned ON. |
| **`onDisable`** | Function | **Yes** | **The Kill Switch.** Runs when the extension is turned OFF. |

### Basic Skeleton Code

Copy this code to start a new extension:

```javascript
forge.register({
    name: "My First Extension",
    version: "1.0.0",
    developer: "Mr. Williams",
    description: "Logs a message to the console.",

    onEnable: function() {
        console.log("System Online.");
        // Your logic goes here
    },

    onDisable: function() {
        console.log("System Offline.");
        // Cleanup logic goes here
    }
});
```

---

## 3. Accessing the Engine (Advanced)

This is the most important section for making powerful tools. **Writer's Forge v1.5.5** exposes its internal state globally. You can access the script data directly.

### A. `window.store` (The Database)
This is the source of truth. It contains the screenplay data.

* **`window.store.blocks`**: An array of objects representing every line in the script.
    * *Example:* `{ id: "uuid...", type: "type-scene", content: "INT. HOUSE - DAY" }`
* **`window.store.characters`**: An array of all character names detected.
* **`window.store.scenes`**: An array of all scene headings.

### B. `window.editor` (The Interface)
This controls the visual editor.

* **`window.editor.render()`**: Forces the editor to redraw the script. Call this if you modify data.
* **`window.editor.calculateStats()`**: Triggers the Pulse Engine to recount words and timing.

### C. The DOM (The Visuals)
Writer's Forge uses specific HTML classes you can target:

* **`#editor-container`**: The main scrollable area containing the pages.
* **`.script-element`**: The class applied to every line of text.
* **Block Types:**
    * `.type-scene` (Scene Headings)
    * `.type-action` (Action lines)
    * `.type-character` (Character Names)
    * `.type-dialogue` (Dialogue)
    * `.type-parenthetical` (Wrylies)
    * `.type-transition` (Transitions)

---

## 4. Tutorial: Building a "Focus Mode" Button

Let's build an extension that adds a floating button to toggle a "High Contrast" mode.

### Step 1: The Setup
Create a new file named `focus_button.js`. Add the registration block.

### Step 2: The Logic (`onEnable`)
We need to create a button, style it, and add it to the screen.

```javascript
onEnable: function() {
    // 1. Create the button
    const btn = document.createElement('button');
    btn.innerText = "FOCUS";
    btn.id = "my-focus-btn"; // Give it an ID so we can find it later!
    
    // 2. Style it (using CSS strings or standard JS styles)
    btn.style.position = "fixed";
    btn.style.bottom = "20px";
    btn.style.right = "20px";
    btn.style.zIndex = "1000";
    btn.style.padding = "10px 20px";
    btn.style.background = "red";
    btn.style.color = "white";
    btn.style.borderRadius = "5px";
    btn.style.border = "none";
    btn.style.cursor = "pointer";
    
    // 3. Add functionality
    btn.onclick = function() {
        alert("Focus Mode Clicked! (You could inject CSS here)");
    };

    // 4. Inject into the page
    document.body.appendChild(btn);
}
```

### Step 3: The Cleanup (`onDisable`)
**Crucial:** If you do not remove the button in `onDisable`, it will stay on the screen even after the user disables your extension.

```javascript
onDisable: function() {
    const btn = document.getElementById("my-focus-btn");
    if (btn) {
        btn.remove(); // Delete the element
    }
}
```

---

## 5. Style & Dark Mode

Writer's Forge v1.5.5 has a robust **Dark Mode**. If you create UI elements (like panels or bars), you should respect the user's theme.

**The `dark-mode` Class:**
The `<body>` tag will have the class `dark-mode` if the user has enabled it.

**Example Check:**
```javascript
function applyTheme(myElement) {
    if (document.body.classList.contains('dark-mode')) {
        myElement.style.background = "#1f2937"; // Dark gray
        myElement.style.color = "#f3f4f6";      // Light text
    } else {
        myElement.style.background = "#ffffff"; // White
        myElement.style.color = "#1f2937";      // Dark text
    }
}
```

---

## 6. Best Practices (Do's and Don'ts)

* **DO** use unique IDs for your elements (e.g., `ext-my-tool-panel`) so you can easily find and remove them.
* **DO** use `try...catch` blocks. If your code crashes, you don't want to crash the whole editor.
* **DON'T** rely on global variables (`var x = 10`). Keep variables inside your `onEnable` function or attach them to `this` (e.g., `this.myCounter = 0`).
* **DON'T** edit the `window.store.blocks` array directly unless you are sure to call `window.editor.render()` immediately after.

---

## 7. How to Install

1.  Save your code as a `.js` file.
2.  Open **Writer's Forge**.
3.  Click the **Extensions Icon** (Puzzle Piece) in the toolbar.
4.  Click **+ Load Custom Extension**.
5.  Select your file.

Happy Coding.
