# kacchiOS Project Explanation

## 1. Introduction

Hello! This document explains **kacchiOS**, a minimal educational operating system designed to teach OS fundamentals. It covers the project structure, boot process, kernel logic, subsystem details, outputs, and verification methods. This guide is suitable for preparing a video or presentation.

---

## 2. Project Structure Overview

- **boot.S**: Bootloader and entry point.
- **kernel.c**: Main kernel logic and test harness.
- **serial.c/h**: Serial port driver for I/O.
- **string.c/h**: Basic string utilities.
- **types.h**: Type definitions.
- **io.h**: Low-level I/O port operations.
- **link.ld**: Linker script.
- **src/**: Core OS modules:
    - **memory.c/h**: Memory manager.
    - **process.c/h**: Process manager.
    - **scheduler.c/h**: Scheduler.
    - **context_switch.S/h**: Context switching (assembly + interface).

---

## 3. Boot Process (`boot.S`)

- System starts at `boot.S`, which sets up the stack and clears the BSS section.
- Calls the `kmain` function in `kernel.c`, the C entry point for the kernel.

---

## 4. Kernel Initialization (`kernel.c`)

- Initializes the serial port for output.
- Prints a boot message:
  ```
  [BOOT] Initializing kacchiOS...
  ```
- Initializes memory manager, process manager, and scheduler:
    - `memory_init()`
    - `process_init()`
    - `scheduler_init()`

---

## 5. Testing and Self-Verification

The kernel runs tests to verify each subsystem:

### a. Memory Manager Test

- `test_memory_manager()`:
    - Allocates heap and stack memory.
    - Frees some allocations.
    - Prints memory statistics.
- Output example:
  ```
  [TEST] Heap allocation...
  [OK] Multiple heap allocations
  [TEST] Stack allocation...
  [OK] Stack allocations
  [TEST] Memory deallocation...
  [OK] Deallocations completed
  ========== MEMORY STATISTICS ==========
  Total allocated: ...
  Total freed: ...
  ...
  ```

### b. Process Manager Test

- `test_process_manager()`:
    - Creates a test process.
    - Changes its state (BLOCKED, READY).
    - Uses process utilities to fetch process info.
    - Lists all processes.
- Output example:
  ```
  [TEST] Create test process...
  [OK] Process creation
  [TEST] State transitions...
  [OK] State change to BLOCKED
  [OK] State change to READY
  [OK] process_get() works
  [OK] Active processes: ...
  ========== PROCESS TABLE ==========
  PID ...: state=READY, priority=...
  ...
  ```

### c. Scheduler Test

- `test_scheduler()`:
    - Initializes the scheduler.
    - Sets the time quantum.
    - Selects the next process to run.
    - Applies the aging algorithm.
    - Prints scheduler statistics.
- Output example:
  ```
  [TEST] Initialize scheduler...
  [OK] Scheduler initialized
  [TEST] Set time quantum to 20ms...
  [OK] Quantum set correctly
  [TEST] Select next process...
  [OK] Selected process PID ...
  [TEST] Scheduler statistics...
  [OK] Scheduler test completed
  [TEST] Apply aging algorithm...
  ========== SCHEDULER STATISTICS ==========
  System ticks: ...
  Context switches: ...
  ...
  ```

### d. IPC Test

- `test_ipc()`:
    - Creates sender and receiver processes.
    - Simulates sending and receiving messages.
    - Verifies IPC functionality.
- Output example:
  ```
  [TEST] Create IPC processes...
  [OK] IPC processes created
  [TEST] IPC simulation...
  [OK] Message sent
  [OK] Message received
  ```

---

## 6. Main Kernel Loop

- After tests, the kernel prints a welcome banner and enters a command loop.
- User can type commands via the serial terminal:
    - `help` — lists available commands.
    - `memstat` — prints memory stats.
    - `proclist` — lists all processes.
    - `schedstat` — prints scheduler stats.
    - `test` — reruns all tests.
    - `exit` — halts the system.
- Example prompt:
  ```
  kacchiOS> 
  ```

---

## 7. Subsystems in Detail (`src/`)

### a. Memory Manager (`src/memory.c/h`)

- Manages a fixed-size heap and stack area.
- Provides `kmalloc`, `kfree` for heap, and `alloc_stack`, `free_stack` for stacks.
- Tracks allocations and prints statistics for debugging.

### b. Process Manager (`src/process.c/h`)

- Manages process control blocks (PCBs) in a table.
- Supports process creation, termination, state changes, and IPC.
- Each process has its own stack, priority, and message queue.

### c. Scheduler (`src/scheduler.c/h`)

- Implements a simple priority-based scheduler with aging.
- Selects the next READY process with the highest priority (lowest number).
- Handles context switching using assembly routines in `src/context_switch.S`.
- Tracks and prints statistics like context switches and system ticks.

### d. Context Switching (`src/context_switch.S/h`)

- Provides low-level routines to save and restore CPU state.
- Enables switching between process stacks during scheduling.

---

## 8. Outputs and Verification

- All outputs are sent to the serial port (COM1), visible in QEMU’s terminal.
- Each subsystem prints detailed logs during initialization, operation, and testing.
- The kernel’s built-in tests and command interface allow you to verify that:
    - Memory allocation/deallocation works.
    - Processes can be created, managed, and listed.
    - The scheduler selects and switches processes correctly.
    - IPC between processes is functional.

---

## 9. Conclusion

- kacchiOS is a modular, testable educational OS.
- Each subsystem is self-verifying via kernel tests and runtime commands.
- The project is easy to extend for further OS concepts, such as filesystems or device drivers.

---

## 10. Demo Flow (for Video)

1. **Boot the OS in QEMU**:  
   Show the boot messages and welcome banner.
2. **Demonstrate Commands**:  
   Type `help`, `memstat`, `proclist`, `schedstat`, and `test` to show live outputs.
3. **Explain Each Output**:  
   As each command/test runs, explain what it verifies and how it proves the subsystem is working.
4. **Show Source Code Highlights**:  
   Briefly show the relevant code in `src/` as you explain each subsystem.

---

**Thank you for reading! This concludes the kacchiOS project walkthrough.**
