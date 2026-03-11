# Automotive SPICE (ASPICE)

Automotive SPICE (Software Process Improvement and Capability dEtermination) is a **framework for assessing and improving software development processes** in the automotive industry. It defines what a mature, capable development process looks like and provides a structured way to measure how close a supplier's process is to that standard.

---

## Why Automotive SPICE Exists

Modern vehicles contain over 100 Electronic Control Units (ECUs) running hundreds of millions of lines of code. A failure in that software can cause accidents, recalls, or death.

```
  1970s vehicle:
  +----------------------------------+
  | Mechanical systems               |
  | Minimal electronics              |
  | ~0 lines of software             |
  +----------------------------------+

  2024 vehicle:
  +----------------------------------+
  | Engine control       (ECU)       |
  | Transmission control (ECU)       |
  | ABS / ESC            (ECU)       |
  | ADAS / Autopilot     (ECU)       |
  | Infotainment         (ECU)       |
  | Body control         (ECU)       |
  | Gateway / networking (ECU)       |
  | ... 100+ ECUs total              |
  | ~100-300 million lines of code   |
  +----------------------------------+

  OEM (car maker) does not write all this software —
  it is supplied by Tier 1 and Tier 2 suppliers.

  [OEM]
    |
    +---> [Tier 1 Supplier A]  (e.g. Bosch, Continental, ZF)
    |          |
    |          +---> [Tier 2 Supplier X]  (e.g. software house)
    |          +---> [Tier 2 Supplier Y]
    |
    +---> [Tier 1 Supplier B]

  OEM needs confidence that every supplier in the chain
  follows a rigorous, auditable development process.
  --> Automotive SPICE provides the measurement framework.
```

---

## Origins and Governance

```
  1990s: ISO/IEC 15504 (SPICE) published
         General software process assessment standard
              |
              v
  2005: Automotive SPICE v2.0 released
        Developed by AUTOMOTIVESPICE consortium
        (major OEMs: BMW, Audi, Daimler, Ford, VW, Porsche, etc.)
              |
              v
  2010: Automotive SPICE v2.5 — widely adopted
        Became de facto requirement in supplier contracts
              |
              v
  2015: Automotive SPICE v3.0
        Aligned with ISO/IEC 33000 series (replaced 15504)
              |
              v
  2017: Automotive SPICE v3.1 — current mainstream version
        Added support for agile, hardware processes, cybersecurity
              |
              v
  2023: Automotive SPICE for Cybersecurity (separate extension)
        Aligned with ISO/SAE 21434

  Governed by: VDA QMC (German Association of the Automotive Industry)
  Freely available at: automotivespice.com
```

---

## Core Concept — What ASPICE Measures

ASPICE measures two independent dimensions for every process:

```
  DIMENSION 1: PROCESS                DIMENSION 2: CAPABILITY
  What you are doing                  How well you are doing it

  +---------------------------+       +---------------------------+
  | Which processes exist     |       | Capability Level 0–5      |
  | in your organisation?     |       | Are they performed,       |
  | (process reference model) |       | managed, established,     |
  |                           |       | predictable, optimising?  |
  +---------------------------+       +---------------------------+
              |                                   |
              +-------------------+---------------+
                                  |
                          ASPICE Assessment
                          "Process X is at Capability Level Y"
```

---

## The Capability Levels

Every process in an organisation is rated on a six-level scale (0–5).

```
  Level 5  Optimising    +-----------------------------------------------+
           Process       | Continuously improved to meet current and     |
                         | projected business goals                      |
  Level 4  Predictable  ++-----------------------------------------------+
           Process       | Operates within defined limits to achieve     |
                         | its outcomes — quantitatively managed         |
  Level 3  Established  ++-----------------------------------------------+
           Process       | Standard process defined, used, and           |
                         | maintained across the organisation            |
  Level 2  Managed      ++-----------------------------------------------+
           Process       | Planned, monitored, and adjusted; work        |
                         | products are controlled                       |
  Level 1  Performed    ++-----------------------------------------------+
           Process       | Process achieves its purpose — work products  |
                         | exist, outcomes are met (but informally)      |
  Level 0  Incomplete   ++-----------------------------------------------+
                         | Process not implemented or fails to achieve   |
                         | its purpose                                   |
                         +-----------------------------------------------+

  Most automotive supplier contracts require:
  - Safety-relevant processes:   Level 3
  - Standard supply processes:   Level 2
  - Minimum acceptable:          Level 1 (rare — most OEMs require 2+)
```

