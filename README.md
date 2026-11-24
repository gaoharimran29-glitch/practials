# Linux System Programming – Practical Questions & Answers (4–12)

> **Question 4:** Python (Linux/Unix only)  
> **Questions 5–12:** C programs  
> **Platform:** Linux / Ubuntu / WSL

---

## 4. Write a program (using `fork()` and/or `exec()` commands) where parent and child execute:

### (i) Same program, same code

**Program (Python):**

```python
import os

pid = os.fork()

if pid == 0:
    print("Child process: same code", flush=True)
    print("Child PID:", os.getpid(), flush=True)
else:
    print("Parent process: same code", flush=True)
    print("Parent PID:", os.getpid(), flush=True)
```

**Explanation:** `os.fork()` creates a child process. Both parent and child continue executing the same script after the fork. `pid == 0` identifies the child; a positive PID identifies the parent branch.

**Run:**

```bash
python3 same_code.py
```

### (ii) Same program, different code

**Program (Python):**

```python
import os

pid = os.fork()

if pid == 0:
    print("Child process: executing child-specific code", flush=True)
    print("Child PID:", os.getpid(), flush=True)
else:
    print("Parent process: executing parent-specific code", flush=True)
    print("Parent PID:", os.getpid(), flush=True)
```

**Explanation:** Both processes are created from the same script, but the child executes the `if` block while the parent executes the `else` block, so they run different statements.

**Run:**

```bash
python3 different_code.py
```

### (iii) Parent waits for child to finish before terminating

**Program (Python):**

```python
import os

pid = os.fork()

if pid == 0:
    print("Child process is working...", flush=True)
    print("Child PID:", os.getpid(), flush=True)
else:
    print("Parent is waiting for the child...", flush=True)
    os.wait()
    print("Child has finished. Parent terminates.", flush=True)
```

**Explanation:** `os.wait()` blocks the parent process until a child process terminates. The parent continues after the child finishes.

**Run:**

```bash
python3 wait_child.py
```

> **Note:** `os.fork()` is available on Linux and Unix-like systems (including WSL), but not native standard Windows Python.

---

## Practical Questions 5–12 (C)

> **Language:** C\
> **Platform:** Linux / Ubuntu / WSL\
> Compile programs using `gcc`. For the Pthreads program, add
> `-pthread`.

------------------------------------------------------------------------

## 5. Write a program to report behaviour of Linux kernel including kernel version, CPU type and CPU information.

### Program

``` c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/utsname.h>

int main(void)
{
    struct utsname info;

    if (uname(&info) == -1) {
        perror("uname");
        return 1;
    }

    printf("Linux System Information\\n");
    printf("----------------------\\n");
    printf("System Name  : %s\\n", info.sysname);
    printf("Node Name   : %s\\n", info.nodename);
    printf("Kernel      : %s\\n", info.release);
    printf("Version      : %s\\n", info.version);
    printf("Machine Type : %s\\n", info.machine);

    printf("\\nCPU Information (from /proc/cpuinfo)\\n");
    printf("------------------------------------\\n");

    FILE *fp = fopen("/proc/cpuinfo", "r");
    if (fp == NULL) {
        perror("fopen");
        return 1;
    }

    char line[256];
    int printed_model = 0, printed_cores = 0;

    while (fgets(line, sizeof(line), fp)) {
        if (!printed_model && (strstr(line, "model name") || strstr(line, "Hardware"))) {
            printf("%s", line);
            printed_model = 1;
        }
        if (!printed_cores && strstr(line, "cpu cores")) {
            printf("%s", line);
            printed_cores = 1;
        }
        if (strstr(line, "processor")) {
            printf("%s", line);
        }
    }

    fclose(fp);
    return 0;
}
```

### Explanation

-   `uname()` retrieves system and kernel details through the `utsname`
    structure.
-   `info.release` displays the kernel release/version string.
-   `info.machine` reports the system architecture (such as x86_64).
-   `/proc/cpuinfo` is a Linux virtual file containing CPU details. The
    program prints processor entries and some CPU model/core
    information.

