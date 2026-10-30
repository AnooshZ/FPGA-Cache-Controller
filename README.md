# FPGA-Based Cache Controller

A 256-byte direct-mapped cache controller designed in VHDL and implemented on a Xilinx Spartan-3E FPGA for my COE758 Digital Systems Engineering course at Toronto Metropolitan University.

I worked on this project with a partner. The goal was to build a cache controller that could handle CPU read/write requests, manage data transfers between the cache and main memory, and correctly handle cache hits and misses.

The controller was designed around an 11-state finite state machine (FSM) and tested using both RTL simulation and on-chip debugging with Xilinx ChipScope Pro.

## Project Overview

The system consists of three main components:

- **CPU transaction generator:** Generates read and write requests to different memory addresses.
- **Cache controller:** Processes CPU requests, checks for cache hits or misses, and manages memory transfers.
- **Memory system:** A 256-byte BRAM cache connected to a 2 KB SDRAM-style main-memory model.

The CPU transaction generator was provided as part of the course. Our work focused on implementing the cache controller, integrating the memory system, and verifying its behaviour.

### Cache Configuration

| Parameter | Value |
|---|---|
| Cache organization | Direct-mapped |
| Cache capacity | 256 bytes |
| Number of blocks | 8 |
| Block size | 32 bytes |
| Data width | 8 bits |
| Address width | 16 bits |
| Address breakdown | 8-bit tag, 3-bit index, 5-bit offset |
| Write policy | Write-back, write-allocate |
| Controller | 11-state FSM |
| FPGA clock | 50 MHz |

## How It Works

The cache controller sits between the CPU and main memory. Whenever the CPU sends a read or write request, the controller checks whether the requested memory block is already stored in the cache.

There are four main cases:

- **Read hit:** The requested data is already in the cache, so it can be returned without accessing main memory.
- **Write hit:** The data is updated directly in the cache, and the block is marked dirty.
- **Clean miss:** The requested block is not cached. The controller fetches the block from main memory without needing to write back the existing block.
- **Dirty miss:** The block being replaced has been modified. The controller first writes it back to main memory, then fetches the new block.

The FSM controls the sequence of operations, including SRAM read/write enables, memory strobes, address selection, and the CPU-ready signal. During a cache miss, the CPU is held in a waiting state until the required memory transfers are complete.

## Implementation

The cache controller was written in **VHDL**, using an FSM to coordinate the different memory operations.

The design was integrated with a 256 × 8-bit BRAM cache and a synthesizable SDRAM-style memory model. The memory model was used to represent main-memory transactions without requiring a physical SDRAM device.

We used **Xilinx ISE** for development and synthesis, and **ISim** to check the controller's behaviour before testing it on the FPGA.

The design was then implemented on a **Xilinx Spartan-3E (XC3S500E)** FPGA, where we used **ChipScope Pro** to inspect internal signals and debug memory transactions.

## Testing and Verification

We tested the four main cache scenarios using ChipScope Pro's Integrated Logic Analyzer (ILA).

The captured waveforms allowed us to check whether the controller was correctly switching between memory operations, handling cache hits and misses, and keeping the CPU waiting during longer transactions.

One of the more useful tests was the dirty miss. We could see the controller perform a memory write to preserve the modified block, followed by a memory read to load the replacement block.

### Measured Latency

The following transaction latencies were measured with a 50 MHz clock (20 ns per cycle):

| Operation | Clock Cycles | Latency |
|---|---:|---:|
| Write hit | 3 | 60 ns |
| Read hit | 5 | 100 ns |
| Clean miss | 67 | 1.34 µs |
| Dirty miss | 131 | 2.62 µs |

These measurements show the difference between accessing data already in the cache and having to transfer an entire block from main memory. Dirty misses take the longest because they require both a write-back and a block refill.

**[View hardware verification results](verification/README.md)**

The verification folder includes ChipScope screenshots and explanations for each of the four test cases.

## Repository Structure

```text
FPGA-Cache-Controller/
├── rtl/
│   ├── CacheController.vhd
│   ├── CacheSystem.vhd
│   ├── CacheTop.vhd
│   └── SDRAM_model.vhd
├── support/
│   └── CPU_gen.vhd
├── verification/
│   ├── read_hit.png
│   ├── write_hit.png
│   ├── clean_miss.png
│   ├── dirty_miss.png
│   └── README.md
└── README.md
```

The `rtl/` folder contains the cache-controller and memory-system design files. The `support/` folder contains the CPU transaction generator provided for the course.

## Tools Used

- **VHDL** — RTL design
- **Xilinx ISE** — FPGA development and synthesis
- **ISim** — RTL simulation
- **Xilinx ChipScope Pro** — On-chip signal capture and debugging
- **Xilinx Spartan-3E** — FPGA implementation and testing

## What I Learned

This project helped me better understand how cache controllers work beyond the theory covered in class.

The most useful part was seeing how the FSM, memory control signals, and CPU-ready handshaking all had to work together. A cache hit is relatively straightforward, but handling a dirty miss requires several operations to happen in the correct order without losing modified data.

Testing the design on the FPGA was also valuable because I could compare the expected behaviour from simulation with the actual signals captured through ChipScope.

The project gave me more experience with VHDL, memory-system design, FPGA implementation, and hardware debugging.
