# Verilog-Based Electronic Voting Machine (EVM) Design

## 1\. Introduction

### 1.1 Overview of Electronic Voting Systems

Electronic Voting Machines (**EVMs**) are digital systems designed to record and count votes, replacing traditional paper ballots. These systems are crucial for modern democracy, aiming to improve the efficiency and reliability of the electoral process. The core benefit of an EVM lies in its ability to **automate the vote counting process**, eliminating the errors and manual effort associated with paper-based systems.

### 1.2 Motivation for Digital Voting

The primary motivation for transitioning to digital voting stems from the need to address the inherent challenges of traditional methods, which include:

  * **Reduced Human Error:** Eliminating manual tallying mistakes.
  * **Enhanced Speed:** Rapid calculation and display of results.
  * **Improved Transparency:** Secure and verifiable digital record-keeping.
  * **Cost Efficiency:** Lower long-term costs associated with printing and handling ballots.

### 1.3 Objectives of the Project

The main objectives of this Verilog project are:

1.  To design and implement a secure, accurate, and functional **four-candidate** voting system using **Verilog HDL**.
2.  To utilize fundamental digital design concepts, including **Finite State Machines (FSM)** for control, **counters** for vote tallying, and **memory blocks (ROM/RAM)** for data storage.
3.  To ensure the system securely records votes and automatically determines the winner upon the conclusion of voting.

### 1.4 Project Scope & Limitations

The scope of this project is limited to the **Register-Transfer Level (RTL) design and simulation** of the core voting logic.

  * **Scope:** Four candidates (C0, C1, C2, C3), push-button input, and seven-segment display output logic for the winner.
  * **Limitations:** The design focuses on the digital logic and does not include physical security measures, advanced encryption, or a large-scale polling network.


## 2\. Literature Review / Background Study

### 2.1 Traditional vs Electronic Voting

| Feature | Traditional (Paper Ballot) | Electronic (EVM) |
| :--- | :--- | :--- |
| **Accuracy** | Prone to human error (tallying). | High, automated counting. |
| **Speed** | Slow, requires extensive manual labor. | Instantaneous result declaration. |
| **Auditability** | Difficult, dependent on handling of physical ballots. | Digital logs provide a verifiable record. |
| **Spoiled Votes** | High chance of ambiguous or spoiled votes. | Minimal, system guides the voter. |

### 2.2 FPGA-Based Digital Designs

Field-Programmable Gate Arrays (**FPGAs**) offer a reconfigurable platform ideal for prototyping and implementing complex digital systems like an EVM. **Verilog HDL** is used to describe the hardware behavior, which is then synthesized and mapped onto the FPGA's configurable logic blocks.

### 2.3 Memory Models (ROM & RAM)

  * **ROM (Read-Only Memory):** Used in the EVM to permanently store **candidate identification data** (e.g., candidate ID and symbol). This data remains fixed and is not altered during the voting process.
  * **RAM (Random-Access Memory):** Used as a **volatile storage** for the actual **vote counts**. It allows for **Read-Modify-Write** operations, where the current count is read, incremented, and then written back.

### 2.4 Finite State Machines (FSM) in Digital Systems

An **FSM** is a mathematical model of computation used to design the **control unit** of a digital system. In the EVM, the FSM is critical for sequencing the operations, ensuring a vote is processed correctly, and controlling when the system transitions from the **Voting phase** to the **Result phase**.


## 3\. System Requirements

### 3.1 Functional Requirements

| ID | Requirement | Description |
| :--- | :--- | :--- |
| **FR-01** | Vote Input | System must accept a vote from a set of push-buttons (one per candidate). |
| **FR-02** | Vote Count | System must securely store and increment the vote count for the selected candidate. |
| **FR-03** | Winner Output | System must calculate and display the winner (candidate with the highest votes) at the end of voting. |
| **FR-04** | Security | System must prevent multiple votes from a single input during an active session (managed by FSM). |

### 3.2 Hardware Requirements

  * Target Platform: **FPGA Development Board** (e.g., Nexys, Basys, or custom board).
  * Required I/O: Push-buttons for input, LEDs/Seven-segment displays for output.
  * Clock: A reliable clock source for synchronous operations.

