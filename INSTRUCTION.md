# ProgressBar

Purpose: Main CLI progress bar implementation

```js
constructor(total: number, description: string, renderer: ProgressRenderer)
start(): ProgressBar
stop(): ProgressBar
update(increment: number = 1): ProgressData
complete(): ProgressData
reset(): ProgressBar
onProgress(callback: Function): ProgressBar
```

# ProgressBarBuilder

Purpose: Fluent interface for configuring progress bars

```js
withTotal(total: number): ProgressBarBuilder
withDescription(description: string): ProgressBarBuilder
withBarLength(length: number): ProgressBarBuilder
withChars(filled: string, empty: string): ProgressBarBuilder
withColors(enabled: boolean): ProgressBarBuilder
showETA(show: boolean): ProgressBarBuilder
showSpeed(show: boolean): ProgressBarBuilder
forSpinner(): ProgressBarBuilder
build(): ProgressBar
```

# CLIProgressHelper

Purpose: Utility functions for common CLI patterns

```js
static async withProgress(
  total: number,
  description: string,
  asyncTask: Function,
  config: Object = {}
): Promise<any>

static async withSpinner(
  description: string,
  asyncTask: Function
): Promise<any>
```

# Configuration Options:

```json
{
  barLength: number,           // Width of progress bar
  filledChar: string,          // Character for completed portion
  emptyChar: string,           // Character for remaining portion
  useColors: boolean,          // Enable terminal colors
  showETA: boolean,            // Show estimated time remaining
  showSpeed: boolean,          // Show items per second
  showPercentage: boolean,     // Show percentage complete
  precision: number,           // Decimal places for percentage
  template: string            // Custom template string
}
```

# SilentProgressRenderer

Purpose: Non-interactive renderer for testing/CI

```js
render(progressData: ProgressData): void
getLastProgress(): ProgressData
getHistory(): Array<ProgressData>
clear(): void
```

# Data Structures - ProgressData

```js
{
  current: number,             // Current progress value
  total: number,               // Total progress value
  percentage: number,          // Completion percentage (0-100)
  elapsed: number,             // Elapsed time in seconds
  eta: number,                 // Estimated time remaining in seconds
  speed: number,               // Items per second
  description: string,         // Progress description
  isComplete: boolean,         // Whether progress is complete
  isIndeterminate: boolean     // Whether total is unknown
}
```

# Utility Classes - TerminalUtils

Purpose: Terminal feature detection and control

```js
static get isInteractive(): boolean
static get columns(): number
static get supportsColor(): boolean
static moveCursor(dx: number, dy: number): void
static clearLine(): void
static hideCursor(): void
static showCursor(): void
```

# Colors

Purpose: Terminal color formatting

```js
static colorize(text: string, color: string): string
static success(text: string): string
static error(text: string): string
static warning(text: string): string
static info(text: string): string
static dim(text: string): string
```

# Indeterminate Spinner

```js
const spinner = ProgressBar.createSpinner("Loading data");
spinner.start();

const data = await fetchData();
spinner.stop();
```

# Custom Styled Progress Bar

```js
import { ProgressBarBuilder } from "./main.mjs";

const progressBar = new ProgressBarBuilder()
  .withTotal(1000)
  .withDescription("Custom Processing")
  .withBarLength(50)
  .withChars("▓", "▒")
  .withColors(true)
  .showETA(true)
  .showSpeed(true)
  .withPrecision(2)
  .build();

progressBar.start();
```

# Template-Based Progress

```js
const progressBar = new ProgressBarBuilder()
  .withTotal(100)
  .withTemplate(
    "{description}: {current}/{total} ({percentage}%) [{bar}] ETA: {eta}s"
  )
  .build();
```

# Async Task with Progress

```js
import { CLIProgressHelper } from "./main.mjs";

const result = await CLIProgressHelper.withProgress(
  totalItems,
  "Processing items",
  async (updateProgress) => {
    for (const item of items) {
      const result = await processItem(item);
      updateProgress(1);
    }
    return results;
  }
);
```

# Spinner for Unknown Duration

```js
const config = await CLIProgressHelper.withSpinner(
  "Fetching configuration",
  async () => {
    return await fetch("/api/config").then((r) => r.json());
  }
);
```

# Multiple Concurrent Progress Bars

```js
import { MultiProgressManager } from "./main.mjs";

const manager = new MultiProgressManager();

// Add multiple progress trackers
const download1 = manager.add("download1", 100, "Downloading file 1");
const download2 = manager.add("download2", 200, "Downloading file 2");

// Update them independently
manager.update("download1", 10);
manager.update("download2", 25);

// Complete individual tasks
manager.complete("download1");
```

