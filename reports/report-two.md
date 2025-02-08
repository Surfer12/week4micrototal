# Lab Two: Building with Gates  

**Due Date:** Monday, 2/3  
**Week 3 (23, 25) – CIS 240 Micro Arch & Prog (31563)**  

---

## 1. Introduction

### 1.1 Objective
>
> “Design, simulate, and validate a single‑bit adder with Carry‑In = 0 using basic logic gates in LTSpice. Then, optimize the LSB block by eliminating the Carry‑In input to create a streamlined half‑adder.”

### 1.2 Overview

- **Part 1:** Implementation of a half‑adder (using Carry‑In = 0).
- **Part 2:** Optimization of the LSB block by removing the Carry‑In.
- Verification involves truth tables, Boolean SOP derivations, LTSpice simulations, and integration tests.

---

## 2. Part 1: Main Block Design (Carry‑In = 0)

### 2.1 Understanding the Problem

- **Task:** Add two input bits, A[0] and B[0], with no incoming carry.
- **Result:** The circuit functions as a half‑adder.

### 2.2 Create Truth Tables

#### 2.2.1 SUM Truth Table

| A[0] | B[0] | Sum[0] | Explanation         |
|------|------|--------|---------------------|
| 0    | 0    | 0      | 0 + 0 = 0           |
| 0    | 1    | 1      | 0 + 1 = 1           |
| 1    | 0    | 1      | 1 + 0 = 1           |
| 1    | 1    | 0      | 1 + 1 = 2 → 0 (carry 1)|

#### 2.2.2 CARRY‑OUT Truth Table

| A[0] | B[0] | Carry_Out | Explanation                         |
|------|------|-----------|-------------------------------------|
| 0    | 0    | 0         | No carry generated                  |
| 0    | 1    | 0         |                                     |
| 1    | 0    | 0         |                                     |
| 1    | 1    | 1         | 1 + 1 = 2 → carry generated         |

**Block Function:**  
The block computes:
\[
\text{Sum}[0] = A[0] \oplus B[0] \quad \text{and} \quad \text{Carry\_Out} = A[0] \cdot B[0]
\]

### 2.3 Derive the Logic (SOP Method)

- **Sum[0]:**
  \[
  \text{Sum}[0] = \overline{A[0]}\,B[0] + A[0]\,\overline{B[0]}
  \]
- **Carry_Out:**
  \[
  \text{Carry\_Out} = A[0] \cdot B[0]
  \]

### 2.4 Circuit Implementation Steps

- **LTSpice Entry:** Draw and simulate the half‑adder.
- **Symbol Creation:** Design and store the relevant symbol.
- **Test Schematic:** Set up sources, probes, and simulation to check all logic levels.
- **Backup:** Archive `.asc` files, `.asy` symbols, and simulation screenshots.

---

## 3. Part 2: LSB Block Optimization (No Carry‑In Input)

### 3.1 Schematic Layout

- **Inputs:** `A[0]` and `B[0]`.
- **XOR-Like Sum:**  
  Create the XOR function by using:
  - Two NAND gates with inverters (one for (A, ¬B) and one for (¬A, B)).
  - Combine these outputs (via an OR or NAND‑invert stage) to generate `Sum[0]`.

- **Carry Logic:**  
  Implement using an AND gate:  
  \[
  \text{Carry\_Out} = A[0] \cdot B[0]
  \]

### 3.2 Why Use NAND + Inverters?

- **Universality:** NAND gates can build any Boolean function.
- **Efficiency:** Removing the Carry‑In simplifies the design while using inverters to form the XOR function.

### 3.3 Symbol Creation in LTSpice

1. Open the symbol editor.
2. Sketch the symbol and add pins: `A[0]`, `B[0]`, `Sum[0]`, and `Carry_Out`.
3. Save as a `.asy` file.

### 3.4 Top-Level Test Schematic & Simulation

- Place the new optimized symbol.
- Drive test patterns (00, 01, 10, 11) using pulse/voltage sources.
- Monitor outputs with waveform probes to ensure:
  - `Sum[0]` toggles only when inputs differ.
  - `Carry_Out` is high only when both inputs are 1.

### 3.5 Integration

- Remove the original LSB block.
- Replace it with the newly optimized block.
- Re-run simulations to validate overall performance.

---

## 4. Discussion

- Compare the original half‑adder design with the optimized LSB design.
- Discuss challenges (e.g., achieving XOR functionality using NAND and inverters).
- Address how these blocks can be cascaded in larger adders.

## 5. Conclusion

- Both designs yield the correct half‑adder behavior.
- The optimized LSB (without Carry‑In) simplifies the circuit while maintaining output correctness.

## 6. Next Steps

- Extend the design to a full 4‑bit or n‑bit adder.
- Optimize further based on timing, power consumption, and layout considerations.

## 7. Appendix

- **Truth Tables & SOP Derivations:** Refer to Sections 2.2 and 2.3.
- **LTSpice Files:** Include `.asc` and `.asy` files plus simulation screenshots.
- **References:** List textbooks, lecture slides, and online resources.

---
