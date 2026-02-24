# Unix Pipe Implementation in C

## Overview

This project is a low-level reimplementation of Unix shell piping behavior in C.

It reproduces the behavior of:

```
< infile cmd1 | cmd2 > outfile
```

using direct system calls and manual process control, without relying on the shell.

The goal of this project is to demonstrate deep understanding of:

- Inter-process communication (IPC)
- File descriptor management
- Process creation and execution
- Environment path resolution
- Error handling at the system call level

---

## How It Works

The program creates a unidirectional pipe between two child processes:

1. First child:
   - Reads from `infile`
   - Writes to the pipe
   - Executes `cmd1`

2. Second child:
   - Reads from the pipe
   - Writes to `outfile`
   - Executes `cmd2`

The parent process:
- Creates the pipe
- Forks children
- Waits for execution to finish

---

## System Calls Used

This implementation directly uses:

- `fork()` – process creation
- `pipe()` – inter-process communication
- `dup2()` – file descriptor redirection
- `execve()` – command execution
- `wait()` – process synchronization
- `open()` – file handling
- `access()` – path resolution
- `close()` – file descriptor management

No shell helpers are used.

---

## Execution Flow

```
Parent
 ├── pipe()
 ├── fork() → Child 1
 │     ├── open(infile)
 │     ├── dup2(infile → STDIN)
 │     ├── dup2(pipe write → STDOUT)
 │     └── execve(cmd1)
 │
 ├── wait()
 ├── fork() → Child 2
 │     ├── open(outfile)
 │     ├── dup2(pipe read → STDIN)
 │     ├── dup2(outfile → STDOUT)
 │     └── execve(cmd2)
 │
 └── wait()
```

---

## Compilation

```
make
```

This produces:

```
./pipex
```

---

## Usage

```
./pipex infile "cmd1" "cmd2" outfile
```

Example:

```
./pipex input.txt "grep hello" "wc -l" output.txt
```

Equivalent shell behavior:

```
< input.txt grep hello | wc -l > output.txt
```

---

## Path Resolution Strategy

Commands are resolved using the `PATH` environment variable.

The program:
1. Parses `PATH`
2. Splits it into directories
3. Concatenates each directory with the command
4. Uses `access()` to verify executable presence
5. Executes using `execve()`

---

## Error Handling Strategy

- Invalid file descriptors are checked immediately.
- Failed system calls trigger proper error reporting via `perror()`.
- Memory allocated for command parsing is freed before exit.
- Pipe endpoints are closed in all failure paths to prevent leaks.

---

## Memory Management

- All dynamically allocated command arrays are freed after execution failure.
- No orphan file descriptors remain open.
- Each process cleans its own resources before exiting.

---

## Key Concepts Demonstrated

- Unix process model
- File descriptor duplication
- Shell command emulation
- IPC through anonymous pipes
- Environment variable parsing
- Defensive programming in C

---

## What This Project Demonstrates

This implementation shows strong understanding of:

- Low-level Unix architecture
- Process lifecycle management
- Descriptor table manipulation
- Controlled execution environments

It reflects practical systems programming skills applicable to:

- Shell development
- Backend infrastructure tools
- Container runtimes
- Process orchestration systems

---

## Author

Khaled Adas  
Systems Programming | C | Unix Internals