### Compile and run

``` bash
gcc kernel_info.c -o kernel_info
./kernel_info
```

------------------------------------------------------------------------

## 6. Write a program to report behaviour of Linux kernel including information on configured memory, amount of free and used memory. (Memory information)

### Program

``` c
#include <stdio.h>
#include <stdlib.h>
#include <sys/sysinfo.h>

int main(void)
{
    struct sysinfo mem;

    if (sysinfo(&mem) == -1) {
        perror("sysinfo");
        return 1;
    }

    unsigned long unit = mem.mem_unit;
    unsigned long long total = (unsigned long long)mem.totalram * unit;
    unsigned long long free_mem = (unsigned long long)mem.freeram * unit;
    unsigned long long used = total - free_mem;

    printf("Memory Information\\n");
    printf("-----------------\\n");
    printf("Configured/Total RAM : %llu MB\\n", total / (1024ULL * 1024ULL));
    printf("Free RAM            : %llu MB\\n", free_mem / (1024ULL * 1024ULL));
    printf("Used RAM (approx.)   : %llu MB\\n", used / (1024ULL * 1024ULL));
    printf("Total Swap           : %llu MB\\n",
           ((unsigned long long)mem.totalswap * unit) / (1024ULL * 1024ULL));
    printf("Free Swap           : %llu MB\\n",
           ((unsigned long long)mem.freeswap * unit) / (1024ULL * 1024ULL));

    return 0;
}
```

### Explanation

-   `sysinfo()` obtains system memory and swap statistics.
-   `totalram` is the total RAM reported by the system.
-   `freeram` is currently free RAM.
-   Used RAM is calculated approximately as total RAM minus free RAM.
-   `mem_unit` is the multiplier used to convert the reported values
    into bytes.

### Compile and run

``` bash
gcc memory_info.c -o memory_info
./memory_info
```

------------------------------------------------------------------------

## 7. Write a program to copy files using system calls.

### Program

``` c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    if (argc != 3) {
        fprintf(stderr, "Usage: %s source_file destination_file\\n", argv[0]);
        return 1;
    }

    int source = open(argv[1], O_RDONLY);
    if (source == -1) {
        perror("open source");
        return 1;
    }

    int destination = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (destination == -1) {
        perror("open destination");
        close(source);
        return 1;
    }

    char buffer[4096];
    ssize_t bytes_read;

    while ((bytes_read = read(source, buffer, sizeof(buffer))) > 0) {
        ssize_t total_written = 0;

        while (total_written < bytes_read) {
            ssize_t bytes_written = write(destination,
                                         buffer + total_written,
                                         bytes_read - total_written);
            if (bytes_written == -1) {
                perror("write");
                close(source);
                close(destination);
                return 1;
            }
            total_written += bytes_written;
        }
    }

    if (bytes_read == -1) {
        perror("read");
    }

    close(source);
    close(destination);

    if (bytes_read == -1)
        return 1;

    printf("File copied successfully.\\n");
    return 0;
}
```

### Explanation

-   `open()` opens the source and creates/opens the destination.
-   `read()` reads bytes from the source file into a buffer.
-   `write()` writes those bytes to the destination.
-   `O_TRUNC` clears an existing destination file before copying.
-   `close()` releases both file descriptors.

### Compile and run

``` bash
gcc file_copy.c -o file_copy
./file_copy input.txt copy.txt
```

------------------------------------------------------------------------

## 8. Write a program to implement First Come First Serve (FCFS) scheduling algorithm.

### Program

