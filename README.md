# Multiprocessing and Multiprogramming

Resources:

1. [Watch the video on YouTube](https://www.youtube.com/watch?v=Fn0xBsmact4&list=PLvv0ScY6vfd_ocTP2ZLicgqKnvq50OCXM)
2. https://github.com/methylDragon/coding-notes/blob/master/C++/07%20C++%20-%20Threading%20and%20Concurrency.md
---

## Difference between Concurrency (Multiprogramming) and Parallelism (Multiprocessing)

### **Concurrency**
- **Definition**:  
  Concurrency refers to the ability of a system to manage multiple tasks (processes or threads) at the same time, giving the illusion of simultaneous execution. However, these tasks might not actually be executing at the same moment. Instead, the CPU switches between tasks rapidly (context switching).

- **Analogy**:  
  Think of a single chef preparing multiple dishes by switching between tasks like chopping vegetables, boiling water, and stirring a pot.

- **Example**:  
  If two threads share a single-core CPU, only one thread executes at any moment, but the OS schedules them in a way that they appear to run concurrently.

---

### **Parallelism**
- **Definition**:  
  Parallelism involves executing multiple tasks simultaneously on multiple processors or cores. Each task runs on its own processor without interference from others.

- **Analogy**:  
  Think of multiple chefs, each preparing their own dish simultaneously in a kitchen.

- **Example**:  
  On a multi-core CPU, two threads can run on separate cores at the same time, genuinely executing in parallel.

---

### **Key Differences**

| Feature          | Concurrency                              | Parallelism                              |
|------------------|------------------------------------------|------------------------------------------|
| Execution Model  | Appears simultaneous (time-sliced)       | Truly simultaneous (multi-core execution) |
| Hardware         | Single-core CPU                         | Multi-core CPU                           |
| Analogy          | Single chef switching between tasks      | Multiple chefs working simultaneously    |

---

## `<thread>` Library in C++

The `<thread>` library in C++ enables the creation, management, and deletion of threads.

### **Example Code:**
```cpp
#include <iostream>
#include <thread>
using namespace std;

void threadFn(int x) {
    cout << "The thread-id is: " << this_thread::get_id() << endl;
    cout << "The argument passed is: " << x << endl;
}

int main() {
    std::thread t(threadFn, 10); // Create a new thread and pass an argument
    t.join(); // Wait for the thread to complete
    cout << "The main process is complete, exiting now" << endl;
    return 0;
}

```

## Lambda Functions:
Lambda functions can be declared in variables, inline functions, etc.

Example Code:

```cpp
#include <iostream>
#include <thread>
using namespace std;

int main() {
    auto lambda = [](int x) {
        cout << "The process-id is: " << this_thread::get_id() << " and argument passed is: " << x << endl;        
    };
    
    thread t(lambda, 10);
    t.join();
    cout << "The main process is complete, exiting now" << endl;
    return 0;
}

```


## Concurrency vs Parallelism in Threading
Creating threads does not mean that the code is executing in parallel; they must be managed to work concurrently.

Consider the difference between these two snippets:

First Snippet:

```cpp
for(int i=0; i < 10; i++) {
    threads.push_back(std::thread(lambda, i));
}

for(int i=0; i < 10; i++) {
    threads[i].join();
}

```

Second Snippet:


```cpp
for(int i=0; i < 10; i++) {
    threads.push_back(std::thread(lambda, i));
    threads[i].join();
}

```


## Need for `<jthread>` Library
The `<jthread>` library in C++ simplifies thread management and solves issues associated with std::thread. It addresses automatic joining, cancellation, and thread resource management. Here's how jthread differs from std::thread:

Automatic joining of threads after execution
No need for manual joining or detaching threads
Easier handling of exceptions and cancellations
Need for <mutex> Library
The mutex library helps avoid data races and data inconsistency when it comes to shared resources between multiple threads. Here's an example:

Code with Potential Data Race:
```cpp
void increment() {
    // gLock.lock();
    critical_resource = critical_resource + 1;
    cout << "The critical resource is: " << critical_resource << endl;
    // gLock.unlock();
    return;
}

int main() {
    vector<thread> arr;
    for(int i=0; i<1000; i++)
        arr.push_back(thread(increment));
    
    for(int i=0; i<1000; i++)
        arr[i].join();
    
    return 0;
}

```

Without a lock, two threads may read the same value of critical_resource and both increment it, causing data inconsistency.

Fixed Code with Mutex:
```cpp
mutex gLock;
static int critical_resource = 0;

void increment() {
    lock_guard<mutex> lockGuard(gLock);
    critical_resource = critical_resource + 1;
    cout << "The critical resource is: " << critical_resource << endl;
    return;
}

int main() {
    vector<thread> arr;
    for(int i=0; i<1000; i++)
        arr.push_back(thread(increment));
    
    for(int i=0; i<1000; i++)
        arr[i].join();
    
    return 0;
}

```

## Atomic Variables
If you don't want to manage the mutex lock but still need to read/write shared data, you can use atomic variables.

Code with Atomic Variable:
```cpp
static std::atomic<int> shared_value = 0;

void shared_value_increment() {
    shared_value += 1;
}

int main() {
    std::vector<std::thread> threads;
    for(int i=0; i < 1000; i++) {
        threads.push_back(std::thread(shared_value_increment));
    }

    for(int i=0; i < 1000; i++) {
        threads[i].join();
    }

    std::cout << "Shared value: " << shared_value << std::endl;
    return 0;
}

```

## Conditional Variables
To save unnecessary computation when checking for mutex lock, we use conditional variables. If a thread is already holding a mutex, others wait for it to be released, and once released, it can notify other waiting threads to resume.

Example with Conditional Variable:
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <vector>
#include <condition_variable>
using namespace std;

mutex gLock;
condition_variable con;

int main() {
    bool notify = false;
    int data = 0;

    thread reporter([&] {
        unique_lock<mutex> lock(gLock);
        con.wait(lock);
        cout << "Reporter function reporting value: " << data << endl;
    });

    thread worker([&] {
        unique_lock<std::mutex> lock(gLock);
        data = 69 + 96;
        notify = true;
        this_thread::sleep_for(std::chrono::seconds(5));
        con.notify_one();
    });

    reporter.join();
    worker.join();

    return 0;
}

```

## Asynchronous Operations
Asynchronous functions allow you to call a function and continue with the remaining code without waiting for the result. The <future> library is used to achieve this functionality.

Example with Future:
```cpp
#include <iostream>
#include <future>
#include <thread>
#include <chrono>

bool bufferedFileLoader() {
    size_t bytesLoaded = 0;
    while (bytesLoaded < 20000) {
        std::cout << "thread: loading file..." << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(250));
        bytesLoaded += 1000;
    }
    return true;
}

