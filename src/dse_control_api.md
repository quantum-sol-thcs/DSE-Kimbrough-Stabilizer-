# ⚙️ DSE Control API: System Interface Definition

This document defines the high-level control interface for the Deterministic State Evolution (DSE) stabilizer loop. The API manages the flow of fixed-point data between the quantum measurement unit and the dedicated **Pipelined Goldschmidt Core (Claim 2)**.

## I. Control Loop Cycle

The DSE operates on a rapid, fixed-latency cycle ($\mathbf{\tau_{calc} < 40 \, ns}$) to correct quantum errors deterministically before decoherence can dominate.

### 1. Input Interface (`DSE_IN`)

The input is the fixed-point representation of the measured state fidelity ($x$).

| Signal | Data Type | Precision | Description |
| :--- | :--- | :--- | :--- |
| **`x_in_fidelity`** | Fixed-Point | 26-bit | Measured observable fidelity of the quantum state ($x_n$). This value is fed directly into the Harmonic Map Function Unit. |
| **`Reset_n`** | Boolean | 1-bit | Active-low reset signal for pipeline initialization. |
| **`Clock_500MHz`** | Clock | 1-bit | The high-frequency clock source driving the Goldschmidt Core. |

### 2. Output Interface (`DSE_OUT`)

The output is the mathematically guaranteed control signal ($f(x)$) required to drive the state back to the fixed point ($x^* = 1/\Phi$).

| Signal | Data Type | Precision | Description |
| :--- | :--- | :--- | :--- |
| **`U_corr_Phase`** | Fixed-Point | 14-bit | The **phase component** of the corrective control pulse, derived from $f(x)$. |
| **`U_corr_Amplitude`** | Fixed-Point | 8-bit | The **amplitude component** of the corrective control pulse, derived from $f(x)$. |
| **`Correction_Enable`** | Boolean | 1-bit | Active-high signal indicating that the output data is stable and ready for actuation ($\sim 20 \, ns$ after input). |

## II. Function Definition

The core functionality of the API is enforced by the **Harmonic Control Map, $f(x)$**, which implements the **Kimbrough Contraction Principle (Claim 1)**:

$$f(x) = \frac{x^*}{x}$$

Where the fixed point is $x^* = 1/\Phi \approx 0.618034$. The Goldschmidt Core (Claim 2) performs this division using rapid iterative multiplication.
