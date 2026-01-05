# Operating System Project - Video Script

## Introduction
Welcome! This video walks you through an educational operating system kernel. I'll focus on the main files and how they implement core OS concepts.

---

## src/process.c & process.h - Process Management

This file implements process control. It defines:

- **Process Control Block (PCB)**: A structure holding process information (PID, state, stack pointer, priority, message queue)
- **Process table**: An array of up to 16 processes

Key functions:
- `process_init()`: Initializes the process table
- `process_create(entry, priority)`: Creates a new process with an entry point and priority, allocates its stack, assigns a PID
- `process_exit()`: Terminates the current process

---

## src/memory.c & memory.h - Memory Management

This file implements dynamic memory allocation. It divides the kernel heap (64KB) into:

- **Heap region** (bottom-up): For general allocation via `kmalloc()`
- **Stack region** (top-down): For process stacks via `alloc_stack()`

Key functions:
- `memory_init()`: Resets heap and stack pointers
- `kmalloc(size)`: Allocates memory from the heap, tracks allocation metadata
- `kfree(ptr)`: Frees allocated memory
- `alloc_stack()` / `free_stack()`: Allocate and deallocate process stacks
- `memory_print_stats()`: Prints memory statistics (allocated, freed, failed allocations)

---

## src/scheduler.c & scheduler.h - CPU Scheduling

This file implements the process scheduler using priority-based scheduling with aging.

Configuration:
- `DEFAULT_TIME_QUANTUM` = 10ms (time slice per process)
- `AGING_THRESHOLD` = 50 ticks (when waiting processes get priority boost)

Key functions:
- `scheduler_init()`: Initializes scheduler state
- `scheduler_next()`: Selects the highest-priority READY process, implements aging to prevent starvation
- `scheduler_tick()`: Called regularly, decrements time quantum, performs context switch when expired
- `scheduler_context_switch()`: Saves current process state, loads next process state
- `scheduler_print_stats()`: Prints context switch count and tick count

---

## src/context_switch.S - Context Switching

This assembly file implements low-level process switching.

Key function:
- `context_switch_asm(current_sp, next_sp)`: Saves all CPU registers to current process's stack, swaps stack pointers, restores registers from next process's stack

This is pure assembly because it directly manipulates the CPU stack and registers.

---

## kernel.c - Main Kernel

This file ties everything together and runs test processes.

Key functions:
- `kmain()`: The kernel entry point
  - Initializes serial port for I/O
  - Initializes memory manager
  - Initializes process table
  - Initializes scheduler
  - Creates test processes
  - Starts the scheduler loop

- `worker_process_high()`: A high-priority test process
  - Prints 5 iterations with busy-wait (simulating work)
  - Calls `process_exit()` when done

- `worker_process_low()`: A low-priority test process
  - Prints 3 iterations
  - Demonstrates scheduling priority

- `test_simple_process()`: A simple test process for basic testing

The kernel creates these processes and the scheduler runs them concurrently, switching between them based on priority.

---

## Execution Flow

1. **Boot**: CPU loads boot.S, sets up stack, clears memory, jumps to `kmain()`
2. **Initialization**: All managers and schedulers are set up
3. **Process Creation**: Test processes are created and placed in READY state
4. **Scheduling Loop**: Scheduler runs continuously
   - Selects highest-priority READY process
   - Context switches to it
   - Process runs for its time quantum (10ms)
   - When time expires, scheduler picks next process
5. **Output**: Interleaved execution shows processes taking turns running

---

## Key Concepts Implemented

- **Process Management**: Create, track, and manage multiple processes
- **Memory Management**: Allocate memory for processes with separate stacks
- **CPU Scheduling**: Priority-based scheduling with aging to prevent starvation
- **Context Switching**: Save/restore CPU registers to switch between processes
- **Multitasking**: Multiple processes running concurrently on one CPU
- **Serial I/O**: Debug output and interaction via serial port

---

## End of Script
