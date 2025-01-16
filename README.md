# Multiprocessing and Multiprogramming

[Watch the video on YouTube](https://www.youtube.com/watch?v=Fn0xBsmact4&list=PLvv0ScY6vfd_ocTP2ZLicgqKnvq50OCXM)

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
Lambda Functions:
Lambda functions can be declared in variables, inline functions, etc.

Example Code:
cpp
Copy code
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
Concurrency vs Parallelism in Threading
Creating threads does not mean that the code is executing in parallel; they must be managed to work concurrently.

Consider the difference between these two snippets:

First Snippet:

cpp
Copy code
for(int i=0; i < 10; i++) {
    threads.push_back(std::thread(lambda, i));
}

for(int i=0; i < 10; i++) {
    threads[i].join();
}
Second Snippet:

cpp
Copy code
for(int i=0; i < 10; i++) {
    threads.push_back(std::thread(lambda, i));
    threads[i].join();
}
Need for <jthread> Library
The <jthread> library in C++ simplifies thread management and solves issues associated with std::thread. It addresses automatic joining, cancellation, and thread resource management. Here's how jthread differs from std::thread:

Automatic joining of threads after execution
No need for manual joining or detaching threads
Easier handling of exceptions and cancellations
Need for <mutex> Library
The mutex library helps avoid data races and data inconsistency when it comes to shared resources between multiple threads. Here's an example:

Code with Potential Data Race:
cpp
Copy code
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
Without a lock, two threads may read the same value of critical_resource and both increment it, causing data inconsistency.

Fixed Code with Mutex:
cpp
Copy code
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
Atomic Variables
If you don't want to manage the mutex lock but still need to read/write shared data, you can use atomic variables.

Code with Atomic Variable:
cpp
Copy code
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
Conditional Variables
To save unnecessary computation when checking for mutex lock, we use conditional variables. If a thread is already holding a mutex, others wait for it to be released, and once released, it can notify other waiting threads to resume.

Example with Conditional Variable:
cpp
Copy code
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
Asynchronous Operations
Asynchronous functions allow you to call a function and continue with the remaining code without waiting for the result. The <future> library is used to achieve this functionality.

Example with Future:
cpp
Copy code
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
Example: SFML Grid Update with Threads
Here’s an example that uses threads to update a grid and draw shapes in SFML.

cpp
Copy code
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