``` c
#include <stdio.h>

int main(void)
{
    int n, bt[20], wt[20], tat[20];
    float avg_wt = 0, avg_tat = 0;

    printf("Enter number of processes: ");
    scanf("%d", &n);

    for (int i = 0; i < n; i++) {
        printf("Enter burst time for P%d: ", i + 1);
        scanf("%d", &bt[i]);
    }

    wt[0] = 0;

    for (int i = 1; i < n; i++)
        wt[i] = wt[i - 1] + bt[i - 1];

    for (int i = 0; i < n; i++) {
        tat[i] = wt[i] + bt[i];
        avg_wt += wt[i];
        avg_tat += tat[i];
    }

    printf("\\nProcess\\tBurst Time\\tWaiting Time\\tTurnaround Time\\n");
    for (int i = 0; i < n; i++)
        printf("P%d\\t%d\\t\\t%d\\t\\t%d\\n",
               i + 1, bt[i], wt[i], tat[i]);

    printf("\\nAverage Waiting Time: %.2f", avg_wt / n);
    printf("\\nAverage Turnaround Time: %.2f\\n", avg_tat / n);

    return 0;
}
```

### Explanation

-   FCFS executes processes in the order they arrive (here, input order
    is treated as arrival order).
-   The first process has waiting time 0.
-   Each next process waits for the total burst time of all previous
    processes.
-   **Turnaround time = Waiting time + Burst time.**

### Compile and run

``` bash
gcc fcfs.c -o fcfs
./fcfs
```

------------------------------------------------------------------------

## 9. Write a program to implement Shortest Job First (SJF) scheduling algorithm.

> This is **non-preemptive SJF**: among the processes waiting to run,
> the one with the smallest burst time is selected next. All processes
> are assumed to be available at time 0.

### Program

``` c
#include <stdio.h>

int main(void)
{
    int n, bt[20], p[20], wt[20], tat[20];
    float avg_wt = 0, avg_tat = 0;

    printf("Enter number of processes: ");
    scanf("%d", &n);

    for (int i = 0; i < n; i++) {
        p[i] = i + 1;
        printf("Enter burst time for P%d: ", i + 1);
        scanf("%d", &bt[i]);
    }

    /* Sort processes by burst time (ascending) */
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (bt[i] > bt[j]) {
                int temp = bt[i]; bt[i] = bt[j]; bt[j] = temp;
                temp = p[i]; p[i] = p[j]; p[j] = temp;
            }
        }
    }

    wt[0] = 0;
    for (int i = 1; i < n; i++)
        wt[i] = wt[i - 1] + bt[i - 1];

    for (int i = 0; i < n; i++) {
        tat[i] = wt[i] + bt[i];
        avg_wt += wt[i];
        avg_tat += tat[i];
    }

    printf("\\nProcess\\tBurst Time\\tWaiting Time\\tTurnaround Time\\n");
    for (int i = 0; i < n; i++)
        printf("P%d\\t%d\\t\\t%d\\t\\t%d\\n",
               p[i], bt[i], wt[i], tat[i]);

    printf("\\nAverage Waiting Time: %.2f", avg_wt / n);
    printf("\\nAverage Turnaround Time: %.2f\\n", avg_tat / n);

    return 0;
}
```

### Explanation

-   SJF selects the process with the shortest burst time first.
-   Processes are sorted by burst time.
-   Waiting time is calculated from the cumulative burst time of earlier
    scheduled processes.
-   **Turnaround time = Waiting time + Burst time.**

### Compile and run

``` bash
gcc sjf.c -o sjf
./sjf
```

------------------------------------------------------------------------

## 10. Write a program to implement non-preemptive priority-based scheduling algorithm.

> A smaller priority number is considered a higher priority (for
> example, priority 1 is highest). All processes are assumed to arrive
> at time 0.

### Program

