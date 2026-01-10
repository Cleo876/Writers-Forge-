# Writer's Forge Extension Developer Manual

Welcome to the **Writer's Forge Extension System**. This guide provides the standards and best practices for creating `.js` extensions that enhance the capabilities of the editor.

Extensions in Writer's Forge are lightweight JavaScript modules injected at runtime. They allow you to manipulate the DOM, interact with the application state, modify CSS, or add entirely new features.

## 1. The Core Mechanism

Writer's Forge exposes a global `forge` object. Your extension script communicates with the application by calling `forge.register()`.

When a user loads your `.js` file, the system executes it immediately. Your script **must** call `register` with a configuration object to be recognized and saved to the user's database.

### Basic Structure

```javascript
// my_awesome_extension.js

forge.register({
    name: "My Awesome Tool",
    version: "1.0.0",
    developer: "Your Name", // Vital for attribution!
    description: "Adds a cool feature to the editor.",
    onEnable: function() {
        console.log("Extension Enabled!");
        // Your activation logic here
    },
    onDisable: function() {
        console.log("Extension Disabled!");
        // Your cleanup logic here
    }
});
```

## 2. Configuration Object Reference

The configuration object passed to `forge.register` supports the following properties:

| Property | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| **`name`** | String | Yes | The display name of your extension in the manager. |
| **`version`** | String | Yes | Semantic versioning string (e.g., "1.0.5"). |
| **`developer`** | String | **Yes** | **Your Name or Brand.** Displayed in the UI to credit you. |
| **`description`** | String | No | A short summary of what the tool does. |
| **`onEnable`** | Function | Yes | Executed when the user loads the extension or toggles it ON. |
| **`onDisable`** | Function | Yes | Executed when the user toggles the extension OFF. |
| **`id`** | String | No | Optional unique identifier. If omitted, the system generates one. |

### The `developer` Tag
**Crucial:** Always include the `developer` field. This ensures that whether a user purchased your tool or downloaded it for free, your contribution is visibly recognized in the Extensions Manager.

## 3. Creating Your First Extension

Here is a complete example of a functional extension. This script adds a "Word Goal" tracker to the top of the editor.

```javascript
forge.register({
    name: "Daily Word Goal",
    version: "1.1.0",
    developer: "Cleon",
    description: "Displays a simple progress bar for a 500-word daily goal.",

    onEnable: function() {
        // 1. Create the UI element
        const goalBar = document.createElement('div');
        goalBar.id = 'ext-word-goal';
        goalBar.style.position = 'fixed';
        goalBar.style.top = '60px'; // Below navbar
        goalBar.style.right = '20px';
        goalBar.style.background = 'white';
        goalBar.style.padding = '10px';
        goalBar.style.border = '1px solid #ccc';
        goalBar.style.borderRadius = '8px';
        goalBar.style.zIndex = '1000';
        goalBar.style.boxShadow = '0 4px 6px rgba(0,0,0,0.1)';
        goalBar.innerHTML = `
            <div style="font-size:12px; font-weight:bold; color:#555;">DAILY GOAL</div>
            <div style="width:100px; height:6px; background:#eee; margin-top:5px; border-radius:3px;">
                <div id="ext-goal-fill" style="width:0%; height:100%; background:#4f46e5; border-radius:3px; transition:width 0.3s;"></div>
            </div>
            <div id="ext-goal-text" style="font-size:10px; color:#888; margin-top:3px;">0 / 500</div>
        `;
        document.body.appendChild(goalBar);

        // 2. Define the logic
        this.updateGoal = () => {
            const text = document.body.innerText || "";
            const words = text.split(/\s+/).length;
            const percentage = Math.min(100, (words / 500) * 100);
            
            const fill = document.getElementById('ext-goal-fill');
            const label = document.getElementById('ext-goal-text');
            
            if(fill) fill.style.width = percentage + '%';
            if(label) label.innerText = `${words} / 500`;
        };

        // 3. Attach listeners
        document.addEventListener('input', this.updateGoal);
        
        // Initial run
        this.updateGoal();
    },

    onDisable: function() {
        // CLEANUP IS CRITICAL
        const el = document.getElementById('ext-word-goal');
        if(el) el.remove();
        
        // Remove listeners to prevent memory leaks
        document.removeEventListener('input', this.updateGoal);
    }
});
```

## 4. Best Practices

1.  **Clean Up:** Your `onDisable` function must remove *everything* your extension added. If you added event listeners, styles, or DOM elements, remove them.
2.  **Scoping:** Avoid declaring global variables (`var x = 10`). Keep logic inside your `onEnable` function or attached to the extension object (`this.myFunc = ...`).
3.  **Selectors:** When targeting editor elements, use the class `.script-element` or the ID `#editor-container`. Be aware that Writer's Forge uses Tailwind CSS, so standard utility classes are available for your use.
4.  **Error Handling:** Wrap risky code in `try...catch` blocks so your extension doesn't crash the main application.

## 5. Distributing Your Extension

Simply save your code as a `.js` file (e.g., `word_goal.js`). Users can load it directly via the **Extensions Manager > Load Custom Extension** button in Writer's Forge.

Happy Coding!
    