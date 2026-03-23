# Operating System Concepts — Learning Roadmap (Win32 / Windows API)

A systematic guide to learning OS concepts using Microsoft Learn's Win32 documentation.

---

## Learning Path

```
┌──────────────────────────────────────────────────────────────────┐
│  1. Processes & Threads                                          │
│     Creating, scheduling, and managing execution units           │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  2. Memory Management                                            │
│     Virtual memory, paging, heaps, memory-mapped files           │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  3. Synchronization                                              │
│     Mutexes, semaphores, events, critical sections               │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  4. Interprocess Communication (IPC)                             │
│     Pipes, shared memory, sockets, DDE, file mapping             │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  5. File Systems & I/O                                           │
│     File I/O, directories, volumes, async I/O, disk mgmt         │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  6. DLLs & Dynamic Linking                                       │
│     Loading, linking, runtime binding, symbol resolution          │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  7. Diagnostics & Debugging                                      │
│     Error handling, event tracing, performance counters, WER     │
└───────────────────────────────┬──────────────────────────────────┘
                                │
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│  8. Security & Identity                                          │
│     Authentication, access control, tokens, cryptography         │
└──────────────────────────────────────────────────────────────────┘
```

> Build bottom-up: each layer depends on understanding the one above it.

---

## Documentation Links

### 1. Processes & Threads
- [Processes and Threads — Win32](https://learn.microsoft.com/en-us/windows/win32/procthread/processes-and-threads)

### 2. Memory Management
- [Memory Management — Win32](https://learn.microsoft.com/en-us/windows/win32/memory/memory-management) — virtual memory, heaps, memory-mapped files, large memory support

### 3. Synchronization
- [Synchronization — Win32](https://learn.microsoft.com/en-us/windows/win32/sync/synchronization) — mutexes, semaphores, events, critical sections
- [Synchronization and Multiprocessor Issues](https://learn.microsoft.com/en-us/windows/win32/sync/synchronization-and-multiprocessor-issues) — memory barriers, acquire/release semantics

### 4. Interprocess Communication (IPC)
- [IPC — Win32](https://learn.microsoft.com/en-us/windows/win32/ipc/interprocess-communications) — pipes, mailslots, shared memory, sockets, file mapping

### 5. File Systems & I/O
- [File Management — Win32](https://learn.microsoft.com/en-us/windows/win32/fileio/file-management) — sync/async I/O, directories, volumes

### 6. DLLs & Dynamic Linking
- [Dynamic-Link Libraries — Win32](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-libraries) — loading, symbol resolution, implicit vs explicit linking

### 7. Diagnostics & Debugging
- [Debugging — Win32](https://learn.microsoft.com/en-us/windows/win32/debug/debugging) — breakpoints, structured exception handling, post-mortem debugging
- [Event Tracing — Win32](https://learn.microsoft.com/en-us/windows/win32/etw/event-tracing-portal) — ETW, performance counters

### 8. Security & Identity
- [Authorization — Win32](https://learn.microsoft.com/en-us/windows/win32/secauthz/authorization-portal) — access tokens, ACLs, privileges

---

## Master Reference

All of the above topics live under the [System Services](https://learn.microsoft.com/en-us/windows/win32/system-services) section of the [Windows API Index](https://learn.microsoft.com/en-us/windows/win32/apiindex/windows-api-list) — the master table of contents for OS internals via Win32.

---

## Supplementary Resource

For a theory-first approach alongside these docs, [Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/) (free) pairs very well with this Win32 doc path.
