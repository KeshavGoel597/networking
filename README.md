# S.H.A.M. - A Reliable UDP Protocol

## Description

This project implements a custom reliable transport protocol over UDP, named S.H.A.M. (Simulated Handshake and Acknowledgement Mechanism). It provides reliable data transfer for both file transfers and a real-time chat application. The protocol includes mechanisms for connection establishment (3-way handshake), connection termination (4-way handshake), reliable data transfer using acknowledgements and retransmissions, and flow control.

## Features

- **Reliable Data Transfer**: Ensures data is delivered completely and in order.
- **Connection Management**: Implements a 3-way handshake for connection setup and a 4-way handshake for teardown.
- **Flow Control**: Uses a sliding window protocol to manage the amount of data in flight, preventing receiver buffer overflow.
- **Two Modes of Operation**:
    - **File Transfer Mode**: Reliably transfers a file from a client to a server. The server calculates and prints the MD5 checksum of the received file.
    - **Chat Mode**: A bi-directional chat application where both client and server can send and receive messages.
- **Packet Loss Simulation**: The user can specify a packet loss rate to test the protocol's reliability.
- **Event Logging**: Detailed logging of protocol events for debugging and analysis, controlled by an environment variable.

## How to Compile and Run

### Prerequisites

- `gcc` compiler
- `make` utility
- `openssl` library (for MD5 checksum)

### Compilation

To compile the client and server applications, run the `make` command in the project directory:

```bash
make
```

This will generate two executables: `client` and `server`.

To clean up the build files, you can run:

```bash
make clean
```

### Running the Applications

The applications can be run in either File Transfer mode or Chat mode.

#### File Transfer Mode

**Server:**

```bash
./server <port> [loss_rate]
```

- `<port>`: The port number for the server to listen on.
- `[loss_rate]`: (Optional) A floating-point value between 0.0 and 1.0 to simulate packet loss.

**Client:**

```bash
./client <server_ip> <server_port> <input_file> <output_file_name> [loss_rate]
```

- `<server_ip>`: The IP address of the server.
- `<server_port>`: The port number of the server.
- `<input_file>`: The path to the file to be sent.
- `<output_file_name>`: The name of the file to be saved on the server.
- `[loss_rate]`: (Optional) A floating-point value between 0.0 and 1.0 to simulate packet loss.

#### Chat Mode

**Server:**

```bash
./server <port> --chat [loss_rate]
```

**Client:**

```bash
./client <server_ip> <server_port> --chat [loss_rate]
```

In chat mode, both the client and server can type messages into the console and press Enter to send them. To quit the chat, type `/quit` and press Enter.

## File Descriptions

- **`server.c`**: The source code for the server application. It can handle both file transfer and chat modes.
- **`client.c`**: The source code for the client application. It can initiate file transfers or a chat session.
- **`sham.h`**: The header file containing the definition of the S.H.A.M. protocol header and function prototypes for the protocol's implementation.
- **`Makefile`**: The makefile for compiling the project.

## Protocol Details

### S.H.A.M. Header

The protocol uses a custom header for all its packets:

```c
struct sham_header {
    uint32_t seq_num;      // Sequence Number
    uint32_t ack_num;      // Acknowledgment Number
    uint16_t flags;        // Control flags (SYN, ACK, FIN)
    uint16_t window_size;  // Flow control window size
    uint16_t datalen;      // Data length
    uint16_t reserved;     // Reserved
};
```

- **Flags**: `SHAM_SYN` (0x1), `SHAM_ACK` (0x2), `SHAM_FIN` (0x4).

### Connection Establishment (3-Way Handshake)

1.  **Client to Server**: `SYN` (with initial sequence number `x`)
2.  **Server to Client**: `SYN-ACK` (with its own initial sequence number `y` and acknowledgment `x+1`)
3.  **Client to Server**: `ACK` (with acknowledgment `y+1`)

### Connection Termination (4-Way Handshake)

The protocol implements a graceful 4-way handshake to terminate the connection, ensuring that both sides have finished transmitting data.

### Reliable Data Transfer

- **Sequence Numbers**: Each byte of data is assigned a sequence number.
- **Acknowledgments (ACKs)**: The receiver sends cumulative ACKs for the data it has received in order.
- **Retransmission Time-Out (RTO)**: If the sender does not receive an ACK for a packet within a certain time, it retransmits the packet. The RTO is currently fixed at 500ms.

### Flow Control

A sliding window mechanism is used for flow control. The receiver advertises its available buffer space (window size) in the `window_size` field of the S.H.A.M. header. The sender ensures it does not send more data than the receiver's advertised window.

## Logging

To enable logging, set the `RUDP_LOG` environment variable to `1`.

```bash
export RUDP_LOG=1
```

When enabled, the client and server will create `client_log.txt` and `server_log.txt` respectively, containing detailed, timestamped logs of protocol events. This is very useful for debugging and understanding the protocol's behavior.
