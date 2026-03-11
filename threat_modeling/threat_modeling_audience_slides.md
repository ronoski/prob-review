# Threat Modeling: IoT Medical Device System
### Technical Security Review — Audience Reference

---

# SLIDE 0 — Methodology Overview

## What Is Threat Modeling?

> A structured, repeatable engineering process for identifying security weaknesses **before** attackers find them.

**Overall framework: PASTA** (Process for Attack Simulation and Threat Analysis)
- Risk-centric — asks *what is the business impact?* not just *what can go wrong?*
- Aligns technical threat analysis with real-world attacker motivation

---

## STRIDE — Threat Enumeration Tool

| Letter | Threat Category        | Property Violated  |
|--------|------------------------|--------------------|
| **S**  | Spoofing               | Authentication     |
| **T**  | Tampering              | Integrity          |
| **R**  | Repudiation            | Non-repudiation    |
| **I**  | Information Disclosure | Confidentiality    |
| **D**  | Denial of Service      | Availability       |
| **E**  | Elevation of Privilege | Authorization      |

## DREAD — Risk Scoring Tool

| Letter | Dimension         | Scored 1–10                          |
|--------|-------------------|--------------------------------------|
| **D**  | Damage            | How bad if exploited?                |
| **R**  | Reproducibility   | How easily repeated?                 |
| **E**  | Exploitability    | How much skill required?             |
| **A**  | Affected Users    | How many impacted?                   |
| **D**  | Discoverability   | How easy to find the vuln?           |

> Score **> 40** = Critical &nbsp;|&nbsp; **25–40** = High &nbsp;|&nbsp; **< 25** = Medium/Low

---

## Today's 12-Step Roadmap

```
 1  System Introduction      →  What is the system?
 2  Asset Identification     →  What are we protecting?
 3  System Decomposition     →  How do components fit together?
 4  Data Flow Diagram        →  What data moves where?
 5  Trust Boundaries         →  Where does trust change hands?
 6  Attack Surface           →  Where can an attacker get in?
 7  STRIDE Enumeration       →  What threats exist per component?
 8  Attacker Mindset         →  Who attacks and why?
 9  Attack Path Analysis     →  How do attacks chain together?
10  Attack Tree              →  Structure of the worst-case attack?
11  DREAD Scoring            →  How bad is each threat?
12  Mitigations              →  What do we do about it?
```

---

# SLIDE 1 — What Are We Modeling?
## Drug Infusion Pump System
> A hospital-deployed IoT system that delivers controlled medication doses directly into patients' bloodstreams.

```
                                                          [NURSE]
                                                             │ PIN · drug · rate
╔════════════════════════════════════════════════════════════▼═══════════════╗
║  HOSPITAL (ONSITE)                                                         ║
║                                                                            ║
║  ┌───────────────────────┐                   ┌──────────────────────────┐ ║
║  │      DRUG PUMP        │◄── infusion cmds ─│                          │ ║
║  │      (bedside)        │     Wi-Fi ⚠️      │    CONTROL SERVER        │ ║
║  │                       │─── sensor/alarms─►│    (server room)         │ ║
║  │  · pump service       │   [UNENCRYPTED]   │                          │ ║
║  │  · OS / RTOS          │                   │  ┌──────────────────────┐│ ║
║  │  · firmware           │                   │  │         RUI          ││ ║
║  └───────────────────────┘                   │  └──────────────────────┘│ ║
║            ▲                                 │  ┌──────────────────────┐│ ║
║            │ [PATIENT] physical              │  │     Drug Library     ││ ║
║                                              │  └──────────────────────┘│ ║
║                                              └──────────────────────────┘ ║
╚═══════════════════════════════════════════╤════════════════════════════════╝
                                            │
                       ┌────────────────────┴───────────────────┐
                       │                                         │
                HTTPS ✓│                         Custom TCP ⚠️  │
            ──► patient │ID · session logs         ──► serial    │
            ◄── PHI     │· allergies · orders      ◄── firmware  │
                        │                          ⚠️ may → PHI  │
                        ▼                                         ▼
                ┌───────────────┐                  ┌─────────────────────┐
                │      EHR      │                  │    UPDATE SERVER    │
                │   (OFFSITE)   │                  │      (OFFSITE)      │
                └───────────────┘                  └─────────────────────┘
```

