# xv6 Completely Fair Scheduler (CFS) with htop

## 📌 Operating Systems Project

This project modifies the **xv6 operating system kernel** to replace the default Round-Robin scheduler with a **Completely Fair Scheduler (CFS)** and implements a **user-space htop-like system monitor** to visualize scheduling behavior in real time.

---

## 🎯 Objective

- Implement a **Completely Fair Scheduler (CFS)** in xv6 using virtual runtime (`vruntime`)
- Support **process priorities** through weights
- Prevent starvation using intelligent process initialization
- Build an **htop-like user utility** to monitor scheduling metrics in real time

---

## 🧠 Key Concepts

### Completely Fair Scheduler (CFS)
- Schedules the process with the **minimum virtual runtime**
- Uses **weights (priorities)** to proportionally allocate CPU time
- Higher weight → slower vruntime growth → more CPU time
- Ensures fairness while preventing starvation

### htop
- User-space system monitor
- Displays live scheduling and process metrics
- Helps verify correctness of CFS behavior

---

## 🛠️ Implementation Details

### 1️⃣ Process Control Block Modifications (`proc.h`)
Added the following fields to `struct proc`:
- `weight` – Process priority (default: 10)
- `vruntime` – Virtual runtime used by CFS
- `rtime` – Actual running time
- `stime` – Sleeping time
- `rnbletime` – Runnable (waiting) time

---

### 2️⃣ Scheduler Logic (`proc.c`)

#### 🔹 Process Initialization (`allocproc`)
- New processes inherit the **minimum vruntime** of existing runnable processes
- Prevents new processes from unfairly preempting older ones

#### 🔹 Scheduling Algorithm (`scheduler`)
- Scans process table to select the **RUNNABLE process with lowest vruntime**
- Updates:
  - `rtime`
  - `vruntime += (1024 * 10) / weight`

#### 🔹 Time Accounting (`update_proc_times`)
- Updates:
  - `stime` for SLEEPING processes
  - `rnbletime` for RUNNABLE processes
- Invoked on every timer interrupt

---

### 3️⃣ Kernel ↔ User Space Interface

#### `getprocs` System Call
- Copies process information from kernel to user space
- Enables user programs like `htop` to access scheduling data

#### `set_priority` System Call
- Allows a process to change its weight (range: 1–100)
- Triggers rescheduling immediately

---

### 4️⃣ Interrupt Handling (`trap.c`)
- Hooks into timer interrupt
- Ensures accurate accounting of runtime, sleep time, and wait time

---

## 👨‍💻 User-Space Utilities

### `htop.c`
- Displays:
  - PID, State, CPU %
  - Weight, Runtime, Sleep Time, Wait Time
  - Virtual Runtime
- Calculates CPU usage over a fixed sampling window

### `loadgen.c`
Creates three types of workloads:
1. CPU-bound
2. IO-bound
3. Mixed workload

Used to test scheduler fairness under load.

### `cputest_with_priority.c`
- Assigns custom weights to processes
- Demonstrates proportional CPU allocation

---

## ▶️ How to Run

### Compile and Start xv6
```bash
make clean
make qemu
