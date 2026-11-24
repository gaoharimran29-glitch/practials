# Linux System Programming Practicals --- Python (Questions 4--12)

> **Language:** Python 3\
> **Platform:** Linux / Ubuntu / WSL\
> Save each program in a separate `.py` file and run with
> `python3 filename.py`.

------------------------------------------------------------------------

## 4. Write a program using `fork()` and/or `exec()` commands where parent and child execute:

### (i) Same program, same code

Both parent and child continue running the same code after `fork()`.
They have separate process IDs.

``` python
import os

pid = os.fork()

print("Hello from process!")
print("PID:", os.getpid())

if pid == 0:
    print("This is the child process.")
else:
    print("This is the parent process.")
```

**Explanation:** - `os.fork()` creates a child process. - After
`fork()`, both parent and child continue from the next line. -
`os.getpid()` returns the current process ID. - The `pid` return value
is `0` in the child and the child's PID in the parent.

**Run:**

``` bash
python3 same_program.py
```

### (ii) Same program, different code

The parent and child are created from the same program, but each runs a
different branch of code.

``` python
import os

pid = os.fork()

if pid == 0:
    print("Child process")
    print("Child is executing its own code.")
    print("Child PID:", os.getpid())
else:
    print("Parent process")
    print("Parent is executing its own code.")
    print("Parent PID:", os.getpid())
```

**Explanation:** - `fork()` creates a child process. - The child
executes the `if` branch (`pid == 0`). - The parent executes the `else`
branch. - Both processes originate from the same program but run
different statements.

**Run:**

``` bash
python3 different_code.py
```

### (iii) Parent waits for child to finish before terminating

``` python
import os

pid = os.fork()

if pid == 0:
    print("Child process is running.")
    print("Child PID:", os.getpid())
    print("Child task completed.")
    os._exit(0)
else:
    print("Parent is waiting for the child...")
    os.wait()
    print("Child has finished.")
    print("Parent process terminates.")
```

**Explanation:** - `os.fork()` creates the child process. - `os.wait()`
makes the parent wait until a child process exits. - After the child
terminates, the parent continues and prints the final messages. -
`os._exit(0)` exits the child process immediately without running
inherited exit handlers.

**Run:**

``` bash
python3 parent_wait.py
```

> **Note:** `fork()` is supported on Linux/Unix-like systems, not native
> standard Windows. Run this practical in Linux, Ubuntu, or WSL.

------------------------------------------------------------------------

## 5. Write a program to report Linux kernel behaviour/information including kernel version, CPU type and CPU information.

``` python
import platform

print("Linux System Information")
print("--------------------")
print("System:", platform.system())
print("Node:", platform.node())
print("Kernel Release:", platform.release())
print("Kernel Version:", platform.version())
print("Machine/CPU Architecture:", platform.machine())
print("Processor:", platform.processor() or "Not reported")

print("\nCPU Information")
print("-------------")

try:
    with open("/proc/cpuinfo", "r") as file:
        for line in file:
            if line.startswith(("model name", "Hardware", "processor", "cpu cores")):
                print(line.strip())
except OSError as e:
    print("Unable to read CPU information:", e)
```

**Explanation:** - The `platform` module reports operating system and
kernel information. - `platform.release()` gives the kernel release. -
`platform.machine()` reports the architecture (e.g., x86_64). -
`/proc/cpuinfo` contains detailed CPU information on Linux.

**Run:**

``` bash
python3 kernel_info.py
```

------------------------------------------------------------------------

## 6. Write a program to report Linux memory information including configured memory, free and used memory.

``` python
def read_meminfo():
    values = {}

    with open("/proc/meminfo", "r") as file:
        for line in file:
            key, value = line.split(":", 1)
            amount_kb = int(value.strip().split()[0])
            values[key] = amount_kb

    return values


memory = read_meminfo()

total = memory["MemTotal"]
available = memory.get("MemAvailable", memory.get("MemFree", 0))
used = total - available
free = memory.get("MemFree", 0)

print("Memory Information")
print("----------------")
print(f"Total RAM     : {total / 1024:.2f} MB")
print(f"Free RAM      : {free / 1024:.2f} MB")
print(f"Available RAM  : {available / 1024:.2f} MB")
print(f"Used RAM      : {used / 1024:.2f} MB")
```

**Explanation:** - `/proc/meminfo` contains memory statistics provided
by the Linux kernel. - `MemTotal` is total physical RAM. - `MemFree` is
RAM that is completely unused right now. - `MemAvailable` estimates
memory available for new applications without swapping. - Used memory is
estimated as `MemTotal - MemAvailable`.

**Run:**

``` bash
python3 memory_info.py
```

------------------------------------------------------------------------

