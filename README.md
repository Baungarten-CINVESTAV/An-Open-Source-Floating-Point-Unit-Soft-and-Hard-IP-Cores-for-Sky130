# Open-Source IEEE 754 Floating-Point Unit for Sky130

A 32-bit Verilog floating-point unit (FPU) supporting IEEE 754 single-precision addition, subtraction, multiplication, and division. The repository includes synthesizable RTL and Cadence Genus static-timing-analysis artifacts for Sky130.

## Repository layout

```text
rtl/
  FPU.v                  Top-level FPU interface
  add_sub_main.v         Addition/subtraction datapath
  mul_div.v              Multiplication/division datapath
  exception_block.v      IEEE exception and special-value handling
  normalize_rounder.v    Normalization and rounding logic
  *_logic.v              Sign and exponent logic
  *_divider.v            Division and normalization modules

static_timing_analysis/
  run.tcl                Genus synthesis script
  rtl_list               Ordered RTL source list
  constraints.sdc        Clock and timing constraints
  sky130_fd_sc_hd.lib    Sky130 timing library
  results/
    netlist_mapped.v     Synthesized Verilog netlist
    qor.txt              Quality-of-results report
    timing_max.txt       Maximum-delay report
    area.txt             Area report
    gates.txt            Gate statistics
```

## Features

- 32-bit IEEE 754 single-precision operands and result
- Addition and subtraction
- Multiplication and division
- Guard, round, and sticky rounding information
- Overflow, underflow, inexact, invalid, and divide-by-zero flags
- Handling for zero, infinity, and NaN operands
- Sky130 synthesis and static timing scripts
- 23-stage pipelining between the arithmetic datapaths and top-level output

## Interface

```verilog
module FPU(
    input  [31:0] A,
    input  [31:0] B,
    input  [1:0]  op,
    input         clk,
    input         arst,
    input         en,
    output [4:0]  flags,
    output [31:0] Z
);
```

### Operation encoding

| `op` | Operation |
|---|---|
| `00` | Floating-point subtraction: `A - B` |
| `01` | Floating-point addition: `A + B` |
| `10` | Floating-point multiplication: `A × B` |
| `11` | Floating-point division: `A ÷ B` |

### Flags

| Bit | Flag |
|---|---|
| `0` | Invalid operation |
| `1` | Divide by zero |
| `2` | Overflow |
| `3` | Underflow |
| `4` | Inexact |

The exception block overrides the normal arithmetic result for special operands or exceptional cases. It generates a canonical quiet NaN for invalid operations.

## Architecture

`FPU.v` selects between the parallel add/subtract and multiply/divide datapaths. Its delayed operation signal keeps the selected result aligned with the arithmetic pipeline. The exception block uses a delayed copy of `A`, `B`, and the operation code to provide special-value results.

The add/subtract path performs sign selection, exponent adjustment, mantissa alignment, mantissa arithmetic, normalization, and rounding. The multiply/divide path computes the exponent and mantissa, normalizes the result, and emits guard, round, and sticky bits.

## Synthesis

The `static_timing_analysis` directory contains a Cadence Genus flow for synthesis and timing analysis. Run it from that directory:

```bash
cd static_timing_analysis
genus -f run.tcl
```

Before running, update the absolute library path in `run.tcl`:

```tcl
/home/chuy/FPU_LAEDC/static_timing_analysis/sky130_fd_sc_hd__ff_100C_1v65.lib
```

The script expects the RTL sources to be accessible through `rtl_list`, reads `constraints.sdc`, and produces synthesis reports and a mapped netlist.

### Reported synthesis result

The checked-in Genus report for the `FPU` module shows:

- 12,695 leaf instances
- 4,965 sequential instances
- 7,730 combinational instances
- 176,694.462 total reported cell area
- 10.8 ps critical-path slack
- No timing violations

These values describe the existing synthesis result and are not a place-and-route timing guarantee.

## RTL compilation example

The RTL can be compiled with Icarus Verilog:

```bash
iverilog -g2012 -o fpu_sim rtl/*.v
```

No testbench or automated simulation suite is currently included.

## Design status

This repository provides the FPU RTL and an initial synthesis result. Before production use, validate:

- IEEE 754 rounding and subnormal behavior
- Pipeline-cycle alignment under `en` and `arst`
- Exception timing and flag generation
- Formal and randomized test coverage
- Physical implementation and post-place-and-route timing

## License

No license file is currently included. Add a license before redistributing or integrating the IP into other projects.

## Citation

If you use this work in academic research, cite the repository and its contributing authors:

- Daniel
- Lino
- Kevin
- Emmanuel
- CINVESTAV