### The System
- **Drug Infusion Pump** — bedside IoT device physically connected to patient via IV line; delivers medication at a programmed rate over Wi-Fi
- **Control Server** — hospital server room hub; manages a fleet of pumps simultaneously; runs the RUI and all business logic
- **RUI (Restrictive User Interface)** — nurse-facing kiosk app; locked down like an ATM; 4-digit PIN, one shared account for all staff
- **Drug Library** — onsite database defining every drug's approved concentrations, min/max doses, and hard limits; the last safety gate before a command reaches the pump
- **EHR** — offsite patient records system; control server pulls patient context (allergies, physician orders) per session via HTTPS
- **Update Server** — vendor-managed, internet-facing; delivers firmware and drug library updates via a **custom TCP protocol** (not HTTPS)

### ⚠️ Key Context
- Wrong infusion rate = cardiac arrest, respiratory failure, or death — every vulnerability has a potential body count
- Wi-Fi network is **flat and unsegmented** — medical devices, admin, staff, and guest Wi-Fi share the same segment
- A visitor in the lobby can passively capture pump traffic
- IoT constraints: no antivirus, no NAC, RTOS trades security for speed, vendor patches stop within 12 months

---

# SLIDE 2 — System Components

## Full Architecture Breakdown
![Alt Text](sytem_breakdown.png)
```
ONSITE (Inside Hospital)
├── Drug Infusion Pump
│   ├── Pump Service          ← talks to control server
│   ├── Operating System
│   ├── Firmware
│   └── Physical System
│
└── Control Server
    ├── RUI                   ← kiosk app, nurses interact here
    ├── Control Server Service← backend business logic
    ├── Drug Library          ← database of drugs + dose limits
    ├── Operating System
    ├── Firmware
    └── Physical System

OFFSITE (Outside Hospital)
├── EHR                       ← patient records (HTTPS)
└── Update Server             ← vendor updates (Custom TCP ⚠️)
```

### What Each Component Does

| Component | Role |
|---|---|
| Drug Pump | Physically delivers medication to patient |
| Control Server | Manages all pumps; central command hub |
| RUI | Nurse-facing kiosk interface (ATM-like) |
| Drug Library | Defines safe drug doses; last safety gate |
| EHR | Patient records pulled per session |
| Update Server | Delivers software & drug library updates |

---

# SLIDE 3 — Asset Identification

## What Are We Protecting?

| # | Asset | Where It Lives | Why It Matters |
|---|---|---|---|
| 1 | **Patient Safety** | Drug Pump (bedside) | Wrong dose rate = patient death |
| 2 | **Patient Health Data** | EHR · Control Server · Pump logs | PHI under HIPAA; legal & financial liability |
| 3 | **Auth Credentials** | RUI · Control Server · Vendor infra | PIN, API keys, vendor remote access |
| 4 | **Drug Library** | Control Server (onsite DB) | Defines what "safe dose" means to the pump |
| 5 | **Firmware Integrity** | Pump · Control Server hardware | Persists through OS reinstall & disk replacement |
| 6 | **System Availability** | Pump · Control Server · Wi-Fi link | Pump downtime = medical emergency |

## Asset Location Map

```
 ONSITE                                      OFFSITE
 ┌─────────────────────────────────────┐     ┌──────────────────┐
 │                                     │     │                  │
 │  ┌─────────────┐                    │     │  ┌───────────┐   │
 │  │ DRUG PUMP   │                    │     │  │   EHR     │   │
 │  │             │                    │     │  │           │   │
 │  │ [A1] Safety │◄──────────────┐    │     │  │ [A2] PHI  │   │
 │  │ [A5] FW     │               │    │     │  └───────────┘   │
 │  │ [A6] Avail  │               │    │     │                  │
 │  └─────────────┘               │    │     │  ┌───────────┐   │
 │                                │    │     │  │  UPDATE   │   │
 │  ┌─────────────────────────┐   │    │     │  │  SERVER   │   │
 │  │    CONTROL SERVER       │   │    │     │  │           │   │
 │  │                         │───┘    │     │  │ [A3] Cred │   │
 │  │  RUI  → [A3] Credentials│        │     │  └───────────┘   │
 │  │  CSS  → [A6] Avail      │        │     │                  │
 │  │  DB   → [A4] Drug Lib   │        │     └──────────────────┘
 │  │         [A2] PHI        │        │
 │  │  FW   → [A5] Integrity  │        │
 │  └─────────────────────────┘        │
 │                                     │
 └─────────────────────────────────────┘

 Asset Key:
 [A1] Patient Safety    [A2] Patient Health Data   [A3] Auth Credentials
 [A4] Drug Library      [A5] Firmware Integrity    [A6] System Availability
```

