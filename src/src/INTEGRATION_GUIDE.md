# 🛠️ DSE Source Code Integration Guide

This document lists the proprietary source code files that must be integrated into the final build but **must NOT be committed** to this public repository. These files contain the patented implementation details of the DSE-Kimbrough-Stabilizer.

## ⚠️ Proprietary Code Warning

The code related to the Kimbrough Contraction Principle and the Goldschmidt Core are protected under active patent filings (Claims 1, 2, 3, and 4). **Do not commit final synthesis or simulation code.**

---

## I. Core Goldschmidt Apparatus (Claim 2)

* **Location:** VHDL implementation of the Goldschmidt Pipelined Core.
* **Purpose:** Enforces the $\mathbf{\tau_{calc} \ll 100 \, ns}$ latency constraint.

| File Path | Description |
| :--- | :--- |
| `vhdl/kimbrough_goldschmidt_core.vhd` | Main 5-stage pipeline logic. |
| `vhdl/kimbrough_fixed_point_scaler.vhd` | Proprietary Normalization & Scaling unit. |
| `vhdl/kimbrough_phase_amplitude_mux.vhd` | Final correction signal generation. |

## II. Harmonic Control Map (Claim 1)

* **Location:** Python model for rapid co-simulation and verification.
* **Purpose:** Implements the core control function $f(x) = x^*/x$ using $\Phi$.

| File Path | Description |
| :--- | :--- |
| `python/kimbrough_invariant_model.py` | Implements $f(x)$ and defines the fixed point $x^*=1/\Phi$. |
| `python/qfsi_risk_model.py` | Generates QFSI (5.77) and risk multipliers. |

## III. Testbenches and Simulation (Claim 3)

* **Location:** Verification environments.
* **Purpose:** Must verify the guaranteed $\mathbf{85.4\%}$ error reduction rate.

| File Path | Description |
| :--- | :--- |
| `test/goldschmidt_timing_tb.vhd` | Verifies $\mathbf{\tau_{calc} < 40 \, ns}$ constraint. |
| `test/exponential_convergence_tester.py` | Generates data for `stability_visualization.png`. |