### Capability Level Detail

```
  LEVEL 1 — PERFORMED
  Evidence required:
  - Process outcomes are achieved
  - Work products exist
  No formal planning or tracking needed at this level.

  LEVEL 2 — MANAGED
  Evidence required:
  PA 2.1 Performance Management:
    - Process is planned
    - Roles and responsibilities assigned
    - Resources allocated
    - Progress monitored against plan
  PA 2.2 Work Product Management:
    - Work products are defined
    - Requirements for WPs established
    - WPs are reviewed and adjusted
    - WPs are version controlled

  LEVEL 3 — ESTABLISHED
  Evidence required:
  PA 3.1 Process Definition:
    - Standard process defined for the organisation
    - Process assets maintained (templates, guidelines)
  PA 3.2 Process Deployment:
    - Standard process tailored for the project
    - Skilled personnel assigned
    - Process infrastructure in place

  Levels 4 and 5 are rarely assessed in supplier audits
  (require statistical process control and organisational
  improvement programmes).
```

---

## The Process Reference Model — Process Groups

ASPICE organises all processes into a two-tier hierarchy: **process groups** and **individual processes**.

```
  AUTOMOTIVE SPICE PROCESS REFERENCE MODEL

  +------------------------------------------------------------------+
  |                    PRIMARY LIFE CYCLE                            |
  |                                                                  |
  |  ACQ — Acquisition        SYS — System Engineering              |
  |  SPL — Supply             SWE — Software Engineering            |
  |                           HWE — Hardware Engineering            |
  +------------------------------------------------------------------+
  |                  SUPPORTING LIFE CYCLE                           |
  |                                                                  |
  |  SUP — Support            MAN — Management                      |
  |  REU — Reuse              PIM — Process Improvement             |
  +------------------------------------------------------------------+
```

### Full Process List

```
  GROUP    ID       PROCESS NAME
  -------  -------  -----------------------------------------
  ACQ      ACQ.3    Contract Agreement
           ACQ.4    Supplier Monitoring
           ACQ.13   Project Requirements
           ACQ.14   Supplier Qualification
           ACQ.15   Supplier Request and Selection

  SPL      SPL.1    Supplier Tendering
           SPL.2    Product Release

  SYS      SYS.1    Requirements Elicitation
           SYS.2    System Requirements Analysis
           SYS.3    System Architectural Design
           SYS.4    System Integration and Integration Test
           SYS.5    System Qualification Test

  SWE      SWE.1    Software Requirements Analysis
           SWE.2    Software Architectural Design
           SWE.3    Software Detailed Design & Unit Construction
           SWE.4    Software Unit Verification
           SWE.5    Software Integration and Integration Test
           SWE.6    Software Qualification Test

  HWE      HWE.1    Hardware Requirements Analysis
           HWE.2    Hardware Architectural Design
           HWE.3    Hardware Detailed Design
           HWE.4    Hardware Integration and Integration Test
           HWE.5    Hardware Qualification Test

  SUP      SUP.1    Quality Assurance
           SUP.2    Verification
           SUP.4    Joint Review
           SUP.7    Documentation
           SUP.8    Configuration Management
           SUP.9    Problem Resolution Management
           SUP.10   Change Request Management

  MAN      MAN.3    Project Management
           MAN.5    Risk Management
           MAN.6    Measurement

  PIM      PIM.3    Process Improvement

  REU      REU.2    Reuse Programme Management
```

---

## The Software Engineering Processes (SWE) in Detail

SWE.1 through SWE.6 form a **V-model** — requirements and design on the left, verification and testing on the right.

```
  SWE.1                                               SWE.6
  Software                                            Software
  Requirements  ---------------------------------->   Qualification
  Analysis                                            Test
      |                                                   ^
      v                                                   |
  SWE.2                                               SWE.5
  Software                                            Software
  Architectural ---------------------------------->   Integration
  Design                                              Test
      |                                                   ^
      v                                                   |
  SWE.3                                               SWE.4
  Software                                            Software Unit
  Detailed Design -------------------------------->   Verification
  & Unit Construction

  Left side:  WHAT to build and HOW to build it
  Right side: PROOF that it was built correctly
  Arrows:     Each left side process feeds the test criteria
              for its corresponding right side process
```

### SWE.1 — Software Requirements Analysis

