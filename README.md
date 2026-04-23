# Packet Logger using SDN Controller

> This is QS: Capture and log packets traversing the network using controller events.

## Project Overview

This project implements a simple SDN-based packet logger using the Ryu controller framework. It captures incoming packets at the OpenFlow switch, logs packet metadata to the console and `log.txt`, and forwards packets by flooding them to connected ports.

The main goal is to:
- capture packet headers
- identify protocol types (ICMP, TCP, UDP)
- maintain a persistent log file
- display packet information in real time

## Features

- Uses Ryu SDN controller and OpenFlow 1.3
- Installs a table-miss flow entry to send unmatched packets to the controller
- Logs packet timestamp, source/destination IP, protocol, and port
- Writes packet log entries to `log.txt`
- Supports ICMP, TCP, and UDP packet inspection

## Architecture Flow

The packet logger follows this flow:

1. SDN controller starts and initializes the Ryu app
2. Switch connects and sends `EventOFPSwitchFeatures`
3. Controller installs a table-miss flow entry
4. Switch sends unmatched packets to controller via `PacketIn`
5. Controller parses packet headers
6. Controller floods packet back into the network
7. Controller logs packet metadata to console and `log.txt`

### Flow Architecture Diagram

```mermaid
flowchart TD
    A[OpenFlow Switch] -->|PacketIn| B(Ryu Controller)
    B --> C{Parse Packet}
    C --> D[Ethernet Header]
    C --> E[IPv4 Header]
    E --> F{Protocol}
    F -->|TCP| G[Log TCP src/dst/port]
    F -->|UDP| H[Log UDP src/dst/port]
    F -->|Other| I[Log ICMP / Default]
    B -->|PacketOut (FLOOD)| A
    B --> J[Append to log.txt]
```

## File Structure

- `packet_logger.py` - main Ryu application for packet logging
- `log.txt` - generated packet log output file

## Requirements

- Python 3.x
- Ryu SDN framework
- OpenFlow 1.3 compatible switch or network simulator (Mininet, Open vSwitch)

## Setup

1. Install Ryu:

```bash
pip install ryu
```

2. Ensure your OpenFlow switch supports OpenFlow 1.3.

## Run Commands

This project uses two terminals:

- Terminal 1: start the Ryu controller and packet logger
- Terminal 2: start Mininet with a switch and hosts

### Terminal 1

```bash
ryu-manager packet_logger.py
```

### Terminal 2

Start Mininet with a remote controller and a simple topology, for example:

```bash
sudo mn --topo=single,3 --switch=ovsk --controller=remote
```

This creates one Open vSwitch and three hosts connected to it. Mininet will connect to the Ryu controller running in Terminal 1.
```

Then use Mininet CLI commands to generate traffic.

### ICMP test

```bash
mininet> pingall
```

This will send ICMP echo requests between all hosts and generate log entries in `log.txt`.

### TCP test with iperf

```bash
mininet> h2 iperf -s &
mininet> h1 iperf -c h2
```

### UDP test with iperf

```bash
mininet> h2 iperf -s -u &
mininet> h1 iperf -c h2
```

These commands start an `iperf` server on host `h2` and connect from `h1`.

## Example Output

`log.txt` receives entries like:

```text
06:29:29 | 10.0.0.1 -> 10.0.0.2 | ICMP | Port: -
06:30:02 | 10.0.0.2 -> 10.0.0.3 | ICMP | Port: -
06:50:51 | 10.0.0.1 -> 10.0.0.2 | TCP | Port: 5001
06:50:52 | 10.0.0.2 -> 10.0.0.1 | TCP | Port: 49272
06:52:10 | 10.0.0.1 -> 10.0.0.3 | UDP | Port: 5201
06:52:11 | 10.0.0.3 -> 10.0.0.1 | UDP | Port: 5201
```

## Notes

- The current implementation floods all packets after logging.
- TCP and UDP port values are pulled from packet headers when available.
- Non-TCP/UDP packets are logged as `ICMP` with `Port: -` by default.

## Improvements

Possible enhancements:
- add VLAN or ARP support
- log MAC addresses
- filter and categorize packets before forwarding
- build a UI dashboard for live packet statistics