## 7. Write a program to copy files using system calls.

``` python
import os
import sys

if len(sys.argv) != 3:
    print("Usage: python3 file_copy.py source destination")
    sys.exit(1)

source_path = sys.argv[1]
destination_path = sys.argv[2]

try:
    source_fd = os.open(source_path, os.O_RDONLY)
    try:
        destination_fd = os.open(
            destination_path,
            os.O_WRONLY | os.O_CREAT | os.O_TRUNC,
            0o644
        )
    except OSError:
        os.close(source_fd)
        raise

    try:
        while True:
            data = os.read(source_fd, 4096)
            if not data:
                break

            offset = 0
            while offset < len(data):
                written = os.write(destination_fd, data[offset:])
                if written == 0:
                    raise OSError("Write made no progress")
                offset += written

        print("File copied successfully.")

    finally:
        os.close(source_fd)
        os.close(destination_fd)

except OSError as error:
    print("File copy failed:", error)
    sys.exit(1)
```

**Explanation:** - `os.open()` opens the source and destination files. -
`os.read()` reads data from the source in chunks. - `os.write()` writes
the data to the destination. - `os.close()` closes the file
descriptors. - This demonstrates low-level file operations rather than
using `shutil.copy()`.

**Run:**

``` bash
python3 file_copy.py source.txt destination.txt
```

------------------------------------------------------------------------

## 8. Write a program to implement First Come First Serve (FCFS) scheduling algorithm.

> For simplicity, processes are assumed to arrive in the listed order at
> time 0.

``` python
n = int(input("Enter number of processes: "))
burst = []

for i in range(n):
    bt = int(input(f"Enter burst time for P{i + 1}: "))
    burst.append(bt)

waiting = [0] * n
turnaround = [0] * n

for i in range(1, n):
    waiting[i] = waiting[i - 1] + burst[i - 1]

for i in range(n):
    turnaround[i] = waiting[i] + burst[i]

print("\nProcess\tBurst\tWaiting\tTurnaround")
for i in range(n):
    print(f"P{i + 1}\t{burst[i]}\t{waiting[i]}\t{turnaround[i]}")

avg_waiting = sum(waiting) / n
avg_turnaround = sum(turnaround) / n

print(f"\nAverage Waiting Time: {avg_waiting:.2f}")
print(f"Average Turnaround Time: {avg_turnaround:.2f}")
```

**Explanation:** - FCFS stands for First Come First Serve. - Processes
execute in the order they are entered. - The first process has waiting
time 0. - Each later process waits for the sum of the previous
processes' burst times. - Turnaround Time = Waiting Time + Burst Time.

**Run:**

``` bash
python3 fcfs.py
```

------------------------------------------------------------------------

## 9. Write a program to implement Shortest Job First (SJF) scheduling algorithm.

> This program implements **non-preemptive SJF**. All processes are
> assumed to arrive at time 0.

``` python
n = int(input("Enter number of processes: "))
processes = []

for i in range(n):
    bt = int(input(f"Enter burst time for P{i + 1}: "))
    processes.append([i + 1, bt])

# Sort processes in ascending order of burst time
processes.sort(key=lambda p: p[1])

waiting = [0] * n
turnaround = [0] * n

for i in range(1, n):
    waiting[i] = waiting[i - 1] + processes[i - 1][1]

for i in range(n):
    turnaround[i] = waiting[i] + processes[i][1]

print("\nProcess\tBurst\tWaiting\tTurnaround")
for i in range(n):
    pid, bt = processes[i]
    print(f"P{pid}\t{bt}\t{waiting[i]}\t{turnaround[i]}")

print(f"\nAverage Waiting Time: {sum(waiting) / n:.2f}")
print(f"Average Turnaround Time: {sum(turnaround) / n:.2f}")
```

**Explanation:** - SJF selects the process with the shortest burst time
first. - The processes are sorted by burst time in ascending order. -
The process with the smallest burst time executes first. - Waiting time
is calculated using the burst times of preceding processes. - Turnaround
Time = Waiting Time + Burst Time.

**Run:**

``` bash
python3 sjf.py
```

------------------------------------------------------------------------

## 10. Write a program to implement non-preemptive priority-based scheduling algorithm.

> A smaller priority number means higher priority. All processes are
> assumed to arrive at time 0.

