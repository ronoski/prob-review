# Automotive SPICE (ASPICE)

**Automotive Software Process Improvement and Capability Determination**

A **process framework** used to evaluate and improve automotive software development.

---

## 1. Why ASPICE Exists

Modern vehicles contain over 100 Electronic Control Units (ECUs) running hundreds of millions of lines of code. A failure in that software can cause accidents, recalls, or death.

```
  1970s vehicle:                       2024 vehicle:
  +----------------------------------+  +----------------------------------+
  | Mechanical systems               |  | Engine control       (ECU)       |
  | Minimal electronics              |  | Transmission control (ECU)       |
  | ~0 lines of software             |  | ABS / ESC            (ECU)       |
  +----------------------------------+  | ADAS / Autopilot     (ECU)       |
                                        | Infotainment         (ECU)       |
                                        | ... 100+ ECUs total              |
                                        | ~100–300 million lines of code   |
                                        +----------------------------------+
```

The OEM (car maker) does not write all this software — it is supplied by Tier 1 and Tier 2 suppliers:

```
  [OEM]
    |
    +---> [Tier 1 Supplier A]  (e.g. Bosch, Continental, ZF)
    |          |
    |          +---> [Tier 2 Supplier X]  (software house)
    |          +---> [Tier 2 Supplier Y]
    |
    +---> [Tier 1 Supplier B]
```

OEMs need confidence that every supplier in the chain follows a rigorous, auditable development process. **ASPICE provides that measurement framework.**

---

## 2. Origins and Governance

```
  1990s  ISO/IEC 15504 (SPICE) — general software process assessment standard
           |
  2005   Automotive SPICE v2.0 — developed by AUTOMOTIVESPICE consortium
         (BMW, Audi, Daimler, Ford, VW, Porsche, etc.)
           |
  2010   Automotive SPICE v2.5 — widely adopted, de facto supplier contract requirement
           |
  2015   Automotive SPICE v3.0 — aligned with ISO/IEC 33000 series
           |
  2017   Automotive SPICE v3.1 — current mainstream version
         Added agile support, hardware processes, cybersecurity
           |
  2023   Automotive SPICE for Cybersecurity — aligned with ISO/SAE 21434

  Governed by: VDA QMC (German Association of the Automotive Industry)
  Available at: automotivespice.com
```

---

## 3. Core Concept — What ASPICE Measures

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

## 4. Capability Levels

Every process is rated on a six-level scale (0–5):

| Level | Name        | Description                                                |
|-------|-------------|-----------------------------------------------------------|
| 5     | Optimising  | Continuously improved to meet current and projected goals  |
| 4     | Predictable | Operates within defined limits; quantitatively managed     |
| 3     | Established | Standard process defined, used, and maintained org-wide    |
| 2     | Managed     | Planned, monitored, adjusted; work products controlled     |
| 1     | Performed   | Process achieves its purpose (but informally)              |
| 0     | Incomplete  | Process not implemented or fails to achieve its purpose    |

**Most OEM supplier contracts require:**
- Safety-relevant processes: **Level 3**
- Standard supply processes: **Level 2**

### Level Detail

```
  LEVEL 2 — MANAGED
  PA 2.1 Performance Management:  process planned, roles assigned, progress tracked
  PA 2.2 Work Product Management: WPs defined, reviewed, version controlled

  LEVEL 3 — ESTABLISHED
  PA 3.1 Process Definition:  standard process defined org-wide, assets maintained
  PA 3.2 Process Deployment:  standard process tailored per project, infrastructure in place

  (Levels 4 and 5 require statistical process control — rarely assessed in supplier audits)
```

---

## 5. Process Groups

```
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

The most important groups for engineers: **SYS, SWE, SUP, MAN**

---

## 6. The V-Model (SWE + SYS)

ASPICE development follows the **V-Model** — design and requirements on the left, verification and testing on the right:

```
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


  Left side (↓):  WHAT to build and HOW to build it
  Right side (↑): PROOF that it was built correctly
  Horizontal (─):  each left-side process defines the test criteria for its right-side counterpart
