# Operating System Project - Detailed Video Script

---

## SECTION 1: INTRODUCTION

Hello and welcome! In this video, I'll walk you through a complete educational operating system project from the ground up. This project demonstrates core OS concepts including bootloading, memory management, process scheduling, context switching, and inter-process communication.

What you're about to see is a real, functional OS kernel written in C and assembly. We'll explore how hardware is initialized, how processes are managed, and how the CPU switches between them.

Let's start with the project structure and then dive into each component.

---

## SECTION 2: PROJECT OVERVIEW

### File Structure

Our OS is organized into several key areas:

**Boot and Build Files:**
- `boot.S` - The assembly bootloader
- `link.ld` - The linker script
- `Makefile` - Build automation

**Utility Libraries:**
- `io.h` - Low-level I/O operations
- `types.h` - Standard data type definitions
- `serial.h` / `serial.c` - Serial port driver
- `string.h` / `string.c` - String utilities

**Kernel Core:**
- `kernel.c` - Main kernel logic and test processes

**Core Modules (in src/):**
- `process.h` / `process.c` - Process management
- `scheduler.h` / `scheduler.c` - CPU scheduler
- `memory.h` / `memory.c` - Memory management
- `context_switch.h` / `context_switch.S` - Context switching assembly

---

## SECTION 3: BOOTLOADING & INITIALIZATION

### boot.S - The Bootloader

Let me show you the bootloader. This is where everything starts.

The bootloader does three critical things:

**First**, it defines the Multiboot header. This is a magic marker that tells the bootloader (like GRUB) that this is a valid kernel image. It contains:
- Magic number: `0x1BADB002` - identifies a Multiboot kernel
- Flags and checksum for validation

**Second**, it allocates a kernel stack. We reserve 16KB of memory in the `.bss` section for the stack. The CPU uses this to store local variables and function returns.

**Third**, it performs the initial setup:
- `cli` - Disables interrupts so nothing interrupts initialization
- Sets the stack pointer to the top of our allocated stack
- Clears the BSS section (uninitialized data area)
- Calls `kmain()` - the C kernel entry point

