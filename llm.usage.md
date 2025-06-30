# Quick Guide: @thinkeloquent/cli-progressor` Progress Bar Library

### Key Imports

```js
import {
  ProgressBar, // The core progress bar class
  ProgressBarBuilder, // For creating a customized bar
  CLIProgressHelper, // Helper for wrapping loops with a progress bar
  MultiProgressManager, // For managing multiple progress bars at once
  ProgressRenderer, // Base class for creating custom output formats
  Colors, // For adding color to terminal text
} from "@thinkeloquent/cli-progressor";
```

---

### 1. Basic Progress Bar

- **`new ProgressBar(total, description)`**: Creates a new progress bar instance.
- **`.start()`**: Displays the bar in the console.
- **`.update(increment)`**: Advances the bar's progress by a given amount.

```js
const bar = new ProgressBar(100, "Downloading..."); // Create a bar for 100 items.
bar.start(); // Display the bar.
bar.update(10); // Advance progress by 10.
```

### 2. Customized Bar (Builder Pattern)

- **`ProgressBarBuilder`**: A class to build a `ProgressBar` with custom options.
- **`.with...()` / `.show...()`**: Chainable methods to set configuration.
- **`.build()`**: Creates the final `ProgressBar` instance.

```js
const customBar = new ProgressBarBuilder()
  .withTotal(500) // [Method] Sets the total value.
  .withDescription("Processing data") // [Method] Sets the description text.
  .withBarLength(40) // [Method] Sets the visual width of the bar.
  .showETA(true) // [Method] Toggles the ETA display.
  .build(); // [Method] Creates the configured bar instance.

customBar.start(); // Start the custom bar.
```

### 3. Automatic Progress for Loops

- **`CLIProgressHelper.withProgress(total, desc, taskFn)`**: Wraps an async task with a progress bar, handling start/update/stop automatically.
- **`taskFn(updateProgress)`**: Your async function where you call `updateProgress(1)` after each step.

```js
const files = ["a.txt", "b.txt", "c.txt"];
await CLIProgressHelper.withProgress(
  files.length, // The total number of items to process.
  "Processing files", // The description for the progress bar.
  async (update) => {
    // Your task function receives an `update` callback.
    for (const file of files) {
      await processFile(file); // Do your work for one item.
      update(1); // Increment the progress bar.
    }
  }
);
```

### 4. Multiple Progress Bars

- **`MultiProgressManager`**: Manages and displays several progress bars at once.
- **`.add(id, total, description)`**: Adds a new bar to the manager and returns it for individual updates.

```js
const manager = new MultiProgressManager(); // Create a manager instance.
const bar1 = manager.add("dl1", 100, "File 1"); // Add the first bar.
const bar2 = manager.add("dl2", 200, "File 2"); // Add the second bar.

bar1.update(50); // Update bars individually as needed.
bar2.update(25);
```

### 5. Reacting to Progress (Observers)

- **`.onProgress(callback)`**: Attaches a function to run on every single update.
- **`callback(progressData)`**: Receives an object with details like `percentage`, `speed`, and `isComplete`.

```js
const bar = new ProgressBar(100, "Observing Progress")
  .onProgress((data) => {
    // Attach an observer function.
    if (data.percentage === 50) console.log("\nReached 50%!");
  })
  .onProgress((data) => {
    // You can chain multiple observers.
    if (data.isComplete) console.log("\nDone!");
  });

bar.start();
// In your loop: bar.update(1);
```

### 6. Custom Output (Renderer)

- **`ProgressRenderer`**: Extend this base class to create your own output format (e.g., JSON, spinner).
- **`render(progressData)`**: Required method to define how to display the progress.
- **`cleanup()`**: Optional method that runs after the progress is complete.

```js
class JsonRenderer extends ProgressRenderer {
  // Extend the base class.
  render(data) {
    // Implement the `render` method for custom output.
    console.log(JSON.stringify({ percent: data.percentage }));
  }
}

// Pass an instance of your custom renderer to the ProgressBar constructor.
const jsonBar = new ProgressBar(100, "JSON Output", new JsonRenderer());
jsonBar.start();
```

### 7. Utilities

- **`Colors`**: An object for easily adding color to terminal text.

```js
console.log(Colors.success("✓ Operation successful!")); // Green text.
console.log(Colors.error("✗ An error occurred.")); // Red text.
console.log(Colors.warning("⚠️ Warning.")); // Yellow text.
console.log(Colors.info("ℹ️ Information.")); // Blue text.
```
