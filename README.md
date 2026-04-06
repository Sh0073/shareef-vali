## System Architecture

The proposed system implements an **Adaptive Fault-Tolerant RISC-V Processor** that integrates Triple Modular Redundancy (TMR), multi-dimensional Hamming code (MDMHC)-based error correction, and dynamic recovery mechanisms.

The architecture is designed to ensure high reliability while optimizing resource utilization through adaptive execution modes.
<img width="761" height="601" alt="architecure " src="https://github.com/user-attachments/assets/7354a095-192c-4445-8e9e-a2b57a599123" />

### 1. TMR Computation Core

The processor consists of three identical RISC-V cores:

- Core A  
- Core B  
- Core C  

All three cores execute the same instructions in parallel.

A **fault injection unit** introduces controlled errors into selected cores to emulate real-world fault conditions such as transient and permanent faults.

---

### 2. Majority Voter

The outputs of the three cores are fed into a **Majority Voter**, which:

- Compares outputs from all cores  
- Detects mismatches  
- Selects the correct output using majority logic  

This enables masking of single-core faults.

---

### 3. Adaptive Controller

The **Adaptive Controller** dynamically changes the operating mode based on system health:

| Healthy Cores | Mode        |
|--------------|------------|
| 3            | TMR Mode   |
| 2            | Dual Mode  |
| 1            | Single Mode|

This reduces unnecessary power and area overhead during fault-free operation.

---

### 4. Reliability Monitoring Unit

This block continuously tracks system behavior using:

- Mismatch counter  
- Partial fault counter  
- Full fault counter  
- Injected fault counter  
- Fault coverage  
- Reliability estimation  
- Availability calculation  

These metrics are used to evaluate system robustness.

---

### 5. ECC-Based Memory Protection (MDMHC)

The architecture includes a **Multi-Dimensional Hamming Code (MDMHC)** module:

- Encoder: Adds vertical and horizontal parity  
- Decoder: Detects and corrects errors  

Capabilities:
- ✔ Single-bit error correction  
- ✔ Multi-bit error detection  

This ensures data integrity in memory operations.

---

### 6. Lockstep Controller

The lockstep mechanism ensures synchronized execution between cores:

- Detects timing mismatches  
- Maintains execution consistency  

---

### 7. Recovery and Rollback Unit

The system includes advanced recovery mechanisms:

- Partial rollback (recover specific faulty core)  
- Full rollback (restart system state)  

This allows the processor to recover from faults without full system failure.

---

### 8. Fault Injection Unit

A hardware-based fault injection module is used to:

- Inject bit-level faults  
- Select target core  
- Simulate real-world fault scenarios  

This is critical for validating fault tolerance.

---

## 🔥 Key Features

- ✔ Triple Modular Redundancy (TMR)  
- ✔ Adaptive mode switching  
- ✔ ECC-based memory protection  
- ✔ Hardware fault injection  
- ✔ Recovery and rollback support  
- ✔ Reliability monitoring  

---

## 🎯 Summary

This architecture provides a **robust, scalable, and efficient fault-tolerant processor design**, suitable for safety-critical and high-reliability applications.


## RISC-V Processor Datapath

The proposed fault-tolerant system is built on a **5-stage pipelined RISC-V processor (RV32I)**. The datapath follows a standard pipeline structure with enhancements for forwarding and hazard handling.

