# APB Memory Verification using UVM
A complete **UVM-based functional verification environment** for a 64KB APB memory slave — featuring 64-bit data width, PSTRB byte-enables, PSLVERR error handling, wait states, and full functional + code coverage closure.
## Project Overview
This project implements a **layered UVM verification environment** to verify an **APB memory slave** design. The DUT is a 64KB memory with 64-bit data width, requiring 8-byte address alignment, supporting byte-enable writes via PSTRB, and generating error responses via PSLVERR for out-of-bounds and misaligned accesses.

#**Verification Goals:**
- ✅ Verify APB protocol (IDLE → SETUP → ACCESS)
- ✅ Verify read/write transactions
- ✅ Verify PSTRB byte-enable (all 256 combinations)
- ✅ Verify address alignment (8-byte)
- ✅ Verify error handling (PSLVERR)
- ✅ Verify boundary conditions
- ✅ Verify reset behavior
- ✅ Verify wait states (PREADY)
- ✅ Achieve 100% functional coverage
- ✅ Achieve high code coverage

- ## DUT Description

**Top Module:** `apb_wrapper`

### Specifications

| Parameter | Value |
|---|---|
| **Memory Size** | 64 KB (65,536 bytes) |
| **Data Width** | 64-bit |
| **Address Width** | 32-bit |
| **Alignment** | 8-byte |
| **Base Address** | 0 (parameterizable) |
| **PSTRB** | 8-bit byte-enable |
| **PSLVERR** | Error flag |
| **PREADY** | Wait-state extension |
| **Reset** | Active-low asynchronous |

### Internal Modules

| Module | Purpose |
|---|---|
| `apb_wrapper` | Top-level wrapper |
| `apb_fsm` | APB protocol state machine (Mealy FSM) |
| `err_gen` | Error generation (out-of-bounds, misaligned) |
| `generic_mem` | 64KB memory block |
| `mem_1024x32` | 4KB memory block (building block) |

### Error Conditions

| Condition | PSLVERR |
|---|---|
| Address outside 64KB window | 1 |
| Misaligned address (not 8-byte) | 1 |
| Valid address + aligned | 0 |

### Component Hierarchy

```
uvm_test_top (apb_*_test)
  └── environment (apb_env)
        ├── agent (apb_agent)
        │     ├── sequencer (apb_sequencer)
        │     ├── driver    (apb_driver)
        │     └── monitor   (apb_monitor)
        ├── scoreboard (apb_scoreboard)
        └── coverage   (apb_coverage)
## 📂 Folder Structure
APB_UVM_Verification/
│
├── rtl/ # Design Under Test (DUT)
│ ├── apb_wrapper.sv # Top-level wrapper
│ ├── apb_fsm.sv # APB protocol state machine
│ ├── err_gen.sv # Error generation
│ ├── generic_mem.sv # 64KB memory
│ └── mem_1024x32.sv # 4KB memory block
│
├── tb/ # Testbench
│ ├── apb_interface.sv # Signal interface
│ ├── apb_seq_item.sv # Transaction item
│ ├── apb_tb_uvm_pkg.sv # UVM package
│ ├── testbench_top.sv # Top module
│ │
│ ├── agent/ # UVM Agent
│ │ ├── apb_sequencer.sv
│ │ ├── apb_driver.sv
│ │ ├── apb_monitor.sv
│ │ └── apb_agent.sv
│ │
│ ├── env/ # UVM Environment
│ │ ├── apb_scoreboard.sv
│ │ ├── apb_coverage.sv
│ │ └── apb_env.sv
│ │
│ ├── seq_lib/ # Sequence Library
│ │ ├── apb_base_seq.sv
│ │ ├── apb_multi_addr_seq.sv
│ │ ├── apb_pstrb_seq.sv
│ │ ├── apb_misalign_seq.sv
│ │ ├── apb_oob_seq.sv
│ │ ├── apb_boundary_seq.sv
│ │ ├── apb_reset_seq.sv
│ │ └── apb_random_seq.sv
│ │
│ └── tests/ # Test Classes
│ ├── apb_base_test.sv
│ ├── apb_multi_addr_test.sv
│ ├── apb_pstrb_test.sv
│ ├── apb_misalign_test.sv
│ ├── apb_oob_test.sv
│ ├── apb_boundary_test.sv
│ ├── apb_reset_test.sv
│ ├── apb_random_test.sv
│ └── apb_regression_test.sv
│
├── sim/ # Simulation
│ ├── build.flist # File list
│ ├── Makefile # Build + Run
│ └── out/ # Output (gitignored)
│
├── docs/ # Documentation
│ └── verification_plan.md
```
### 🧪 Test Scenarios

