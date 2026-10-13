# Synchronous FIFO in Verilog

A small, parameterizable **synchronous FIFO** (First-In, First-Out buffer) written in Verilog. Data written first is read first. Reads and writes use the same clock.
## Parameters

| Parameter | Default | Meaning |
|---|---|---|
| `FIFO_DEPTH` | 8 | Number of words stored. **Must be a power of 2 and at least 2** (2, 4, 8, 16, ...). |
| `DATA_WIDTH` | 32 | Width of each word in bits |

## Ports

| Port | Direction | Description |
|---|---|---|
| `clk` | input | Clock. Everything happens on the rising edge. |
| `rst_n` | input | Reset, **active low** and asynchronous. Empties the FIFO. |
| `cs` | input | Chip select. Reads and writes are ignored when `cs` is 0. |
| `wr_en` | input | Write enable |
| `rd_en` | input | Read enable |
| `data_in` | input | Data to write |
| `data_out` | output | Data read from the FIFO |
| `empty` | output | 1 when the FIFO has no data |
| `full` | output | 1 when the FIFO cannot take more data |

## How it works

- Data is stored in an array of `FIFO_DEPTH` words.
- A **write pointer** and a **read pointer** say where to write next and where to read next. Each pointer has **one extra bit** on top of the bits needed to address the array (4 bits for depth 8).
- **Empty:** both pointers are exactly equal.
- **Full:** the lower bits are equal but the extra top bit is different. This means the write pointer has gone around the array once more than the read pointer. The extra bit is what lets the FIFO tell "full" from "empty".

Rules:
- A **write** happens on a rising clock edge when `cs && wr_en && !full`. If the FIFO is full, the write is ignored.
- A **read** happens on a rising clock edge when `cs && rd_en && !empty`. If the FIFO is empty, the read is ignored and `data_out` keeps its old value.
- `data_out` is a register: the data appears right after the clock edge on which the read happens.
- Reset clears the pointers only. The stored words and `data_out` are not cleared, so `data_out` is unknown (`x`) until the first read.

## Testbench scenarios

| Scenario | What it does | Expected reads |
|---|---|---|
| 1 | Write 1, 10, 100, then read 3 times | 1, 10, 100 |
| 2 | Write then immediately read, 8 times (values 1, 2, 4, ... 128) | 1, 2, 4, 8, 16, 32, 64, 128 |
| 3 | Write 9 values into the 8-deep FIFO, then read 8 times | 1, 2, 4, 8, 16, 32, 64, 128 (the 9th value, 256, is rejected because the FIFO is full) |

The testbench prints the values; check them against the table above.

## Limitation

- One clock for both read and write. Crossing between two different clocks needs an asynchronous FIFO.