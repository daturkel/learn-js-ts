# Exercise: Promises Basics

Convert callback-based functions to Promises, then to async/await.

## Setup

Create `promises-basics.ts` and run with `npx tsx promises-basics.ts`.

## The Callback Functions

Here are 5 functions that use the old callback pattern. Each simulates an async operation with `setTimeout`.

```typescript
// Simulates reading a file (callback-style)
function readFileCallback(filename: string, callback: (err: Error | null, data?: string) => void) {
  setTimeout(() => {
    if (filename === "missing.txt") {
      callback(new Error("File not found: missing.txt"));
    } else {
      callback(null, `Contents of ${filename}`);
    }
  }, 100);
}

// Simulates a database query
function queryDbCallback(query: string, callback: (err: Error | null, rows?: string[]) => void) {
  setTimeout(() => {
    if (query.includes("DROP")) {
      callback(new Error("Dangerous query blocked"));
    } else {
      callback(null, ["row1", "row2", "row3"]);
    }
  }, 150);
}

// Simulates an API call
function fetchApiCallback(url: string, callback: (err: Error | null, data?: object) => void) {
  setTimeout(() => {
    if (!url.startsWith("https://")) {
      callback(new Error("Only HTTPS URLs supported"));
    } else {
      callback(null, { status: 200, data: "response" });
    }
  }, 200);
}

// Simulates processing data
function processDataCallback(data: string, callback: (err: Error | null, result?: string) => void) {
  setTimeout(() => {
    callback(null, data.toUpperCase());
  }, 50);
}

// Simulates sending a notification
function sendNotificationCallback(message: string, callback: (err: Error | null) => void) {
  setTimeout(() => {
    if (message.length === 0) {
      callback(new Error("Message cannot be empty"));
    } else {
      console.log(`Notification sent: ${message}`);
      callback(null);
    }
  }, 100);
}
```

## Tasks

### Task 1: Wrap Each in a Promise

Convert each function to return a `Promise` instead of using a callback.

Example for the first one:

```typescript
function readFile(filename: string): Promise<string> {
  // `resolve` and `reject` are not keywords — they're parameter names for two
  // functions the Promise runtime provides. Call resolve(value) to fulfill the
  // promise, or reject(reason) to fail it. You must call exactly one of them.
  return new Promise((resolve, reject) => {
    readFileCallback(filename, (err, data) => {
      if (err) reject(err);
      else resolve(data!); // `data!` — non-null assertion: we know data is defined
                           // when err is null (the callback contract guarantees it)
    });
  });
}
```

Now do the same for `queryDb`, `fetchApi`, `processData`, and `sendNotification`.

<details>
<summary>Solution: Promise wrappers</summary>

```typescript
function readFile(filename: string): Promise<string> {
  return new Promise((resolve, reject) => {
    readFileCallback(filename, (err, data) => {
      if (err) reject(err);
      else resolve(data!);
    });
  });
}

function queryDb(query: string): Promise<string[]> {
  return new Promise((resolve, reject) => {
    queryDbCallback(query, (err, rows) => {
      if (err) reject(err);
      else resolve(rows!);
    });
  });
}

function fetchApi(url: string): Promise<object> {
  return new Promise((resolve, reject) => {
    fetchApiCallback(url, (err, data) => {
      if (err) reject(err);
      else resolve(data!);
    });
  });
}

function processData(data: string): Promise<string> {
  return new Promise((resolve, reject) => {
    processDataCallback(data, (err, result) => {
      if (err) reject(err);
      else resolve(result!);
    });
  });
}

function sendNotification(message: string): Promise<void> {
  return new Promise((resolve, reject) => {
    sendNotificationCallback(message, (err) => {
      if (err) reject(err);
      else resolve();
    });
  });
}
```

The pattern is always the same: `new Promise((resolve, reject) => { ... })` wrapping the callback.

`resolve` and `reject` are just parameter names — not JS keywords. The Promise
constructor passes two functions into your executor: call one to settle the promise.
The `!` (non-null assertion) after `data` and `rows` tells TypeScript "I know this
isn't undefined here" — safe because the callback only passes a value when `err` is null.
</details>

### Task 2: Chain with .then()

Using your Promise-based functions, chain these operations:
1. Read "config.json"
2. Process the data (uppercase it)
3. Send a notification with the result

Do this using `.then()` chains (not async/await yet):

<details>
<summary>Solution: .then() chain</summary>

```typescript
readFile("config.json")
  .then(data => processData(data))
  .then(processed => sendNotification(processed))
  .then(() => console.log("All done!"))
  .catch(error => console.error("Pipeline failed:", error));
```

Compare to the callback version:
```typescript
readFileCallback("config.json", (err, data) => {
  if (err) return console.error(err);
  processDataCallback(data!, (err, processed) => {
    if (err) return console.error(err);
    sendNotificationCallback(processed!, (err) => {
      if (err) return console.error(err);
      console.log("All done!");
    });
  });
});
```

The Promise chain is flat and readable. The callback version is nested.
</details>

### Task 3: Rewrite with async/await

Rewrite the same pipeline using `async/await`:

<details>
<summary>Solution: async/await</summary>

```typescript
async function pipeline() {
  try {
    const data = await readFile("config.json");
    const processed = await processData(data);
    await sendNotification(processed);
    console.log("All done!");
  } catch (error) {
    console.error("Pipeline failed:", error);
  }
}

pipeline();
```

This reads like synchronous Python code. Each `await` pauses until the Promise resolves, then continues.
</details>

### Task 4: Run operations in parallel

Use `Promise.all` to run `readFile`, `queryDb`, and `fetchApi` concurrently:

<details>
<summary>Solution: parallel</summary>

```typescript
async function parallelOps() {
  try {
    const [fileData, dbRows, apiData] = await Promise.all([
      readFile("config.json"),
      queryDb("SELECT * FROM experiments"),
      fetchApi("https://api.example.com/data"),
    ]);

    console.log("File:", fileData);
    console.log("DB rows:", dbRows);
    console.log("API:", apiData);
  } catch (error) {
    console.error("One or more operations failed:", error);
  }
}

parallelOps();
```

Array destructuring `[fileData, dbRows, apiData]` extracts the results in order. `Promise.all` runs all three concurrently and waits for all to finish.
</details>

### Task 5: Handle mixed results

Use `Promise.allSettled` to run `readFile("config.json")`, `readFile("missing.txt")`, and `queryDb("SELECT 1")`. Log which succeeded and which failed:

<details>
<summary>Solution: allSettled</summary>

```typescript
async function mixedResults() {
  const results = await Promise.allSettled([
    readFile("config.json"),
    readFile("missing.txt"),
    queryDb("SELECT 1"),
  ]);

  results.forEach((result, i) => {
    if (result.status === "fulfilled") {
      console.log(`Operation ${i}: Success —`, result.value);
    } else {
      console.log(`Operation ${i}: Failed —`, result.reason.message);
    }
  });
}

mixedResults();
// Operation 0: Success — Contents of config.json
// Operation 1: Failed — File not found: missing.txt
// Operation 2: Success — row1,row2,row3
```

Unlike `Promise.all`, `Promise.allSettled` never rejects. It waits for everything and tells you which succeeded and which failed.
</details>

---

**Next:** [Exercise 2: Async/Await →](/03-async-patterns/exercises/02-async-await.md)