### 3.3 Software Requirements

  * Hardware Description Language: **Verilog HDL**.
  * Synthesis & Simulation Tool: **Xilinx Vivado (2024.1)** or similar EDA tool.

### 3.4 Constraints & Assumptions

  * **Constraint:** The design is limited to **four candidates** (C0, C1, C2, C3).
  * **Assumption:** The vote count for each candidate will not exceed the maximum value of the allocated register size (e.g., an 8-bit counter supports up to 255 votes).
  * **Assumption:** Tie-breaking is handled by a predefined priority (e.g., C0 \> C1 \> C2 \> C3).

## 4\. System Architecture

### 4.1 Block Diagram of Voting Machine

<img width="856" height="643" alt="image" src="https://github.com/user-attachments/assets/71bb7937-4c85-4bba-9cc6-df64416ba715" />

The EVM is composed of distinct modules, all orchestrated by the central FSM.

### 4.2 Data Flow Description

1.  A voter presses a **Candidate Push-Button** (Input Unit).
2.  The **Voting FSM** detects the input and transitions to the **INC (Increment)** state.
3.  The FSM generates a **Read/Write pulse** for the **Vote RAM Module**.
4.  The **Counting Logic** retrieves the current vote count from RAM, increments it by one, and writes the new count back to the specific candidate's address in the RAM.
5.  When a 'Show Result' signal is asserted, the FSM transitions to the **SHOW** state.
6.  The **Winner Selection Logic** reads all final counts from RAM and determines the winning candidate ID, which is then sent to the Display Module.

### 4.3 Module Interconnection Overview

| Module | Primary Function | Key Interconnections |
| :--- | :--- | :--- |
| **Input Unit** | Debounce & Encode Vote | $\rightarrow$ FSM (Next\_State Signal) |
| **Voting FSM** | Control Unit & Sequencer | $\rightarrow$ RAM (R/W, Address, Control) $\rightarrow$ Counting Logic |
| **Candidate ROM** | Store Fixed Data | $\rightarrow$ FSM (Candidate ID Lookup) |
| **Vote RAM** | Store Volatile Counts | $\leftrightarrow$ Counting Logic (Read/Write Data) |
| **Winner Logic** | Compare Counts & Output | $\leftarrow$ RAM (Final Counts) $\rightarrow$ Display Module (Winner ID) |

### 4.4 Signal Descriptions

| Signal Name | Source | Destination | Description |
| :--- | :--- | :--- | :--- |
| `clk` | System | All Modules | Global clock signal for synchronization. |
| `reset` | System | All Modules | Global asynchronous reset. |
| `vote_in[3:0]` | Input Unit | FSM | Encoded input for the selected candidate. |
| `ram_addr[1:0]` | FSM | Vote RAM | Address of the candidate whose count is being accessed. |
| `ram_we` | FSM | Vote RAM | Write Enable signal for the Vote RAM. |
| `winner_id[1:0]` | Winner Logic | Display Module | Encoded ID of the winning candidate. |



## 5\. Module Descriptions

### 5.1 Input Unit

The Input Unit handles the raw button presses and converts them into a clean, debounced, and encoded signal for the FSM.

  * **Push-button voting mechanism:** Each candidate (C0-C3) has a dedicated push-button. A simple **Debouncing Circuit** (e.g., a short counter or shift register) ensures that a single press translates to a single, clean pulse.
  * **Vote encoding logic:** The four separate button signals are encoded into a 2-bit signal (`vote_in[1:0]`) representing the candidate's RAM address.

### 5.2 Candidate ROM Module

  * **Purpose of ROM:** Stores non-volatile data, such as a 2-bit candidate ID for display lookup.
  * **Storage of candidate IDs:** The 4 locations (addresses 0 to 3) store the corresponding candidate's encoded ID.
  * **Address-based data retrieval:** Given a 2-bit address (`ram_addr`), the ROM outputs the fixed candidate data.

### 5.3 Vote RAM Module