```
  INPUT                   PROCESS                   OUTPUT
  +----------------+      +------------------+      +------------------+
  | System         |      | Elicit, analyse, |      | Software         |
  | requirements   |----->| and document     |----->| requirements     |
  | (from SYS.2)   |      | software         |      | specification    |
  |                |      | requirements     |      | (SRS)            |
  | Customer       |      |                  |      |                  |
  | needs          |      | Consistency and  |      | Traceability to  |
  |                |      | feasibility      |      | system reqs      |
  +----------------+      | checked          |      |                  |
                          +------------------+      | Test criteria    |
                                                    | for SWE.6        |
                                                    +------------------+
  Key outcomes:
  - Each requirement is: unique, complete, consistent, testable, feasible
  - Requirements are rated by criticality
  - Bidirectional traceability to system requirements established
  - Interface requirements to other software and hardware defined
```

### SWE.2 — Software Architectural Design

```
  INPUT                   PROCESS                   OUTPUT
  +----------------+      +------------------+      +------------------+
  | Software       |      | Define top-level |      | Software         |
  | requirements   |----->| structure:       |----->| architectural    |
  | (SWE.1)        |      | components,      |      | design doc       |
  |                |      | interfaces,      |      |                  |
  |                |      | data flows,      |      | Static view:     |
  |                |      | resource usage   |      | components +     |
  |                |      |                  |      | interfaces       |
  |                |      |                  |      |                  |
  |                |      |                  |      | Dynamic view:    |
  |                |      |                  |      | behaviour,       |
  |                |      |                  |      | timing, state    |
  +----------------+      +------------------+      +------------------+
  Key outcomes:
  - All requirements allocated to components
  - Interfaces between components specified
  - Criteria for SWE.5 integration test defined
  - Dynamic behaviour, concurrency, timing constraints documented
```

### SWE.3 — Software Detailed Design and Unit Construction

```
  INPUT                   PROCESS                   OUTPUT
  +----------------+      +------------------+      +------------------+
  | Architectural  |      | Design each unit |      | Detailed design  |
  | design (SWE.2) |----->| to implementable |----->| (per unit)       |
  |                |      | level, then code |      |                  |
  |                |      | it               |      | Source code      |
  |                |      |                  |      |                  |
  |                |      | Coding           |      | Unit test        |
  |                |      | guidelines       |      | criteria for     |
  |                |      | applied          |      | SWE.4            |
  +----------------+      +------------------+      +------------------+
  Key outcomes:
  - Each unit fully specified before coding (or concurrent)
  - Coding standards applied (MISRA C, AUTOSAR guidelines)
  - Traceability from code to detailed design to architecture
  - Unit test criteria derived from detailed design
```

### SWE.4 — Software Unit Verification

```
  INPUT                   PROCESS                   OUTPUT
  +----------------+      +------------------+      +------------------+
  | Source code    |      | Verify each unit |      | Unit test        |
  | (SWE.3)        |----->| meets its design |----->| results          |
  |                |      | specification    |      |                  |
  | Unit test      |      |                  |      | Coverage metrics |
  | criteria       |      | Static analysis  |      | (statement,      |
  |                |      | Code review      |      |  branch, MC/DC)  |
  |                |      | Dynamic testing  |      |                  |
  +----------------+      +------------------+      | Defects logged   |
                                                    +------------------+
  Key outcomes:
  - Unit tested in isolation (stub/mock dependencies)
  - Code coverage measured and meets target (e.g. MC/DC for ASIL D)
  - Static analysis results reviewed and addressed
  - All failed tests result in defect reports (SUP.9)
```

### SWE.5 — Software Integration and Integration Test

```
  INPUT                   PROCESS                   OUTPUT
  +----------------+      +------------------+      +------------------+
  | Verified units |      | Combine units    |      | Integrated       |
  | (SWE.4)        |----->| into subsystems  |----->| software build   |
  |                |      | and full         |      |                  |
  | Integration    |      | software stack,  |      | Integration test |
  | test criteria  |      | verify interfaces|      | results          |
  | (SWE.2)        |      | behave correctly |      |                  |
  +----------------+      +------------------+      | Regression test  |
                                                    | suite            |
                                                    +------------------+
  Key outcomes:
  - Integration order defined and followed
  - Interface behaviour between components verified
  - Regression tests run on every integration build
  - Test results traceable to architectural design
```