```

---

## 7. Software Engineering Processes (SWE) in Detail

### SWE.1 — Software Requirements Analysis

- Analyse and document software requirements from system requirements (SYS.2)
- Each requirement must be: unique, complete, consistent, testable, feasible
- Bidirectional traceability to system requirements established
- **Output:** Software Requirements Specification (SRS), test criteria for SWE.6

### SWE.2 — Software Architectural Design

- Define top-level structure: components, interfaces, data flows, resource usage
- All requirements allocated to components; interfaces between components specified
- **Output:** Architectural design doc (static + dynamic views), criteria for SWE.5

### SWE.3 — Software Detailed Design and Unit Construction

- Design each unit to implementable level, then code it
- Coding standards applied (MISRA C, AUTOSAR guidelines)
- Traceability from code → detailed design → architecture
- **Output:** Detailed design docs, source code, unit test criteria for SWE.4

### SWE.4 — Software Unit Verification

- Verify each unit in isolation (stub/mock dependencies)
- Static analysis, code review, dynamic testing
- Code coverage measured (e.g. MC/DC for ASIL C/D)
- **Output:** Unit test results, coverage report, defects logged in SUP.9

### SWE.5 — Software Integration and Integration Test

- Combine verified units into subsystems and full software stack
- Verify interface behaviour between components
- Regression tests run on every integration build
- **Output:** Integrated software build, integration test results

### SWE.6 — Software Qualification Test

- Test complete software against original software requirements (SWE.1)
- Every software requirement has at least one test case
- Tests run in target environment or representative simulator
- **Output:** Qualification test report, release decision, requirements coverage proof

---

## 8. System Engineering Processes (SYS)

Mirror the SWE V-model at the system level — covering hardware, software, and their integration:

```
  SYS.1  Requirements Elicitation
  SYS.2  System Requirements Analysis
  SYS.3  System Architectural Design
              |-------> SWE.1 (software requirements)
              |-------> HWE.1 (hardware requirements)
  SYS.4  System Integration and Integration Test
  SYS.5  System Qualification Test
```

---

## 9. Supporting Processes (SUP)

Supporting processes run **throughout the entire project** — not lifecycle phases, but continuous activities.

```
  PROJECT TIMELINE
  |-----SWE.1--SWE.2--SWE.3--SWE.4--SWE.5--SWE.6-----|
  |                                                     |
  |  SUP.8  Configuration Management  ................ | track every artefact
  |  SUP.9  Problem Resolution Mgmt   ................ | log and fix defects
  |  SUP.10 Change Request Management ................ | control scope changes
  |  SUP.1  Quality Assurance         ................ | audit process compliance
  |  SUP.2  Verification              ................ | review every work product
  |  SUP.4  Joint Review              ................ | milestone reviews with OEM
  |  SUP.7  Documentation             ................ | maintain all records
  |-----------------------------------------------------|
```

### SUP.8 — Configuration Management

Every artefact (requirement, design, code, test, tool) is a **configuration item (CI)**:

```
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
```

### SUP.9 — Problem Resolution Management

```
  Detected → New → Assigned → In Fix → Verified → Closed

  Every defect record must contain:
  - Unique ID, date found, severity, priority
  - Description and reproduction steps
  - Root cause (after analysis)
  - Fix description + verification that fix works without regression
```

---

## 10. Management Processes (MAN)

### MAN.3 — Project Management

```
  PLANNING               MONITORING              CONTROL
  Define scope           Track progress          Take corrective action
  Estimate effort        Monitor risks           Update plan
  Schedule tasks         Review milestones       Escalate issues
  Allocate resources     Collect metrics