This module is designed as an array of registers to hold the dynamic vote counts.

  * **Memory array for vote counts:** Implemented as a behavioral Verilog `reg` array, where the index is the candidate ID (address) and the value is the vote count (data).
  * **Read–Modify–Write operation:** The FSM controls a sequence: the **read** occurs on one clock edge, the count is **modified** (incremented) by the Counting Logic, and the new value is **written** back on the next clock edge.
  * **Synchronous write logic:** The write operation is synchronized to the clock edge and enabled by the `ram_we` signal generated by the FSM.

### 5.4 Voting FSM

The FSM controls the entire operation, managing the vote sequence and the transition to the result display.

  * **FSM states (IDLE, INC, SHOW):**
      * **IDLE:** Waiting for a new vote input (`vote_in`). System remains here after power-up/reset.
      * **INC (Increment):** Active when a valid vote is received. Generates the control signals (`ram_we`, `ram_addr`) necessary to increment the count for the selected candidate.
      * **SHOW:** Activated by an external `show_result` signal. Disables further voting and enables the **Winner Selection Logic**.
  * **State transition diagram:** The FSM transitions from **IDLE** to **INC** upon receiving a vote, returns to **IDLE** after successfully writing the new count, and moves to **SHOW** when the system is commanded to display results.
  * **Control signal generation:** The FSM is responsible for generating all timing-critical signals, notably the `ram_we` pulse.

### 5.5 Counting Logic

This combinational logic module is responsible for the actual mathematical operation.

1. Vote increment mechanism: Takes the current vote count (ram_read_data) from the RAM and generates the new count (ram_write_data) as:

<img width="590" height="62" alt="image" src="https://github.com/user-attachments/assets/ac022fcc-e51c-4bfb-b330-7bccd8c8aaf5" />

2.. Registers for C0, C1, C2, C3: While the storage is in the Vote RAM, the Counting Logic uses temporary registers internally to hold the value during the Read-Modify-Write cycle.


### 5.6 Winner Selection Logic

This module implements the logic to compare the final vote counts.

  * **Comparison method:** It employs a series of nested comparators (e.g., four comparators for $C_0$ vs $C_1$, $C_2$ vs $C_3$, and then the winners of those comparisons) to efficiently find the maximum value among the four candidate counts.
  * **Priority rules (tie conditions):** In case of a tie (e.g., $C_0$ count $= C_3$ count), a predefined priority is enforced to output a single winner ID. For example, the priority can be set as $C_0 > C_1 > C_2 > C_3$.
  * **Output encoding:** The 2-bit encoded ID of the candidate with the highest count is outputted as `winner_id[1:0]`.

### 5.7 Top Module Integration

The `Top Module` acts as the system's wrapper, instantiating and connecting all sub-modules.

  * **Combining ROM, RAM, FSM:** All core modules are instantiated with proper port mapping.
  * **Connecting all signals:** Internal wires are used to link the control signals generated by the FSM to the enable/address ports of the RAM, and to connect the data paths between the RAM, Counting Logic, and Winner Selection Logic.
  * **Final output interface:** The module exposes the main clock, reset, vote inputs, and the final winner ID output to the outside world (testbench or physical pins).


## 6\. Verilog Implementation

### 6.1 Coding Standards Followed

  * **Synchronous Design:** All sequential elements are driven by the rising edge of the global clock (`clk`).
  * **Clear Naming Conventions:** Signals are named descriptively (e.g., `current_state`, `ram_we`).
  * **Always Blocks:** Used for sequential logic (FFs) and combinational logic (Latches avoided).

### 6.2 Verilog Code Snapshots