### SWE.6 — Software Qualification Test

```
  INPUT                   PROCESS                   OUTPUT
  +----------------+      +------------------+      +------------------+
  | Fully          |      | Test the complete|      | Qualification    |
  | integrated     |----->| software against |----->| test report      |
  | software       |      | the original     |      |                  |
  |                |      | software         |      | Release decision |
  | SW requirement |      | requirements     |      |                  |
  | test criteria  |      | (from SWE.1)     |      | Requirements     |
  | (SWE.1)        |      |                  |      | coverage proof   |
  +----------------+      +------------------+      +------------------+
  Key outcomes:
  - Every software requirement has at least one test case
  - Tests run in target environment or representative simulator
  - Regression testing performed
  - Release criteria checked and signed off
```

---

## System Engineering Processes (SYS)

The SYS processes mirror the SWE V-model but at the **system level** — covering hardware, software, and their integration together.

```
  SYS.1                                               SYS.5
  Requirements  ---------------------------------->   System
  Elicitation                                         Qualification
                                                      Test
      |                                                   ^
      v                                                   |
  SYS.2                                               SYS.4
  System                                              System
  Requirements  ---------------------------------->   Integration
  Analysis                                            Test
      |                                                   ^
      v                                                   |
  SYS.3
  System
  Architectural
  Design
         |
         |-------> SWE.1 (software requirements)
         |-------> HWE.1 (hardware requirements)
```

---

## Supporting Processes (SUP)

Supporting processes run **throughout the project** — they are not lifecycle phases but continuous activities that underpin everything else.

```
  PROJECT TIMELINE
  |-----SWE.1--SWE.2--SWE.3--SWE.4--SWE.5--SWE.6-----|
  |                                                     |
  |  SUP.8  Configuration Management  ................. | track every artefact
  |  SUP.9  Problem Resolution Mgmt   ................. | log and fix defects
  |  SUP.10 Change Request Management ................. | control scope changes
  |  SUP.1  Quality Assurance         ................. | audit process compliance
  |  SUP.2  Verification              ................. | review every work product
  |  SUP.4  Joint Review              ................. | milestone reviews with OEM
  |  SUP.7  Documentation             ................. | maintain all records
  |                                                     |
  |-----------------------------------------------------|
```

### SUP.8 — Configuration Management

```
  Everything produced by the project is a configuration item (CI):
  requirements, designs, source code, test cases, tools, test results.

  +-------------------+
  | Configuration     |
  | Item (CI)         |
  +-------------------+
  | Unique ID         |  e.g. SWE-REQ-0042 v1.3
  | Version           |
  | Status            |  Draft / Reviewed / Approved / Obsolete
  | Owner             |
  | Change history    |
  +-------------------+

  Configuration Management ensures:
  - You always know which version of what is in the product
  - You can reproduce any previous build exactly
  - No uncontrolled changes reach integration or test
```

### SUP.9 — Problem Resolution Management

```
  Defect lifecycle:
  Detected
     |
     v
  +----------+    +----------+    +----------+    +----------+
  | New       |--->| Assigned |--->| In Fix   |--->| Verified |
  +----------+    +----------+    +----------+    +----------+
                                                       |
                                                       v
                                                  +----------+
                                                  | Closed   |
                                                  +----------+

  Every defect record must contain:
  - Unique ID and date found
  - Description and how to reproduce
  - Severity and priority
  - Root cause (after analysis)
  - Fix description
  - Verification that fix works and did not regress anything
```

---

## Management Processes (MAN)

### MAN.3 — Project Management

```
  Project Management in ASPICE covers:

  PLANNING                    MONITORING                 CONTROL
  +--------------------+      +--------------------+      +--------------------+
  | Define scope       |      | Track progress vs  |      | Take corrective    |
  | Estimate effort    |      | plan               |      | action when off    |
  | Schedule tasks     |      | Monitor risks      |      | track              |
  | Allocate resources |      | Review milestones  |      | Update plan        |
  | Identify risks     |      | Collect metrics    |      | Escalate issues    |
  +--------------------+      +--------------------+      +--------------------+
```

### MAN.5 — Risk Management