# Progress Notifications

```js
const progressBar = new ProgressBar(100, "Processing")
  .onProgress((progressData) => {
    if (progressData.percentage >= 50) {
      console.log("Halfway there!");
    }
  })
  .onProgress((progressData) => {
    // Log progress to file
    logProgress(progressData);
  });
```

# Silent Mode for Tests

```js
const progressBar = ProgressBar.createSilent(100, "Test progress");

// Run your operations
for (let i = 0; i < 100; i++) {
  progressBar.update(1);
}

// Access progress data for assertions
const finalProgress = progressBar.getProgress();
assert.equal(finalProgress.percentage, 100);

// Access history for detailed testing
const history = progressBar.renderer.getHistory();
assert.equal(history.length, 100);
```

# Environment-Based Configuration

```js
const createProgressBar = (total, description) => {
  if (process.env.CI === "true") {
    return ProgressBar.createSilent(total, description);
  }
  return new ProgressBarBuilder()
    .withTotal(total)
    .withDescription(description)
    .withColors(process.stdout.isTTY)
    .build();
};
```

# Graceful Error Recovery

```js
const progressBar = new ProgressBar(100, "Processing files").start();

try {
  for (let i = 0; i < 100; i++) {
    await processFile(files[i]);
    progressBar.update(1);
  }
} catch (error) {
  progressBar.stop();
  console.error(Colors.error(`Processing failed: ${error.message}`));
  process.exit(1);
}
```

# Creating a Custom Renderer

```js
import { ProgressRenderer } from "./main.mjs";

class CustomRenderer extends ProgressRenderer {
  render(progressData) {
    const { current, total, description } = progressData;
    console.log(`${description}: ${current}/${total}`);
  }

  cleanup() {
    console.log("Custom renderer cleanup");
  }
}

const progressBar = new ProgressBar(100, "Custom", new CustomRenderer());
```

# File Processing CLI Tool

```js
#!/usr/bin/env node
import { ProgressBar, CLIProgressHelper, Colors } from "./main.mjs";
import fs from "fs/promises";
import path from "path";

async function processFiles(directory) {
  try {
    const files = await fs.readdir(directory);

    await CLIProgressHelper.withProgress(
      files.length,
      `Processing files in ${directory}`,
      async (updateProgress) => {
        for (const file of files) {
          const filePath = path.join(directory, file);
          await processFile(filePath);
          updateProgress(1);
        }
      }
    );

    console.log(
      Colors.success(`✓ Successfully processed ${files.length} files`)
    );
  } catch (error) {
    console.error(Colors.error(`✗ Error: ${error.message}`));
    process.exit(1);
  }
}

async function processFile(filePath) {
  // Simulate file processing
  const stats = await fs.stat(filePath);
  await new Promise((resolve) => setTimeout(resolve, 100));
  return stats;
}

// CLI argument handling
const directory = process.argv[2] || ".";
processFiles(directory);
```

=========

# Simple Progress Bar Implementation

```js
import { ProgressBar } from "./main.mjs";

// Basic progress bar
const progressBar = new ProgressBar(100, "Downloading files");
progressBar.start();

// Simulate work
for (let i = 0; i < 100; i++) {
  await new Promise((resolve) => setTimeout(resolve, 50));
  progressBar.update(1);
}
```

# Builder Pattern Usage

```js
import { ProgressBarBuilder } from "./main.mjs";

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
```

# Custom Renderer Implementation

```js
import { ProgressRenderer, ProgressBar } from "./main.mjs";

class JSONProgressRenderer extends ProgressRenderer {
  render(progressData) {
    const output = {
      timestamp: new Date().toISOString(),
      progress: progressData,
    };
    console.log(JSON.stringify(output));
  }

  cleanup() {
    console.log('{"event": "progress_complete"}');
  }
}

// Usage
const jsonProgress = new ProgressBar(
  100,
  "API Processing",
  new JSONProgressRenderer()
);
jsonProgress.start();
```

# Multiple Import Pattern

```js
import {
  ProgressBar,
  ProgressBarBuilder,
  CLIProgressHelper,
  Colors,
  MultiProgressManager,
} from "./main.mjs";

// Use multiple classes together
const manager = new MultiProgressManager();
const download1 = manager.add("file1", 100, "Downloading file1.zip");
const download2 = manager.add("file2", 200, "Downloading file2.zip");

console.log(Colors.info("Starting multiple downloads..."));
```

