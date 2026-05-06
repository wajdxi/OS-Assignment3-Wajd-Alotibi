# Assignment 3 - Complete Documentation

**Student Name**: [Wajd Moiedh Alotibi]  
**Student ID**: [445052101]  
**Date Submitted**: [6/5/2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [6/5/2026, 6 am]
**What I implemented**:  Set up the repository, changed student ID, made first commit.

**Challenges encountered**: None

**How I solved it**: -

**Testing approach**:  Compiled and ran the original unsynchronized code to see the race con
ditions (inconsistent log counts)

**Time spent**: 15 min

---

### Entry 2 - [6/5/2026, 6:16 am]
**What I implemented**:  Task 1 – fine-grained ReentrantLocks for the three counters

**Challenges encountered**:  Understanding why fine-grained locking is better than a single 
lock.

**How I solved it**:  Read about lock granularity; decided to use three separate locks becau
se the counters are independent.

**Testing approach**:  Ran the program 10 times; counters now give the same values each run.

**Time spent**: 1 hr

---

### Entry 3 - [6/5/2026, 7:20 am]
**What I implemented**: Task 2 – ReentrantLock for the execution log (ArrayList).

**Challenges encountered**:  Initially forgot to unlock in finally block.

**How I solved it**:  Corrected to lock/unlock inside try-finally.

**Testing approach**:  Added heavy logging; never saw ConcurrentModificationException.

**Time spent**: 40 min

---

### Entry 4 - [6/5/2026, 8 am]
**What I implemented**:  Task 3 – Semaphore for CPU control (binary semaphore with 1 permi
t).

**Challenges encountered**:  Ensuring semaphore is also added to runToCompletion() method.

**How I solved it**:  Wrapped both run() and runToCompletion() with acquire()/release() in finally.

**Testing approach**:  Set Semaphore(2) temporarily to see concurrency effects; then reverte
d to 1.

**Time spent**: 1 hr

---

### Entry 5 - [6/5/2026, 9 am]
**What I implemented**:  Task 4 – Completed ASSIGNMENT_DOCUMENTATION.md, recorded video, mad
e final commits.

**Challenges encountered**:  Explaining lock granularity clearly.

**How I solved it**:  Drew a small diagram and wrote the explanation in Q4.

**Testing approach**:  Ran final code 5 times, all statistics identical.

**Time spent**: 2 hr

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[ **First race condition** –`contextSwitchCount++` (and the other counters).  
Shared resource: the integer counters.  
Problem: `++` is not atomic; two threads can read the same value, increment, and write 
back, causing a lost update.  
Incorrect behaviour: The final counter value is less than the actual number of incremen
ts.
 **Second race condition** – `executionLog.add(message)`.  
Shared resource: `ArrayList<String>`.  
Problem: `ArrayList` is not thread-safe; concurrent `add()` calls can corrupt internal 
structure, throw `ConcurrentModificationException`, or lose entries.  
Incorrect behaviour: Program may crash or log entries may disappear
]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[- **ReentrantLock** is a mutual exclusion lock (binary). It guarantees that only one thre
ad holds the lock at a time. I used it for the counters and the log because those resourc
es require exclusive access.- **Semaphore** maintains a set of permits. A binary semaphore (permits = 1) acts like a 
lock, but semaphores can also allow N concurrent accesses (e.g., a connection pool). I us
ed a `Semaphore(1)` to limit CPU execution – only one process can run at any moment, exac
tly matching a single-core CPU.]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[- **Deadlock** occurs when two or more threads wait forever for each other’s locked resou
rces.- Prevention techniques I used:
1. **Lock ordering** – I never acquire more than one lock at a time, so cyclic wait can
not happen.
2. **try-finally blocks** – Every `lock()` or `acquire()` is followed by a `finally` bl
ock that releases the resource. This guarantees release even if an exception occurs, prev
enting resource leaks.- Additionally, the semaphore is acquired at the very beginning of the critical section a
nd released immediately after, so there is no nested locking.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[- I chose **fine-grained locking** – three separate `ReentrantLock`s, one per counter (`c
ontextSwitchLock`, `completedProcessLock`, `waitingTimeLock`).- **Why:** The three counters are completely independent (updating one does not depend on 
the others). With a single coarse-grained lock, threads updating different counters would 
still block each other, creating unnecessary contention. Fine-grained locking allows true 
parallelism: while one thread increments `contextSwitchCount`, another can simultaneously 
increment `completedProcessCount`.- **Trade-offs:** Fine-grained requires more code and careful reasoning, but for independ
ent resources the concurrency gain is worth it. Coarse-grained is simpler but reduces thr
oughput.- Because the counters are independent, fine-grained locking provides **better concurrenc
y** – it exactly follows the principle: protect each shared resource with its own lock.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**:  `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`

**Why they need protection**:  The read-modify-write operations (increment, addition) are not atomic; 
without locks, updates can be lost.

**Synchronization mechanism used**: Three separate `ReentrantLock`s (fine-grained)

**Code snippet**:
```java
public static void incrementContextSwitch() {
contextSwitchLock.lock();
try { contextSwitchCount++; } finally { contextSwitchLock.unlock(); }
}```

**Justification**: Each counter is independent, so separate locks maximise concurrency.

---

### Critical Section #2: Execution Log

**What resource**: List<String> executionLog

**Why it needs protection**: ArrayList is not thread‑safe; concurrent add() calls cause
corruption or exceptions.


**Synchronization mechanism used**: ReentrantLock logLock

**Code snippet**:
```java
public static void logExecution(String message) {
logLock.lock();
try { executionLog.add(message); } finally { logLock.unlock(); }
}```

**Justification**:  Exclusive access is required to preserve the logʼs integrity.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**:  Simulate a single‑core CPU – only one process can execute at a time.

**Number of permits and why**: 1 (binary semaphore)

**Where implemented**: Process.run() and Process.runToCompletion()

**Code snippet**:
```java
SharedResources.cpuSemaphore.acquire();
try {
// ... execution code ...
} finally {
SharedResources.cpuSemaphore.release();
}```

**Effect on program behavior**:  Guarantees that even though many threads are ready, only one proceeds into
the CPU at any moment – exactly like a real uniprocessor system.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: Ran java SchedulerSimulationSync five times.
```bash

**Results**: 
( Every run produced the exact same numbers:
Context switches: always 29
Completed processes: always 16 
Total waiting time: always 864052ms
Average waiting time: 54003ms)

**Why synchronization is necessary**: 
( Without locks, the counters could lose increments
and the log could throw exceptions. The fact that results are now deterministic proves
race conditions are eliminated)

**Conclusion**: 
consistent results
---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: : Added a loop that ran the program 20 times.

**Results**: No ConcurrentModificationException or any other exception occurred.

**What this proves**:  The logLock successfully serialises access to the ArrayList ,
making it thread‑safe.

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**:  completedProcessCount should equal number of processes created; total
waiting time should be consistent with per‑process waiting times.

**Actual values**:  All values matched the manual calculation (checked with print statements).

**Analysis**: The synchronisation does not change the logical behaviour – it only ensures
correctness under concurrency.
---
### Test 4: Different Scenarios
**Scenario tested**: [ Changed Semaphore(1) to Semaphore(2) temporarily.]

**Purpose**:  Observe effect of allowing two concurrent processes.

**Results**:  With 2 permits, execution overlapped (interleaving in output). Still no race
conditions because counters remained protected.

**What I learned**: Semaphores are extremely flexible – they control the degree of
concurrency without changing the core logic. 

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[-Race conditions are subtle: the code can run correctly many times and then suddenly
fail. Synchronisation makes concurrent programs predictable.

-Fine‑grained locking is powerful: protecting independent resources with separate
locks unlocks real parallelism.

-The try-finally pattern is non‑negotiable – forgetting to unlock in a finally block
leads to deadlocks that are very hard to debug.

-A binary semaphore ( Semaphore(1) ) is functionally similar to a mutex, but a mutex
(ReentrantLock) is usually preferred for mutual exclusion because it provides
ownership and reentrancy.

-Synchronisation adds overhead, but the safety it buys is essential for any
multithreaded program.]

---

### Real-world applications:  

Give TWO examples where synchronization is critical:

**Example 1**:  Banking systems – When multiple tellers update the same account balance, locks
prevent lost deposits or withdrawals.

**Example 2**:  Print spooler – A semaphore with a limit equal to the number of printers controls
access to physical printers.

---

### How I would explain synchronization to others:

[Imagine a shared whiteboard where many students want to write. If two write at the
same time, their notes become unreadable. A mutex lock is like giving the marker to
only one student at a time. A semaphore is like having a few markers – it lets a limited
number of students write together. Without these rules, the whiteboard would be a
mess. Thatʼs exactly what synchronisation does for shared data in a program.]

---

## Part 6: GitHub Repository Information

**Repository URL**: 

**Number of commits**: 

**Commit messages**: 
1. Set my student ID: 445052101
2. Task 1 : Added fine-grained ReentrantLock for counters
3. Task 2 : Added ReentrantLock for execution log
4. Task 3 : Implemented Semaphore for CPU control
5. Task 4 : Completed comprehensive documentation
6. Final cleanup and verification of synchronisa
---

## Summary

**Total time spent on assignment**: 

**Key takeaways**: 
1. 
2. 
3. 

**Most challenging aspect**: 

**What I'm most proud of**: 

---

**End of Documentation**
