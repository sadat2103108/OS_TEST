# kacchiOS - Test Suite Documentation

## Overview

The kacchiOS kernel includes comprehensive built-in tests for all three major subsystems:
1. Memory Management
2. Process Management  
3. Scheduling System

All tests are automatically executed during kernel boot and can be re-run at any time.

## Automatic Boot Tests

When the kernel starts, it automatically executes the following test sequence:

### 1. Memory Manager Tests
```
[BOOT] Initializing kacchiOS...
[memory] initialized (heap=6KB)

========== MEMORY TEST ==========
[TEST] Heap allocation...
[memory] kmalloc 50B at 0
[memory] kmalloc 100B at 50
[memory] kmalloc 200B at 150
[OK] Multiple heap allocations
[TEST] Stack allocation...
[memory] alloc_stack 4KB at 2048
[memory] alloc_stack 4KB at 1024
[OK] Stack allocations
[TEST] Memory deallocation...
[memory] kfree 50B
[memory] kfree 100B
[memory] free_stack 4KB
[OK] Deallocations completed

========== MEMORY STATISTICS ==========
Total allocated: 6KB
Total freed: 2KB
Heap allocations: 4
Stack allocations: 2
Failed allocations: 0
Heap used: 2KB / 64KB
======================================
```

**What's Tested:**
- Multiple heap allocations of different sizes
- Stack allocation (top-down from heap end)
- Memory deallocation and tracking
- Metadata management
- Memory statistics accuracy
- Alignment validation (4-byte)

**Success Criteria:**
- All allocations return valid pointers
- Deallocations reduce used memory
- Statistics accurately reflect operations
- No overlapping allocations

### 2. Process Manager Tests
```
========== PROCESS TEST ==========
[TEST] Create high priority process...
[process] created PID 1 (priority=2)
[OK] High priority creation
[TEST] Create low priority process...
[process] created PID 2 (priority=10)
[OK] Low priority creation
[TEST] State transitions...
[process] PID 1 state changed
[OK] State change to BLOCKED
[process] PID 1 state changed
[OK] State change to READY
[TEST] Get process utilities...
[OK] process_get() works
[OK] Active processes: 2

========== PROCESS TABLE ==========
PID 1: state=READY, priority=2
PID 2: state=READY, priority=10
Total processes: 2
===================================
```

**What's Tested:**
- Process creation with priority support
- Process table management
- State transitions (READY ↔ BLOCKED)
- Process utilities (get, list, count)
- Priority assignment validation
- Process cleanup

**Success Criteria:**
- Process creation succeeds for valid parameters
- Processes added to process table
- State transitions work correctly
- Process list shows all active processes
- Utilities return correct information

### 3. Scheduler Tests
```
========== SCHEDULER TEST ==========
[TEST] Initialize scheduler...
[scheduler] initialized with quantum=10ms
[OK] Scheduler initialized
[TEST] Set time quantum to 20ms...
[scheduler] time quantum set to 20ms
[OK] Quantum set correctly
[TEST] Select next process...
[OK] Selected process PID 1
[TEST] Context switch...
[scheduler] starting first process PID 1
[TEST] Apply aging algorithm...
[scheduler] aging applied, 1 processes promoted

========== SCHEDULER STATISTICS ==========
System ticks: 5
Context switches: 3
Current quantum: 20ms
Current process PID: 1

Ready processes:
  PID 1: priority=2, age=5
  PID 2: priority=9, age=10
=========================================
```

**What's Tested:**
- Scheduler initialization
- Time quantum configuration
- Process selection algorithm
- Context switching
- Aging algorithm implementation
- Statistics tracking

**Success Criteria:**
- Scheduler initializes without errors
- Quantum configuration works (1-100ms)
- Highest priority process selected first
- Context switches logged correctly
- Aging increases priority of waiting processes
- Statistics accurately track operations

### 4. IPC Tests
```
========== IPC TEST ==========
[TEST] Create IPC processes...
[process] created PID 1 (priority=5)
[process] created PID 2 (priority=5)
[OK] IPC processes created
[TEST] IPC simulation...
[IPC] message sent from PID 1 to PID 2
[OK] Message sent
[IPC] received message value
[OK] Message received
```

**What's Tested:**
- Message sending between processes
- Message queue management
- Message reception and dequeuing
- Message value preservation
- Queue ordering (FIFO)

**Success Criteria:**
- Messages successfully sent
- Messages successfully received
- Queue order maintained
- No message loss or corruption

## Runtime Testing

### Running Tests from Shell

You can re-run the full test suite at any time using the shell command:

```
kacchiOS> test

Running comprehensive tests...

========== MEMORY TEST ==========
[TEST] Heap allocation...
[memory] kmalloc 50B at 0
...
```

### Individual Component Testing

**Memory Statistics:**
```
kacchiOS> memstat

========== MEMORY STATISTICS ==========
Total allocated: 6KB
Total freed: 2KB
Heap allocations: 4
Stack allocations: 2
Failed allocations: 0
Heap used: 2KB / 64KB
======================================
```

Shows current memory usage and statistics.

**Process List:**
```
kacchiOS> proclist

========== PROCESS TABLE ==========
PID 1: state=READY, priority=2
PID 2: state=RUNNING, priority=10
PID 3: state=BLOCKED, priority=5
Total processes: 3
===================================
```

Lists all active processes with their states and priorities.