int main() {
    std::future<bool> backgroundThread = std::async(std::launch::async, bufferedFileLoader);

    std::future_status status;
    while (true) {
        std::cout << "Main thread is running" << std::endl;
        std::this_thread::sleep_for(std::chrono::milliseconds(50));
        status = backgroundThread.wait_for(std::chrono::milliseconds(1));

        if (status == std::future_status::ready) {
            std::cout << "Our data is ready..." << std::endl;
            break;
        }
    }

    std::cout << "Program is complete" << std::endl;
    return 0;
}

```


## Example: SFML Grid Update with Threads
Here’s an example that uses threads to update a grid and draw shapes in SFML.

```cpp
#include <iostream>
#include <thread>
#include <vector>
#include <mutex>
#include <algorithm> // for fill
#include <memory>
#include <chrono>  // For sleep

// Third party libraries
#include <SFML/Graphics.hpp>

// Globally available grid
static std::vector<int> grid;
// Global array of all of our objects
std::vector<std::unique_ptr<sf::Shape>> shapes;
// Keeps track of the program running
bool isRunning = true;

// Function to update grid
void update_grid(int x, int y) {
    while(isRunning) {
        std::this_thread::sleep_for(std::chrono::milliseconds(1000));
        grid[y*2+x] = rand() % 4;
    }
}

