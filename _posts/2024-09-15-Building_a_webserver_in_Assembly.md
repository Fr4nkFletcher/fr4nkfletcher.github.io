---
layout: post
title: "Building a Web Server in Assembly: An In-Depth Exploration"
date: 2024-09-15
tags: [assembly, assembly webserver, socket programming, file serving]
---

In this post, we will explore how to build a simple web server in assembly that can handle HTTP requests and serve a file in Linux.

## Introduction

Building a web server in assembly language is a great way to dive deep into how low-level socket programming works. In this project, we will set up a server that listens for incoming connections, accepts them, and responds with the contents of a file. For this example, we will serve a file called `flag` from the `/home/ffletch/` directory.

## Prerequisites

Make sure you have the following installed before proceeding:

- **`gcc` or any assembler tool like `as` (GNU Assembler)**  
- **`ld` (GNU Linker)**  
- **`strace` for debugging (optional)**

Ensure you have created a `flag` file at `/home/ffletch/flag` or wherever for the server to serve. You can do this by running:

```bash
echo "This is the flag file content!" > /home/ffletch/flag
```
## Understanding System Calls

In Linux, system calls are how user-space programs interact with the kernel. In assembly, system calls are made by setting the appropriate system call number in the `rax` register and executing the `syscall` instruction. Arguments are passed through registers such as `rdi`, `rsi`, and `rdx`.

Here are some system calls we will use:
1. **`socket()`**: Create an endpoint for communication.
2. **`bind()`**: Associate the socket with a local address (IP/port).
3. **`listen()`**: Mark the socket to accept incoming connections.
4. **`accept()`**: Extract the first connection on the queue of pending connections.
5. **`read()`**: Read data from a file descriptor (like a socket).
6. **`write()`**: Write data to a file descriptor (such as a socket).
7. **`close()`**: Close the file descriptor, freeing the associated resources.

## System Call Number Reference

In your assembly code, you directly use system call numbers in the `rax` register to invoke system calls via the `syscall` instruction. Here is a table mapping the system call names to their corresponding numbers on the x86_64 Linux architecture:

| System Call | Number (`rax`) | Description                                         |
|-------------|----------------|-----------------------------------------------------|
| `socket`    | 41             | Create an endpoint for communication.               |
| `bind`      | 49             | Bind a socket to an address.                        |
| `listen`    | 50             | Listen for incoming connections on a socket.        |
| `accept`    | 43             | Accept a connection on a socket.                    |
| `read`      | 0              | Read data from a file descriptor.                   |
| `write`     | 1              | Write data to a file descriptor.                    |
| `close`     | 3              | Close a file descriptor.                            |
| `exit`      | 60             | Terminate the calling process.                      |

For a more complete list of Linux system calls, you can check the [full list here](https://lxr.linux.no/linux+v3.2/arch/x86/include/asm/unistd_64.h).

## Full Code

```bash
.intel_syntax noprefix
.global _start

.section .data
sockaddr_in:  
    .word 2                # AF_INET (IPv4)
    .word 0x5000           # Port 80 in network byte order (0x5000)
    .long 0                # INADDR_ANY (bind to any address)

response_header:
    .ascii "HTTP/1.0 200 OK\r\n\r\n"  # HTTP header

buffer:                     
    .space 1024             # Buffer to store the incoming request

file_buffer:                
    .space 1024             # Buffer to store the file contents

file_path:
    .ascii "/home/ffletch/flag"  # Path to the file

.section .text
_start:
    # Create a socket
    mov rdi, 2              # AF_INET (IPv4)
    mov rsi, 1              # SOCK_STREAM (TCP)
    mov rdx, 0              # Protocol
    mov rax, 41             # socket()
    syscall
    mov rdi, rax            # Save socket file descriptor

    # Bind the socket
    lea rsi, [rip + sockaddr_in] 
    mov rdx, 16             # sockaddr_in size
    mov rax, 49             # bind()
    syscall

    # Listen for connections
    mov rsi, 0              # Backlog of 0
    mov rax, 50             # listen()
    syscall

    # Accept a connection
    xor rsi, rsi            # NULL for client address
    xor rdx, rdx            # NULL for addrlen
    mov rax, 43             # accept()
    syscall
    mov rdi, rax            # Save client socket file descriptor

    # Read the HTTP request
    lea rsi, [rip + buffer]  
    mov rdx, 1024            
    mov rax, 0               # read()
    syscall

    # Open the requested file
    lea rdi, [rip + file_path]  
    mov rsi, 0                  # O_RDONLY
    mov rax, 2                  # open()
    syscall
    mov r8, rax                 # Save file descriptor

    # Read the file contents
    lea rsi, [rip + file_buffer] 
    mov rdx, 1024                # Number of bytes to read
    mov rdi, r8                  # Use file descriptor
    mov rax, 0                   # read()
    syscall
    mov rdx, rax                 # Store the number of bytes read

    # Send HTTP response header
    mov rdi, 4                   # File descriptor (the client connection)
    lea rsi, [rip + response_header] 
    mov rdx, 19                  # Length of HTTP header (19 bytes)
    mov rax, 1                   # write()
    syscall

    # Send file contents
    lea rsi, [rip + file_buffer] 
    mov rax, 1                   # write()
    syscall

    # Close the file and connection
    mov rdi, r8                  # Close file descriptor
    mov rax, 3                   # close()
    syscall

    mov rdi, 4                   # Close socket
    mov rax, 3                   # close()
    syscall

    # Exit the program
    mov rdi, 0                   # Exit status 0
    mov rax, 60                  # exit()
    syscall
```

## How to Compile and Run

Here’s how you can compile and run the assembly web server step-by-step:

### Step 1: Compile the Assembly Code

To compile the assembly code, you will use the GNU Assembler (`as`) and Linker (`ld`).

```bash
as -o server.o server.s && ld -o server server.o
```

This will create an executable named `server`.

### Step 2: Run the Server

Once compiled, run the server to start listening on port 80.

```bash
sudo ./server
```

You will need `sudo` because port 80 is a privileged port, and you will only be able to bind to it with elevated permissions.

### Step 3: Verify Server is Running

You can use `netstat` or `ss` to check that the server is indeed listening on port 80:

```bash
sudo ss -tuln | grep :80
```

## Interacting with the Server using Curl

Now that the web server is up and running, you can interact with it using `curl` to make HTTP requests. Follow the steps below to fetch the contents of the file.

### Example 1: Fetch the File

You can use `curl` to send a GET request to the server. Open a new terminal and run the following command:

```bash
curl http://localhost:80
```

If everything is working correctly, the server will respond with the contents of the file `/home/ffletch/flag`.

### Example 2: Save the Output to a File

You can use `curl` to fetch the content and save it to a file:

```bash
curl http://localhost:80 -o output.txt
cat output.txt
This is the flag file content!
```

This will save the response to `output.txt`.

### Step 3: Debugging with `strace`

If you're encountering issues, you can use `strace` to trace the system calls being made by the server. This can help you understand where things might be going wrong.

```bash
sudo strace -f -e trace=open,read,write ./server
```

With `strace`, you'll see the system calls related to opening the file, reading its contents, and writing the response back to the client.

## Conclusion

This project walks you through building a simple web server in assembly that listens on port 80, accepts connections, and serves a file in response to HTTP requests. By interacting with the server using `curl` and tracing the system calls with `strace`, you can gain a deep understanding of low-level socket programming and file serving in assembly. We also covered various ways to interact with the server using `curl`, including saving output to a file.