```

### MAN.5 — Risk Management

```
  Identify risks  →  Analyse (Probability × Impact)  →  Treat  →  Monitor

  Risk register entry:
  | Risk ID | Prob | Impact | Exposure | Treatment          | Status |
  | RSK-007 | High | High   | Critical | Add code review    | Open   |
```

---

## 11. Traceability — The Spine of ASPICE

Traceability links every artefact back to its origin and forward to its evidence:

```
  Customer need
       │
       ▼
  System requirement  (SYS.2)
       │
       ▼
  Software requirement  (SWE.1)  ←── test case (SWE.6)
       │
       ▼
  Architectural component  (SWE.2)  ←── integration test (SWE.5)
       │
       ▼
  Detailed design / unit  (SWE.3)  ←── unit test (SWE.4)
       │
       ▼
  Source code  (SWE.3)
```

| System Req   | Software Req | Code Module    | Test Case |
|--------------|--------------|----------------|-----------|
| SYS_REQ_01   | SW_REQ_10    | module_init.c  | test_001  |

A **gap in traceability** means either:
- An orphaned requirement (never implemented)
- Dead code (no requirement justifies it)
- An untested requirement (no test coverage)

---

## 12. Work Products (Evidence)

| Process | Key Work Products |
|---------|------------------|
| SWE.1   | Software Requirements Specification (SRS), traceability matrix, review records |
| SWE.2   | Architectural design doc, interface specs, SRS ↔ architecture traceability |
| SWE.3   | Detailed design docs, source code, coding standard compliance, unit test spec |
| SWE.4   | Unit test plan & results, code coverage report, static analysis results |
| SWE.5   | Integration test plan & results, build/integration records |
| SWE.6   | Qualification test plan & results, release note |
| SUP.8   | CM plan, configuration item list (baseline), change history |
| SUP.9   | Problem report log, root cause analysis records |
| MAN.3   | Project plan, progress reports, resource records |

---

## 13. Tools Used in Practice

| Category                | Tools |
|-------------------------|-------|
| Requirements Management | Codebeamer, IBM DOORS, Jama |
| Architecture Design     | Enterprise Architect, MagicDraw |
| Testing                 | VectorCAST, Squish, Jenkins |
| Configuration Management| Git, Gerrit, SVN |

---

## 14. ASPICE Assessment

### Types of Assessment

| Type | Description |
|------|-------------|
| Self-assessment | Supplier assesses own processes — not accepted by most OEMs |
| 2nd party | OEM assesses the supplier — most common in automotive |
| 3rd party | Independent intacs-certified assessor — used for critical safety components |

### Assessment Process

```
  BEFORE                   DURING                    AFTER
  Define scope:            Document review:          Findings report:
  - which processes        check work products       - Ratings per process
  - which projects         exist and are adequate    - Capability level
  - which OUs                                        - Strengths
  Team prepares            Interviews:               - Weaknesses
  evidence                 question engineers        - Action items
                           and managers
                                                     Supplier drives
                           Process observation:      improvement
                           watch team perform task
```

**Rating scale per process attribute:**

| Rating | Name             | Fulfilment |
|--------|------------------|------------|
| N      | Not achieved     | 0–15%      |
| P      | Partially        | 15–50%     |
| L      | Largely          | 50–85%     |
| F      | Fully achieved   | 85–100%    |

To achieve **Level 2**: all Level 1 + Level 2 attributes rated F or L
To achieve **Level 3**: all Level 1–2 + Level 3 attributes rated F or L

### Typical OEM Requirements

| Process | Typical Requirement | Rationale |
|---------|---------------------|-----------|
| SWE.1   | Level 3 | Requirements are the foundation of everything |
| SWE.2   | Level 3 | Architecture errors are costly to fix late |
| SWE.3   | Level 3 | Coding consistency critical |
| SWE.4   | Level 3 | Unit test rigour needed |
| SWE.5   | Level 3 | Integration errors expensive to find late |
| SWE.6   | Level 3 | Qualification = final proof |
| SUP.8   | Level 2–3 | Config management is baseline |
| SUP.9   | Level 2 | Defects must be tracked |
| MAN.3   | Level 2–3 | Project must be managed |

---

## 15. Relationship to ISO 26262 (Functional Safety)

ASPICE and ISO 26262 are **complementary, not overlapping**:

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
```