## Highest-Value Targets at a Glance

```
  Drug Library  ──►  Redefine "safe dose"  ──►  Patient harmed, no alert
  Firmware      ──►  Persist through wipe  ──►  Undetectable long-term access
  Credentials   ──►  Impersonate nurse     ──►  Unlimited pump control
```

> Every threat we identify maps back to at least one of these six assets.

---

# SLIDE 4 — Data Flow Diagram

## DFD 1: Labeled Data Flows

```
                [NURSE]
                   │
                   │ ① PIN · patient selection · drug · rate config · alarm ack
                   ▼
╔══════════════════════════════ HOSPITAL (ONSITE) ═══════════════════════════════╗
║                                                                                ║
║  ┌──────────────────────────────────────────────────────────────────────────┐ ║
║  │                               RUI                                        │ ║
║  └──────────────────────────────────┬───────────────────────────────────────┘ ║
║                                     │ ② commands · drug queries               ║
║                                     ▼                                          ║
║  ┌──────────────────────────────────────────┐  ③ dose queries  ┌───────────┐ ║
║  │           Control Server Svc             │ ───────────────► │   Drug    │ ║
║  │                                          │ ◄─── params ───  │  Library  │ ║
║  └────────────────────┬─────────────────────┘    credentials   └───────────┘ ║
║                       │                                                        ║
║            ┌────── ④ Wi-Fi ⚠️  [UNENCRYPTED] ──────┐                        ║
║            │  ──► infusion cmds · rate (ml/hr) · OTA │                        ║
║            │  ◄── sensor readings · alarms · status   │                        ║
║            └──────────────────┬──────────────────────┘                        ║
║                               ▼                                                ║
║  ┌──────────────────────────────────────────────────────────────────────────┐ ║
║  │                            DRUG PUMP                                     │ ║
║  └──────────────────────────────────────────────────────────────────────────┘ ║
║                               ▲                                                ║
║                               │ [PATIENT] — physical contact                   ║
╚══════════════════╤════════════════════════════════════════╤════════════════════╝
                   │                                        │
      ⑤ HTTPS ✓   │                        ⑥ Custom TCP ⚠️ │
  ──► patient ID   │                         ──► serial · FW version
  ──► session logs │                         ──► drug lib version
  ◄── PHI          │                         ◄── firmware packages
  ◄── allergies    │                         ◄── drug lib updates
  ◄── orders       │                         ⚠️  may also carry PHI
                   ▼                                        ▼
            ┌─────────────┐                      ┌──────────────────┐
            │     EHR     │                      │  UPDATE SERVER   │
            │  (OFFSITE)  │                      │    (OFFSITE)     │
            └─────────────┘                      └──────────────────┘
```

## DFD 2: Data Flow Inventory

| # | From | To | Protocol | Data | Risk |
|---|---|---|---|---|---|
| 1 | Nurse | RUI | Physical | PIN · patient ID · drug selection | 🔴 |
| 2 | RUI | Control Svc | Internal | User commands · queries | 🟡 |
| 3 | Control Svc | Drug Library | Internal SQL | Dose queries · auth | 🔴 |
| 4 | Drug Library | Control Svc | Internal SQL | Drug params · credentials | 🔴 |
| 5 | Control Server | Drug Pump | **Wi-Fi ⚠️** | Infusion commands · rate · OTA | 🔴 |
| 6 | Drug Pump | Control Server | **Wi-Fi ⚠️** | Sensor data · alarms · status | 🔴 |
| 7 | Control Server | EHR | HTTPS ✓ | Patient queries · infusion logs | 🔴 |
| 8 | EHR | Control Server | HTTPS ✓ | Demographics · allergies · orders | 🔴 |
| 9 | Control Server | Update Server | **Custom TCP ⚠️** | FW version · ⚠️ PHI? | 🔴 |
| 10 | Update Server | Control Server | **Custom TCP ⚠️** | Firmware · drug lib · patches | 🔴 |

## DFD 3: PHI (Patient Data) — Where It Should vs. Should NOT Go

