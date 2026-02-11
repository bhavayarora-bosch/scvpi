# SCVPI

SCVPI is a VPI-compatible interface for SystemC that enables **cocotb**
and cocotb-based frameworks such as **PyUVM** to run on SystemC models.

By providing a VPI-compatible layer, SCVPI allows Python-based
verification environments to be reused across both RTL and SystemC,
eliminating duplicated verification infrastructure and enabling a
unified verification flow.

---

## Motivation

Python-based verification frameworks such as cocotb and PyUVM are widely
used for RTL verification due to their productivity and reusability.
However, these frameworks rely on the Verilog Procedural Interface (VPI)
and therefore cannot be applied directly to SystemC models.

SystemC is commonly used for high-level modeling and virtual prototyping.
This separation often leads to duplicated testbenches and inconsistent
verification behavior between RTL and high-level models.

SCVPI bridges this gap by exposing a VPI-compatible interface for
SystemC, allowing cocotb and PyUVM testbenches to execute on SystemC
models without modification.

---

## What SCVPI Provides

- A VPI-compatible interface for SystemC
- Coroutine scheduling support required by cocotb
- Signal access and callback mechanisms required by PyUVM
- Reuse of Python testbenches without modification
- A unified verification flow across RTL and SystemC models

---

## Supported SystemC Signal Types

SCVPI currently supports direct value access for a subset of SystemC
signal types, including:

- `sc_signal<bool>`
- `sc_signal<int>` (treated as 32-bit)
- `sc_signal<sc_int<4>>`
- `sc_signal<sc_uint<3>>`
- `sc_signal<sc_uint<8>>`
- `sc_signal<sc_uint<16>>`

Signals with unsupported types are reported as unknown.

---

## Examples

The repository includes several examples demonstrating different usage
scenarios:

- **examples/basic**  
  Minimal sanity check demonstrating cocotb coroutine scheduling on a
  SystemC model.

- **examples/adder**  
  A simple functional example with deterministic and randomized tests.

- **examples/TinyALU_SystemC**  
  A SystemC implementation of the official PyUVM TinyALU example.
  The original PyUVM testbench is reused unchanged, demonstrating
  verification reuse between RTL and SystemC using SCVPI.

These examples serve as reference implementations for integrating SCVPI
with SystemC models and reusing Python-based verification environments.

---

## Scope and Limitations

SCVPI is intended for verification of high-level SystemC models.
It does not aim to replace RTL simulators or provide full Verilog
compatibility. Performance characteristics may differ from RTL-based
verification flows.

---

## License

This project is released under the applicable open-source license.
Third-party examples and testbenches retain their original licenses.