int main() {
    grid.reserve(4);
    std::fill(begin(grid), end(grid), 0);
    
    for(int x = 0; x < 2; x++) {
        for(int y = 0; y < 2; y++) {
            shapes.push_back(std::make_unique<sf::CircleShape>(100.0f));
        }
    }

    // Launch threads
    std::vector<std::thread> threads;
    for(int x = 0; x < 2; x++) {
        for(int y = 0; y < 2; y++) {
            threads.push_back(std::thread(update_grid, x, y));
        }
    }

    sf::RenderWindow window(sf::VideoMode(400, 400), "SFML with C++ threads");

    while (window.isOpen() && isRunning) {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed) {
                window.close();
                isRunning = false;
            }
        }

        window.clear();
        for (int x = 0; x < 2; x++) {
            for (int y = 0; y < 2; y++) {
                shapes[y*2+x]->setPosition(x*200, y*200);
                if (0 == grid[y*2+x]) {
                    shapes[y*2+x]->setFillColor(sf::Color::Red);
                } else if (1 == grid[y*2+x]) {
                    shapes[y*2+x]->setFillColor(sf::Color::Green);
                } else if (2 == grid[y*2+x]) {
                    shapes[y*2+x]->setFillColor(sf::Color::Blue);
                } else if (3 == grid[y*2+x]) {
                    shapes[y*2+x]->setFillColor(sf::Color::White);
                }
            }
        }

        for (auto& shape : shapes) {
            window.draw(*shape);
        }
        window.display();
    }

    for (auto& th : threads) {
        th.join();
    }

    std::cout << "Program Terminating" << std::endl;
    return 0;
}
```


## Semaphores
They are quite similar to mutex used as synchronization tool for multithreading to access the shared resources.
There are 2 types of semaphores,
1. Binary - Same as mutex with only 2 possible value 0,1
2. Coutning - allows multiple threads to access at same time, counter like behaviour.


Why Use Semaphores?
Semaphores are useful for scenarios where:
1. You need to synchronize access to shared resources (e.g., producer-consumer problems).
2. You want to limit the number of concurrent operations (e.g., controlling the number of threads accessing a resource).

Example code,

```cpp
#include <iostream>
#include <thread>
#include <semaphore>
#include <queue>
#include <mutex>
#include <vector>
#include <chrono>

std::queue<int> data_queue;           // Shared resource
std::binary_semaphore empty_slots(1); // Binary semaphore for producer
std::counting_semaphore<10> full_slots(0); // Counting semaphore for consumers
std::mutex queue_mutex;               // Mutex for consumer synchronization

void producer() {
    for (int i = 1; i <= 10; ++i) {
        empty_slots.acquire(); // Wait for an empty slot
        {
            std::lock_guard<std::mutex> lock(queue_mutex); // Protect access to data_queue
            data_queue.push(i); // Produce an item
            std::cout << "Produced: " << i << "\n";
        }
        full_slots.release(); // Signal that an item is available
        std::this_thread::sleep_for(std::chrono::milliseconds(100)); // Simulate production delay
    }
}

void consumer(int id) {
    while (true) {
        full_slots.acquire(); // Wait for an available item
        int item;
        {
            std::lock_guard<std::mutex> lock(queue_mutex); // Protect access to data_queue
            if (data_queue.empty()) break; // Exit if no more items (for safety)
            item = data_queue.front();
            data_queue.pop(); // Consume the item
        }
        std::cout << "Consumer " << id << " consumed: " << item << "\n";
        empty_slots.release(); // Signal that a slot is empty
        std::this_thread::sleep_for(std::chrono::milliseconds(150)); // Simulate processing delay
    }
}

