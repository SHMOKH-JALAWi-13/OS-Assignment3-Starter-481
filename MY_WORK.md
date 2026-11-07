# Assignment 3: Synchronization - Work Documentation

## 1. Development Log

* **Date: 2026-11-05** - Forked the starter repository (`OS-Assignment3-Starter-481`) and configured the project workspace structure on GitHub.
* **Date: 2026-11-06** - Injected the unique student ID (`445052013`) into `SchedulerSimulationSync.java` to properly seed the random number generator and match the expected verification outputs.
* **Date: 2026-11-06** - Implemented Feature 1 by adding a `ReentrantLock` (`queueLock`) inside the `ReadyQueue` class to protect critical sections (`add`, `poll`, `snapshot`) from race conditions.
* **Date: 2026-11-07** - Implemented Feature 2 by adding a `statsLock` in `SharedStats` to safely protect shared variables (`contextSwitchCount`, `completedProcessCount`, `totalServiceTime`) during multithreaded updates.
* **Date: 2026-11-07** - Implemented Feature 3 by integrating a counting `Semaphore` in `ReadyQueue` and updating `CpuWorker` to eliminate busy-waiting, concluding with clean thread termination logic.

---

## 2. Reflection

* **What was the most challenging part of this assignment?**
  The most challenging part was ensuring that multiple concurrent CPU worker threads could safely poll from and add processes back to the ready queue without missing updates or triggering race conditions, while also correctly managing the semaphore permits so no CPU thread remained blocked forever upon simulation completion.

* **How did you approach solving or debugging concurrency issues?**
  I approached debugging by analyzing the critical sections where shared states or collections were read and written concurrently. By systematically wrapping these blocks in `ReentrantLock` with `try/finally` structures, I ensured mutual exclusion and guaranteed lock release even if exceptions occurred.

* **What did you learn about thread synchronization and race conditions?**
  I learned firsthand how dangerous and intermittent race conditions can be when multiple threads access shared resources without synchronization. Using tools like `ReentrantLock` and semaphores highlighted the importance of enforcing strict control over critical sections to maintain data integrity.

* **How does replacing busy-waiting with a semaphore improve system efficiency?**
  Replacing busy-waiting with a counting semaphore allows CPU threads to sleep efficiently until work is actually available, rather than continuously polling an empty queue and wasting valuable CPU cycles and processor time.

---

## 3. Technical Answers

1. **Why is a `ReentrantLock` preferred over basic synchronized blocks in certain scenarios?**
   `ReentrantLock` provides advanced features such as fairness policies, the ability to attempt locking (`tryLock()`), and more flexible scoping for critical sections across different methods compared to standard `synchronized` blocks.

2. **Explain why a race condition occurs in `SharedStats` when `RaceWindow.pause()` is active.**
   A race condition occurs because updating a counter involves reading the current value, modifying it, and writing it back. If a thread is paused between the read and write operations, another thread can read the stale value, leading to a lost update when both write back their increments.

3. **How does the counting semaphore prevent busy-waiting in `CpuWorker`?**
   The semaphore maintains an internal counter of available processes. When the ready queue is empty, a CPU worker calls `acquire()`, which blocks and puts the thread to sleep instead of looping, and it wakes up only when `release()` is called upon process addition.

4. **Why is it necessary to release multiple semaphore permits when the last process completes?**
   When the simulation finishes, multiple CPU worker threads might still be blocked waiting on the semaphore (`acquire()`). Releasing permits equal to the total number of CPUs ensures all sleeping threads wake up, check the completion condition, and terminate cleanly.