```
  ✅ LEGITIMATE path:
  [EHR] ──HTTPS──► [Control Server] ──internal──► [Pump logs]
                         │
                         └── stays onsite, session ends here

  ⚠️ ACTUAL / POTENTIAL path:
  [EHR] ──HTTPS──► [Control Server] ──Custom TCP──► [Update Server]
                                         OFFSITE · VENDOR-CONTROLLED
                                         NO TLS · NO AUDIT · NO CONTROL
```

### 🚨 First Threat — Found by Following the Data
> Update transaction needs only: **serial number + FW version + drug lib version**
> Anything beyond that = **data minimization violation + HIPAA breach**

---

# SLIDE 5 — Trust Boundaries
![Alt Text](trust_boudary.png)

## Where Trust Changes Hands

Trust boundaries are the lines in our diagram where data crosses from one security context to another. At every boundary, ask:

- **Who is authenticating whom?**
- **Who is authorizing the action?**
- **Is the data validated?**
- **Can it be tampered with in transit?**

```
  [NURSE] ──PIN──► [RUI] ──commands──► [Control Server Svc] ──SQL──► [Drug Library]
                     │                          │
              ❶ 4-digit PIN            ┌────────┴──────────┐
                Shared account         │                    │
                No individual      Wi-Fi ⚠️           HTTPS ✓
                  identity         Unencrypted          ❹ Relatively safe
                                       │                    │
                                  ❸ [DRUG PUMP]          [EHR]
                                   No device cert      CS compromise =
                                   MITM possible       EHR lateral path

                              Custom TCP ⚠️
                              No TLS · No auth
                                       │
                                ❺ [UPDATE SERVER]
                                  Highest-risk external boundary
                                  MITM = firmware injection
```

| # | Boundary | Protocol | Key Weakness | Risk |
|---|----------|----------|--------------|------|
| ❶ | Nurse → RUI | 4-digit PIN | Shared account — no individual identity; 10,000 combinations | 🔴 |
| ❷ | RUI → Control Server | Internal | CS trusts RUI blindly — kiosk escape collapses this entirely | 🟡 |
| ❸ | Control Server → Pump | Wi-Fi (unencrypted) | No crypto device identity — any node on segment can impersonate either side | 🔴 |
| ❹ | Control Server → EHR | HTTPS | Reasonably protected — but CS compromise opens lateral path to EHR | 🟢 |
| ❺ | Control Server → Update Server | Custom TCP | No TLS, custom protocol, unclear auth — attacker on flat network can MITM and inject firmware | 🔴 |
| ❻ | Onsite → Offsite | Mixed | PHI should never cross this boundary — update channel may silently carry it | 🔴 |

### Key Points
- **Boundary ❶**: 4-digit PIN = 10,000 combinations; common PINs (0000, 1234, 1111) broken in minutes; one shared account means zero forensic attribution
- **Boundary ❷**: The RUI restricts what a nurse *sees* — it does not restrict what the underlying OS can *do*; escape the kiosk and the trust assumption disappears
- **Boundary ❸**: Most dangerous internal boundary — no cryptographic device certificates means a laptop on the hospital Wi-Fi can impersonate either the pump or the server
- **Boundary ❺**: Most dangerous external boundary — sits on a flat, unsegmented network; custom protocol has had no security scrutiny; vendor update infrastructure is a single point of compromise for every hospital running this equipment
- **Boundary ❻**: Data minimization failure — the update transaction needs only a serial number and version string; anything else crossing this line is a HIPAA breach

---

## When Trust Assumptions Break — How Attackers Exploit Them

Designers build trust assumptions into every boundary. Attackers look for where those assumptions are **wrong**.

| Trust Assumption Made | Reality | How Attacker Breaks It |
|-----------------------|---------|------------------------|
| "The nurse entering the PIN is a legitimate staff member" | PIN is 4 digits, shared across all staff, no lockout delay | Shoulder-surf or brute-force the PIN; every nurse becomes the same identity |
| "The RUI controls everything a user can do" | RUI is a UI layer — the OS underneath has no awareness of RUI restrictions | Connect a USB keyboard, trigger Alt-F4 or Sticky Keys, escape to OS shell |
| "Commands arriving over Wi-Fi come from the control server" | No cryptographic device certificates; identity is IP/MAC-based | Sit on the same Wi-Fi segment, spoof the control server's MAC/IP, send commands directly to the pump |
| "The pump readings arriving at the control server are real" | Same unencrypted Wi-Fi channel, no message authentication | MITM the Wi-Fi — intercept sensor readings, substitute falsified values, forward to control server |
| "The update server is a trusted vendor system" | Internet-facing, custom protocol, unclear authentication | Compromise the update server, inject malicious firmware — every hospital pulling updates gets the payload |
| "Patient data stays onsite" | Update channel carries more than version strings | Passively monitor the Custom TCP stream; PHI exits the hospital with no TLS, no audit trail |

