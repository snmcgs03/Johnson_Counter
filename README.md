# Johnson Counter - SystemVerilog Layered Testbench

![Status](https://img.shields.io/badge/Verification-SystemVerilog-blue) ![Type](https://img.shields.io/badge/Project-Layered%20Testbench-green)

## 📖 Overview
This repository contains a **SystemVerilog Layered Testbench** designed to verify a **4-bit Johnson Counter**. The project demonstrates a modular verification environment using Object-Oriented Programming (OOP) principles. It separates the testbench into functional blocks like Generator, Driver, Monitor, and Scoreboard to ensure scalability and reusability.

## 📂 File Structure

| File Name | Component | Description |
| :--- | :--- | :--- |
| **`design.sv`** | **DUT** | The 4-bit Johnson Counter RTL design. It implements twisted-ring logic. |
| **`top.sv`** | **Top Module** | The top-level hierarchy connecting the DUT and Testbench via the Interface. Generates the clock. |
| **`interface.sv`** | **Interface** | Bundles signals (`clk`, `reset`, `out`) and defines directions using `modports`. |
| **`environment.sv`**| **Environment** | Container class that builds and connects the Generator, Driver, Monitor, and Scoreboard. |
| **`driver.sv`** | **Driver** | Drives stimulus (reset) to the DUT. |
| **`monitor.sv`** | **Monitor** | Passive component that samples the DUT output (`out`) and signals the Scoreboard. |
| **`scoreboard.sv`** | **Scoreboard** | Contains the **Reference Model** and compares *Expected* vs. *Actual* values. |
| **`generator.sv`** | **Generator** | Orchestrates the simulation flow by triggering the driver sequences. |

---

## 🧠 Design Choices

In this project, I made specific architectural decisions to optimize the testbench for the Johnson Counter's specific requirements:

### Transaction Class Omission
I intentionally omitted a standard `Transaction` class because the Johnson Counter is a sequential circuit that does not accept variable data inputs (like addresses or payloads). It relies solely on control signals (`clk` and `reset`).

Since the design initializes to a fixed deterministic state of `4'b0001` upon reset and shifts automatically, there is no data to randomize or constrain. Therefore, creating a transaction object just to hold a single "reset" bit would add unnecessary overhead. Instead, the **Generator** directly triggers the control sequences in the **Driver**.

### Integrated Reference Model
I integrated the **Reference Model** (Golden Model) directly into the **`scoreboard.sv`** class rather than creating a separate file. This keeps the environment lightweight while maintaining full verification capability.

* **Logic:** The scoreboard uses a local variable, `expected_value`, to mimic the hardware behavior.
* **Prediction:** On every clock cycle, the scoreboard calculates the next expected state using the twisted-ring logic `{~expected_value[0], expected_value[3:1]}`.
* **Verification:** This calculated value is immediately compared against the `observed_value` received from the Monitor to generate a Match/Mismatch report.

---

## ⚙️ How to Run
1.  Compile all SystemVerilog files:
    ```bash
    xrun -sv -access +rwc +gui top.sv interface.sv design.sv environment.sv driver.sv monitor.sv scoreboard.sv generator.sv 
    ```

## 📊 Sample Output
```text
Driver: Driving reset at time 0
Driver: Reset deasserted at time 15
Scoreboard: Reset expected value to 0001 at time 15
Scoreboard: Observed = 0001 matches Expected = 0001
Scoreboard: Observed = 1000 matches Expected = 1000
Scoreboard: Updated expected value to 1000 at time 25
...