After that, we halt the CPU. If the kernel ever returns (which it shouldn't), we disable interrupts and put the CPU in an infinite halt loop.

This simple assembly code is critical - it's the bridge between the bootloader and the C kernel.

---

### link.ld - The Linker Script

Now let's look at the linker script. This file tells the linker how to organize our code in memory.

Key aspects:

- **Output format**: `elf32-i386` - We're building a 32-bit x86 executable
- **Entry point**: The CPU starts executing at the `start` label in boot.S
- **Memory layout**:
  - Code starts at 1 megabyte (standard for x86 kernels)
  - `.text` section holds code and read-only data
  - `.data` section holds initialized global variables
  - `.bss` section holds uninitialized data
  - `__kernel_end` marks where the kernel memory ends

This layout is important because it determines what's in memory and where.

---

### Makefile - Building the OS

The Makefile automates compilation. It:
- Compiles each `.c` file to `.o` object files
- Assembles `.S` files
- Links everything together using `link.ld`
- Produces the final kernel image

This makes building as simple as `make`.

---

## SECTION 4: UTILITIES & DRIVERS

### types.h - Type Definitions

Let me show the types header. It defines standard integer types:
- `uint32_t`, `uint16_t`, `uint8_t` - unsigned integers
- `int32_t`, `int16_t`, `int8_t` - signed integers
- `size_t` - for sizes

And `NULL` for null pointers. These ensure consistency across different platforms and compilers.

---

### io.h - Low-Level Hardware I/O

This header provides two critical inline functions:

**`outb(port, value)`** - Sends a byte to a hardware port. This uses inline assembly to execute the `outb` instruction, which writes a value to an I/O port. We use this to configure hardware like the serial port.

**`inb(port)`** - Reads a byte from a hardware port. This reads data from an I/O port and returns it. We use this to check serial port status.

These are fundamental - they're how the CPU communicates with hardware.

---

### serial.h & serial.c - Serial Communication Driver

The serial driver is how we interact with the outside world. Let me explain how it works:

**Serial Port Basics:**
The COM1 serial port is at I/O address `0x3F8`. It has several control registers:
- Port+0: Data register (send/receive characters)
- Port+1: Interrupt enable
- Port+3: Line control (baud rate, data bits, etc.)
- Port+4: Modem control
- Port+5: Line status (ready to send? data available?)

**Initialization (`serial_init`):**
- Disable interrupts temporarily
- Enable DLAB (Divisor Latch Access Bit) to set baud rate
- Set baud rate divisor for 38400 baud
- Configure 8-bit data, no parity, 1 stop bit
- Enable FIFO and set it up

**Transmitting (`serial_putc`):**
- Check if the transmit buffer is empty by reading port+5
- If empty, write the character to port+0
- Special handling for newlines - we send both carriage return and line feed

**Receiving (`serial_getc`):**
- Poll the receive status
- When data is ready, read from port+0
- Return the character

This driver lets us print debug messages and receive input!

---

### string.h & string.c - String Utilities

These are standard C string functions:

**`strlen(str)`** - Counts characters until the null terminator. Used to find string length.

**`strcmp(str1, str2)`** - Compares two strings byte by byte. Returns 0 if equal, negative if str1 < str2, positive if str1 > str2. Essential for string comparison.

**`strcpy(dest, src)`** - Copies characters from source to destination until null terminator. Used to copy strings.

These are the fundamental string operations we need in the kernel.

---

## SECTION 5: PROCESS MANAGEMENT

### process.h & process.c - Process Control

This is where we define what a process is and how to manage it.

**Process State Enum:**
- `PROC_UNUSED` - Slot is available
- `PROC_READY` - Process is ready to run
- `PROC_RUNNING` - Process is currently executing
- `PROC_BLOCKED` - Waiting for I/O or event
- `PROC_SLEEPING` - Waiting for a time delay
- `PROC_TERMINATED` - Process finished

**Process Control Block (PCB):**
Each process has a PCB structure containing:
- `pid` - Process ID (unique identifier)
- `state` - Current process state
- `stack_base` - Bottom of process stack
- `stack_ptr` - Current top of process stack (for context switching)
- `priority` - Priority level (lower is higher priority)
- `age` - Used for aging to prevent starvation
- `msg_queue` - Messages from other processes (IPC)
- `msg_count` - Number of pending messages

**Process Table:**
We maintain an array of up to 16 PCBs, allowing 16 concurrent processes.

**Key Functions:**

`process_init()` - Initializes the process table, marking all slots as unused.

`process_create(entry, priority)` - Creates a new process:
- Finds a free slot in the process table
- Allocates a stack for the new process
- Initializes the stack with the entry point address
- Sets the process to READY state
- Assigns a unique PID

`process_exit()` - Terminates the current process:
- Marks it as TERMINATED
- Decrements process count
- Returns to scheduler

The initialization of the stack is clever: it pushes fake register values and the entry point address onto the stack. When context switching restores these values, the process starts execution at the entry point.

---

## SECTION 6: MEMORY MANAGEMENT

### memory.h & memory.c - Heap & Stack Allocation

Memory management is crucial. We divide the kernel heap into two regions:

**Heap Region (bottom-up):**
- Starts at offset 0
- Grows upward as we allocate
- Used for general allocation via `kmalloc()`
- Maximum 64KB

**Stack Region (top-down):**
- Starts at the top of the heap
- Grows downward
- Used by `alloc_stack()` for process stacks
- Prevents heap/stack collision

**Metadata Management:**
Each allocation has metadata:
- `size` - Size of the block
- `is_allocated` - Whether it's in use
- `is_stack` - Whether it's a stack or heap block

We maintain an array of metadata for up to 64 allocations.

**Core Functions:**

`memory_init()` - Resets the heap and stack pointers, clears metadata.

`kmalloc(size)` - Allocates memory:
- Finds a free or previously-freed block that fits
- Marks it as allocated
- Tracks statistics
- Returns a pointer to usable memory

`kfree(ptr)` - Frees allocated memory:
- Finds the metadata for this allocation
- Marks it as freed (not immediately reused, but available)
- Updates statistics

`alloc_stack()` / `free_stack()` - Allocate/deallocate process stacks from the top-down stack region.

**Memory Statistics:**
The memory manager tracks:
- Total bytes allocated
- Total bytes freed
- Number of heap allocations
- Number of stack allocations
- Failed allocations

These statistics are printed for debugging.

---

## SECTION 7: THE SCHEDULER

### scheduler.h & scheduler.c - CPU Scheduling

The scheduler is the heart of multitasking. It decides which process runs next.

**Scheduler Configuration:**
- `DEFAULT_TIME_QUANTUM` = 10 milliseconds - How long each process runs before switching
- `AGING_THRESHOLD` = 50 ticks - After this many ticks waiting, a process ages up (priority improves)
- `MAX_PRIORITY` = 20 - Lower numbers are higher priority

**Scheduler State:**
The scheduler maintains:
- `current_quantum` - Time remaining in current process's time slice
- `time_quantum` - Default quantum
- `ticks` - Number of scheduler ticks elapsed
- `context_switches` - Total number of context switches performed

**Scheduling Algorithm - Priority with Aging:**

`scheduler_next()` - Selects the next process to run:
- Scans all PCBs
- Finds all READY processes
- Selects the one with lowest priority number (highest priority)
- Implements aging: processes waiting too long get priority boosted
- Returns the selected PCB

This prevents low-priority processes from starving forever.

`scheduler_tick()` - Called regularly (e.g., every 10ms):
- Decrements the current quantum
- If quantum expires, performs context switch
- Increments the global tick counter
- Applies aging to waiting processes

`scheduler_context_switch()` - Switches to the next process:
- Saves current process state
- Selects next process via `scheduler_next()`
- Loads next process's saved state
- Increments context switch counter

The scheduler ensures fair CPU time while respecting priorities.

---

## SECTION 8: CONTEXT SWITCHING

### context_switch.S - Assembly Context Switch

This is critical low-level code. Context switching is how we swap between processes.

The CPU has registers: `EAX`, `EBX`, `ECX`, `EDX`, `ESI`, `EDI`, `EBP`, `ESP`.

When we switch processes, we must save the current process's registers and restore the next process's registers.

**context_switch_asm Function:**
```
Input: Two pointers to stack pointers
- First pointer: where to SAVE the current process's ESP
- Second pointer: where to LOAD the next process's ESP
```

The function:
1. Saves current register state by pushing them onto the current stack
2. Writes the current ESP to the first parameter (saving our context)
3. Reads the next process's ESP from the second parameter
4. Restores registers by popping them from the next process's stack
5. Returns - but now we're in the next process's context!

This is pure assembly because it directly manipulates the CPU stack and registers.

---

## SECTION 9: KERNEL.C - PUTTING IT TOGETHER

### kernel.c - The Main Kernel

Now let's see how all these pieces work together in the main kernel code.

**Initialization Phase:**

`kmain()` - The kernel entry point:
1. Calls `serial_init()` - Set up serial port
2. Calls `memory_init()` - Initialize memory manager
3. Calls `process_init()` - Initialize process table
4. Calls `scheduler_init()` - Initialize scheduler
5. Prints welcome message

**Test Process Functions:**

`worker_process_high()` - A high-priority test process:
- Prints that it started
- Runs 5 iterations
- In each iteration, prints a message and does busy-wait (simulate work)
- Prints completion message
- Calls `process_exit()` to terminate

`worker_process_low()` - A low-priority test process:
- Same pattern as high priority
- Runs only 3 iterations
- Lower priority means it runs less frequently

`test_simple_process()` - A simple single process for basic testing

**Main Loop:**

After initialization, the kernel:
1. Creates multiple test processes
2. Starts the scheduler
3. Scheduler picks processes based on priority
4. Context switches between them
5. Each process runs for its time quantum
6. When quantum expires, scheduler picks another process

The output shows the interleaved execution as processes take turns running.

---

## SECTION 10: HOW IT ALL WORKS TOGETHER

Let me trace the execution flow:

**Step 1: Boot**
- GRUB loads boot.S at 1MB
- CPU jumps to `start` in boot.S
- Stack is set up, memory is cleared
- Jumps to `kmain()`

**Step 2: Kernel Initialization**
- Serial port configured
- Memory heap and stack initialized
- Process table zeroed
- Scheduler initialized

**Step 3: Process Creation**
- Kernel creates test processes by calling `process_create()`
- Each process gets:
  - A unique PID
  - A stack allocated from the heap
  - An entry point (function to execute)
  - A priority level

**Step 4: Scheduling Begins**
- Scheduler's timer interrupt (or polling) triggers `scheduler_tick()`
- Scheduler selects the highest-priority READY process
- `scheduler_context_switch()` is called

**Step 5: Context Switch**
- Current process's registers are pushed to its stack
- Current ESP is saved in its PCB
- Next process's ESP is loaded from its PCB
- Next process's registers are restored
- Execution continues in the next process

**Step 6: Process Execution**
- Process runs for its time quantum (10ms)
- It prints messages via `serial_puts()` using the serial driver
- It performs work (busy loop)
- Quantum expires

**Step 7: Loop Back**
- Scheduler picks the next process
- Context switches again
- Processes interleave, creating the illusion of simultaneous execution

**Output Pattern:**
You'll see output like:
```
[P-HIGH] high priority process started
[P-HIGH] iteration 0
[P-LOW] low priority process started
[P-LOW] iteration 0
[P-HIGH] iteration 1
[P-LOW] iteration 1
...
```

The processes are interleaved because the scheduler switches between them.

---

## SECTION 11: ADVANCED FEATURES

### Process Aging
The scheduler implements process aging to prevent starvation:
- High-priority processes run frequently
- Low-priority processes wait
- After 50 ticks, waiting processes get their priority improved
- They eventually get a chance to run

### Inter-Process Communication (IPC)
Each process has a message queue:
- `send_message(pid, value)` - Send a message to another process
- `receive_message()` - Wait for and receive a message
- This allows processes to synchronize and coordinate

### Memory Protection
Each process has its own stack:
- Stack overflow doesn't corrupt other processes
- Each process can work independently
- Memory is tracked and accounted for

### Statistics & Debugging
The kernel provides statistics:
- Memory usage (allocated, freed, pools used)
- Scheduler statistics (context switches, ticks)
- Process information (state, priority, age)
- Printed via serial for debugging

---

## SECTION 12: BUILDING & RUNNING

### The Build Process
```bash
make
```

This:
1. Compiles kernel.c, memory.c, process.c, scheduler.c
2. Compiles string.c and serial.c
3. Assembles boot.S and context_switch.S
4. Links everything together using link.ld
5. Produces a bootable kernel image

### Running the OS
```bash
qemu-system-i386 -kernel kernel.bin -serial stdio
```

This runs the kernel in QEMU (a virtual machine) with serial output visible in the terminal.

---

## SECTION 13: KEY CONCEPTS DEMONSTRATED

This project demonstrates:

1. **Bootloading** - How a CPU starts and loads the kernel
2. **Hardware Interface** - Direct hardware I/O via ports
3. **Memory Management** - Dynamic allocation, stack management
4. **Process Management** - Creating, tracking, managing processes
5. **CPU Scheduling** - Priority scheduling with aging
6. **Context Switching** - Saving/restoring CPU state in assembly
7. **Multitasking** - Multiple processes running concurrently
8. **Inter-Process Communication** - Processes passing messages
9. **Debugging** - Serial port communication for output
10. **Linker Scripts** - Controlling memory layout

Each of these is a core OS concept learned by implementing it here.

---

## SECTION 14: CONCLUSION

What we've built here is a simplified but real operating system. It demonstrates all the core concepts of OS design:

- We start from the bare metal with assembly
- We initialize hardware
- We manage memory and processes
- We implement scheduling and context switching
- We handle multiple processes concurrently
- We provide debugging and inter-process communication

This project is an excellent foundation for understanding modern operating systems. Real operating systems (Linux, Windows, macOS) use the same fundamental concepts, just with much more complexity and optimization.

Thank you for watching! Feel free to explore the code, modify it, add new features, and learn how operating systems work at a deep level.

---

## END OF SCRIPT