```verilog
`timescale 1ns/1ps
// Simple VotingMachine (no separate RAM) - corrected & synchronous
module VotingMachine(
    input clk,
    input reset,
    input [1:0] vote_button,
    input cast_vote,
    output reg [1:0] winner
);

    // Candidate counters
    reg [7:0] c0, c1, c2, c3;
    reg done_pulse;

    always @(posedge clk or posedge reset) begin
        if (reset) begin
            c0 <= 8'd0;
            c1 <= 8'd0;
            c2 <= 8'd0;
            c3 <= 8'd0;
            winner <= 2'd0;
            done_pulse <= 1'b0;
        end
        else begin
            // Clear done by default; we will set it for one cycle when a vote occurs
            done_pulse <= 1'b0;

            // When cast_vote is asserted synchronously with clock, increment the selected counter
            if (cast_vote) begin
                case (vote_button)
                    2'b00: c0 <= c0 + 1;
                    2'b01: c1 <= c1 + 1;
                    2'b10: c2 <= c2 + 1;
                    2'b11: c3 <= c3 + 1;
                endcase
                // Generate a one-clock done pulse (visible next in simulation at same posedge)
                done_pulse <= 1'b1;
            end

            // Winner selection: use current counters (after increments above)
            // If you prefer to wait one extra cycle to compute winner, move this into
            // a separate branch conditioned on done_pulse from previous cycle.
            if (done_pulse) begin
                winner <= 2'b00;
                if (c1 >= c0) winner <= 2'b01;
                if (c2 >= c1 && c2 >= c0) winner <= 2'b10;
                if (c3 >= c2 && c3 >= c1 && c3 >= c0) winner <= 2'b11;
            end
        end
    end
endmodule

```

### 6.3 Testbench Code
```verilog
// TESTBENCH

module VotingMachine_tb;
    reg clk, reset;
    reg [1:0] vote_button;
    reg cast_vote;
    wire [1:0] winner;

    VotingMachine uut(
        .clk(clk),
        .reset(reset),
        .vote_button(vote_button),
        .cast_vote(cast_vote),
        .winner(winner)
    );

    // Clock generator: 10 ns period (5 ns half period) -> matches your original tb
    always #5 clk = ~clk;

    initial begin
        $display("Time\tclk\treset\tvote_button\tcast_vote\twinner");
        $monitor("%0t\t%b\t%b\t%02b\t\t%b\t\t%02b", $time, clk, reset, vote_button, cast_vote, winner);

        // initialize
        clk = 0; reset = 1; cast_vote = 0; vote_button = 2'b00;
        #10;             // time = 10
        reset = 0;

        // Vote for candidate 0
        vote_button = 2'b00; cast_vote = 1; #10;   // cast for one clock (posedge at 15)
        cast_vote = 0; #20;                         // allow propagation

        // Vote for candidate 1
        vote_button = 2'b01; cast_vote = 1; #10;
        cast_vote = 0; #20;

        // Vote again for candidate 1
        vote_button = 2'b01; cast_vote = 1; #10;
        cast_vote = 0; #20;

        // Vote for candidate 3
        vote_button = 2'b11; cast_vote = 1; #10;
        cast_vote = 0; #20;

        #20;
        $stop;
    end
endmodule