Both are required for safety-critical automotive software. Evidence often overlaps.

**ASIL level drives ASPICE rigour:**

| ASIL | Requirements |
|------|-------------|
| A    | Basic process discipline |
| B    | Stronger verification evidence, review records |
| C    | Formal methods encouraged, independent review |
| D    | Highest rigour: MC/DC coverage, fully independent testing, complete traceability |

---

## 16. Common Findings in ASPICE Assessments

| Finding | Affected Processes |
|---------|-------------------|
| Requirements not testable (no acceptance criteria) | SWE.1, SYS.2 |
| Traceability gaps (requirements not linked to tests) | SWE.1–6 |
| No bidirectional traceability (only top-down) | SWE.1–6 |
| Code coverage not measured or targets not met | SWE.4 |
| Work products not version controlled | SUP.8, all |
| Defects not logged or root cause not analysed | SUP.9 |
| No regression testing after bug fixes | SWE.4, SWE.5, SWE.6 |
| Reviews not recorded (no minutes, no action items) | SUP.2, SWE.1–6 |
| Project plan not updated during execution | MAN.3 |
| Risk register never updated after project start | MAN.5 |

---

## 17. Full Process Flow

```
  CUSTOMER / OEM
       │
  SYS.1  Requirements Elicitation
       │
  SYS.2  System Requirements Analysis
       │
  SYS.3  System Architectural Design
       │                  │
       │                  └──────────> HWE.1–5 Hardware Engineering
       │
  SWE.1  Software Requirements Analysis
       │
  SWE.2  Software Architectural Design
       │
  SWE.3  Detailed Design & Unit Construction
       │
  SWE.4  Software Unit Verification
       │
  SWE.5  Software Integration Test
       │
  SYS.4  System Integration Test
       │
  SWE.6  Software Qualification Test
       │
  SYS.5  System Qualification Test
       │
  SPL.2  Product Release
       │
  CUSTOMER DELIVERY

  Throughout the entire flow:
  SUP.8  Configuration Management   (every artefact version controlled)
  SUP.9  Problem Resolution         (every defect tracked)
  SUP.10 Change Request Management  (every change controlled)
  SUP.1  Quality Assurance          (audits at each phase)
  MAN.3  Project Management         (plan, monitor, control)
  MAN.5  Risk Management            (risks tracked throughout)
```

---

## Quick Reference

| Term | Meaning |
|------|---------|
| ASPICE | Automotive SPICE — process assessment framework for automotive SW |
| Capability level | 0–5 rating of how well a process is performed and managed |
| Process attribute | Measurable property used to rate capability (PA 1.1, 2.1, 2.2…) |
| Rating | N/P/L/F — Not/Partially/Largely/Fully achieved |
| Work product | Artefact produced by a process (SRS, design doc, test result) |
| Traceability | Links between requirements, design, code, and tests in both directions |
| SRS | Software Requirements Specification — output of SWE.1 |
| V-model | Development model: design on the left, verification on the right |
| OEM | Original Equipment Manufacturer — the car maker |
| Tier 1 / Tier 2 | Direct supplier to OEM / supplier to Tier 1 |
| intacs | International assessor certification scheme for ASPICE assessors |
| ASIL | Automotive Safety Integrity Level A–D (ISO 26262 risk classification) |
| MC/DC | Modified Condition/Decision Coverage — required for ASIL C/D |
| ECU | Electronic Control Unit — embedded computer in a vehicle |
| HARA | Hazard Analysis and Risk Assessment — key ISO 26262 activity |
| CI | Configuration Item — any managed artefact: code, doc, test, tool |