---
![rv32i](https://github.com/user-attachments/assets/315617e6-db7d-4a50-88c6-ce29dfa6d0b2)

### Pipeline Stages

The processor is divided into five stages:

1. **Instruction Fetch (IF)**
   - Program Counter (PC) generates instruction address  
   - Instruction is fetched from Instruction Memory  
   - PC increment logic (PC + 4) updates next instruction  

---

2. **Instruction Decode (ID)**
   - Instruction is decoded using the control unit  
   - Register file reads operands (RS1, RS2)  
   - Immediate values are generated using Immediate Generator  

---

3. **Execute (EX)**
   - ALU performs arithmetic/logic operations  
   - Branch target address is computed  
   - ALU control determines operation type  

---

4. **Memory Access (MEM)**
   - Data memory is accessed for load/store instructions  
   - Read/Write operations controlled by control signals  

---

5. **Write Back (WB)**
   - Result is written back to register file  
   - Data comes from ALU or memory  

---

### Pipeline Registers

To maintain pipeline flow, the design includes:

- IF/ID  
- ID/EX  
- EX/MEM  
- MEM/WB  

These registers isolate each stage and enable parallel execution.

---

### Forwarding Unit (Hazard Resolution)

A **Forwarding Unit** is implemented to handle data hazards:

- Detects dependency between instructions  
- Forwards data from later stages (EX/MEM, MEM/WB)  
- Avoids pipeline stalls  

This improves performance and pipeline efficiency.

---

### ALU and Control Logic

- ALU performs arithmetic and logical operations  
- ALU Control determines operation using:
  - Opcode  
  - Function fields  

---

### Branch Handling

- Branch comparator checks conditions  
- PC is updated based on branch decision  
- Ensures correct control flow  

---

## 🔥 Key Features of Processor

- ✔ 5-stage pipelined architecture  
- ✔ Hazard detection and forwarding  
- ✔ Efficient instruction execution  
- ✔ Modular RTL design  

---

## 🎯 Role in Fault-Tolerant System

This processor is instantiated **three times (Core A, B, C)** in the TMR architecture.

Each core operates independently, and their outputs are compared by the majority voter to ensure correctness.

---

## 🚀 Summary

The datapath design ensures:

- High performance through pipelining  
- Efficient hazard handling  
- Compatibility with fault-tolerant extensions  

This forms the computational backbone of the adaptive fault-tolerant system.


## Adaptive Control and Recovery FSM

The proposed architecture includes an intelligent **Control Finite State Machine (FSM)** that manages system behavior under different fault conditions.

This FSM enables dynamic transitions between normal execution, fault detection, and recovery phases.

---
<img width="1054" height="1658" alt="adaptive+lock+recovery" src="https://github.com/user-attachments/assets/a1e749ae-12fb-49d7-ab66-a096f4028f7f" />

### 1. Majority Voter Output

The FSM receives inputs from the **Majority Voter**:

- `voted_data`
- `voter_state` (match / mismatch)

This determines whether the system is operating correctly or requires intervention.

---

### 2. Control FSM States

The system operates in three main states:

| State        | Description |
|-------------|------------|
| Normal      | Fault-free execution |
| Lockstep    | Re-execution and comparison |
| Recovery    | Error correction and rollback |

---

### 3. Normal Operation

- All cores produce matching outputs  
- System remains in **Normal state**  
- Adaptive mode selector operates in **TMR mode**  

---

### 4. Lockstep Phase

When a mismatch is detected:

- FSM enters **Lockstep Phase**
- Instructions are re-executed
- Outputs are compared again

**Outcomes:**

- Match → return to Normal  
- Mismatch → trigger Recovery  

---

### 5. Recovery Phase

If mismatch persists:

The system activates the **Recovery Unit**, which supports:

- Partial rollback → recover only faulty core  
- Full rollback → restore entire system state  

---

### 6. Recovery Register

A dedicated **Recovery Register** stores:

- Backup Program Counter (PC)  
- Backup data values  

This enables restoration of correct execution state.

---

### 7. Adaptive Mode Selector

Based on system health:

| Condition | Mode |
|----------|------|
| All cores healthy | TMR |
| One faulty core | Dual |
| Two faulty cores | Single |

This ensures optimal trade-off between:
- Reliability  
- Power consumption  
- Performance  

---

### 8. Final Output Generation

After recovery and mode selection:

- System produces **final correct output**  
- Fault impact is minimized  

---

## 🔥 Key Advantages

- ✔ Intelligent fault handling using FSM  
- ✔ Dynamic switching between execution modes  
- ✔ Efficient rollback and recovery  
- ✔ Reduced unnecessary redundancy  

---

## 🎯 Summary

The Control FSM enables:

- Real-time fault detection  
- Adaptive system behavior  
- Reliable recovery from faults  

This significantly enhances system robustness compared to static fault-tolerant designs.

## Multi-Dimensional Hamming Code (MDMHC) ECC

To enhance memory reliability, the proposed architecture integrates a **Multi-Dimensional Hamming Code (MDMHC)** based Error Correction Code (ECC) module.

This ECC scheme improves error detection and correction capability beyond conventional single-dimensional Hamming codes.

---
<img width="2102" height="3473" alt="mdmhc" src="https://github.com/user-attachments/assets/7a2131d9-11ce-42c8-a32c-f57e1ff44a75" />

## 🔹 1. Encoder Architecture

The encoder takes a **32-bit data input** and generates additional parity bits using two dimensions:

### Vertical Parity (Column-wise)
- XOR operation across corresponding bit positions  
- Example:
  - x0 ⊕ x16  
  - x1 ⊕ x17  
  - ...  

### Horizontal Parity (Row-wise)
- XOR across grouped bits  
- Example:
  - (x3..x0 + x11..x8)  
  - (x7..x4 + x15..x12)  

---

### Frame Generation

- Combines:
  - Original data (32 bits)
  - Vertical parity bits
  - Horizontal parity bits  

👉 Final encoded output: **67-bit protected data**

---

## 🔹 2. Memory Storage

- Encoded data is stored in memory  
- Ensures data integrity during read/write operations  

---

## 🔹 3. Decoder Architecture

Upon data retrieval:

### Step 1: Frame Splitting
- Extract:
  - Data
  - Vertical parity
  - Horizontal parity  

---

### Step 2: Parity Recalculation

- Recompute:
  - Vertical XOR  
  - Horizontal XOR  

---

### Step 3: Syndrome Generation

- Generate:
  - `Vsyn` (Vertical syndrome)  
  - `Hsyn` (Horizontal syndrome)  

These indicate the location of potential errors.

---

### Step 4: Error Localization

- Identify exact faulty bit position  
- Based on syndrome combination  

---

### Step 5: Error Correction

- Perform **bit-flip correction**  
- Restore original data  

---

## 🔹 4. Error Detection Capability

| Error Type | Detection | Correction |
|----------|----------|------------|
| Single-bit | ✔ | ✔ |
| Multi-bit | ✔ | ✖ (detected only) |

---

### Output Signals

- `single_error_corrected = 1` → Successful correction  
- `multiple_error_detected = 1` → Unrecoverable error  

---

## 🔥 Key Advantages

- ✔ Enhanced error coverage using 2D parity  
- ✔ Accurate single-bit error correction  
- ✔ Reliable multi-bit error detection  
- ✔ Improved memory robustness  

---

## 🎯 Summary

The MDMHC ECC module ensures:

- Reliable memory operations  
- Protection against soft errors  
- Integration with fault-tolerant processor pipeline  

This significantly improves system reliability when combined with TMR and adaptive recovery mechanisms.

