# Concurrency
Multihreading/Multiprocessing in cpp


The use of multithreading library in cpp.
What are libraries, modules and code used to understand this concept.

1. <thread>
2. <jthread>
3. <future>

# Difference between concurrency and parallelism:

Concurrency
Definition: Concurrency refers to the ability of a system to manage multiple tasks (processes or threads) at the same time, giving the illusion of simultaneous execution. However, these tasks might not actually be executing at the same moment. Instead, the CPU switches between tasks rapidly (context switching).
Analogy: Think of a single chef preparing multiple dishes by switching between tasks like chopping vegetables, boiling water, and stirring a pot.
Example:
If two threads share a single-core CPU, only one thread executes at any moment, but the OS schedules them in a way that they appear to run concurrently.

Parallelism
Definition: Parallelism involves executing multiple tasks simultaneously on multiple processors or cores. Each task runs on its own processor without interference from others.
Analogy: Think of multiple chefs, each preparing their own dish simultaneously in a kitchen.
Example:
On a multi-core CPU, two threads can run on separate cores at the same time, genuinely executing in parallel.

Concurrency gives the user the perception that multiple programs are running simultaneously, but in reality, the CPU switches between tasks. Threads within a process often execute this way on a single-core CPU.
Parallelism, on the other hand, involves multiple processors executing multiple tasks simultaneously, as seen in a multi-core CPU system.
For example, in concurrency, at any moment, the instruction pointer of a thread might be at a different address because threads share resources. In parallelism, multiple threads or processes can execute genuinely at the same time, often working on different parts of a task across different processors.


Mutltithreading is not just about spawning new threads, but to manage them concurrently.