``` c
#include <stdio.h>

int main(void)
{
    int n, bt[20], priority[20], p[20], wt[20], tat[20];
    float avg_wt = 0, avg_tat = 0;

    printf("Enter number of processes: ");
    scanf("%d", &n);

    for (int i = 0; i < n; i++) {
        p[i] = i + 1;
        printf("Enter burst time for P%d: ", i + 1);
        scanf("%d", &bt[i]);
        printf("Enter priority for P%d (smaller number = higher priority): ", i + 1);
        scanf("%d", &priority[i]);
    }

    /* Sort by priority in ascending order */
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (priority[i] > priority[j]) {
                int temp = priority[i]; priority[i] = priority[j]; priority[j] = temp;
                temp = bt[i]; bt[i] = bt[j]; bt[j] = temp;
                temp = p[i]; p[i] = p[j]; p[j] = temp;
            }
        }
    }

    wt[0] = 0;
    for (int i = 1; i < n; i++)
        wt[i] = wt[i - 1] + bt[i - 1];

    for (int i = 0; i < n; i++) {
        tat[i] = wt[i] + bt[i];
        avg_wt += wt[i];
        avg_tat += tat[i];
    }

    printf("\\nProcess\\tPriority\\tBurst Time\\tWaiting Time\\tTurnaround Time\\n");
    for (int i = 0; i < n; i++)
        printf("P%d\\t%d\\t%d\\t\\t%d\\t\\t%d\\n",
               p[i], priority[i], bt[i], wt[i], tat[i]);

    printf("\\nAverage Waiting Time: %.2f", avg_wt / n);
    printf("\\nAverage Turnaround Time: %.2f\\n", avg_tat / n);

    return 0;
}
```

### Explanation

-   Each process has a burst time and priority.
-   Processes are sorted by priority (ascending priority number).
-   The highest-priority process runs first and completes before the
    next process begins.
-   This is non-preemptive, so a running process is not interrupted.
-   **Turnaround time = Waiting time + Burst time.**

### Compile and run

``` bash
gcc priority.c -o priority
./priority
```

------------------------------------------------------------------------

## 11. Write a program to calculate sum of n numbers using Pthreads.

### Program

``` c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

struct thread_data {
    int n;
    long long sum;
};

void *calculate_sum(void *arg)
{
    struct thread_data *data = (struct thread_data *)arg;
    data->sum = 0;

    for (int i = 1; i <= data->n; i++)
        data->sum += i;

    return NULL;
}

int main(void)
{
    pthread_t thread;
    struct thread_data data;

    printf("Enter n: ");
    scanf("%d", &data.n);

    if (data.n < 0) {
        printf("Please enter a non-negative number.\\n");
        return 1;
    }

    if (pthread_create(&thread, NULL, calculate_sum, &data) != 0) {
        perror("pthread_create");
        return 1;
    }

    if (pthread_join(thread, NULL) != 0) {
        perror("pthread_join");
        return 1;
    }

    printf("Sum of numbers from 1 to %d = %lld\\n", data.n, data.sum);
    return 0;
}
```

### Explanation

-   `pthread_create()` creates a new thread to execute
    `calculate_sum()`.
-   The thread calculates the sum from 1 to `n`.
-   `pthread_join()` waits for the thread to finish.
-   The result is stored in the shared `data.sum` field.

### Compile and run

``` bash
gcc pthread_sum.c -o pthread_sum -pthread
./pthread_sum
```

------------------------------------------------------------------------

## 12. Write a program to implement first-fit, best-fit and worst-fit allocation strategies.

### Program