int main() {
    std::thread producer_thread(producer);
    std::vector<std::thread> consumers;

    for (int i = 1; i <= 2; ++i) { // Two consumers
        consumers.emplace_back(consumer, i);
    }

    producer_thread.join();
    for (auto& c : consumers) c.join();

    return 0;
}
```

## Type of locks to handle concurrency

1. Mutual Exclusion
2. Recursive Lock (Reentrant Mutex)
3. Read/Write Lock
4. Spinlock
5. Timed Lock
6. Fair Lock
7. Bias Lock
8. Deadlock-free
9. Transactional Lock
10. Semaphores
11. Distributed Lock



## Locking and Unlocking

Atomic Operations - The operations which cant be divisible further.

Modern CPUs provide hardware support to make locking efficient and avoid race conditions:

Atomic Instructions:
Operations like compare-and-swap (CAS), test-and-set, or fetch-and-add are used to manipulate the lock variable atomically, ensuring no two threads can modify it simultaneously.

Memory Barriers (Fences):
Memory barriers ensure that read and write operations to shared variables are not reordered by the CPU or compiler, preserving consistency in multi-threaded environments.
Cache Coherency:

The lock variable is typically stored in CPU cache. The cache coherency protocol (e.g., MESI) ensures that changes to the lock variable are visible to all processors.

In summary, when a process acquires a lock on a critical section:
The lock variable is updated atomically.
The kernel might get involved if the thread must wait for the lock.
The CPU ensures atomicity and consistency using hardware-level mechanisms. 

## Atomic Operations

What Happens in Atomic Operations?
The CPU locks the memory location temporarily for the operation.

Examples of atomic instructions:

Compare-and-Swap (CAS) --> Read about implementation:
Compares the value of a memory location to an expected value and, if they match, swaps it with a new value.
Executed atomically in one CPU instruction cycle.

Test-and-Set  --> Read about implementation:
Reads a memory location and sets it to a specific value atomically, returning the old value.

Fetch-and-Increment  --> Read about implementation:
Atomically increments a memory location and returns the previous value.

Why It’s Unique:
During the execution of an atomic operation, the CPU prevents other threads or processes from accessing the same memory location, ensuring thread safety without additional synchronization.

## Memory Bus and Cache Lines in play

Memory Bus
The memory bus is a communication pathway that connects the CPU to the system memory (RAM). It plays a key role in transferring data between the CPU and memory during normal process execution.

Components of the Memory Bus
Address Bus: Carries the memory address of the data being accessed.
Data Bus: Transfers the actual data between the CPU and memory.
Control Bus: Manages signals for operations like read, write, and interrupts.
How the Memory Bus Comes into Play in Normal Execution
During normal execution, when a process needs to read or write data from/to memory:

CPU Generates an Address:
The CPU sends the memory address of the data to the address bus.
Control Signals:
The control bus indicates whether the operation is a read or write.
Data Transfer:
The data bus transfers the data between the CPU and memory.
Example:

If a CPU instruction requires reading a variable stored in memory:
The address bus carries the variable's memory location.
The control bus signals a read operation.
The data bus retrieves the variable's value from RAM to the CPU.
Memory Bus in Atomic Operations
During atomic operations, the memory bus may be temporarily locked to ensure exclusive access to a memory location. This prevents other CPUs or devices from interfering with the operation, ensuring atomicity.
2. Cache Line
A cache line is the smallest unit of data that can be transferred between the CPU cache and main memory. It is a contiguous block of memory, typically 32 to 128 bytes, used to optimize memory access.

How Cache Lines Work
Data Loading:

When the CPU accesses a memory address, the entire cache line containing that address is loaded into the CPU cache.
Example: If a cache line is 64 bytes and the CPU requests address 0x1000, memory addresses 0x1000 to 0x103F (64 bytes) are loaded into the cache.
Spatial Locality:

Since programs often access nearby memory addresses, caching entire lines improves performance by reducing memory access latency.
Cache Line in Atomic Operations
During atomic operations, the CPU ensures that no other core or thread can modify the same cache line.
Cache coherency protocols (e.g., MESI) manage synchronization:
If multiple cores attempt to access the same cache line, the protocol ensures only one has write access at a time.

## Deadlocks
Deadlocks are situtation when process1 has hold resource1 and requesting for resource2, but resource2 has been hold by process2 and process2 is requesting resource1.
Such conditions lead to deadlock as both process are requesting resources hold by each other respectively.


Below are some common methods to deal with deadlocks,
 Deadlock Avoidance
Goal: Prevent the system from entering an unsafe state where a deadlock could occur.
Approach: The system analyzes resource allocation requests in advance and ensures that granting a request does not lead to a deadlock.
Key Technique:
Use the Banker’s Algorithm:
Before allocating a resource, the system checks if fulfilling the request will still allow the system to remain in a safe state.
A safe state is one where all processes can complete without deadlock by potentially waiting.
Trade-offs:
Requires knowledge of future resource requests, which is not always feasible.
Increases system overhead due to frequent checks.
Example: A process cannot proceed if granting its resource request would leave the system in a state where another process cannot complete.



2. Deadlock Prevention
Goal: Eliminate one or more necessary conditions for deadlock.
Approach: Modify the system or resource allocation policies to prevent the four necessary conditions for deadlock:
Mutual Exclusion: Resources cannot be shared (e.g., printers).
Make resources sharable if possible.
Hold and Wait: A process holds resources while waiting for others.
Require processes to request all resources at once or release resources before requesting new ones.
No Preemption: Resources cannot be forcibly removed.
Allow preemption, i.e., take resources from a process and assign them to others.
Circular Wait: Processes are waiting in a circular chain.
Impose a global ordering on resource allocation to break the cycle.
Trade-offs:
May lead to reduced resource utilization.
Processes may face starvation due to overly restrictive policies.



3. Deadlock Detection and Recovery
Goal: Allow deadlocks to occur but detect and resolve them when they happen.
Approach:
The system periodically checks for deadlocks using algorithms like:
Wait-for Graphs: If a cycle is detected, a deadlock exists.
Once detected, take recovery measures such as:
Terminate Processes: Abort one or more processes to break the deadlock.
Preempt Resources: Take resources away from some processes to allow others to proceed.
Trade-offs:
Requires system overhead for periodic checks.
Termination or resource preemption can lead to data loss or inconsistency.
Example: Operating systems like UNIX may periodically run deadlock detection routines to identify and handle deadlocks.



4. Deadlock Ignorance
Goal: Ignore the problem of deadlocks altogether.
Approach: Assume deadlocks are rare and let them occur without taking specific measures to prevent or detect them.
Rationale:
In many systems (e.g., most general-purpose operating systems like Windows or Linux), deadlocks are considered infrequent and are left to the user or administrator to resolve (e.g., by restarting the system or killing processes).
Trade-offs:
Simplifies system design and reduces overhead.
May lead to system hangs or require manual intervention in case of a deadlock.
Example: Most desktop operating systems use this approach, relying on users to deal with deadlocks by forcefully terminating processes.


## Banker's Algorithm
The banker alogrithm is used to avoid/prevent/detect deadlock in system.

Youtbe - https://youtu.be/7gMLNiEz3nw?si=Ct2ghP2MfgIOQtgP

## Peterson Algorithm (Imp)
Algorithm to access the shared resource between 2 process without performing the atomic operations.
Peterson algorithm is humble algorithm which allows the another process first to enter the critical section, and waits itself.

Youtube - https://www.youtube.com/watch?v=gYCiTtgGR5Q&t=1145s


## What happens when we start a process
Summary of Process Execution Flow

1. Process Creation: The OS allocates virtual memory (user space) and kernel memory (for the PCB, page table, etc.).

2. CPU Execution in User Mode: The CPU runs instructions in the user space (e.g., code segment, heap, stack) and uses virtual memory translated by the MMU.

3. System Call / Interrupt: The process makes a system call, causing the CPU to switch to kernel mode and use the kernel stack and other kernel resources.

4. Memory Management: The MMU handles the translation of virtual addresses to physical addresses, possibly using the memory bus to fetch data from physical RAM or swap space.

5. Shared Resources and Locks: If the process needs access to shared resources, it acquires the necessary locks. If the resource is locked by another process, it waits or spins until it can acquire the lock.

6. Context Switching: If the process is preempted, the CPU saves the process state in the PCB, and another process may execute, with its own state loaded from its PCB.

In this way, the CPU, memory bus, kernel and user space, and locks work together to manage execution, memory access, and synchronization of processes in a multitasking system.



## Spurious Wakeups of Threads:

A spurious wakeup occurs when a thread, which is waiting on a condition or synchronization primitive (like a mutex or a condition variable), is awakened without any explicit signal or notification from other threads. Essentially, a thread can wake up even though the condition it is waiting for has not been met.
This can happen in multi-threaded environments for various reasons, often related to the underlying system's implementation of synchronization primitives.


 
## When to use CAS/Test-Set/Atmoic/Mutex/Semaphores

If you want to avoid writing low-level atomic methods manually (like Compare-and-Swap or Test-and-Set):

Use compiler intrinsics like __sync_bool_compare_and_swap or __sync_lock_test_and_set.
These intrinsics internally translate to atomic CPU instructions, providing a low-level yet easier interface.
If you want to avoid even using low-level atomic methods:

Use atomic variables from the <atomic> library in C++.
Atomic variables abstract the need for explicit CAS or Test-and-Set operations. They provide higher-level, thread-safe operations directly.
If you want to avoid atomic variables and require more generalized thread synchronization:

Use mutexes, semaphores, or other synchronization primitives.
These are better suited for complex scenarios, such as protecting access to larger critical sections or managing multiple resources.

When to Choose What:
Use CAS or Test-and-Set (manual or intrinsics):
When writing custom synchronization mechanisms or lock-free data structures.
Example: Implementing your own spinlock or lock-free queue.

Use Atomic Variables (std::atomic):
When you need lightweight, thread-safe operations on variables.
Example: Incrementing a shared counter or updating a flag in a multi-threaded program.

Use Mutexes/Semaphores:
When protecting access to complex critical sections or ensuring resource sharing between threads.
Example: Synchronizing access to a shared file or database.