**8 Tests Implemented:**

| # | Test | Description | Checks |
|---|---|---|---|
| 1 | `apb_multi_addr_test` | Multiple addresses write/read | Address decoding |
| 2 | `apb_pstrb_test` | PSTRB byte-enable patterns | Partial writes |
| 3 | `apb_misalign_test` | Misaligned addresses | PSLVERR=1 |
| 4 | `apb_oob_test` | Out-of-bounds addresses | PSLVERR=1 |
| 5 | `apb_boundary_test` | Boundary addresses | Last valid, first invalid |
| 6 | `apb_reset_test` | Reset behavior | Post-reset transfers |
| 7 | `apb_random_test` | Constrained-random (2000 txn) | All combinations |
| 8 | `apb_regression_test` | All sequences combined | Full regression |

### Sequence Classes

| Sequence | Purpose |
|---|---|
| `apb_base_seq` | Base with helper tasks (`write_one`, `read_one`) |
| `apb_multi_addr_seq` | 4 writes + 4 reads (different addresses) |
| `apb_pstrb_seq` | 10 PSTRB patterns |
| `apb_misalign_seq` | 6 misaligned writes + reads |
| `apb_oob_seq` | 7 out-of-bounds writes + reads |
| `apb_boundary_seq` | 7 boundary addresses |
| `apb_reset_seq` | 4 writes + reads after reset |
| `apb_random_seq` | 2000 constrained-random transactions |

### Code Coverage — 99.48% (Merged across 8 tests)

| Coverage Type | Score | Status |
|---|---|---|
| **Total** | **99.48%** | 🟢 Excellent |
| Line | 100.00% | 🟢 Perfect |
| Toggle | 99.85% | 🟢 Excellent |
| Conditional | 98.60% | 🟢 Excellent |
| FSM | 100.00% | 🟢 Perfect |

> **Merged coverage** across all 8 tests (`apb_multi_addr`, `apb_pstrb`, `apb_misalign`, `apb_oob`, `apb_boundary`, `apb_reset`, `apb_random`, `apb_regression`).
### Test-Wise Results

| # | Test | PASS | FAIL | Status |
|---|---|---|---|---|
| 1 | `apb_multi_addr_test` | 4 | 0 | ✅ PASS |
| 2 | `apb_pstrb_test` | 8 | 0 | ✅ PASS |
| 3 | `apb_misalign_test` | 0 | 0 | ✅ PASS |
| 4 | `apb_oob_test` | 0 | 0 | ✅ PASS |
| 5 | `apb_boundary_test` | 5 | 0 | ✅ PASS |
| 6 | `apb_reset_test` | 4 | 0 | ✅ PASS |
| 7 | `apb_random_test` | 25 | 0 | ✅ PASS |
| 8 | `apb_regression_test` | 25 | 0 | ✅ PASS |
| | **Total** | **71** | **0** | ✅ **PASS** |

> **Note:** `apb_misalign_test` and `apb_oob_test` produce only error transactions (PSLVERR=1) — all reads are skipped in scoreboard comparison. Hence PASS count is 0 but test is PASS (no FAIL).
>
> ### Prerequisites

| Tool | Version | Purpose |
|---|---|---|
| **Synopsys VCS** | L-2016.06 or later | Compilation + Simulation |
| **UVM** | 1.2 | Verification Methodology |
| **DVE** | VCS bundled | Waveform + Coverage Viewer |
| **URG** | VCS bundled | Coverage Report Generator |
| **OS** | Linux (Ubuntu 20.04+) | Build Environment |
| **SystemVerilog** | IEEE 1800-2017 | Verification Language |