```
  Risk lifecycle:
  +------------------+
  | Identify risks   |  What could go wrong?
  +--------+---------+
           |
           v
  +------------------+
  | Analyse risks    |  Probability x Impact = Exposure
  +--------+---------+
           |
           v
  +------------------+
  | Treat risks      |  Avoid / Mitigate / Transfer / Accept
  +--------+---------+
           |
           v
  +------------------+
  | Monitor risks    |  Track treatment actions, watch for new risks
  +------------------+

  Risk register entry:
  +-----------+--------+--------+----------+------------------+----------+
  | Risk ID   | Prob   | Impact | Exposure | Treatment        | Status   |
  +-----------+--------+--------+----------+------------------+----------+
  | RSK-007   | High   | High   | Critical | Add code review  | Open     |
  |           |        |        |          | step to SWE.3    |          |
  +-----------+--------+--------+----------+------------------+----------+
```

---

## Traceability — The Spine of ASPICE

Traceability links every artefact back to its origin and forward to its evidence. Without it, you cannot prove a requirement was implemented and tested.

```
  BIDIRECTIONAL TRACEABILITY CHAIN

  Customer need
       |  (traced down)
       v
  System requirement  (SYS.2)
       |
       v
  Software requirement  (SWE.1)  <--- test case (SWE.6)
       |
       v
  Architectural component  (SWE.2)  <--- integration test (SWE.5)
       |
       v
  Detailed design / unit  (SWE.3)  <--- unit test (SWE.4)
       |
       v
  Source code  (SWE.3)

  Tracing UP (from code to need):   "Why does this code exist?"
  Tracing DOWN (from need to code): "Where is this requirement implemented?"
  Tracing ACROSS (to tests):        "How do we know it works?"

  A gap in traceability = either:
  - An orphaned requirement (never implemented)
  - Dead code (no requirement justifies it)
  - An untested requirement (no test coverage)
```

---

## Work Products

Each ASPICE process requires specific **work products** as evidence. The assessor verifies these exist, are complete, and are version controlled.

```
  Process    Key Work Products
  ---------  ---------------------------------------------------------------
  SWE.1      Software requirements specification (SRS)
             Requirements traceability matrix (to system reqs)
             Requirements review records

  SWE.2      Software architectural design document
             Interface specifications
             Traceability (SRS <-> architecture components)

  SWE.3      Detailed design documents (per unit)
             Source code
             Coding standard compliance reports
             Unit test specification

  SWE.4      Unit test plan and specification
             Unit test results
             Code coverage report
             Static analysis results

  SWE.5      Integration test plan and specification
             Integration test results
             Build / integration records

  SWE.6      Software qualification test plan and spec
             Software qualification test results
             Release note

  SUP.8      Configuration management plan
             Configuration item list (baseline)
             Change history records

  SUP.9      Problem report log
             Root cause analysis records

  MAN.3      Project plan
             Progress reports
             Resource records
```

---

## ASPICE Assessment

### Who Assesses

```
  Three types of assessment:

  +---------------------+--------------------------------------------+
  | Self-assessment     | Supplier assesses their own processes      |
  |                     | Internal use; not accepted by most OEMs    |
  +---------------------+--------------------------------------------+
  | 2nd party assessment| OEM or customer assesses the supplier      |
  |                     | Most common in automotive; required for     |
  |                     | new programmes and re-assessments          |
  +---------------------+--------------------------------------------+
  | 3rd party assessment| Independent assessor (intacs certified)    |
  |                     | Most rigorous; used for critical safety     |
  |                     | components or where OEM lacks assessors    |
  +---------------------+--------------------------------------------+
```

### Assessment Process

```
  BEFORE                    DURING                    AFTER
  +-------------------+     +-------------------+     +-------------------+
  | Define scope:     |     | Document review:  |     | Findings report   |
  | - which processes |     | check work        |     | with:             |
  | - which projects  |     | products exist    |     | - Ratings per     |
  | - which OUs       |     | and are adequate  |     |   process         |
  |                   |     |                   |     | - Capability      |
  | Assessor briefs   |     | Interviews:       |     |   level per       |
  | team; team        |     | question engineers|     |   process         |
  | prepares evidence |     | and managers on   |     | - Strengths       |
  |                   |     | how they work     |     | - Weaknesses      |
  |                   |     |                   |     | - Action items    |
  |                   |     | Process           |     |                   |
  |                   |     | observation:      |     | Supplier uses     |
  |                   |     | watch team        |     | findings to       |
  |                   |     | perform a task    |     | drive improvement |
  +-------------------+     +-------------------+     +-------------------+

  Assessment rating scale per process attribute:
  N — Not achieved    (0–15% fulfilled)
  P — Partially       (15–50%)
  L — Largely         (50–85%)
  F — Fully achieved  (85–100%)

  To achieve Level 2: all Level 1 + Level 2 attributes rated F or L
  To achieve Level 3: all Level 1-2 + Level 3 attributes rated F or L
```

