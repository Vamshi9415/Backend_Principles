# The Complete Guide to Python AsyncIO & Asynchronous Programming
### Master Concurrency, Coroutines, Tasks, and Synchronization Primitives

---

## Table of Contents

1. [The Big Picture: Synchronous vs. Asynchronous Programming](#1-the-big-picture-synchronous-vs-asynchronous-programming)
2. [When to Use AsyncIO vs. Threads vs. Processes](#2-when-to-use-asyncio-vs-threads-vs-processes)
3. [Concept 1: The Event Loop](#3-concept-1-the-event-loop)
4. [Concept 2: Coroutines](#4-concept-2-coroutines)
5. [The `await` Keyword — Deep Dive](#5-the-await-keyword--deep-dive)
6. [Concept 3: Tasks — True Concurrency](#6-concept-3-tasks--true-concurrency)
7. [The `gather()` Function](#7-the-gather-function)
8. [Task Groups — The Modern Approach](#8-task-groups--the-modern-approach)
9. [Concept 4: Futures](#9-concept-4-futures)
10. [Concept 5: Synchronization Primitives](#10-concept-5-synchronization-primitives)
11. [Locks](#11-locks)
12. [Semaphores](#12-semaphores)
13. [Events](#13-events)
14. [Comparison Tables and Visual Breakdowns](#14-comparison-tables-and-visual-breakdowns)
15. [Common Mistakes and Troubleshooting](#15-common-mistakes-and-troubleshooting)
16. [Best Practices](#16-best-practices)
17. [Real-World Project Patterns](#17-real-world-project-patterns)
18. [Quick Reference Cheat Sheet](#18-quick-reference-cheat-sheet)

---

## 1. The Big Picture: Synchronous vs. Asynchronous Programming

### The Journey Metaphor

Imagine your program is a journey from **Point A to Point D**. Each point represents a task your program needs to complete.

**Synchronous Programming (Traditional):**
```
[A] ──────► [B] ──────► [C] ──────► [D]
 Wait here    Wait here    Wait here
 until done   until done   until done
```

You travel in a perfectly straight line. You **stop at each point** and wait for it to finish before moving on. If Point B takes 10 seconds (because it's downloading a file from the internet), your entire program sits frozen for those 10 seconds.

**Asynchronous Programming:**
```
[A] starts ──────────────────────────────► [A] finishes
[B] starts ──────────────────────► [B] finishes
[C] starts ──────────────────────────────────────► [C] finishes
[D] starts ──► [D] finishes (fast!)

All running simultaneously, overlapping!
```

You send out multiple "scouts" to explore Points B, C, and D at the same time, without waiting for the first scout to return before dispatching the next. Your program handles multiple tasks simultaneously, making it far more efficient when tasks involve waiting.

**Real-world analogy — The Restaurant Kitchen:**

- **Synchronous chef:** Takes one order, cooks it completely, plates it, serves it, then takes the next order. If pasta takes 15 minutes, nobody else gets food for 15 minutes.
- **Asynchronous chef:** Takes all orders, puts the pasta on the stove (which cooks itself while he's away), starts the grill for steak, chops vegetables for salad. While the pasta boils (waiting), the chef does other work. Everything finishes faster.

The key insight: **the chef (your CPU) is never idle. While waiting on one thing, it works on another.**

### Why This Matters in Real Software

Consider loading a modern web page. Your browser must:
1. Download the HTML
2. Download 30+ CSS files
3. Download 100+ images
4. Download JavaScript files
5. Make API calls for dynamic content

If done synchronously, you'd download them one at a time. With asynchronous programming, all downloads happen concurrently. This is why modern web browsers and servers use async I/O extensively — it's the difference between a 200ms page load and a 30-second one.

---

## 2. When to Use AsyncIO vs. Threads vs. Processes

One of the most important decisions in concurrent Python programming is **which concurrency model to use**. The wrong choice can actually harm performance.

### The Three Models Explained

#### AsyncIO — For Tasks That Wait A Lot

> *"AsyncIO is your choice for tasks that wait a lot — like network requests or reading files. It excels in handling many tasks concurrently without using much CPU power. This makes your application more efficient and responsive when you're waiting on a lot of different tasks."*

AsyncIO is ideal for **I/O-bound** work — tasks where your program spends most of its time waiting for external operations:
- Network requests (calling APIs, downloading data)
- File I/O (reading/writing to disk)
- Database queries
- Web scraping
- Web servers handling many simultaneous connections

**How it works:** A single thread runs the event loop, which switches between tasks whenever one is waiting. No CPU wasted sitting idle.

**Real-world analogy:** A single bank teller who handles multiple customers at once — while one customer's transaction is processing (waiting), they assist another customer.

#### Threads — For Parallel I/O-Bound Tasks That Share Data

> *"Threads are suited for tasks that may need to wait but also share data. They can run in parallel within the same application making them useful for tasks that are I/O-bound but less CPU intensive."*

Threads run within the same process and share memory. In Python, the **Global Interpreter Lock (GIL)** prevents true parallel CPU execution in threads, but threads can still run concurrently for I/O operations.

Best for:
- Tasks that need to share data structures
- Interfacing with libraries that don't support async
- Tasks with some waiting AND some computation

**Real-world analogy:** Multiple bank tellers at the same branch — they share the same vault (memory), can communicate easily, but only one can access certain shared resources at a time.

#### Processes — For CPU-Heavy Tasks

> *"For CPU heavy tasks, processes are the way to go. Each process operates independently, maximizing CPU usage by running in parallel across multiple cores. This is ideal for intensive computations."*

Processes bypass the GIL entirely. Each has its own memory space and Python interpreter. True parallelism across CPU cores.

Best for:
- Mathematical computations (machine learning, scientific computing)
- Image/video processing
- Any task that keeps the CPU busy (not waiting)

**Real-world analogy:** Multiple bank branches — completely independent, with their own staff and vaults. No sharing, full independence, maximum throughput for heavy workloads.

### Decision Table

| Factor | AsyncIO | Threads | Processes |
|--------|---------|---------|-----------|
| **Primary use case** | Many waiting tasks | Shared-data parallel I/O | CPU-intensive work |
| **CPU usage** | Low | Low-moderate | High (maximized) |
| **Memory overhead** | Very low | Low | High (separate memory) |
| **Shared data** | Easy (single thread) | Yes (with care) | Hard (need IPC) |
| **Python GIL affected?** | No (cooperative) | Yes (limits parallelism) | No (separate interpreters) |
| **Complexity** | Medium | High (race conditions) | Medium-High |
| **Best example** | Web scraper, API server | Background worker threads | ML training, video encoding |

### Quick Decision Flowchart

```
Start
  │
  ▼
Is the task CPU-intensive?
  │
  ├── YES ──► Use PROCESSES (multiprocessing)
  │
  └── NO (I/O-bound)
        │
        ▼
      Do tasks need to share mutable data?
        │
        ├── YES ──► Use THREADS (threading)
        │
        └── NO
              │
              ▼
            Are you handling many concurrent connections/tasks?
              │
              ├── YES ──► Use ASYNCIO ✓
              │
              └── NO ──► Synchronous is probably fine
```

---

## 3. Concept 1: The Event Loop

### What Is the Event Loop?

> *"In Python's AsyncIO, the Event Loop is the core that manages and distributes tasks. Think of it as a central hub with tasks circling around it, waiting for their turn to be executed. Each task takes its turn in the center where it's either executed immediately or paused if it's waiting for something — like data from the internet."*

The event loop is the **beating heart** of async Python. It is a single, continuously running loop that:
1. Keeps track of all registered tasks/coroutines
2. Decides which task gets to run next
3. Switches between tasks when one is waiting
4. Ensures no CPU time is wasted

### How the Event Loop Works — Step by Step

```
Event Loop Starts
      │
      ▼
┌─────────────────────────────────────────┐
│           EVENT LOOP                    │
│                                         │
│  Task Queue: [Task A, Task B, Task C]  │
│                                         │
│  1. Pick Task A → Run it               │
│  2. Task A hits `await` (waiting...)   │
│  3. Task A steps aside                 │
│  4. Pick Task B → Run it               │
│  5. Task B hits `await` (waiting...)   │
│  6. Task B steps aside                 │
│  7. Pick Task C → Run it               │
│  8. Task A's wait is done! Resume A    │
│  9. Task A finishes                    │
│  10. Task B's wait is done! Resume B   │
│  ...continues until all done           │
└─────────────────────────────────────────┘
```

> *"When a task awaits, it steps aside, making room for another task to run, ensuring the loop is always efficiently utilized. Once the awaited operation is complete, the task will resume, ensuring a smooth and responsive program flow."*

**Real-world analogy — Air Traffic Control:**

The event loop is like an air traffic controller managing an airport. Multiple planes (tasks) want to land (execute). The controller gives each plane a turn on the runway. When a plane needs to circle and wait for clearance (await), the controller directs another plane to land. No runway (CPU) sits idle. The moment a plane gets clearance (await resolves), the controller fits it back into the queue.

### Starting the Event Loop

The entry point for all async Python code:

```python
import asyncio

async def main():
    print("Hello from async world!")

# This is how you start the event loop
asyncio.run(main())
```

**What `asyncio.run()` does:**
1. Creates a new event loop
2. Runs the coroutine you pass to it
3. Closes the event loop when done
4. Returns the result of the coroutine

**Important:** You should only call `asyncio.run()` **once** in your program — it's the entry point. Everything else flows from there.

### The Event Loop and Cooperative Multitasking

Unlike threads (which can be interrupted by the OS at any time), asyncio uses **cooperative multitasking**. Tasks voluntarily yield control at `await` points. This means:

- **No race conditions** on normal Python objects (only one task runs at a time)
- **Predictable execution** — you know a task won't be interrupted mid-operation
- **You must use `await` correctly** — a task that never awaits will block everyone else

```python
import asyncio

# GOOD — yields to event loop, allows other tasks to run
async def good_task():
    await asyncio.sleep(1)  # yields here, others can run

# BAD — blocks the entire event loop!
async def bad_task():
    import time
    time.sleep(1)  # blocks the thread, NO other tasks can run
```

**Critical rule:** In async code, always use `asyncio.sleep()` not `time.sleep()`. Always use async-compatible libraries for I/O (aiohttp not requests, aiosqlite not sqlite3, etc.)

---

## 4. Concept 2: Coroutines

### What Is a Coroutine?

A coroutine is a special kind of function that can **pause its execution** and **resume later**. It is the fundamental building block of async Python.

There are two things you need to distinguish:

**1. Coroutine Function** — defined with `async def`:
```python
async def fetch_data(item_id, delay):
    # This is a coroutine FUNCTION
    print(f"Fetching data for {item_id}...")
    await asyncio.sleep(delay)  # simulate network wait
    data = f"Data for {item_id}"
    return data
```

**2. Coroutine Object** — what you get when you *call* a coroutine function:
```python
# Calling the coroutine function returns a coroutine OBJECT
coro = fetch_data("item_1", 2)
# coro is now a coroutine object — it has NOT started executing yet!
```

This distinction is crucial and trips up many beginners.

### The Critical Insight: Calling ≠ Executing

> *"When you call a function defined with the `async` keyword, it returns a coroutine object, and that coroutine object needs to be awaited in order for it to actually execute."*

```python
import asyncio

async def main():
    print("Start of main coroutine")

# WRONG — this does NOT run main(), it creates a coroutine object
result = main()
print(result)  # Output: <coroutine object main at 0x...>
# Warning: "coroutine 'main' was never awaited"
```

To actually execute it, you need either:
- `asyncio.run(main())` — at the top level
- `await main()` — inside another async function

```python
import asyncio

async def main():
    print("Start of main coroutine")

# CORRECT — asyncio.run() handles awaiting the coroutine
asyncio.run(main())
# Output: Start of main coroutine
```

### A Complete Coroutine Example

```python
import asyncio

# Simulates an I/O-bound operation (network request, file read, etc.)
async def fetch_data(item_id: str, delay: float) -> str:
    print(f"Fetching data for {item_id}...")
    await asyncio.sleep(delay)  # Pause here, let other tasks run
    data = f"Result for {item_id}"
    print(f"Data fetched for {item_id}!")
    return data

async def main():
    print("Start of main coroutine")
    
    # Create the coroutine object (NOT yet running)
    task = fetch_data("item_1", 2)
    print(f"Coroutine object created: {task}")
    # Output: <coroutine object fetch_data at 0x...>
    
    # NOW it starts running — we await it
    result = await task
    print(f"Received: {result}")
    
    print("End of main coroutine")

asyncio.run(main())
```

**Output:**
```
Start of main coroutine
Coroutine object created: <coroutine object fetch_data at 0x...>
Fetching data for item_1...
Data fetched for item_1!
Received: Result for item_1
End of main coroutine
```

### Sequential Coroutines (The Non-Concurrent Version)

This example shows what happens when you await coroutines one at a time — **they run sequentially, not concurrently:**

```python
import asyncio
import time

async def fetch_data(item_id: str, delay: float) -> str:
    print(f"Fetching data for {item_id}...")
    await asyncio.sleep(delay)
    return f"Data for {item_id}"

async def main():
    start = time.time()
    
    # Create coroutine objects
    coro_1 = fetch_data("id1", 2)
    coro_2 = fetch_data("id2", 2)
    
    # Await them one at a time — SEQUENTIAL, not concurrent
    result_1 = await coro_1  # Waits 2 seconds
    result_2 = await coro_2  # Waits another 2 seconds
    
    elapsed = time.time() - start
    print(f"Total time: {elapsed:.1f}s")  # ~4 seconds
    print(result_1)
    print(result_2)

asyncio.run(main())
```

> *"This might seem counterintuitive because you may have guessed that when we created these two coroutine objects they were going to start running concurrently... but remember — a coroutine doesn't start running until it's awaited. In this case we actually wait for the first coroutine to finish and only once this has finished do we even start executing the second coroutine."*

**The key takeaway:** Creating coroutine objects does not start them. Awaiting them one after another creates sequential execution. To get true concurrency, you need **Tasks**.

---

## 5. The `await` Keyword — Deep Dive

### What `await` Actually Does

The `await` keyword does three things:
1. **Suspends** the current coroutine at that point
2. **Yields control** back to the event loop
3. **Resumes** the coroutine once the awaited operation completes

```python
async def example():
    print("Before await")
    result = await some_async_operation()  # Suspend here
    # ... event loop runs other tasks while waiting ...
    # ... once done, resume here with the result ...
    print(f"After await: {result}")
```

### Rules for Using `await`

**Rule 1: You can only use `await` inside an `async` function**

```python
# WRONG — regular function cannot use await
def bad_function():
    result = await fetch_data()  # SyntaxError!

# CORRECT — async function can use await
async def good_function():
    result = await fetch_data()  # Works fine
```

**Rule 2: You can only await "awaitable" objects**

Awaitables in Python are:
- **Coroutine objects** (from calling `async def` functions)
- **Task objects** (created with `asyncio.create_task()`)
- **Future objects**
- **Any object with `__await__` method**

```python
# WRONG — can't await a regular function's result
async def bad():
    result = await regular_function()  # TypeError!

# CORRECT — await a coroutine
async def good():
    result = await async_function()    # Works
    result = await asyncio.sleep(1)    # Works
    result = await asyncio.create_task(async_function())  # Works
```

### `await` Controls Execution Order

```python
import asyncio

async def task_a():
    print("Task A: start")
    await asyncio.sleep(2)
    print("Task A: end")

async def task_b():
    print("Task B: start")
    await asyncio.sleep(1)
    print("Task B: end")

async def sequential():
    """Runs A then B — total ~3 seconds"""
    await task_a()  # Waits for A to completely finish
    await task_b()  # Then starts B
    # Output order: A start → A end → B start → B end

async def concurrent():
    """Runs A and B together — total ~2 seconds"""
    t1 = asyncio.create_task(task_a())
    t2 = asyncio.create_task(task_b())
    await t1
    await t2
    # Output order: A start → B start → B end → A end

asyncio.run(sequential())
asyncio.run(concurrent())
```

---

## 6. Concept 3: Tasks — True Concurrency

### Why Tasks Exist

The problem with plain coroutines: you can only do one thing at a time if you `await` them sequentially. Tasks solve this.

> *"A task is a way to schedule a coroutine to run as soon as possible and to allow us to run multiple coroutines simultaneously. The issue we saw previously is that we needed to wait for one coroutine to finish before we could start executing the next. With a task we don't have that issue — as soon as a coroutine is sleeping or waiting on something not in our control, we can move on and start executing another task."*

### How Tasks Work

```
Without Tasks (Sequential):
─────────────────────────────────────────────────────
Coroutine 1: ████████████████████ (2s wait)
Coroutine 2:                      ██████████████████████████████ (3s wait)
Coroutine 3:                                                      ██████████ (1s wait)
Total time: 6 seconds

With Tasks (Concurrent):
─────────────────────────────────────────────────────
Task 1: ████████████████████ (2s wait)
Task 2: ██████████████████████████████ (3s wait)
Task 3: ██████████ (1s wait)
Total time: 3 seconds (only as long as the longest task)
```

> *"We're never going to be executing these tasks at the exact same time — we're not using multiple CPU cores — but if one task isn't doing something, if it's idle, blocked, waiting on something, we can switch over and start working on another task. The whole goal here is that our program is always attempting to do something."*

### Creating Tasks with `asyncio.create_task()`

```python
import asyncio
import time

async def fetch_data(item_id: str, delay: float) -> str:
    print(f"Starting fetch for {item_id}")
    await asyncio.sleep(delay)  # Simulates I/O wait
    print(f"Finished fetch for {item_id}")
    return f"Data for {item_id}"

async def main():
    start = time.time()
    
    # Create tasks — scheduling coroutines to run ASAP
    # These start running immediately (as soon as we hit an await)
    task1 = asyncio.create_task(fetch_data("item_1", 2))
    task2 = asyncio.create_task(fetch_data("item_2", 3))
    task3 = asyncio.create_task(fetch_data("item_3", 1))
    
    # Without tasks, this would take 2+3+1 = 6 seconds
    # With tasks, it takes max(2,3,1) = 3 seconds
    
    # Await results (tasks are already running concurrently)
    result1 = await task1
    result2 = await task2
    result3 = await task3
    
    elapsed = time.time() - start
    print(f"\nAll done in {elapsed:.1f}s")
    print(result1, result2, result3)

asyncio.run(main())
```

**Output:**
```
Starting fetch for item_1
Starting fetch for item_2
Starting fetch for item_3
Finished fetch for item_3
Finished fetch for item_1
Finished fetch for item_2

All done in 3.0s
Data for item_1 Data for item_2 Data for item_3
```

All three tasks **started immediately**. The one with the shortest wait (item_3, 1s) finished first. Total time is 3 seconds — not 6.

### Controlling Task Order with `await`

> *"Using asynchronous programming gives us that control and allows us to synchronize our code in whatever manner we see fit."*

```python
async def main():
    # Create all tasks (all start running immediately)
    task1 = asyncio.create_task(fetch_data("item_1", 2))
    task2 = asyncio.create_task(fetch_data("item_2", 3))
    task3 = asyncio.create_task(fetch_data("item_3", 1))
    
    # Wait for task1 AND task2 before starting to process task3's result
    result1 = await task1
    result2 = await task2
    
    # task3 has likely already finished by now, 
    # but we only retrieve its result here
    result3 = await task3
    
    # Use results in guaranteed order
    process_results(result1, result2, result3)
```

**Important nuance:** Even if you `await task1` first, tasks 2 and 3 are **already running** concurrently. `await` just means "wait here for this particular task's result before continuing this code path."

### Task Lifecycle

```
asyncio.create_task(coro())
        │
        ▼
  [SCHEDULED] — Task is queued to run
        │
        ▼ (event loop picks it up)
  [RUNNING] — Task is actively executing
        │
        ├── hits `await` ──► [WAITING] — Paused, event loop runs others
        │                          │
        │                          └── wait done ──► [RUNNING] again
        │
        └── returns ──► [DONE] — Result is available via await
```

---

## 7. The `gather()` Function

### What `gather()` Does

`asyncio.gather()` is a convenience function that:
1. Takes multiple coroutines (or tasks) as arguments
2. Schedules them all to run concurrently
3. Waits for all of them to complete
4. Returns their results as a list, in the order you provided them

> *"Rather than creating a task for every single one of the coroutines using the create_task function, we can simply use gather and it will automatically run these concurrently for us and collect the results in a list."*

```python
import asyncio

async def fetch_data(item_id: str, delay: float) -> str:
    print(f"Starting: {item_id}")
    await asyncio.sleep(delay)
    return f"Result for {item_id}"

async def main():
    # Pass coroutine objects directly — gather handles the rest
    results = await asyncio.gather(
        fetch_data("item_1", 2),
        fetch_data("item_2", 3),
        fetch_data("item_3", 1),
    )
    
    # results is a list in the ORDER YOU PASSED THEM, not completion order
    for result in results:
        print(result)

asyncio.run(main())
```

**Output:**
```
Starting: item_1
Starting: item_2
Starting: item_3
Result for item_1   ← index 0 (even though item_3 finished first)
Result for item_2   ← index 1
Result for item_3   ← index 2
```

### `gather()` Result Ordering

This is a key feature: results are **always in the order you passed the coroutines**, regardless of which finished first.

```python
results = await asyncio.gather(
    slow_task(),    # takes 5 seconds
    fast_task(),    # takes 1 second
    medium_task(),  # takes 3 seconds
)

# results[0] = slow_task result   (even though it finished last)
# results[1] = fast_task result   (even though it finished first)
# results[2] = medium_task result
```

### `gather()` Error Handling Limitations

> *"One thing you should know about gather is that it's not that great at error handling and it's not going to automatically cancel other coroutines if one of them were to fail."*

```python
import asyncio

async def failing_task():
    await asyncio.sleep(1)
    raise ValueError("Something went wrong!")

async def other_task():
    await asyncio.sleep(2)
    return "I finished fine"

async def main():
    try:
        results = await asyncio.gather(
            failing_task(),
            other_task(),
        )
    except ValueError as e:
        print(f"Error caught: {e}")
        # BUT: other_task() may still be running in the background!
        # This can cause unpredictable state in your application.
```

**The problem:** When `failing_task()` raises an exception, `gather()` propagates it to the caller. But `other_task()` continues running — you've lost control of it. This is why **Task Groups** (Section 8) are generally preferred.

**The `return_exceptions=True` option:**
```python
# Alternative: don't raise, collect exceptions as results
results = await asyncio.gather(
    failing_task(),
    other_task(),
    return_exceptions=True
)

for result in results:
    if isinstance(result, Exception):
        print(f"Task failed: {result}")
    else:
        print(f"Task succeeded: {result}")
```

### When to Use `gather()` vs. Create Tasks Manually

| Scenario | Use |
|----------|-----|
| Fixed number of tasks, simple error handling | `gather()` |
| Need results in order | `gather()` |
| Dynamic number of tasks | Task Group or `gather(*task_list)` |
| Need automatic cancellation on error | Task Group |
| Need individual task control | Create tasks manually |

---

## 8. Task Groups — The Modern Approach

### Why Task Groups?

Task Groups (introduced in Python 3.11) are the preferred modern way to run multiple tasks concurrently. They address `gather()`'s main weakness: error handling.

> *"This is a slightly more preferred way to actually create multiple tasks and to organize them together. The reason for this is it provides some built-in error handling, and if any of the tasks inside of our task group were to fail it will automatically cancel all of the other tasks, which is typically preferable when we are dealing with some advanced errors or larger applications where we want to be a bit more robust."*

### Basic Task Group Usage

```python
import asyncio

async def fetch_data(item_id: str, delay: float) -> str:
    print(f"Starting fetch for {item_id}")
    await asyncio.sleep(delay)
    print(f"Finished fetch for {item_id}")
    return f"Data for {item_id}"

async def main():
    tasks = []
    
    # `async with` creates the task group context
    async with asyncio.TaskGroup() as tg:
        # Create tasks inside the context
        task1 = tg.create_task(fetch_data("item_1", 2))
        task2 = tg.create_task(fetch_data("item_2", 3))
        task3 = tg.create_task(fetch_data("item_3", 1))
        
        tasks = [task1, task2, task3]
    
    # This line is ONLY reached after ALL tasks in the group complete
    # (or if an exception was raised and all tasks were cancelled)
    
    for task in tasks:
        print(task.result())

asyncio.run(main())
```

### Understanding the `async with` Block

> *"The idea is this is a little bit cleaner. It's automatically going to execute all of the tasks that we add inside of the task group. Once all of those tasks have finished, this will stop blocking — meaning we can move down to the next line of code — and at this point we can retrieve all of the different results from our tasks."*

The execution flow:
```
Enter `async with asyncio.TaskGroup() as tg:`
    │
    ▼
Create tasks with tg.create_task() — they start running immediately
    │
    ▼
Reach end of `async with` block
    │
    ▼ BLOCKS HERE until ALL tasks complete (or one fails)
    │
    ▼
Continue past the `async with` block — all tasks are done
    │
    ▼
Retrieve results with task.result()
```

### Error Handling in Task Groups

This is where Task Groups shine:

```python
import asyncio

async def reliable_task(name: str) -> str:
    await asyncio.sleep(1)
    return f"{name} succeeded"

async def failing_task() -> str:
    await asyncio.sleep(0.5)
    raise ValueError("This task failed!")

async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            task1 = tg.create_task(reliable_task("task_1"))
            task2 = tg.create_task(failing_task())
            task3 = tg.create_task(reliable_task("task_3"))
        
        # If we get here, all tasks succeeded
        print(task1.result())
        print(task3.result())
        
    except* ValueError as eg:
        # `except*` handles ExceptionGroup (Python 3.11+)
        print(f"One or more tasks failed:")
        for exc in eg.exceptions:
            print(f"  {exc}")
        
        # task1 and task3 were AUTOMATICALLY CANCELLED when task2 failed
        # No zombie tasks left running!
```

**Note:** Task Groups use `except*` (exception group syntax) from Python 3.11+.

### Task Group vs. `gather()` Comparison

| Feature | `asyncio.gather()` | `asyncio.TaskGroup` |
|---------|-------------------|---------------------|
| **Python version** | 3.x | 3.11+ |
| **Auto-cancel on error** | ❌ No | ✅ Yes |
| **Error handling** | Basic | Robust (ExceptionGroup) |
| **Result collection** | Returns list | Access via task.result() |
| **Dynamic task creation** | Before gather | Inside `async with` block |
| **Preferred for production** | Simple cases | Complex/robust apps |

---

## 9. Concept 4: Futures

### What Is a Future?

> *"A future is a promise of a future result. All it's saying is that a result is to come in the future — you don't know exactly when that's going to be. That's all a future is."*

A `Future` is a low-level object that represents an eventual result. Think of it as a **placeholder** — a box that will eventually contain a value.

**Real-world analogy:** A coat check ticket. When you hand in your coat, you get a ticket (Future). The coat isn't in your hands yet, but you have a *promise* that when you present the ticket, you'll get the coat back. The ticket represents the future result.

### Futures vs. Tasks

| | `asyncio.Future` | `asyncio.Task` |
|--|-----------------|----------------|
| **What it represents** | A promised result | A running coroutine |
| **Created by** | Manual creation, low-level APIs | `asyncio.create_task()` |
| **Who sets result** | External code calls `set_result()` | The coroutine itself returns |
| **Common use** | Library internals, bridges | Application code |
| **Typical user?** | Library authors | Application developers |

### A Future Example

```python
import asyncio

async def set_future_result(future: asyncio.Future, value: str):
    """Simulates some operation that eventually sets a result"""
    print("Starting background operation...")
    await asyncio.sleep(2)  # Simulate work
    future.set_result(value)  # Set the promised value
    print("Future result has been set!")

async def main():
    # Get the running event loop
    loop = asyncio.get_event_loop()
    
    # Create a Future — an empty promise
    future = loop.create_future()
    
    # Schedule a task that will eventually fulfill the promise
    asyncio.create_task(set_future_result(future, "The promised value"))
    
    print("Waiting for the future...")
    # Await the future — we're waiting for set_result() to be called
    result = await future
    print(f"Got result: {result}")

asyncio.run(main())
```

**Output:**
```
Waiting for the future...
Starting background operation...
Future result has been set!
Got result: The promised value
```

### The Key Distinction

> *"When we use a future we're just waiting for some value to be available. We're not waiting for an entire task or an entire coroutine to finish."*

```python
# Awaiting a Task → wait for the entire coroutine to complete
result = await some_task  

# Awaiting a Future → wait for set_result() to be called
# The task that calls set_result() may continue running after!
result = await some_future
```

### When You'll Encounter Futures

You probably won't create Futures yourself in typical application code, but you'll encounter them in:
- `asyncio.run_in_executor()` — running sync code in a thread pool
- Low-level network/protocol implementations
- Integration between async and sync code
- Third-party async library internals

```python
import asyncio
import concurrent.futures

def sync_blocking_function(x):
    """A synchronous function we need to call from async code"""
    import time
    time.sleep(1)  # Blocking! Can't use await here
    return x * 2

async def main():
    loop = asyncio.get_event_loop()
    
    with concurrent.futures.ThreadPoolExecutor() as pool:
        # run_in_executor returns a Future
        future = loop.run_in_executor(pool, sync_blocking_function, 21)
        result = await future  # Wait for the blocking function
        print(f"Result: {result}")  # 42

asyncio.run(main())
```

---

## 10. Concept 5: Synchronization Primitives

### Why Synchronization Is Needed

Even though AsyncIO runs on a single thread, problems can arise when multiple coroutines access shared resources. Because tasks switch at `await` points, you can have:

```
Task A reads shared_value (= 5)
    → Task A awaits (switches to Task B)
Task B reads shared_value (= 5)
Task B modifies shared_value (= 6)
    → Task B awaits (switches back to Task A)
Task A modifies shared_value based on its old reading (5+1 = 6 instead of 7!)
```

This is a **race condition** — the final value depends on the timing of task switching.

**Real-world analogy:** Two people editing the same Google Doc simultaneously without seeing each other's changes in real time — one person's edits overwrite the other's.

AsyncIO provides **synchronization primitives** to control access to shared resources:

| Primitive | Purpose |
|-----------|---------|
| **Lock** | Only one coroutine can access a resource at a time |
| **Semaphore** | N coroutines can access a resource simultaneously |
| **Event** | Signal other coroutines when something has happened |
| **Condition** | More complex conditional synchronization |

---

## 11. Locks

### What Is a Lock?

> *"Let's say we have some shared resource — maybe a database, a table, a file — and we want to make sure that no two coroutines are working on this at the same time. What we can do is create a lock."*

A Lock is a binary synchronization mechanism — it's either **locked** or **unlocked**. Only one coroutine can hold the lock at a time. Any other coroutine trying to acquire the lock must wait until it's released.

**Real-world analogy:** A single-occupancy bathroom. The door lock ensures only one person is inside at a time. If someone tries the door and it's locked, they wait. When the occupant leaves and unlocks the door, the next person can enter.

### Using a Lock

```python
import asyncio

async def modify_resource(lock: asyncio.Lock, name: str, shared_data: dict):
    print(f"{name}: Attempting to acquire lock...")
    
    async with lock:  # Acquire lock — waits if another coroutine holds it
        print(f"{name}: Lock acquired! Modifying resource...")
        print(f"  Resource before: {shared_data['value']}")
        
        await asyncio.sleep(1)  # Simulate some I/O during modification
        
        shared_data['value'] += 1  # Critical section
        print(f"  Resource after: {shared_data['value']}")
        
    # Lock is automatically released when exiting `async with`
    print(f"{name}: Lock released.")

async def main():
    lock = asyncio.Lock()
    shared_data = {"value": 0}
    
    # Run 5 coroutines that all want to modify the same resource
    await asyncio.gather(
        modify_resource(lock, "Task-1", shared_data),
        modify_resource(lock, "Task-2", shared_data),
        modify_resource(lock, "Task-3", shared_data),
        modify_resource(lock, "Task-4", shared_data),
        modify_resource(lock, "Task-5", shared_data),
    )
    
    print(f"\nFinal value: {shared_data['value']}")  # Should be 5

asyncio.run(main())
```

**What happens without the lock:**
```
# Without lock — unpredictable results
# Multiple tasks read value=0, all increment to 1, final value = 1 (WRONG!)

# With lock — guaranteed correctness
# Tasks execute the critical section one at a time, final value = 5 (CORRECT!)
```

### The `async with lock:` Pattern

> *"When we create a lock, we have the ability to acquire the lock... async with lock will check if any other coroutine is currently using the lock. If it is, it's going to wait until that coroutine is finished. If it's not, it's going to go into this block of code."*

Using `async with` is the **recommended** way to use locks because it automatically releases the lock even if an exception occurs:

```python
# CORRECT — always releases, even if exception occurs
async with lock:
    await risky_operation()  # If this throws, lock is still released!

# FRAGILE — if exception occurs, lock is never released!
await lock.acquire()
await risky_operation()
lock.release()  # Never reached if exception thrown above!
```

### Lock Use Cases

```python
# Use case 1: Protecting a file
async def append_to_file(lock, filename, content):
    async with lock:
        with open(filename, 'a') as f:
            await asyncio.sleep(0.01)  # Simulate I/O
            f.write(content + '\n')

# Use case 2: Protecting a counter
async def increment_counter(lock, counter_dict):
    async with lock:
        current = counter_dict['count']
        await asyncio.sleep(0)  # Yield to event loop (simulate work)
        counter_dict['count'] = current + 1

# Use case 3: Protecting a database connection
async def safe_db_operation(lock, db_conn, query):
    async with lock:
        result = await db_conn.execute(query)
        return result
```

---

## 12. Semaphores

### What Is a Semaphore?

> *"A semaphore works very similarly to a lock, however it allows multiple coroutines to have access to the same object at the same time — but we can decide how many we want that to be."*

A Semaphore maintains a counter. When a coroutine acquires it, the counter decrements. When it releases, the counter increments. If the counter reaches zero, new coroutines must wait.

**Real-world analogy:** A parking garage with 5 spaces. When all 5 are taken, incoming cars must wait until a space opens. The garage can serve 5 cars simultaneously — not 1 (like a lock) and not unlimited.

### Using a Semaphore

```python
import asyncio

async def access_resource(semaphore: asyncio.Semaphore, name: str):
    print(f"{name}: Waiting for semaphore...")
    
    async with semaphore:  # Acquire (decrements counter)
        print(f"{name}: Access granted! Working...")
        await asyncio.sleep(2)  # Simulate work
        print(f"{name}: Done, releasing semaphore.")
    
    # Semaphore released automatically (counter incremented)

async def main():
    # Allow maximum 2 concurrent accesses
    semaphore = asyncio.Semaphore(2)
    
    # Launch 5 coroutines, but only 2 will run at a time
    await asyncio.gather(
        access_resource(semaphore, "Task-1"),
        access_resource(semaphore, "Task-2"),
        access_resource(semaphore, "Task-3"),
        access_resource(semaphore, "Task-4"),
        access_resource(semaphore, "Task-5"),
    )

asyncio.run(main())
```

**Output (conceptual):**
```
Task-1: Access granted! Working...
Task-2: Access granted! Working...
Task-3: Waiting for semaphore...    ← blocked! semaphore at 0
Task-4: Waiting for semaphore...    ← blocked!
Task-5: Waiting for semaphore...    ← blocked!
Task-1: Done, releasing semaphore.
Task-3: Access granted! Working...  ← Task-3 gets the freed slot
Task-2: Done, releasing semaphore.
Task-4: Access granted! Working...  ← Task-4 gets the freed slot
...
```

### Real-World Semaphore Use Case: Rate-Limited API Calls

> *"We're going to send a bunch of different network requests. We can do a few of them at the same time but we can't do a thousand or 10,000 at the same time. So in that case we would create a semaphore."*

```python
import asyncio
import aiohttp  # pip install aiohttp

async def fetch_url(session, semaphore, url):
    async with semaphore:  # Max N concurrent requests
        async with session.get(url) as response:
            return await response.text()

async def scrape_many_urls(urls):
    # Limit to 5 concurrent requests (be a good citizen!)
    semaphore = asyncio.Semaphore(5)
    
    async with aiohttp.ClientSession() as session:
        tasks = [
            fetch_url(session, semaphore, url)
            for url in urls
        ]
        results = await asyncio.gather(*tasks)
    
    return results

# Without semaphore: might send 1000 requests simultaneously → API ban!
# With semaphore(5): always at most 5 simultaneous → polite and safe
```

### Lock vs. Semaphore

| | Lock | Semaphore |
|--|------|-----------|
| **Concurrent access** | 1 coroutine | N coroutines |
| **Use when** | Complete exclusivity needed | Limiting (not eliminating) concurrent access |
| **Analogy** | Single-occupancy bathroom | Parking garage with N spaces |
| **Counter** | Binary (0 or 1) | Integer (0 to N) |

---

## 13. Events

### What Is an Event?

> *"The event allows us to do some simpler synchronization. We can create an event and what we can do is we can await the event to be set and we can set the event. This acts as a simple Boolean flag and allows us to block other areas of our code until we've set this flag to be true."*

An `asyncio.Event` is a simple signaling mechanism. One coroutine sets the event; others wait for it to be set.

**Real-world analogy:** A starting pistol in a race. All runners (coroutines) crouch at the starting line (await the event). When the pistol fires (event is set), they all start running simultaneously.

### Using an Event

```python
import asyncio

async def waiter(event: asyncio.Event, name: str):
    print(f"{name}: Waiting for the event...")
    await event.wait()  # Blocks until event.set() is called
    print(f"{name}: Event received! Continuing execution.")

async def setter(event: asyncio.Event):
    print("Setter: Starting some operation...")
    await asyncio.sleep(2)  # Simulate work
    print("Setter: Operation complete! Setting the event.")
    event.set()  # Unblocks all waiters simultaneously!

async def main():
    event = asyncio.Event()
    
    # Run waiters and setter concurrently
    await asyncio.gather(
        waiter(event, "Waiter-1"),
        waiter(event, "Waiter-2"),
        waiter(event, "Waiter-3"),
        setter(event),
    )

asyncio.run(main())
```

**Output:**
```
Waiter-1: Waiting for the event...
Waiter-2: Waiting for the event...
Waiter-3: Waiting for the event...
Setter: Starting some operation...
Setter: Operation complete! Setting the event.
Waiter-1: Event received! Continuing execution.
Waiter-2: Event received! Continuing execution.
Waiter-3: Event received! Continuing execution.
```

All three waiters unblock **simultaneously** when the event is set.

### Event vs. Lock vs. Semaphore

| | Event | Lock | Semaphore |
|--|-------|------|-----------|
| **Purpose** | Signal/notify | Exclusive access | Limited concurrent access |
| **State** | Set/unset | Locked/unlocked | Counter (0 to N) |
| **How many unblocked** | All waiters at once | One at a time | Up to N at a time |
| **Stays triggered** | Yes (until `clear()`) | No (per acquire) | No |
| **Use case** | "Ready" signals, startup | Protecting shared data | Rate limiting |

### Resetting an Event

```python
async def main():
    event = asyncio.Event()
    
    # Use the event
    event.set()
    await some_task_that_waits_for_event(event)
    
    # Reset the event to use it again
    event.clear()
    
    # Now new waiters will block again
    await another_task(event)
    event.set()
```

### Event Use Cases

```python
# Use case 1: Initialization signal
async def worker(ready_event):
    await ready_event.wait()  # Wait for initialization
    # Now do actual work safely

async def initializer(ready_event):
    await load_config()
    await connect_database()
    ready_event.set()  # Signal: everything is ready!

# Use case 2: Shutdown signal
async def long_running_task(shutdown_event):
    while not shutdown_event.is_set():
        await do_some_work()
        await asyncio.sleep(0.1)
    print("Shutting down gracefully...")

# Use case 3: Producer-consumer coordination
async def producer(event, queue):
    data = await fetch_data()
    await queue.put(data)
    event.set()  # Signal: new data available!

async def consumer(event, queue):
    await event.wait()  # Wait for data
    event.clear()  # Reset for next batch
    data = await queue.get()
    await process(data)
```

---

## 14. Comparison Tables and Visual Breakdowns

### Async Primitives at a Glance

| Primitive | Creates | Controls | Releases | Best For |
|-----------|---------|----------|----------|---------|
| `asyncio.run()` | Event loop | Program entry | On completion | Starting async programs |
| `async def` | Coroutine function | Execution flow | On return/await | Defining async operations |
| `await` | — | Execution pause | When awaited resolves | Waiting for async results |
| `create_task()` | Task | Concurrent scheduling | On completion | Running coroutines concurrently |
| `gather()` | Multiple tasks | Concurrent run + results | When all done | Collecting multiple results |
| `TaskGroup` | Task group | Concurrent + error handling | When all done or error | Production concurrent code |
| `Future` | Promised value | Result availability | `set_result()` | Low-level libraries |
| `Lock` | Exclusive access | 1 at a time | `async with` exit | Protecting shared data |
| `Semaphore` | Limited access | N at a time | `async with` exit | Rate limiting |
| `Event` | Signal | All waiters | `.set()` | Coordination signals |

### Performance Impact of Concurrency Models

```
Single task (sequential baseline):
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  30 seconds

Sequential coroutines (no tasks):
▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  30 seconds (same!)

With asyncio.gather() / create_task():
▓▓▓▓▓▓▓▓▓▓                        10 seconds (3x faster!)

With multiprocessing (CPU-bound):
▓▓▓▓▓▓▓▓▓▓                        ~7.5 seconds (4 cores)
```
*Example: 30 I/O-bound tasks of 1 second each*

### The Anatomy of an Async Program

```python
import asyncio                          # 1. Import the module

# 2. Define coroutine functions (async def)
async def worker(name: str, data: str) -> str:
    await asyncio.sleep(1)             # 3. await I/O operations
    return f"{name} processed {data}"

async def main():                       # 4. Main entry coroutine
    # 5. Create tasks for concurrency
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(worker("W1", "item_a"))
        task2 = tg.create_task(worker("W2", "item_b"))
    
    # 6. Access results after all tasks done
    print(task1.result())
    print(task2.result())

asyncio.run(main())                     # 7. Run the event loop
```

---

## 15. Common Mistakes and Troubleshooting

### Mistake 1: Using `time.sleep()` Instead of `asyncio.sleep()`

**Problem:**
```python
async def bad_task():
    import time
    time.sleep(2)  # BLOCKS the entire event loop for 2 seconds!
    # No other tasks can run during this time
```

**Solution:**
```python
async def good_task():
    await asyncio.sleep(2)  # Yields to event loop
    # Other tasks run while this sleeps
```

**How to spot it:** All your tasks seem to run sequentially even though you created them as tasks.

### Mistake 2: Forgetting to Await a Coroutine

**Problem:**
```python
async def main():
    result = fetch_data("item_1")  # Missing await!
    print(result)  # Prints: <coroutine object fetch_data at 0x...>
    # Warning: "coroutine 'fetch_data' was never awaited"
```

**Solution:**
```python
async def main():
    result = await fetch_data("item_1")  # Add await!
    print(result)  # Prints: actual result
```

### Mistake 3: Using `await` Outside an Async Function

**Problem:**
```python
def sync_function():
    result = await fetch_data()  # SyntaxError!
```

**Solution:**
```python
# Option 1: Make the function async
async def async_function():
    result = await fetch_data()

# Option 2: Use asyncio.run() if at top level
def sync_function():
    result = asyncio.run(fetch_data())
```

### Mistake 4: Calling `asyncio.run()` Inside an Async Function

**Problem:**
```python
async def main():
    result = asyncio.run(fetch_data())  # RuntimeError!
    # "This event loop is already running"
```

**Solution:**
```python
async def main():
    result = await fetch_data()  # Just await it
```

### Mistake 5: Not Creating Tasks for Concurrency

**Problem:**
```python
async def main():
    # These run SEQUENTIALLY, not concurrently!
    result1 = await fetch_data("id1", 2)
    result2 = await fetch_data("id2", 2)
    # Total time: 4 seconds
```

**Solution:**
```python
async def main():
    # These run CONCURRENTLY
    result1, result2 = await asyncio.gather(
        fetch_data("id1", 2),
        fetch_data("id2", 2),
    )
    # Total time: 2 seconds
```

### Mistake 6: Sharing Non-Thread-Safe State Without Locks

**Problem:**
```python
counter = 0

async def increment():
    global counter
    val = counter          # Read
    await asyncio.sleep(0) # Task switch happens here!
    counter = val + 1      # Write (based on stale read)
```

**Solution:**
```python
counter = 0
lock = asyncio.Lock()

async def safe_increment():
    global counter
    async with lock:
        val = counter
        await asyncio.sleep(0)
        counter = val + 1  # Protected by lock
```

### Mistake 7: Using Synchronous Libraries in Async Code

**Problem:**
```python
import requests  # Synchronous HTTP library

async def bad_fetch(url):
    response = requests.get(url)  # Blocks event loop!
    return response.text
```

**Solution:**
```python
import aiohttp  # Async HTTP library

async def good_fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()
```

**Common synchronous → async library replacements:**

| Sync Library | Async Replacement |
|-------------|-------------------|
| `requests` | `aiohttp`, `httpx` |
| `sqlite3` | `aiosqlite` |
| `redis` (sync) | `aioredis` |
| `psycopg2` | `asyncpg` |
| `time.sleep()` | `asyncio.sleep()` |
| `open()` (files) | `aiofiles` |

### Troubleshooting Checklist

```
□ Program runs but seems slow
    → Are you using await with sequential coroutines? Use tasks/gather.
    → Are you using time.sleep()? Switch to asyncio.sleep().

□ "coroutine was never awaited" warning
    → You called an async function but didn't await it.

□ "RuntimeError: This event loop is already running"
    → You called asyncio.run() inside an async function. Use await instead.

□ "SyntaxError: 'await' outside async function"
    → Move the await inside an async def function.

□ Tasks seem to run one at a time
    → You're awaiting coroutines directly. Wrap them in create_task() or gather().

□ Race conditions on shared data
    → Add asyncio.Lock() around critical sections.

□ Too many concurrent requests (getting rate limited or banned)
    → Add asyncio.Semaphore(N) to limit concurrency.

□ One task failing silently
    → Add try/except inside the task, or use TaskGroup for propagation.
```

---

## 16. Best Practices

### 1. Always Use `asyncio.run()` as the Entry Point

```python
# GOOD — single, clear entry point
async def main():
    await do_work()

if __name__ == "__main__":
    asyncio.run(main())
```

### 2. Use Task Groups Over `gather()` in Production

```python
# Prefer TaskGroup for automatic error handling
async with asyncio.TaskGroup() as tg:
    tasks = [tg.create_task(operation(item)) for item in items]
```

### 3. Set Timeouts on Long-Running Operations

```python
async def fetch_with_timeout(url):
    try:
        async with asyncio.timeout(5.0):  # Python 3.11+
            return await fetch(url)
    except asyncio.TimeoutError:
        print(f"Request to {url} timed out!")
        return None

# Or for older Python:
result = await asyncio.wait_for(fetch(url), timeout=5.0)
```

### 4. Use Semaphores to Avoid Overwhelming External Services

```python
# Never fire unlimited concurrent requests at an API
sem = asyncio.Semaphore(10)

async def safe_request(url):
    async with sem:
        return await fetch(url)
```

### 5. Avoid Blocking the Event Loop

```python
# If you MUST use blocking code, run it in a thread pool
import asyncio
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor()

async def call_blocking_function(arg):
    loop = asyncio.get_running_loop()
    result = await loop.run_in_executor(executor, blocking_function, arg)
    return result
```

### 6. Use Type Hints for Async Functions

```python
from typing import Coroutine, Any
import asyncio

async def fetch_data(url: str) -> dict:
    ...

async def process_all(urls: list[str]) -> list[dict]:
    tasks = [asyncio.create_task(fetch_data(url)) for url in urls]
    return [await task for task in tasks]
```

### 7. Handle Cancellation Gracefully

```python
async def cancellable_task():
    try:
        while True:
            await do_work()
            await asyncio.sleep(1)
    except asyncio.CancelledError:
        await cleanup()  # Clean up resources before cancellation
        raise  # Re-raise so the event loop knows we handled it
```

### 8. Always Close Resources

```python
# Use async context managers for resources
async def main():
    async with aiohttp.ClientSession() as session:
        result = await fetch(session, url)
    # Session is automatically closed

# Or use try/finally
session = aiohttp.ClientSession()
try:
    result = await fetch(session, url)
finally:
    await session.close()
```

---

## 17. Real-World Project Patterns

### Pattern 1: Async Web Scraper with Rate Limiting

```python
import asyncio
import aiohttp
from typing import Optional

async def scrape_page(
    session: aiohttp.ClientSession,
    semaphore: asyncio.Semaphore,
    url: str
) -> Optional[str]:
    async with semaphore:
        try:
            async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as resp:
                if resp.status == 200:
                    return await resp.text()
                return None
        except Exception as e:
            print(f"Failed to fetch {url}: {e}")
            return None

async def scrape_all(urls: list[str], max_concurrent: int = 5) -> list[Optional[str]]:
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async with aiohttp.ClientSession() as session:
        tasks = [
            scrape_page(session, semaphore, url)
            for url in urls
        ]
        return await asyncio.gather(*tasks, return_exceptions=True)

# Usage
urls = [f"https://example.com/page/{i}" for i in range(100)]
results = asyncio.run(scrape_all(urls, max_concurrent=5))
```

### Pattern 2: Producer-Consumer with Queue

```python
import asyncio

async def producer(queue: asyncio.Queue, items: list):
    """Produces items and puts them in the queue"""
    for item in items:
        await queue.put(item)
        print(f"Produced: {item}")
        await asyncio.sleep(0.1)
    
    # Signal consumers to stop
    await queue.put(None)

async def consumer(queue: asyncio.Queue, name: str):
    """Consumes and processes items from the queue"""
    while True:
        item = await queue.get()
        
        if item is None:  # Poison pill — stop signal
            await queue.put(None)  # Pass signal to other consumers
            break
        
        print(f"{name}: Processing {item}")
        await asyncio.sleep(0.5)  # Simulate processing time
        queue.task_done()

async def main():
    queue = asyncio.Queue(maxsize=10)
    items = list(range(20))
    
    async with asyncio.TaskGroup() as tg:
        tg.create_task(producer(queue, items))
        tg.create_task(consumer(queue, "Consumer-1"))
        tg.create_task(consumer(queue, "Consumer-2"))

asyncio.run(main())
```

### Pattern 3: Async Resource Pool

```python
import asyncio
from contextlib import asynccontextmanager

class ConnectionPool:
    def __init__(self, max_connections: int):
        self._semaphore = asyncio.Semaphore(max_connections)
        self._connections = []
    
    @asynccontextmanager
    async def acquire(self):
        async with self._semaphore:
            conn = await self._create_connection()
            try:
                yield conn
            finally:
                await self._release_connection(conn)
    
    async def _create_connection(self):
        # Create actual DB/network connection
        await asyncio.sleep(0.01)  # Simulate connection time
        return {"id": id(object())}
    
    async def _release_connection(self, conn):
        await asyncio.sleep(0)  # Cleanup

# Usage
pool = ConnectionPool(max_connections=5)

async def do_query(pool, query_id):
    async with pool.acquire() as conn:
        print(f"Query {query_id} using connection {conn['id']}")
        await asyncio.sleep(1)
        return f"Result {query_id}"
```

---

## 18. Quick Reference Cheat Sheet

### Essential Syntax

```python
# Import
import asyncio

# Define coroutine
async def my_func():
    ...

# Run event loop (entry point)
asyncio.run(my_func())

# Await a coroutine (inside async function)
result = await my_func()

# Create a task (concurrent scheduling)
task = asyncio.create_task(my_func())
result = await task

# Run multiple coroutines concurrently
results = await asyncio.gather(coro1(), coro2(), coro3())

# Task group (preferred for production)
async with asyncio.TaskGroup() as tg:
    t1 = tg.create_task(coro1())
    t2 = tg.create_task(coro2())
# All tasks done here

# Timeout
async with asyncio.timeout(5.0):
    result = await slow_operation()

# Sleep (non-blocking)
await asyncio.sleep(1.0)
```

### Synchronization Primitives

```python
# Lock — one at a time
lock = asyncio.Lock()
async with lock:
    # critical section

# Semaphore — N at a time
sem = asyncio.Semaphore(5)
async with sem:
    # limited access section

# Event — notify waiters
event = asyncio.Event()
await event.wait()   # block until set
event.set()          # unblock all waiters
event.clear()        # reset
event.is_set()       # check state
```

### When to Use What

```
Need to run async code at all?
  → asyncio.run(main())

Need to wait for one thing?
  → await coroutine()

Need multiple things to run at the same time?
  → asyncio.gather() or TaskGroup

Need to limit concurrent access?
  → asyncio.Semaphore(N)

Need exclusive access to shared data?
  → asyncio.Lock()

Need to signal between coroutines?
  → asyncio.Event()

Need to run blocking code?
  → loop.run_in_executor()
```

### Python Version Compatibility

| Feature | Python Version |
|---------|---------------|
| `asyncio` module | 3.4+ |
| `async`/`await` syntax | 3.5+ |
| `asyncio.run()` | 3.7+ |
| `asyncio.TaskGroup` | 3.11+ |
| `asyncio.timeout()` | 3.11+ |
| `except*` (ExceptionGroup) | 3.11+ |

### The Five Key Concepts — One Liner Summaries

| Concept | One-Line Summary |
|---------|-----------------|
| **Event Loop** | The central manager that runs and switches between all async tasks |
| **Coroutine** | An `async def` function that can pause and resume at `await` points |
| **Task** | A scheduled coroutine that runs concurrently with other tasks |
| **Future** | A placeholder for a result that will be available later |
| **Sync Primitives** | Tools (Lock, Semaphore, Event) to coordinate access to shared resources |

---

*Python version referenced throughout: 3.11+. Update your Python version if features like `TaskGroup`, `asyncio.timeout()`, or `except*` are unavailable.*
