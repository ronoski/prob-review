# Automotive SPICE (ASPICE) — Presentation Version

## 1. What is ASPICE?

**ASPICE (Automotive SPICE)** stands for:

**Automotive Software Process Improvement and Capability Determination**

It is a **process framework** used to evaluate and improve automotive software development.

Modern vehicles contain many software systems such as:

*   Engine Control Units (ECUs)
*   ADAS systems
*   Infotainment systems
*   Battery management systems

Because these systems are *complex and safety‑critical*, the automotive industry needs a standard way to control development quality.

**ASPICE provides that framework.**

### Simple Explanation

ASPICE answers this question:

> **How mature and reliable is a company's software development process?**

Instead of focusing only on code quality, ASPICE evaluates the **entire development process**.

***

## 2. Why ASPICE is Important

Major automotive companies require suppliers to follow ASPICE, including:

*   BMW
*   Volkswagen
*   Mercedes‑Benz
*   General Motors
*   Hyundai

These companies use ASPICE to ensure:

*   High software quality
*   Reliable development processes
*   Good supplier capability
*   Reduced project risk

**Example:**  
A supplier developing ECU software for BMW might be required to achieve:

*   **ASPICE Capability Level 2 or 3**

***

## 3. ASPICE Development Model (V‑Model)

ASPICE development follows the **V‑Model**, ensuring that each development stage has a corresponding testing stage.

    DEVELOPMENT                              TESTING
    ---------------                           --------------------------
     System Requirements                ←→    System Qualification Test
            │                                      ▲
            ▼                                      │
     System Architecture                ←→    System Integration Test
            │                                      ▲
            ▼                                      │
     Software Requirements              ←→    Software Qualification Test
            │                                      ▲
            ▼                                      │
     Software Architecture               ←→    Software Integration Test
            │                                      ▲
            ▼                                      │
     Detailed Design / Coding            ←→    Unit Testing

### Key Concept

*   **Left side (↓)** → Design and Implementation
*   **Right side (↑)** → Verification and Testing

This ensures everything developed is properly tested.

***

## 4. ASPICE Process Groups

ASPICE organizes activities into several process groups—think of them like functional areas of a company.

### ASPICE Process Model

*   **ACQ** – Acquisition
*   **SPL** – Supply
*   **SYS** – System Engineering
*   **SWE** – Software Engineering
*   **SUP** – Supporting Processes
*   **MAN** – Management
*   **REU** – Reuse
*   **PIM** – Process Improvement

However, the most important groups for engineers are:

*   **SYS**
*   **SWE**
*   **SUP**
*   **MAN**

***

## 5. System Engineering Processes (SYS)

System engineering defines how the **entire vehicle system** works.

Key processes:

*   **SYS.1** Requirements Elicitation
*   **SYS.2** System Requirements Analysis
*   **SYS.3** System Architectural Design
*   **SYS.4** System Integration & Testing
*   **SYS.5** System Qualification Testing

### Example

For a brake control system, SYS processes define:

*   Functional requirements
*   System architecture
*   Component interfaces
*   System‑level tests

***

## 6. Software Engineering Processes (SWE)

The SWE group focuses on software development activities.

*   **SWE.1** Software Requirements Analysis
*   **SWE.2** Software Architecture Design
*   **SWE.3** Detailed Design and Coding
*   **SWE.4** Unit Verification
*   **SWE.5** Software Integration Testing
*   **SWE.6** Software Qualification Testing

### Development Flow

    Software Requirements
            │
            ▼
    Software Architecture
            │
            ▼
    Detailed Design
            │
            ▼
    Coding
            │
            ▼
    Unit Testing
            │
            ▼
    Integration Testing
            │
            ▼
    Software Qualification Testing

This ensures software is systematically designed, implemented, and tested.

***

## 7. Supporting Processes (SUP)

Supporting processes ensure **development quality and control**.

Important examples:

*   **SUP.1** Quality Assurance
*   **SUP.8** Configuration Management
*   **SUP.9** Problem Resolution
*   **SUP.10** Change Management

### Example Workflow (Defect Handling)

1.  A problem report is created
2.  Impact analysis is performed
3.  Code and tests are updated
4.  Changes are tracked

***

## 8. Management Processes (MAN)

Management processes organize and support the project.

Examples:

*   **MAN.3** Project Management
*   **MAN.5** Risk Management
*   **MAN.6** Measurement

These processes ensure:

*   Planning
*   Resource management
*   Risk tracking
*   Progress monitoring

***

## 9. ASPICE Capability Levels

ASPICE evaluates **process maturity** using six levels:

| Level | Name        | Description                      |
| ----- | ----------- | -------------------------------- |
| 5     | Optimizing  | Continuously improved            |
| 4     | Predictable | Quantitatively measured          |
| 3     | Established | Standardized across organization |
| 2     | Managed     | Managed and controlled           |
| 1     | Performed   | Process performed                |
| 0     | Incomplete  | Process not implemented          |

Most suppliers aim for:

*   **Capability Level 2 or 3**

***

## 10. Traceability (Critical Concept)

Traceability means **every artifact must be connected**.

    System Requirement
            │
            ▼
    Software Requirement
            │
            ▼
    Architecture Component
            │
            ▼
    Code Module
            │
            ▼
    Test Case

### Example Table

| System Req   | Software Req | Code Module    | Test Case |
| ------------ | ------------ | -------------- | --------- |
| SYS\_REQ\_01 | SW\_REQ\_10  | module\_init.c | test\_001 |

If auditors ask:

> “Where is this requirement implemented?”

You must show:

*   the design
*   the code
*   the test verifying it

***

## 11. Tools Used in ASPICE

Automotive companies use specialized tools to maintain quality and traceability.

### Requirements Management

*   Codebeamer
*   IBM DOORS
*   Jama

### Architecture Design

*   Enterprise Architect
*   MagicDraw

### Testing

*   VectorCAST
*   Squish
*   Jenkins

### Configuration Management

*   Git
*   Gerrit
*   SVN

***

## 12. Why ASPICE Matters

ASPICE improves development by ensuring:

*   Structured development processes
*   Complete requirements traceability
*   Systematic testing
*   Strong change management
*   Consistent supplier evaluation

ASPICE also supports safety standards such as **ISO 26262 (Functional Safety)**.

***

## Final Summary

ASPICE is a process framework used in the automotive industry to ensure **high‑quality software development**.

It defines:

*   structured development processes
*   requirement → code → test traceability
*   standardized project management
*   systematic verification and validation

By following ASPICE, organizations can produce reliable, safe, and high‑quality automotive software.