### Typical OEM Requirements by Process

```
  Process          Typical OEM Requirement   Rationale
  +--------------+-------------------------+-----------------------------+
  | SWE.1        | Level 3                 | Requirements are the        |
  |              |                         | foundation of everything    |
  +--------------+-------------------------+-----------------------------+
  | SWE.2        | Level 3                 | Architecture errors are     |
  |              |                         | costly to fix late          |
  +--------------+-------------------------+-----------------------------+
  | SWE.3        | Level 3                 | Coding consistency critical |
  +--------------+-------------------------+-----------------------------+
  | SWE.4        | Level 3                 | Unit test rigour needed     |
  +--------------+-------------------------+-----------------------------+
  | SWE.5        | Level 3                 | Integration errors are      |
  |              |                         | expensive to find late      |
  +--------------+-------------------------+-----------------------------+
  | SWE.6        | Level 3                 | Qualification = final proof |
  +--------------+-------------------------+-----------------------------+
  | SUP.8        | Level 2 or 3            | Config mgmt is baseline     |
  +--------------+-------------------------+-----------------------------+
  | SUP.9        | Level 2                 | Defects must be tracked     |
  +--------------+-------------------------+-----------------------------+
  | MAN.3        | Level 2 or 3            | Project must be managed     |
  +--------------+-------------------------+-----------------------------+
```

---

## Relationship to ISO 26262 (Functional Safety)

ASPICE and ISO 26262 are **complementary, not overlapping**.

```
  ISO 26262                              Automotive SPICE
  +----------------------------+         +----------------------------+
  | WHAT to do for safety      |         | HOW WELL the process       |
  | - Hazard analysis (HARA)   |         | is executed                |
  | - Safety goals             |         | - Process capability level |
  | - ASIL classification      |         | - Work product quality     |
  | - Safety requirements      |         | - Traceability evidence    |
  | - Verification methods     |         | - Management discipline    |
  | - Safety case              |         |                            |
  +----------------------------+         +----------------------------+
              |                                      |
              +------------------+-------------------+
                                 |
              Both are required for safety-critical
              automotive software development.
              They are assessed separately but
              evidence often overlaps.

  ASIL level drives ASPICE rigour:
  +--------+-------------------------------------------------------+
  | ASIL A | Basic process discipline required                     |
  | ASIL B | Stronger verification evidence, review records        |
  | ASIL C | Formal methods encouraged, independent review         |
  | ASIL D | Highest rigour: MC/DC coverage, fully independent     |
  |        | testing, complete traceability mandatory              |
  +--------+-------------------------------------------------------+
```

---

## Relationship to CMMI

ASPICE and CMMI (Capability Maturity Model Integration) share common roots (both derived from ISO/IEC 15504) but serve different industries.

```
  Feature             Automotive SPICE          CMMI
  +-----------------+--------------------------+------------------------+
  | Industry        | Automotive               | General / defence /    |
  |                 |                          | aerospace              |
  | Measurement     | Process capability       | Organisational         |
  | unit            | per individual process   | maturity level (1–5)   |
  | Granularity     | Fine — per process       | Coarser — per maturity |
  |                 | attribute rating         | level                  |
  | Process model   | Based on V-model         | More flexible          |
  | Assessors       | intacs certified         | CMMI Institute         |
  |                 |                          | authorised             |
  | Common use      | Supplier qualification   | Organisational         |
  |                 | by OEM                   | improvement            |
  +-----------------+--------------------------+------------------------+

  Key difference:
  ASPICE rates each process independently at its own capability level.
  CMMI rates the whole organisation at a single maturity level.
```

---

## Common Findings in ASPICE Assessments

These are the most frequently cited weaknesses in supplier assessments:

```
  Finding                              Affected Processes
  +------------------------------------+----------------------------------+
  | Requirements not testable          | SWE.1, SYS.2                    |
  | (no acceptance criteria defined)   |                                  |
  +------------------------------------+----------------------------------+
  | Traceability gaps                  | SWE.1 to SWE.6, all             |
  | (requirements not linked to tests) |                                  |
  +------------------------------------+----------------------------------+
  | No bidirectional traceability      | SWE.1/2/3/4/5/6                 |
  | (only top-down, not bottom-up)     |                                  |
  +------------------------------------+----------------------------------+
  | Code coverage not measured         | SWE.4                           |
  | or targets not met                 |                                  |
  +------------------------------------+----------------------------------+
  | Work products not version          | SUP.8, all                      |
  | controlled                         |                                  |
  +------------------------------------+----------------------------------+
  | Defects not logged                 | SUP.9                           |
  | or root cause not analysed         |                                  |
  +------------------------------------+----------------------------------+
  | No regression testing after        | SWE.4, SWE.5, SWE.6            |
  | bug fixes                          |                                  |
  +------------------------------------+----------------------------------+
  | Reviews not recorded               | SUP.2, SWE.1–6                 |
  | (no minutes, no action items)      |                                  |
  +------------------------------------+----------------------------------+
  | Project plan not updated           | MAN.3                           |
  | during execution                   |                                  |
  +------------------------------------+----------------------------------+
  | Risk register never updated        | MAN.5                           |
  | after project start                |                                  |
  +------------------------------------+----------------------------------+
```

---

## Full Process Flow Summary

```
  CUSTOMER / OEM
       |
       | Customer requirements
       v
  SYS.1 Requirements Elicitation
       |
       v
  SYS.2 System Requirements Analysis
       |
       v
  SYS.3 System Architectural Design
       |                  |
       |                  +----------------> HWE.1 Hardware Reqs
       v                                          |
  SWE.1 Software Requirements Analysis            v
       |                                    HWE.2 HW Architecture
       v                                          |
  SWE.2 Software Architectural Design             v
       |                                    HWE.3 HW Detailed Design
       v                                          |
  SWE.3 Detailed Design & Unit Construction       v
       |                                    HWE.4 HW Integration Test
       v                                          |
  SWE.4 Software Unit Verification                v
       |                                    HWE.5 HW Qualification Test
       v                                          |
  SWE.5 Software Integration Test                 |
       |                   <--------------------  +
       v
  SYS.4 System Integration Test
       |
       v
  SWE.6 Software Qualification Test
       |
       v
  SYS.5 System Qualification Test
       |
       v
  SPL.2 Product Release
       |
       v
  CUSTOMER DELIVERY

  Throughout the entire flow:
  SUP.8  Configuration Management     (every artefact version controlled)
  SUP.9  Problem Resolution           (every defect tracked)
  SUP.10 Change Request Management    (every change controlled)
  SUP.1  Quality Assurance            (audits at each phase)
  MAN.3  Project Management           (plan, monitor, control)
  MAN.5  Risk Management              (risks tracked throughout)
```

---

## Quick Reference

| Term             | Meaning                                                              |
|------------------|----------------------------------------------------------------------|
| ASPICE           | Automotive SPICE — process assessment framework for automotive SW    |
| Capability level | 0–5 rating of how well a process is performed and managed           |
| Process attribute| Measurable property used to rate capability (PA 1.1, 2.1, 2.2 etc) |
| Rating           | N/P/L/F — Not/Partially/Largely/Fully achieved (0–15/15–50/50–85/85–100%) |
| Work product     | Artefact produced by a process (SRS, design doc, test result)       |
| Traceability     | Links between requirements, design, code, and tests in both directions |
| SRS              | Software Requirements Specification — output of SWE.1              |
| V-model          | Development model with design on left, verification on right        |
| OEM              | Original Equipment Manufacturer — the car maker                     |
| Tier 1           | Direct supplier to OEM                                              |
| Tier 2           | Supplier to Tier 1                                                  |
| intacs           | International assessor certification scheme for ASPICE assessors    |
| HARA             | Hazard Analysis and Risk Assessment — key ISO 26262 activity        |
| ASIL             | Automotive Safety Integrity Level A–D (ISO 26262 risk classification)|
| MC/DC            | Modified Condition/Decision Coverage — code coverage required for ASIL C/D |
| ECU              | Electronic Control Unit — embedded computer in a vehicle            |
| Configuration item | Any managed artefact: code, doc, test, tool, build script         |
| SUP.8            | Configuration Management process                                    |
| SUP.9            | Problem Resolution Management — defect tracking                     |
| MAN.3            | Project Management process                                          |
| MAN.5            | Risk Management process                                             |