**Scheduler Statistics:**
```
kacchiOS> schedstat

========== SCHEDULER STATISTICS ==========
System ticks: 5
Context switches: 3
Current quantum: 10ms
Current process PID: 1

Ready processes:
  PID 2: priority=10, age=5
  PID 3: priority=9, age=10
=========================================
```

Shows scheduler state, timing, and process aging.

## Test Coverage Matrix

| Component | Feature | Test Method | Result |
|-----------|---------|------------|--------|
| **Memory** | Heap allocation | `kmalloc(size)` calls | ✓ Multiple sizes |
| | Heap deallocation | `kfree(ptr)` calls | ✓ Tracking works |
| | Stack allocation | `alloc_stack()` calls | ✓ Top-down from heap |
| | Stack deallocation | `free_stack(ptr)` calls | ✓ Cleanup successful |
| | Optimization | 4-byte alignment | ✓ All allocations aligned |
| **Process** | Creation | `process_create()` | ✓ Multiple processes |
| | Termination | `process_exit()` | ✓ Cleanup completed |
| | State transition | `process_set_state()` | ✓ All state changes |
| | Utilities | `process_get()`, `process_list()` | ✓ Correct info |
| | Additional states | SLEEPING, BLOCKED | ✓ Implemented |
| | IPC sending | `process_send()` | ✓ Messages sent |
| | IPC receiving | `process_receive()` | ✓ Messages received |
| **Scheduler** | Policy | Round-Robin + Priority | ✓ High priority first |
| | Context switch | `scheduler_context_switch()` | ✓ Switches tracked |
| | Time quantum | Configurable (1-100ms) | ✓ Settable |
| | Aging | Every 50 ticks | ✓ Priority boosted |

## Failure Scenarios Tested

The test suite also validates error handling:

### Memory Manager Errors
- Heap exhaustion (not enough memory)
- Invalid free (double free detection)
- Stack exhaustion
- Invalid allocation size

### Process Manager Errors
- Process table full
- Invalid PID
- No current process
- Message queue full
- Invalid state transitions

### Scheduler Errors
- No READY process available
- Invalid quantum value
- Priority range validation (1-20)

## Performance Benchmarks

### Memory Allocation Times
- Single allocation: O(1)
- Multiple allocations: O(n)
- Search for free slot: O(max_allocs)

### Process Operations
- Process creation: O(n) where n = processes
- State transition: O(n)
- Process lookup: O(n)

### Scheduling
- Next process selection: O(n) where n = processes
- Context switch: O(1)
- Aging calculation: O(n)

## Debugging with Logs

All operations are logged with prefixes for easy filtering:

```
[memory]   - Memory manager operations
[process]  - Process management operations
[scheduler] - Scheduler operations
[IPC]      - Inter-process communication
[TEST]     - Test section markers
[OK]       - Test passed
[FAIL]     - Test failed
```

Example: Filter memory operations
```
qemu-system-i386 ... 2>&1 | grep "\[memory\]"
```

Example: Filter all tests
```
qemu-system-i386 ... 2>&1 | grep "\[TEST\]"
```

## Expected Output Patterns

### Successful Boot
```
[BOOT] Initializing kacchiOS...
[memory] initialized
[process] initialized
[scheduler] initialized
...
========== MEMORY TEST ==========
[OK] Multiple heap allocations
...
System initialized successfully!
Type 'help' for commands
kacchiOS>
```

### Memory Stress Test
Allocate until exhaustion:
```
[memory] kmalloc XXB at Y
[memory] FAIL: heap exhausted
```

### Process Table Full
```
[process] FAIL: process table full
```

### IPC Success
```
[IPC] message sent from PID 1 to PID 2
[IPC] received message value
```

## Manual Testing Guide

1. **Boot Kernel**
   ```bash
   make run
   ```

2. **Wait for boot tests to complete** (automatic)

3. **Run individual tests**
   ```
   kacchiOS> memstat
   kacchiOS> proclist
   kacchiOS> schedstat
   ```

4. **Re-run full test suite**
   ```
   kacchiOS> test
   ```

5. **Check system status at any time**
   ```
   kacchiOS> help
   kacchiOS> proclist
   kacchiOS> memstat
   ```

## Troubleshooting

### No output appears
- Check serial port configuration in link.ld
- Verify QEMU with `-serial stdio` flag

### Tests fail immediately
- Check memory.c compilation
- Verify process.c and scheduler.c are linked
- Check Makefile has all .o files

### Memory test fails
- Verify KERNEL_HEAP_SIZE definition
- Check align4() function
- Test manual kmalloc() in serial

### Process test fails
- Verify process_init() called
- Check PCB initialization
- Validate next_pid counter

### Scheduler test fails
- Verify scheduler_init() called
- Check process table has READY processes
- Validate quantum values (1-100)

## Success Verification

A successful implementation should show:
- ✓ All boot tests pass
- ✓ Memory statistics accurate
- ✓ All processes listed correctly
- ✓ Scheduler selects processes in priority order
- ✓ Context switches logged
- ✓ Aging algorithm working (priorities increase)
- ✓ IPC messages sent and received
- ✓ Shell commands responsive
- ✓ No crashes or hangs
- ✓ Proper cleanup on termination

## Notes

- Tests run in sequence, not parallel
- Each test is independent
- Failed test doesn't stop subsequent tests
- Logs help identify which component failed
- All operations are blocking (synchronous)