### The Pattern
- **Designers assume** the environment enforces the boundary (network isolation, UI restriction, trusted vendor)
- **Attackers assume** nothing is enforced unless proven cryptographically
- Every boundary in this system relies on at least one environmental assumption that the flat, unsegmented hospital network actively violates

---

# SLIDE 6 — Attack Surface

## Every Entry Point an Attacker Can Touch

### Network
- [ ] Wi-Fi interface on pump — open to hospital network
- [ ] Wi-Fi interface on control server
- [ ] Custom TCP channel to update server (bidirectional)
- [ ] Management ports: SSH, RDP, web admin

### Physical
- [ ] USB ports on pump and control server
- [ ] JTAG debug interface on circuit board
- [ ] External storage slots (SD cards)
- [ ] Physical power supply

### Software
- [ ] RUI — accepts user input, kiosk-escape vectors
- [ ] Control server API — unauthenticated endpoints
- [ ] Drug Library — SQL-accessible via RUI
- [ ] Firmware OTA update mechanism

### Human
- [ ] Nurse — social engineering, PIN observation
- [ ] Patient — physical pump access
- [ ] IT Admin — elevated internal access
- [ ] Vendor technician — remote support credentials

---

# SLIDE 7 — STRIDE Framework

## Six Threat Categories Applied to Every Component

| Letter | Threat | Question to Ask |
|---|---|---|
| **S** | Spoofing | Can someone pretend to be a legitimate component? |
| **T** | Tampering | Can data or system state be modified? |
| **R** | Repudiation | Can actions be denied or logs be falsified? |
| **I** | Info Disclosure | Can sensitive data be leaked? |
| **D** | Denial of Service | Can availability be disrupted? |
| **E** | Elevation of Privilege | Can someone gain higher access than allowed? |

---

# SLIDE 8 — STRIDE: RUI

| STRIDE | Threat | Detail |
|---|---|---|
| **S** Spoofing | Weak PIN auth | 4-digit PIN = 10,000 combinations; shared across all nurses |
| **T** Tampering | Kiosk escape | Alt-F4, Windows key, Sticky Keys → access to underlying OS |
| **R** Repudiation | No individual identity | Single shared account; logs cannot attribute actions to a person |
| **I** Info Disclosure | Debug error messages | SQL errors reveal DB tech, internal IPs, service names |
| **D** DoS | Brute-force lockout | 5 wrong PINs locks out medical team during emergency |
| **E** EoP | Vendor backdoor | Remote support service may have shared or no credentials |

---

# SLIDE 9 — STRIDE: Control Server Service

| STRIDE | Threat | Detail |
|---|---|---|
| **S** Spoofing | Pump impersonation | Server identifies pumps by IP/MAC → anyone on network can fake a pump |
| **T** Tampering | MITM attack | Attacker modifies sensor readings in transit on flat Wi-Fi |
| **R** Repudiation | World-writable logs | Any process can overwrite or inject fake log entries |
| **I** Info Disclosure | Data minimization failure | Patient data unnecessarily sent to vendor update server |
| **D** DoS | Wi-Fi jamming | Jammer severs control server ↔ pump link |
| **E** EoP | Unauthenticated API | Internal API endpoints exposed without auth bypass RUI entirely |

---

# SLIDE 10 — STRIDE: Drug Library

| STRIDE | Threat | Detail |
|---|---|---|
| **S** Spoofing | DB user impersonation | SQL injection to execute queries as privileged DB user |
| **T** Tampering | SQL injection | Modify max dose limits; nurse programs lethal infusion unaware |
| **R** Repudiation | Log injection | Line-feed chars in input create fake log entries |
| **I** Info Disclosure | Out-of-band SQL exfil | DNS-based data exfiltration via stored procedures |
| **D** DoS | Resource exhaustion | Complex queries (nested joins, Cartesian products) crash DB |
| **E** EoP | `xp_cmdshell` | DB stored procedures execute OS commands with elevated privilege |

---

# SLIDE 11 — STRIDE: Firmware & Physical

### Device Firmware