# File Processing CLI

```js
#!/usr/bin/env node
import { CLIProgressHelper, Colors, ProgressBarBuilder } from "./main.mjs";
import fs from "fs/promises";
import path from "path";

class FileProcessor {
  constructor(inputDir, outputDir) {
    this.inputDir = inputDir;
    this.outputDir = outputDir;
  }

  async processAllFiles() {
    const files = await fs.readdir(this.inputDir);

    return await CLIProgressHelper.withProgress(
      files.length,
      "Processing files",
      async (updateProgress) => {
        const results = [];

        for (const file of files) {
          const result = await this.processFile(file);
          results.push(result);
          updateProgress(1);
        }

        return results;
      }
    );
  }

  async processFile(filename) {
    const inputPath = path.join(this.inputDir, filename);
    const outputPath = path.join(this.outputDir, filename);

    // Simulate file processing
    const data = await fs.readFile(inputPath);
    await fs.writeFile(outputPath, data);

    return { filename, size: data.length };
  }
}

// Usage
const processor = new FileProcessor("./input", "./output");
processor
  .processAllFiles()
  .then((results) => {
    console.log(Colors.success(`✓ Processed ${results.length} files`));
  })
  .catch((error) => {
    console.error(Colors.error(`✗ Error: ${error.message}`));
    process.exit(1);
  });
```

# API Data Fetcher with Progress

```js
import { ProgressBar, CLIProgressHelper, Colors } from "./main.mjs";

class APIDataFetcher {
  constructor(baseUrl, apiKey) {
    this.baseUrl = baseUrl;
    this.apiKey = apiKey;
  }

  async fetchWithProgress(endpoints) {
    return await CLIProgressHelper.withProgress(
      endpoints.length,
      "Fetching API data",
      async (updateProgress) => {
        const results = [];

        for (const endpoint of endpoints) {
          try {
            const data = await this.fetchEndpoint(endpoint);
            results.push({ endpoint, data, success: true });
          } catch (error) {
            results.push({ endpoint, error: error.message, success: false });
          }
          updateProgress(1);
        }

        return results;
      }
    );
  }

  async fetchEndpoint(endpoint) {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      headers: { Authorization: `Bearer ${this.apiKey}` },
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    return await response.json();
  }
}

// Usage
const fetcher = new APIDataFetcher("https://api.example.com", "your-api-key");
const endpoints = ["/users", "/posts", "/comments", "/analytics"];

fetcher.fetchWithProgress(endpoints).then((results) => {
  const successful = results.filter((r) => r.success).length;
  console.log(
    Colors.success(
      `✓ Successfully fetched ${successful}/${results.length} endpoints`
    )
  );
});
```

# Progress Analytics Collector

```js
import { ProgressBar } from "./main.mjs";

class ProgressAnalytics {
  constructor() {
    this.sessions = new Map();
  }

  trackProgress(progressId, progressBar) {
    const session = {
      id: progressId,
      startTime: Date.now(),
      updates: [],
      completed: false,
    };

    progressBar.onProgress((progressData) => {
      session.updates.push({
        timestamp: Date.now(),
        percentage: progressData.percentage,
        speed: progressData.speed,
      });

      if (progressData.isComplete) {
        session.completed = true;
        session.endTime = Date.now();
        session.totalDuration = session.endTime - session.startTime;
        this.generateReport(session);
      }
    });

    this.sessions.set(progressId, session);
    return session;
  }

  generateReport(session) {
    const avgSpeed =
      session.updates.reduce((sum, update) => sum + update.speed, 0) /
      session.updates.length;

    console.log(`\n📊 Progress Analytics Report for ${session.id}:`);
    console.log(`   Duration: ${session.totalDuration}ms`);
    console.log(`   Updates: ${session.updates.length}`);
    console.log(`   Avg Speed: ${avgSpeed.toFixed(2)} items/sec`);
  }
}

// Usage
const analytics = new ProgressAnalytics();
const progressBar = new ProgressBar(1000, "Data Processing");

analytics.trackProgress("data-processing-001", progressBar);
progressBar.start();
```

# Multi-Observer Pattern

