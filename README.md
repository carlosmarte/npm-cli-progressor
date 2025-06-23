# @thinkeloquent/cli-progressor

# Instruction Prompt: How to Use the `@thinkeloquent/cli-progressor` Progress Bar Library

This document provides a comprehensive guide on using the progress bar utilities exported from `@thinkeloquent/cli-progressor`. It covers everything from basic usage to advanced patterns like custom renderers, multi-bar management, and error handling.

### Table of Contents

1.  [**Basic Usage**](#1-basic-usage): Create a simple progress bar.
2.  [**Advanced Configuration with the Builder Pattern**](#2-advanced-configuration-with-the-builder-pattern): Customize your progress bar's appearance and behavior.
3.  [**Handling Multiple Progress Bars**](#3-handling-multiple-progress-bars): Manage several progress bars simultaneously.
4.  [**Using the `CLIProgressHelper` for Common Tasks**](#4-using-the-cliprogresshelper-for-common-tasks): Easily wrap asynchronous tasks with a progress bar.
5.  [**Responding to Events with Observers**](#5-responding-to-events-with-observers): Execute code at different stages of progress.
6.  [**Extending Functionality: Custom Renderers**](#6-extending-functionality-custom-renderers): Change the entire output format (e.g., to JSON or a themed spinner).
7.  [**Building Robust CLI Tools**](#7-building-robust-cli-tools): Implement error recovery and retry logic.
8.  [**Utility Components**](#8-utility-components): Use helper components like `Colors` and `Spinner`.

---

### UI (from test.renderer.mjs) prospective

```sh
Async Test: [████████████████████████████████████████] 100.0% (5/5) 89.7/s ✓ Complete!
```

### 1. Basic Usage

To create a simple progress bar, instantiate the `ProgressBar` class with a total value and an optional description.

**Instructions:**

1.  Import `ProgressBar` from `@thinkeloquent/cli-progressor`.
2.  Create a new `ProgressBar` instance, providing the total number of steps.
3.  Call `.start()` to display the bar.
4.  Call `.update(increment)` inside your loop or task to advance the progress.

**Example:**

```js
import { ProgressBar } from "@thinkeloquent/cli-progressor";

// Create a progress bar for 100 items with a description.
const progressBar = new ProgressBar(100, "Downloading files");
progressBar.start();

// Simulate work and update the progress.
for (let i = 0; i < 100; i++) {
  await new Promise((resolve) => setTimeout(resolve, 50));
  progressBar.update(1); // Increment progress by 1
}
```

---

### 2. Advanced Configuration with the Builder Pattern

For more control over the progress bar's appearance and features, use the `ProgressBarBuilder`. This provides a fluent interface to chain configuration methods.

**Instructions:**

1.  Import `ProgressBarBuilder`.
2.  Create a new `ProgressBarBuilder` instance.
3.  Chain various `.with...()` and `.show...()` methods to set options.
4.  Call `.build()` to create the final `ProgressBar` instance.

**Available Methods:**

- `.withTotal(number)`: Sets the total value.
- `.withDescription(string)`: Sets the text description.
- `.withBarLength(number)`: Sets the visual width of the bar.
- `.withChars(completeChar, incompleteChar)`: Sets the characters for the bar.
- `.withColors(boolean)`: Enables or disables terminal colors.
- `.showETA(boolean)`: Toggles the Estimated Time of Arrival display.
- `.showSpeed(boolean)`: Toggles the items/second display.

**Example:**

```js
import { ProgressBarBuilder } from "@thinkeloquent/cli-progressor";

const customProgress = new ProgressBarBuilder()
  .withTotal(500)
  .withDescription("Processing data")
  .withBarLength(50)
  .withChars("█", "░")
  .withColors(true)
  .showETA(true)
  .showSpeed(true)
  .build();

customProgress.start();
// ... update loop ...
```

---

### 3. Handling Multiple Progress Bars

To display and manage multiple progress bars at once (e.g., for concurrent downloads), use the `MultiProgressManager`.

**Instructions:**

1.  Import `MultiProgressManager`.
2.  Create an instance of the manager.
3.  Use the `manager.add(id, total, description)` method to create and add a new progress bar. This returns the `ProgressBar` instance, which you can then update individually.

**Example:**

```js
import { MultiProgressManager } from "@thinkeloquent/cli-progressor";

const manager = new MultiProgressManager();

// Add two progress bars to the manager
const download1 = manager.add("file1", 100, "Downloading file1.zip");
const download2 = manager.add("file2", 200, "Downloading file2.zip");

// You can now update each bar independently
// download1.update(10);
// download2.update(25);
```

---

### 4. Using the `CLIProgressHelper` for Common Tasks

The `CLIProgressHelper.withProgress()` method is a powerful wrapper that simplifies adding a progress bar to a loop that processes a list of items. It handles starting, updating, and stopping the bar automatically.

**Instructions:**

1.  Import `CLIProgressHelper`.
2.  Call `CLIProgressHelper.withProgress()`, providing:
    - The total number of items.
    - A description string.
    - An async callback function that receives an `updateProgress` function as its argument.
3.  Inside your callback, iterate through your items and call `updateProgress(1)` after each item is processed.

**Example (File Processing):**

```js
import { CLIProgressHelper } from "@thinkeloquent/cli-progressor";

const files = ["file1.txt", "file2.txt", "file3.txt"];

// The helper manages the entire lifecycle of the progress bar.
await CLIProgressHelper.withProgress(
  files.length,
  "Processing files",
  async (updateProgress) => {
    for (const file of files) {
      // Simulate processing a file
      await new Promise((resolve) => setTimeout(resolve, 100));
      updateProgress(1); // The helper increments the bar
    }
  }
);
```

---

### 5. Responding to Events with Observers

You can attach multiple "observer" functions to a progress bar to react to updates. This is useful for logging, analytics, or triggering actions at specific milestones.

**Instructions:**

1.  Use the `.onProgress(callback)` method on a `ProgressBar` instance.
2.  The `callback` function will receive a `progressData` object on every update. This object contains details like `percentage`, `current`, `total`, `speed`, and `isComplete`.
3.  You can chain multiple `.onProgress()` calls to add several observers.

**Example:**

```js
import { ProgressBar, Colors } from "@thinkeloquent/cli-progressor";

class ProgressLogger {
  log(progressData) {
    if (progressData.percentage === 50) {
      console.log(Colors.info("LOG: Halfway there!"));
    }
  }
}

const logger = new ProgressLogger();
const progressBar = new ProgressBar(100, "Observer Demo")
  .onProgress(logger.log.bind(logger)) // Observer 1
  .onProgress((data) => {
    // Observer 2 (anonymous)
    if (data.isComplete) {
      console.log(Colors.success("🎉 Task completed!"));
    }
  });

progressBar.start();
// ... update loop ...
```

---

### 6. Extending Functionality: Custom Renderers

If you need a completely different output format, you can create a custom renderer by extending the `ProgressRenderer` base class.

**Instructions:**

1.  Import `ProgressRenderer`.
2.  Create a new class that `extends ProgressRenderer`.
3.  Implement the `render(progressData)` method to define how the progress is displayed.
4.  Optionally, implement the `cleanup()` method to clear the display after the progress is complete.
5.  Pass an instance of your custom renderer as the third argument to the `ProgressBar` constructor.

**Example (JSON Output):**

```js
import { ProgressRenderer, ProgressBar } from "@thinkeloquent/cli-progressor";

class JSONProgressRenderer extends ProgressRenderer {
  render(progressData) {
    // Output progress data as a JSON string
    console.log(JSON.stringify({ progress: progressData }));
  }
  cleanup() {
    // Optionally log a completion event
    console.log('{"event": "progress_complete"}');
  }
}

// Use the custom renderer with a progress bar
const jsonProgress = new ProgressBar(
  100,
  "API Processing",
  new JSONProgressRenderer()
);
jsonProgress.start();
```

---

### 7. Building Robust CLI Tools

Combine the library's features to build resilient command-line tools with features like automatic retries and error reporting.

**Instructions:**

Use `CLIProgressHelper.withProgress` to manage the main loop. Inside the loop, wrap individual processing steps in `try...catch` blocks to handle failures. For retries, create a helper function with a `for` loop.

**Example (Processing with Error Recovery):**

```js
import { Colors, CLIProgressHelper } from "@thinkeloquent/cli-progressor";

const items = ["task1", "task2", "failing_task", "task4"];

await CLIProgressHelper.withProgress(
  items.length,
  "Robust Processing",
  async (update) => {
    for (const item of items) {
      let attempts = 3;
      while (attempts > 0) {
        try {
          // Simulate a task that might fail
          if (item === "failing_task" && attempts > 1) {
            throw new Error("Temporary failure");
          }
          console.log(`\nProcessed ${item}`);
          break; // Success
        } catch (error) {
          attempts--;
          console.warn(Colors.warning(`\n⚠️  Failed on ${item}, retrying...`));
          if (attempts === 0) {
            console.error(Colors.error(`\n✗ Permanently failed on ${item}`));
          }
        }
      }
      update(1);
    }
  }
);
```

---

### 8. Utility Components

The library also exports standalone utilities.

- `Colors`: An object with methods for colorizing terminal output (`.success()`, `.error()`, `.warning()`, `.info()`, `.dim()`).
- `Spinner`: A class for creating indeterminate loading spinners, often used within a custom renderer.

**Example:**

```js
import { Colors, Spinner } from "@thinkeloquent/cli-progressor";

console.log(Colors.success("✓ Operation successful!"));
console.log(Colors.error("✗ An error occurred."));

// Spinners are typically used inside a custom renderer (see section 6)
const spinner = new Spinner(Spinner.presets.dots);
// console.log(spinner.next()); // Get the next frame of the animation
```