| STRIDE | Threat |
|---|---|
| **T** Tampering | APT malware in firmware — survives OS reinstall and disk replacement |
| **I** Info Disclosure | SMRAM read via exposed management API — reveals crypto keys |
| **D** DoS | Block OTA updates — fleet stays on vulnerable firmware forever |
| **E** EoP | Hardcoded default credentials in firmware management interface |

### Physical Equipment

| STRIDE | Threat |
|---|---|
| **S** Spoofing | Supply chain attack — malicious hardware before delivery |
| **T** Tampering | USB HID injection; JTAG firmware extraction/modification |
| **I** Info Disclosure | Side-channel attacks (EM, power); cold-boot key recovery |
| **D** DoS | Deliberate power disruption |
| **E** EoP | Race conditions in embedded CPU → arbitrary memory write |

---

# SLIDE 12 — Attacker Mindset

## They Don't Attack Components — They Break Trust Assumptions

| Assumption Designers Made | Reality |
|---|---|
| "Hospital network provides isolation" | Network is flat — no segmentation |
| "Update server is trusted" | If vendor is compromised, every hospital is compromised |
| "RUI limits the attack surface" | RUI sits on a full OS it cannot control |
| "Internal device comms are trusted" | Wi-Fi is broadcast — any node can listen |

## Four Attacker Archetypes

```
┌─────────────────────┬──────────────────────────────────────────────┐
│ Attacker            │ Goal & Method                                │
├─────────────────────┼──────────────────────────────────────────────┤
│ Reckless Nurse      │ Unintentional — kiosk shortcut leaves system │
│                     │ exposed; no malicious intent                 │
├─────────────────────┼──────────────────────────────────────────────┤
│ Hospital Thief      │ Opportunistic — 20 sec physical access,      │
│                     │ USB drive, sells patient data                │
├─────────────────────┼──────────────────────────────────────────────┤
│ Data Miner          │ Sophisticated — targets control server as    │
│                     │ weaker path to EHR patient records at scale  │
├─────────────────────┼──────────────────────────────────────────────┤
│ Govt. Cyber Warrior │ Nation-state — compromises vendor update     │
│                     │ server to hit entire hospital fleet at once  │
└─────────────────────┴──────────────────────────────────────────────┘
```

---

# SLIDE 13 — Attack Path 1
## SQL Injection → Drug Library Manipulation

```
Step 1  Connect to hospital guest Wi-Fi (no auth required)
   │
Step 2  Passive scan → discover control server IP from broadcast
   │
Step 3  Navigate to RUI → error message leaks DB technology
   │
Step 4  Fuzz input fields → confirm SQL injection
        Payload: ' OR '1'='1
   │
Step 5  Enumerate schema
        Table: drug_library | Columns: drug_name, max_dose_mg
   │
Step 6  ⚠️ Modify drug limit
        UPDATE drug_library SET max_dose_mg=100
        WHERE drug_name='morphine';
   │
Step 7  Nurse programs normal morphine infusion
        System accepts it — within "limits"
        Patient receives 10× intended dose
   │
Step 8  Restore original value
        Logs were world-writable → no forensic evidence
```
> ⏱️ **Time to execute: under 1 hour for a competent attacker**

---

# SLIDE 14 — Attack Path 2
## Update Server Compromise → Fleet-Wide Firmware Persistence

```
Step 1  Identify vendor → find internet-facing update server
   │
Step 2  Exploit vulnerability in update server (SQLi / RCE)
        Gain access to update distribution system
   │
Step 3  Inject malicious code into firmware update package
        Sign with stolen vendor code-signing key
        (or devices don't verify signatures at all)
   │
Step 4  All hospitals requesting updates receive trojanized firmware
        Malware installs at firmware level
        Survives OS reinstall ✓  Survives disk replacement ✓
   │
Step 5  Malware beacons to attacker C2 infrastructure
        Persistent access across every hospital in vendor's fleet
   │
Step 6  On command:
        → Disable all pumps simultaneously
        → Manipulate drug delivery
        → Exfiltrate patient records at scale
```
> 🎯 **This is the SolarWinds model applied to medical IoT**

---

# SLIDE 15 — Attack Tree

![Alt Text](attack_tree.png)

**How to read this tree:**
- **OR node** → attacker needs **any one** branch to succeed
- **AND node** → attacker needs **all** branches to succeed

---

# SLIDE 16 — Risk Evaluation: DREAD Scoring

## DREAD = Damage · Reproducibility · Exploitability · Affected Users · Discoverability