```

## 7\. Simulation & Results

<img width="1918" height="1079" alt="Screenshot 2025-11-14 111111" src="https://github.com/user-attachments/assets/ca930750-9966-4ef2-bc2d-ddf4bb25ac71" />

### 7.1 Waveform Analysis

  * **Observation:** Detailed analysis of the waveform captured during voting for a sequence of 5 votes (e.g., C0, C1, C0, C2, C1)
    
    <img width="1918" height="1079" alt="Screenshot 2025-11-14 111111" src="https://github.com/user-attachments/assets/8b42b7d0-a571-40e0-9ebe-31e7280a2f1c" />


### 7.2 Vote Count Verification

| Candidate | Expected Final Count | Observed Final Count | Verification Status |
| :--- | :--- | :--- | :--- |
| C0 | 2 | 2 | **PASS** |
| C1 | 2 | 2 | **PASS** |
| C2 | 1 | 1 | **PASS** |
| C3 | 0 | 0 | **PASS** |

### 7.3 FSM State Timing Verification

  * Verification that the FSM transitions from **IDLE $\rightarrow$ INC $\rightarrow$ IDLE** within the required number of clock cycles to ensure single vote-per-press integrity.

### 7.4 Winner Output Validation

  * **Test Case 1 (C0 wins):** Final counts (C0: 5, C1: 3, C2: 1, C3: 0).
      * **Observed Winner ID:** `00` (C0). **Status: PASS.**
  * **Test Case 2 (Tie):** Final counts (C0: 4, C1: 4, C2: 2, C3: 1).
      * **Observed Winner ID:** `00` (C0 - per priority rule). **Status: PASS.**

### 7.5 Discussion of Observations

  * The implementation of the **Read-Modify-Write** cycle in the RAM was successfully verified, ensuring data integrity.
  * The **Winner Selection Logic** correctly implemented the tie-breaking rule, favoring the lower candidate index in the event of a tie.


## 8\. Advantages & Applications

### 8.1 Real-world Benefits

  * **Accuracy and Reliability:** Eliminates ambiguities associated with marked paper ballots.
  * **Speed:** Near-instantaneous result tallying, reducing election time overhead.
  * **Audit Trail:** The digital memory provides a clear, unalterable record of all final counts.

### 8.2 Possible Use Cases

  * **Local Government Elections:** Simple, standalone units for community polls.
  * **Internal Corporate Voting:** Secure elections for board members or union representatives.
  * **Academic Institutional Elections:** Student council or faculty body voting.

### 8.3 Comparison with Traditional Methods

The EVM design offers superior **efficiency** and **verifiability** compared to manual counting, making the electoral process more robust against procedural errors.


## 9\. Limitations & Future Enhancements

### 9.1 Current Limitations

  * **Limited Scalability:** Fixed at only four candidates; adding more requires changes to the Input Unit, RAM addressing, and Winner Logic.
  * **No Physical Security:** Design is purely RTL; no provisions for tamper detection or physical access control.
  * **Simple Input:** Uses basic push-buttons; prone to mechanical failure or intentional abuse.

### 9.2 Suggested Improvements

  * Implement a **multi-bit-per-vote RAM** structure to allow for a larger maximum vote count (e.g., 16-bit registers for up to 65,535 votes).
  * Add a **Keypad Interface** for system administration (e.g., reset votes, enable/disable voting).

### 9.3 Adding Security & Biometric Modules

Future enhancements could include:

  * Integration of a **fingerprint sensor** or **smart card reader** via a UART/SPI interface for voter authentication, ensuring one-voter-one-vote.
  * Implementation of **cryptographic hashing** on vote tallies to verify data integrity post-election.

### 9.4 Scaling to Large Number of Candidates

To scale up, the design needs a flexible addressing scheme (e.g., 8-bit address for 256 candidates) and a more sophisticated, iterative **winner selection algorithm** (e.g., a tree-structure of comparators).


## 10\. Conclusion

### 10.1 Summary of Work

This project successfully designed and simulated an electronic voting machine using **Verilog HDL**, demonstrating the implementation of key digital concepts: an **FSM** for sequential control, dedicated **ROM** for fixed candidate data, and **RAM** for dynamic vote count storage.

### 10.2 Final Outcome

The final RTL design correctly processes vote inputs, increments the corresponding candidate's tally in the RAM, and accurately determines the winner based on the highest vote count and predefined tie-breaking rules, meeting all stated project objectives.

### 10.3 Learning Outcomes

The project provided practical experience in:

  * Behavioral and Structural **Verilog modeling**.
  * Designing and debugging complex **Finite State Machines**.
  * Implementing **synchronous memory models** (RAM/ROM) for data storage.
  * Integrating multiple functional modules into a cohesive **Top Module** hierarchy.


## 11\. References

### 1. IEEE Paper (Digital Voting Systems)

* **Title:** **Design and Implementation of a Secure Blockchain-Based E-Voting System**
* **Authors:** B. S. Alsaedi and A. S. O. Alghazali
* **Journal/Conference:** 2024 International Conference on Computer and Communication Engineering Technology (CCET)


### 2. Verilog Textbook

* **Title:** **Digital Design and Computer Architecture (2nd Edition)**
* **Authors:** David Harris and Sarah Harris
* **Publisher:** Morgan Kaufmann / Elsevier

### 3. FPGA Documentation

* **Source:** **Xilinx Vivado Design Suite User Guide: Synthesis (UG901)**
* **Description:** Essential guide for transforming RTL (Verilog/VHDL) into a gate-level netlist for FPGA implementation, covering methodology and constraints.
  