```js
import { ProgressBar, Colors } from "./main.mjs";

class ProgressLogger {
  log(progressData) {
    if (progressData.percentage % 25 === 0) {
      console.log(`📝 Milestone: ${progressData.percentage}% complete`);
    }
  }
}

class ProgressNotifier {
  notify(progressData) {
    if (progressData.percentage === 50) {
      console.log(Colors.warning("⚠️  Halfway point reached"));
    }
    if (progressData.isComplete) {
      console.log(Colors.success("🎉 Task completed!"));
    }
  }
}

// Usage with multiple observers
const logger = new ProgressLogger();
const notifier = new ProgressNotifier();

const progressBar = new ProgressBar(100, "Multi-Observer Demo")
  .onProgress(logger.log.bind(logger))
  .onProgress(notifier.notify.bind(notifier))
  .onProgress((data) => {
    // Anonymous observer
    if (data.percentage === 75) {
      console.log(Colors.info("🏃 Final stretch!"));
    }
  });

progressBar.start();
```

# Themed Spinner Implementation

```js
import { Spinner, ProgressBar, ProgressRenderer } from "./main.mjs";

class ThemedSpinnerRenderer extends ProgressRenderer {
  constructor(theme = "dots") {
    super();
    this.spinner = new Spinner(Spinner.presets[theme]);
    this.lastFrame = "";
  }

  render(progressData) {
    const frame = this.spinner.next();
    const output = `${frame} ${progressData.description}...`;

    // Clear previous line and write new frame
    process.stdout.write("\r" + " ".repeat(this.lastFrame.length) + "\r");
    process.stdout.write(output);
    this.lastFrame = output;
  }

  cleanup() {
    process.stdout.write("\r" + " ".repeat(this.lastFrame.length) + "\r");
  }
}

// Usage
const themes = ["dots", "line", "arrow", "clock"];

for (const theme of themes) {
  const spinner = new ProgressBar(
    0,
    `Loading with ${theme} theme`,
    new ThemedSpinnerRenderer(theme)
  );
  spinner.start();

  await new Promise((resolve) => setTimeout(resolve, 2000));
  spinner.stop();
  console.log(`✓ ${theme} theme demo complete`);
}
```

# Robust CLI Tool with Error Recovery

```js
import { ProgressBar, Colors, CLIProgressHelper } from "./main.mjs";

class RobustProcessor {
  constructor(config = {}) {
    this.config = {
      retryAttempts: 3,
      retryDelay: 1000,
      continueOnError: false,
      ...config,
    };
  }

  async processWithRecovery(items, processFn) {
    const results = [];
    let errors = [];

    try {
      await CLIProgressHelper.withProgress(
        items.length,
        "Processing with error recovery",
        async (updateProgress) => {
          for (const [index, item] of items.entries()) {
            try {
              const result = await this.processWithRetry(item, processFn);
              results.push({ index, item, result, success: true });
            } catch (error) {
              errors.push({ index, item, error: error.message });

              if (!this.config.continueOnError) {
                throw new Error(
                  `Processing failed at item ${index}: ${error.message}`
                );
              }

              results.push({
                index,
                item,
                error: error.message,
                success: false,
              });
            }

            updateProgress(1);
          }
        }
      );

      this.reportResults(results, errors);
      return results;
    } catch (error) {
      console.error(Colors.error(`\n✗ Processing failed: ${error.message}`));
      throw error;
    }
  }

  async processWithRetry(item, processFn) {
    let lastError;

    for (let attempt = 1; attempt <= this.config.retryAttempts; attempt++) {
      try {
        return await processFn(item);
      } catch (error) {
        lastError = error;

        if (attempt < this.config.retryAttempts) {
          console.log(
            Colors.warning(
              `\n⚠️  Retry ${attempt}/${this.config.retryAttempts} for item: ${item}`
            )
          );
          await new Promise((resolve) =>
            setTimeout(resolve, this.config.retryDelay)
          );
        }
      }
    }

    throw lastError;
  }

  reportResults(results, errors) {
    const successful = results.filter((r) => r.success).length;
    const failed = results.length - successful;

    console.log(Colors.success(`\n✓ Successfully processed: ${successful}`));

    if (failed > 0) {
      console.log(Colors.error(`✗ Failed to process: ${failed}`));
      errors.forEach(({ index, item, error }) => {
        console.log(Colors.dim(`  - Item ${index} (${item}): ${error}`));
      });
    }
  }
}

// Usage
const processor = new RobustProcessor({
  retryAttempts: 3,
  retryDelay: 500,
  continueOnError: true,
});

const items = ["file1.txt", "file2.txt", "file3.txt"];

processor.processWithRecovery(items, async (filename) => {
  // Simulate processing that might fail
  if (Math.random() < 0.3) {
    throw new Error(`Processing failed for ${filename}`);
  }

  await new Promise((resolve) => setTimeout(resolve, 100));
  return `Processed: ${filename}`;
});
```
