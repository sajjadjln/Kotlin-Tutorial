### **The Need for Asynchronous Programming**

- **Historical Context**:
    
    Computer speeds used to double every 18 months (Moore's Law), but this stopped around 2005.
    
    Modern computers focus on increasing cores rather than clock speed.
    
    Applications must leverage multiple cores to improve performance
    
    - **Challenges with Threads**:
    
    Threads are heavyweight, meaning they consume significant resources (e.g., memory) and have high overhead.
    
    Managing threads is complex and can lead to inefficiencies.
    
- **ForkJoinPool for Multithreading in Java**
    - Introduced in Java 7 to manage lightweight, **small tasks efficiently**.
    - Helps leverage **multi-core processors** without manually managing threads.

---

### **2. Cores, Threads, and Concurrency**

**Cores** are independent processing units within a CPU. Each core can handle tasks separately or work together on complex problems. **Threads**, on the other hand, are execution paths within a process, allowing multiple tasks to be executed concurrently on a single core.

Modern processors often support **multithreading**, where each core can manage multiple threads simultaneously, increasing efficiency. For instance, a quad-core processor with two threads per core can handle up to eight threads concurrently.

**Concurrency** refers to the ability to manage multiple tasks at once, often by overlapping their execution. It’s crucial for modern applications that require responsiveness, such as handling user input while performing background tasks.

### **3. Asynchronous Programming: The Key to Responsiveness**

While concurrency focuses on managing multiple tasks, **asynchronous programming** optimizes how tasks wait for resources like I/O operations. Instead of blocking a thread, asynchronous programming allows other tasks to execute while waiting, improving efficiency.

In programming, asynchronous techniques include:

- **Callbacks**: Functions passed as arguments to be executed later.
- **Promises/Futures**: Objects representing a value that will be available in the future.
- **Coroutines**: Lightweight, cooperative routines designed for asynchronous tasks.
- **Callbacks:**
    - calling a function inside another function also can be using threads with it like
    
    ```jsx
    fun main() {
    
        fetchDataFromServer(object : Callback<String> {
            override fun onSuccess(result: String) {
                println("Received: $result")
            }
    
            override fun onError(error: Exception) {
                println("Error: ${error.message}")
            }
        })
    
        println("Fetching data...") // This prints immediately
    
    }
    
    fun fetchDataFromServer(callback: Callback<String>) {
        // Simulating asynchronous work (e.g., network request)
        Thread {
            try {
                Thread.sleep(2000) // Simulate network delay
                val data = "Data from server" // Simulated data
                callback.onSuccess(data) // Notify success
            } catch (e: Exception) {
                callback.onError(e) // Notify error
            }
        }.start()
    }
    
    interface Callback<T> {
        fun onSuccess(result: T)
        fun onError(error: Exception)
    }
    ```
    
    - Work but lead to **callback hell** with deeply nested calls.
- **Futures:**
    - Provide a more **fluent style** than callbacks but have **ceremony** (e.g., `thenCompose`) and require careful exception handling.
- **Coroutines:**
    - Enable **structured concurrency**, making asynchronous code look like synchronous code.
    - Simplify thread switching, looping, and error handling.
- **Virtual Threads:**
    - Focus on addressing **blocking I/O** in Java.
    - Lightweight but lack **structured concurrency**.

---

### **Core Concepts**

1. **Threads vs. Coroutines**:
    - Threads are system-level entities, and they require significant resources like memory and CPU scheduling.
    - Coroutines, on the other hand, are lightweight. They can share threads, and when a coroutine suspends (e.g., using `delay`), it doesn't block the underlying thread, allowing other coroutines to run on it.
2. **Non-blocking Nature of Coroutines**:
    - `Thread.sleep` blocks the thread entirely, making it unavailable for other work.
    - `delay` suspends the coroutine without blocking the thread, enabling better resource utilization. and the thread is still can get to work
3. **Scalability**:
    - Threads are limited by system resources (e.g., memory and process limits).
    - Coroutines scale efficiently, allowing millions of concurrent tasks without overwhelming the system.
4. **Performance Comparison**:
    - Threads with 10,000 tasks:
        - High CPU utilization.
        - Takes longer to finish (88 seconds for 10,000 threads with 500 loops of 10 ms sleep).
    - Coroutines with 10,000 tasks:
        - Minimal CPU usage.
        - Faster completion (30 seconds for 10,000 coroutines with the same workload).
5. **Resource Management**:
    - Threads require memory for stacks and have overhead for scheduling.
    - Coroutines avoid this by leveraging suspensions and efficient resource sharing.

**Why do coroutines not require as much memory as threads?**

- Threads require a fixed amount of memory for their stack, typically several megabytes. Each thread has its stack to store local variables, method call states, etc., which adds significant overhead when creating a large number of threads.
- Coroutines, however, don't have their own stack. They leverage the stack of the thread they run on and save their execution state (e.g., variable values, program counter) in a small object when suspended. This makes coroutines lightweight and enables them to scale efficiently.

```kt
@OptIn(DelicateCoroutinesApi::class)
suspend fun runCoroutineTest(): Long = measureTimeMillis {
    val jobs = mutableListOf<Job>()

    for (i in 1..NUM_TASK) {
        jobs.add(GlobalScope.launch {
            for (x in 1..loops) {
                delay(wait_ms)
            }
        })
    }
    jobs.forEach { it.join() }
}

fun runThreadTest(): Long = measureTimeMillis {
    val threads = mutableListOf<Thread>()

    for (i in 1..NUM_TASK) {
        threads.add(Thread {
            for (x in 1..loops) {
                sleep(wait_ms)
            }
        }.apply { start() })
    }
    threads.forEach { it.join() }
}

@OptIn(DelicateCoroutinesApi::class)
suspend fun main() {
    println("System Information:")
    println("Available processors: ${Runtime.getRuntime().availableProcessors()}")
    println("Max memory: ${Runtime.getRuntime().maxMemory() / 1024 / 1024}MB")
    println("\nRunning benchmarks...")

    repeat(3) { iteration ->
        println("\nIteration ${iteration + 1}")

        val threadTime = runThreadTest()
        println("Thread implementation took: ${threadTime}ms")

        // Give system some time to clean up
        delay(1000)

        val coroutineTime = runCoroutineTest()
        println("Coroutine implementation took: ${coroutineTime}ms")

        // Calculate difference
        val difference = threadTime - coroutineTime
        val faster = if (difference > 0) "Coroutines" else "Threads"
        println("$faster were faster by ${kotlin.math.abs(difference)}ms")
    }
}
```