``` c
#include <stdio.h>

#define MAX 20

void first_fit(int blocks[], int nb, int processes[], int np)
{
    int allocation[MAX];
    for (int i = 0; i < np; i++)
        allocation[i] = -1;

    int b[MAX];
    for (int i = 0; i < nb; i++)
        b[i] = blocks[i];

    for (int i = 0; i < np; i++) {
        for (int j = 0; j < nb; j++) {
            if (b[j] >= processes[i]) {
                allocation[i] = j;
                b[j] -= processes[i];
                break;
            }
        }
    }

    printf("\\nFirst Fit Allocation:\\n");
    for (int i = 0; i < np; i++) {
        if (allocation[i] != -1)
            printf("Process %d (%d) -> Block %d\\n",
                   i + 1, processes[i], allocation[i] + 1);
        else
            printf("Process %d (%d) -> Not Allocated\\n", i + 1, processes[i]);
    }
}

void best_fit(int blocks[], int nb, int processes[], int np)
{
    int allocation[MAX];
    for (int i = 0; i < np; i++)
        allocation[i] = -1;

    int b[MAX];
    for (int i = 0; i < nb; i++)
        b[i] = blocks[i];

    for (int i = 0; i < np; i++) {
        int best = -1;
        for (int j = 0; j < nb; j++) {
            if (b[j] >= processes[i] &&
                (best == -1 || b[j] < b[best]))
                best = j;
        }

        if (best != -1) {
            allocation[i] = best;
            b[best] -= processes[i];
        }
    }

    printf("\\nBest Fit Allocation:\\n");
    for (int i = 0; i < np; i++) {
        if (allocation[i] != -1)
            printf("Process %d (%d) -> Block %d\\n",
                   i + 1, processes[i], allocation[i] + 1);
        else
            printf("Process %d (%d) -> Not Allocated\\n", i + 1, processes[i]);
    }
}

void worst_fit(int blocks[], int nb, int processes[], int np)
{
    int allocation[MAX];
    for (int i = 0; i < np; i++)
        allocation[i] = -1;

    int b[MAX];
    for (int i = 0; i < nb; i++)
        b[i] = blocks[i];

    for (int i = 0; i < np; i++) {
        int worst = -1;
        for (int j = 0; j < nb; j++) {
            if (b[j] >= processes[i] &&
                (worst == -1 || b[j] > b[worst]))
                worst = j;
        }

        if (worst != -1) {
            allocation[i] = worst;
            b[worst] -= processes[i];
        }
    }

    printf("\\nWorst Fit Allocation:\\n");
    for (int i = 0; i < np; i++) {
        if (allocation[i] != -1)
            printf("Process %d (%d) -> Block %d\\n",
                   i + 1, processes[i], allocation[i] + 1);
        else
            printf("Process %d (%d) -> Not Allocated\\n", i + 1, processes[i]);
    }
}

int main(void)
{
    int nb, np, blocks[MAX], processes[MAX];

    printf("Enter number of memory blocks: ");
    scanf("%d", &nb);
    if (nb <= 0 || nb > MAX) {
        printf("Invalid number of blocks.\\n");
        return 1;
    }

    for (int i = 0; i < nb; i++) {
        printf("Enter size of block %d: ", i + 1);
        scanf("%d", &blocks[i]);
    }

    printf("Enter number of processes: ");
    scanf("%d", &np);
    if (np <= 0 || np > MAX) {
        printf("Invalid number of processes.\\n");
        return 1;
    }

    for (int i = 0; i < np; i++) {
        printf("Enter memory required for process %d: ", i + 1);
        scanf("%d", &processes[i]);
    }

    first_fit(blocks, nb, processes, np);
    best_fit(blocks, nb, processes, np);
    worst_fit(blocks, nb, processes, np);

    return 0;
}
```

### Explanation

-   **First Fit:** Selects the first block large enough for the process.
-   **Best Fit:** Selects the smallest available block that can
    accommodate the process.
-   **Worst Fit:** Selects the largest available block that can
    accommodate the process.
-   The remaining size of a selected block is reduced after allocation.
-   Each strategy starts with a fresh copy of the original blocks, so
    the results can be compared independently.

### Compile and run

``` bash
gcc memory_allocation.c -o memory_allocation
./memory_allocation
```

------------------------------------------------------------------------

## Quick compilation reminder

For the regular C programs:

``` bash
gcc filename.c -o filename
./filename
```

For the Pthreads program:

``` bash
gcc pthread_sum.c -o pthread_sum -pthread
./pthread_sum
```

## Important viva points

-   `uname()` reports kernel/system information.
-   `/proc/cpuinfo` provides CPU information on Linux.
-   `sysinfo()` reports memory and swap statistics.
-   `open()`, `read()`, `write()`, and `close()` are file-related system
    calls.
-   FCFS scheduling follows arrival/input order.
-   SJF selects the process with the smallest burst time.
-   Priority scheduling selects the process with the highest priority
    (in this program, the smaller priority number).
-   Non-preemptive scheduling allows a process to finish once it starts.
-   Pthreads allows a program to create and synchronize threads.
-   First Fit, Best Fit and Worst Fit are contiguous-memory allocation
    strategies.
