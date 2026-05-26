# Multi-Threaded TCP Group Chat Server

A concurrent TCP chat server written in **C** using `pthreads` and sockets. Originally built for a **Systems Programming** course project at Simon Fraser University (CMPT 201).  

## Features

- **Multi-Threading:** Handles multiple client connections at the same time and broadcasts messages to everyone.  
- **Global Message Ordering:** Guarantees all clients see messages in the exact same order, while keeping each sender's messages in sequence.  
- **Custom Protocol:** Uses tagged messages (`0` for chat, `1` for shutdown) that include the sender's IP and port.  
- **Two-Phase Commit:** Safely shuts down the server only after all clients finish sending their data.  
- **Stress Testing:** Includes a "fuzzing" client that sends random messages to test server stability under load.  
- **Proven Scalability:** Tested and verified to work smoothly with over 100 concurrent clients.  

## Requirements

- **OS:** Linux (Ubuntu, Docker, or WSL). *Not compatible with Windows.*
- **Compiler:** `clang`  
- **Build Tools:** `cmake` and `make`  

### How to Build
```bash
mkdir build
cd build
cmake ..
make
```
### This creates two executables:
- ./server — The multi-threaded chat server.
- ./client — The automated testing client.

### Running the Server
```Bash
./server <port> <number_of_clients>
```
Example:
```Bash
./server 8000 3
```
### Running the Client
```Bash
./client <server_ip> <port> <number_of_messages> <log_file>
```
Example:
```Bash
./client 127.0.0.1 8000 10 client0.log
```

### Testing

This project was verified using official course testing tools to check for:

- Stable connections and basic message broadcasting.

- Correct message formatting and protocol rules.

- Perfect message ordering across multiple clients.

- Clean shutdowns using shutdown (1) tags.

- High-load stability (100+ clients sending 100+ messages each).

You can also test the server manually using:
```Bash
telnet 127.0.0.1 8000
```