``` python
n = int(input("Enter number of processes: "))
processes = []

for i in range(n):
    bt = int(input(f"Enter burst time for P{i + 1}: "))
    priority = int(input(f"Enter priority for P{i + 1} (smaller number = higher priority): "))
    processes.append([i + 1, bt, priority])

# Sort by priority: lower number has higher priority
processes.sort(key=lambda p: p[2])

waiting = [0] * n
turnaround = [0] * n

for i in range(1, n):
    waiting[i] = waiting[i - 1] + processes[i - 1][1]

for i in range(n):
    turnaround[i] = waiting[i] + processes[i][1]

print("\nProcess\tPriority\tBurst\tWaiting\tTurnaround")
for i in range(n):
    pid, bt, priority = processes[i]
    print(f"P{pid}\t{priority}\t{bt}\t{waiting[i]}\t{turnaround[i]}")

print(f"\nAverage Waiting Time: {sum(waiting) / n:.2f}")
print(f"Average Turnaround Time: {sum(turnaround) / n:.2f}")
```

**Explanation:** - Every process has a burst time and priority number. -
The process with the highest priority runs first. - In this program, a
smaller priority number represents higher priority. - It is
non-preemptive: once a process begins, it runs until it finishes. -
Turnaround Time = Waiting Time + Burst Time.

**Run:**

``` bash
python3 priority.py
```

------------------------------------------------------------------------

## 11. Write a program to calculate sum of n numbers using Pthreads.

> In Python, POSIX threads are accessed through the `threading` module.

``` python
import threading

result = 0

def calculate_sum(n):
    global result
    result = sum(range(1, n + 1))

n = int(input("Enter n: "))

if n < 0:
    print("Enter a non-negative number.")
else:
    thread = threading.Thread(target=calculate_sum, args=(n,))
    thread.start()
    thread.join()

    print(f"Sum of numbers from 1 to {n} = {result}")
```

**Explanation:** - Python's `threading` module is used to create a
thread. - `calculate_sum()` computes the sum from 1 to `n`. -
`thread.start()` starts the thread's work. - `thread.join()` makes the
main program wait until the thread completes. - The result is then
printed.

**Run:**

``` bash
python3 thread_sum.py
```

------------------------------------------------------------------------

## 12. Write a program to implement First Fit, Best Fit and Worst Fit allocation strategies.

``` python
def allocate_memory(blocks, processes, strategy):
    remaining = blocks.copy()
    allocation = [-1] * len(processes)

    for i, size in enumerate(processes):
        candidates = [
            j for j, block_size in enumerate(remaining)
            if block_size >= size
        ]

        if not candidates:
            continue

        if strategy == "first":
            chosen = candidates[0]
        elif strategy == "best":
            chosen = min(candidates, key=lambda j: remaining[j])
        elif strategy == "worst":
            chosen = max(candidates, key=lambda j: remaining[j])
        else:
            raise ValueError("Unknown allocation strategy")

        allocation[i] = chosen
        remaining[chosen] -= size

    return allocation


def display(strategy_name, allocation, processes):
    print(f"\n{strategy_name}")
    print("Process\tSize\tBlock")
    for i, block in enumerate(allocation):
        if block == -1:
            print(f"P{i + 1}\t{processes[i]}\tNot Allocated")
        else:
            print(f"P{i + 1}\t{processes[i]}\tB{block + 1}")


blocks = list(map(int, input("Enter memory block sizes separated by spaces: ").split()))
processes = list(map(int, input("Enter process sizes separated by spaces: ").split()))

first = allocate_memory(blocks, processes, "first")
best = allocate_memory(blocks, processes, "best")
worst = allocate_memory(blocks, processes, "worst")

display("First Fit Allocation", first, processes)
display("Best Fit Allocation", best, processes)
display("Worst Fit Allocation", worst, processes)
```

**Explanation:** - **First Fit:** Allocates the first memory block that
is large enough. - **Best Fit:** Allocates the smallest block that is
large enough. - **Worst Fit:** Allocates the largest block that is large
enough. - After allocation, the remaining space in the selected block is
reduced by the process size. - Each strategy gets a fresh copy of the
original block list for a fair comparison.

**Run:**

``` bash
python3 memory_allocation.py
```

------------------------------------------------------------------------

## Quick run commands

For each program, save it with the filename shown in the Run section and
execute:

``` bash
python3 filename.py
```

### Practical/Viva quick revision

-   `os.fork()` creates a new child process on Linux/Unix.
-   `os.wait()` makes the parent wait for a child process to finish.
-   `platform` and `/proc/cpuinfo` help report Linux/kernel and CPU
    information.
-   `/proc/meminfo` contains memory statistics.
-   `os.open()`, `os.read()`, `os.write()`, `os.close()` are low-level
    file operations.
-   FCFS schedules processes in arrival/input order.
-   SJF selects the process with the shortest burst time.
-   Priority scheduling runs the highest-priority process first (here,
    smaller number means higher priority).
-   Python `threading.Thread` creates and runs a thread; `join()` waits
    for it.
-   First Fit, Best Fit, and Worst Fit are memory allocation strategies.