### Threat 1: Weak 4-Digit PIN

| Category | Score | Note |
|---|---|---|
| Damage | 5 | One patient at a time |
| Reproducibility | 10 | Always works once PIN known |
| Exploitability | 10 | No tools needed |
| Affected Users | 5 | Single patient per session |
| Discoverability | 10 | Visible on RUI immediately |
| **Total** | **8 / 10** | 🔴 Critical |

### Threat 2: SQL Injection in Drug Library

| Category | Score | Note |
|---|---|---|
| Damage | 10 | Lethal dose modification |
| Reproducibility | 8 | Reliable once found |
| Exploitability | 7 | Moderate skill; tools available |
| Affected Users | 9 | All patients on this server |
| Discoverability | 7 | Basic app testing reveals it |
| **Total** | **8.2 / 10** | 🔴 Critical |

### Threat 3: Unencrypted Pump Protocol

| Category | Score | Note |
|---|---|---|
| Damage | 9 | MITM enables falsified readings |
| Reproducibility | 9 | Any time attacker is on network |
| Exploitability | 6 | Protocol analysis needed |
| Affected Users | 8 | All pumps on the network |
| Discoverability | 8 | Passive Wi-Fi capture |
| **Total** | **8 / 10** | 🔴 Critical |

> ⚠️ These three threats **chain together** — each one amplifies the others.

---

# SLIDE 17 — Mitigation Checklist

## Priority Actions

### 🔴 P1 — Authentication
- [ ] Replace 4-digit PIN with individual TOTP-based 2FA
- [ ] Individual accounts per staff member — no shared credentials
- [ ] Vendor remote access: on-demand only, time-limited, nurse-approved

### 🔴 P2 — Network Segmentation
- [ ] Isolate pump traffic on dedicated, isolated VLAN
- [ ] Block guest Wi-Fi access to medical device segments
- [ ] Deploy Network Access Control (NAC)

### 🔴 P3 — Encrypt Everything in Transit
- [ ] Replace custom TCP update protocol with TLS 1.3 + mutual auth
- [ ] Encrypt pump ↔ control server channel with message auth codes
- [ ] Add nonces + sequence numbers to pump protocol (prevent replay)

### 🟠 P4 — Input Validation
- [ ] Parameterized queries everywhere — no string concatenation in SQL
- [ ] Disable `xp_cmdshell` and external-network stored procedures
- [ ] Principle of least privilege on DB service account

### 🟠 P5 — Device Identity & Firmware Integrity
- [ ] Cryptographic certificates on every pump
- [ ] Secure Boot enforced on pump and control server
- [ ] JTAG disabled in production builds

### 🟡 P6 — Logging & Audit
- [ ] Write-once or remote append-only log storage
- [ ] Per-user audit trails on all RUI actions
- [ ] Real-time log forwarding to SIEM

### 🟡 P7 — Physical Security
- [ ] Control server in badge-access locked room
- [ ] USB ports disabled or epoxy-sealed
- [ ] Tamper-evident chassis seals; TPM-based hardware attestation

### 🟡 P8 — Data Minimization
- [ ] Update transaction sends only: serial number + firmware version + drug library version
- [ ] No patient data ever crosses the onsite → offsite boundary via update channel
- [ ] Data flow audit mandatory in change control process

---

# SLIDE 18 — Key Takeaways

## The Three Questions of Threat Modeling

```
┌────────────────────────────────────────────────────────────┐
│  1. WHERE does trust change hands?                         │
│     → Every boundary is a potential attack path           │
├────────────────────────────────────────────────────────────┤
│  2. WHAT happens if that trust is violated?                │
│     → Map it to assets: safety, data, credentials,        │
│       firmware, availability                               │
├────────────────────────────────────────────────────────────┤
│  3. WHAT IS THE WORST CASE if this component is            │
│     fully compromised?                                     │
│     → Work backwards from impact to attack path           │
└────────────────────────────────────────────────────────────┘
```

## Threat Modeling Is a Discipline, Not a Checkbox

| When to do it | Why |
|---|---|
| At design time | Cheapest to fix — nothing is built yet |
| When architecture changes | New components = new trust boundaries |
| When new features are added | New user flows = new attack paths |
| After an incident | Validate your model against reality |

---

> **Think like an attacker. Build like a defender.**

---
*Companion reference for: Threat Modeling an IoT Medical Device System*
*Sections map 1:1 to the presenter's talk script*
