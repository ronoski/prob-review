# TLS 1.3 — Complete Reference
### Cryptography · Certificates · Handshake · Key Exchange · Cipher Suites

---

## 1. Why TLS Exists

Plain HTTP sends everything as readable text. Anyone on the path can read or modify it.

```
WITHOUT TLS (HTTP)

  Browser ──────► Router ──────► ISP ──────► Server
          "password=secret123"
                    ▲
                    Anyone here can read or modify this
                    (wiretapping, rogue Wi-Fi, ISP logging, MITM)

WITH TLS (HTTPS)

  Browser ──────► Router ──────► ISP ──────► Server
          "x8Kp#@!$mQ3z9..."
                    ▲
                    Encrypted — meaningless without session keys
```

**TLS provides three guarantees:**

```
╔═══════════════════╦═══════════════════════════════════════════════════╗
║ Confidentiality   ║ Data is encrypted — only sender and receiver      ║
║                   ║ can read it                                        ║
╠═══════════════════╬═══════════════════════════════════════════════════╣
║ Integrity         ║ Data cannot be modified in transit without         ║
║                   ║ detection (via AEAD authentication tag)            ║
╠═══════════════════╬═══════════════════════════════════════════════════╣
║ Authentication    ║ Server (and optionally client) identity is         ║
║                   ║ verified via digital certificates                  ║
╚═══════════════════╩═══════════════════════════════════════════════════╝
```

---

## 2. Where TLS Sits in the Stack

```
  ┌──────────────────────────────────────────────────────────┐
  │  Application Layer    │  HTTP, SMTP, FTP, IMAP           │
  ├──────────────────────────────────────────────────────────┤
  │  TLS 1.3              │  ◄── sits here, wraps all        │
  │                       │      application data            │
  ├──────────────────────────────────────────────────────────┤
  │  Transport Layer      │  TCP (TLS runs over TCP)          │
  ├──────────────────────────────────────────────────────────┤
  │  Network Layer        │  IP                              │
  ├──────────────────────────────────────────────────────────┤
  │  Data Link            │  Ethernet, Wi-Fi                 │
  └──────────────────────────────────────────────────────────┘

  TLS is not a separate OSI layer — it operates between Application
  and Transport, transparently encrypting all data before it hits TCP.
```

**TLS has two internal phases:**

| Phase | Name | Purpose | Crypto Used |
|-------|------|---------|-------------|
| 1 | **Handshake Protocol** | Authentication + key exchange | Asymmetric (ECDHE, signatures) |
| 2 | **Record Protocol** | Bulk data encryption | Symmetric (AES-GCM, ChaCha20) |

> The Handshake is heavy but happens once. The Record Protocol is lightweight and runs for every byte transferred.

---

## 3. Cryptography Foundations

### 3.1 Symmetric Encryption

Same key encrypts and decrypts. Fast. Used for bulk data after the handshake.

```
  Plaintext + Key ──► [Encrypt] ──► Ciphertext
  Ciphertext + Key ──► [Decrypt] ──► Plaintext

  Alice                                    Bob
    │── "Hello" encrypted with Key123 ────►│
    │   ciphertext: "x8Kp@!"              │  Bob decrypts with same Key123
    │                                      │  Bob reads: "Hello"

  ⚠️  Problem: How do Alice and Bob agree on Key123 without
      an attacker intercepting it?  →  The Key Exchange Problem
```

### 3.2 Asymmetric Encryption (Public Key Cryptography)

Two mathematically linked keys: **public key** (share freely) and **private key** (never share).

```
  ┌─────────────────────┐        ┌──────────────────────┐
  │    Public Key       │        │    Private Key        │
  │    (share freely)   │        │    (never share)      │
  └─────────────────────┘        └──────────────────────┘
          │                               │
   Anyone can encrypt              Only owner can decrypt
   with public key                 with private key

  Alice wants to send Bob a secret:
  ┌──────────────────────────────────────────────────────┐
  │  Alice:  "Hello" + Bob's Public Key → "x8Kp@!"      │
  │  Bob:    "x8Kp@!" + Bob's Private Key → "Hello" ✓   │
  │  Attacker has "x8Kp@!" but not private key → ✗      │
  └──────────────────────────────────────────────────────┘
  Algorithms: RSA, ECC (Elliptic Curve Cryptography)
```

### 3.3 Hashing

One-way function — produces a fixed-size fingerprint. Cannot be reversed.

```
  "Hello World"  ──SHA-256──► a591a6d40bf420404a011733cfb7b190...
  "Hello world"  ──SHA-256──► 64ec88ca00b268e5ba1a35678a1b5316...
                               ▲
                               One bit change = completely different output
                               (avalanche effect)

  Properties:
  ✔ Same input always → same output
  ✔ Tiny change → completely different output
  ✔ Cannot reverse to get original input
  ✔ Collision resistant

  Used in TLS for: integrity checks, digital signatures, certificate fingerprints
```

### 3.4 Digital Signatures

Proves a message came from a specific sender and was not modified.

```
  SIGNING (sender uses PRIVATE key):
  ┌──────────────────────────────────────────────────────────────┐
  │  Message ──► [Hash] ──► Digest ──► [Sign with PRIVATE key]  │
  │                                         │                    │
  │                                      Signature              │
  └──────────────────────────────────────────────────────────────┘

  VERIFYING (receiver uses PUBLIC key):
  ┌──────────────────────────────────────────────────────────────┐
  │  Message ──► [Hash] ──────────────────────► Digest A        │
  │  Signature ──► [Verify with PUBLIC key] ──► Digest B        │
  │                                                              │
  │  Digest A == Digest B  →  ✅ Valid, authentic, unmodified    │
  │  Digest A != Digest B  →  ❌ Tampered or wrong sender       │
  └──────────────────────────────────────────────────────────────┘

  Example:
  Alice signs "Pay Bob $100"  →  signature "xyz789"
  Attacker changes to "Pay Eve $100"  →  hash mismatch  →  INVALID ❌
```

---

## 4. Digital Certificates

A digital certificate binds a **public key** to an **identity** (domain name, organisation).

**Why certificates matter:**

```
  WITHOUT CERTIFICATES — MITM attack succeeds:
  ┌─────────────────────────────────────────────────────────────┐
  │  Browser ◄──────────► Attacker ◄──────────► Real Server    │
  │                ▲                                            │
  │   Attacker presents their own public key,                   │
  │   pretending to be the real server.                         │
  │   Browser encrypts with attacker's key → attacker reads all │
  └─────────────────────────────────────────────────────────────┘

  WITH CERTIFICATES — MITM attack fails:
  ┌─────────────────────────────────────────────────────────────┐
  │  Browser ◄──────────► Attacker ◄──────────► Real Server    │
  │                ▲                                            │
  │   Attacker cannot forge a cert signed by a trusted CA.      │
  │   Browser rejects the connection. Attack fails. ✅          │
  └─────────────────────────────────────────────────────────────┘
```

### 4.1 Certificate Contents (X.509 Format)

```
  ┌─────────────────────────────────────────────────────┐
  │  X.509 Certificate                                  │
  ├─────────────────────────────────────────────────────┤
  │  Version:          3                                │
  │  Serial Number:    0x4A3B2C1D...                    │
  │  Subject:          CN=example.com                   │  ← who this belongs to
  │                    O=Example Corp, C=US             │
  │  Issuer:           CN=DigiCert TLS CA               │  ← who signed it
  │  Valid From:       2024-01-01                       │
  │  Valid Until:      2025-01-01                       │
  │  Public Key:       ECDSA P-256                      │  ← server's public key
  │                    [key bytes...]                   │
  │  Subject Alt Names: example.com                     │  ← domains covered
  │                     www.example.com                 │
  │  Key Usage:        Digital Signature                │
  │  CA Signature:     [signature bytes]                │  ← CA's approval stamp
  └─────────────────────────────────────────────────────┘
```

### 4.2 Certificate Authority & Chain of Trust

```
  ╔═══════════════════════════════════════════════════════════╗
  ║  CHAIN OF TRUST                                           ║
  ╠═══════════════════════════════════════════════════════════╣
  ║                                                           ║
  ║  ┌─────────────────────────────┐                         ║
  ║  │       ROOT CA               │  Self-signed            ║
  ║  │  "DigiCert Root CA"         │  Pre-installed in       ║
  ║  │  (trusted anchor)           │  OS / browser           ║
  ║  └──────────────┬──────────────┘                         ║
  ║                 │ signs                                   ║
  ║                 ▼                                         ║
  ║  ┌─────────────────────────────┐                         ║
  ║  │    INTERMEDIATE CA          │  Issued by Root CA      ║
  ║  │  "DigiCert TLS CA"          │  Keeps Root offline     ║
  ║  └──────────────┬──────────────┘  and safe               ║
  ║                 │ signs                                   ║
  ║                 ▼                                         ║
  ║  ┌─────────────────────────────┐                         ║
  ║  │   END-ENTITY CERTIFICATE    │  Presented during       ║
  ║  │   "example.com"             │  TLS handshake          ║
  ║  └─────────────────────────────┘                         ║
  ║                                                           ║
  ║  Browser validates: example.com ← signed by → Intermediate║
  ║                     Intermediate ← signed by → Root CA   ║
  ║                     Root CA ← in trust store → ✅ TRUSTED ║
  ║  Any break in chain → ❌ connection rejected              ║
  ╚═══════════════════════════════════════════════════════════╝
```

### 4.2.1 Client-Side Trust Chain Verification

When the server sends its certificate during the TLS handshake, the client does **not** just accept it. It walks the chain from bottom to top, verifying each link cryptographically.

```
  ╔══════════════════════════════════════════════════════════════════════╗
  ║  CLIENT TRUST CHAIN VERIFICATION (bottom-up)                         ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                      ║
  ║  Server sends during TLS handshake:                                  ║
  ║  [ example.com cert ] + [ Intermediate CA cert ]                     ║
  ║  (Root CA is NOT sent — client already has it in trust store)        ║
  ║                                                                      ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                      ║
  ║  STEP 1 — Verify server cert signature                               ║
  ║                                                                      ║
  ║  ┌─────────────────────────┐     ┌─────────────────────────┐        ║
  ║  │   example.com cert      │     │   Intermediate CA cert  │        ║
  ║  │   Signature: [sig_A]    │     │   Public Key: [pub_int] │        ║
  ║  └────────────┬────────────┘     └──────────────┬──────────┘        ║
  ║               │                                 │                   ║
  ║               └──── Verify sig_A using pub_int ─┘                   ║
  ║                              │                                       ║
  ║                    ✅ Match → proceed   ❌ Fail → reject             ║
  ║                                                                      ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                      ║
  ║  STEP 2 — Verify Intermediate CA signature                           ║
  ║                                                                      ║
  ║  ┌─────────────────────────┐     ┌─────────────────────────┐        ║
  ║  │   Intermediate CA cert  │     │   Root CA cert          │        ║
  ║  │   Signature: [sig_B]    │     │   Public Key: [pub_root]│        ║
  ║  └────────────┬────────────┘     └──────────────┬──────────┘        ║
  ║               │                                 │                   ║
  ║               └──── Verify sig_B using pub_root ┘                   ║
  ║                              │                                       ║
  ║                    ✅ Match → proceed   ❌ Fail → reject             ║
  ║                                                                      ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                      ║
  ║  STEP 3 — Check Root CA is in local trust store                      ║
  ║                                                                      ║
  ║  ┌──────────────────────────────────────────────────────────┐        ║
  ║  │  OS / Browser Trust Store                                │        ║
  ║  │  (pre-installed, maintained by Apple / Microsoft /       │        ║
  ║  │   Mozilla / Google)                                      │        ║
  ║  │                                                          │        ║
  ║  │  DigiCert Root CA ✅  │  Let's Encrypt Root ✅           │        ║
  ║  │  Comodo Root CA   ✅  │  GlobalSign Root    ✅  ...      │        ║
  ║  └──────────────────────────────────────────────────────────┘        ║
  ║                              │                                       ║
  ║               Root CA found in trust store? ✅ → TRUSTED             ║
  ║               Root CA not found?            ❌ → UNTRUSTED           ║
  ║                                                                      ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                      ║
  ║  RESULT: All 3 steps pass → chain of trust established ✅            ║
  ║          Client can now trust the server's public key                ║
  ║          and proceed with ECDHE key exchange                         ║
  ║                                                                      ║
  ╚══════════════════════════════════════════════════════════════════════╝
```

**What the client is actually doing at each step:**

- It takes the certificate's content (fields, public key, etc.) and **hashes** it
- It takes the **signature** on that cert and **decrypts it** using the issuer's public key — this gives back the original hash
- If both hashes match → the cert was genuinely signed by the issuer and has not been tampered with
- This is repeated for every link up to the Root CA

**Why the Root CA is not sent by the server:**

The client already has the Root CA pre-installed in its OS/browser trust store. Sending it would be redundant — and more importantly, if an attacker is intercepting traffic, they could not fake a root cert that the client trusts, since the trust store is controlled by the OS/browser, not the network.

**What breaks the chain:**

```
  ┌─────────────────────────────────────────────────────────────┐
  │  Self-signed cert    → Root CA not in trust store     ❌    │
  │  Expired cert        → Valid Until date passed        ❌    │
  │  Wrong domain        → SAN does not match hostname    ❌    │
  │  Revoked cert        → CRL / OCSP says invalid        ❌    │
  │  Tampered cert       → Signature verification fails   ❌    │
  │  Missing intermediate→ Chain incomplete, root unreachable ❌│
  └─────────────────────────────────────────────────────────────┘
```

### 4.3 Certificate Types

| Type | Validation | Time | Shows | Used By |
|------|-----------|------|-------|---------|
| **DV** — Domain Validation | Prove domain control only | Minutes | Padlock | Most websites, Let's Encrypt |
| **OV** — Organisation Validation | Domain + org existence | 1–3 days | Org name in cert | Companies |
| **EV** — Extended Validation | Domain + org + legal + address | ~1 week | Cert details | Banks, payment processors |
| **Wildcard** | `*.example.com` | Any | All subdomains | Multi-subdomain sites |

### 4.4 Certificate Validation Steps

```
  Browser receives server certificate and checks in order:

  ┌─ 1. SIGNATURE VALID? ────────────────────────────────────────┐
  │  Verify CA's signature on cert using CA's public key         │
  │  ❌ Invalid → NET::ERR_CERT_AUTHORITY_INVALID                │
  └──────────────────────────────────────────────────────────────┘
                    │ ✅
  ┌─ 2. CHAIN COMPLETE? ─────────────────────────────────────────┐
  │  Follow chain from server cert up to a trusted root CA       │
  │  ❌ Broken → NET::ERR_CERT_AUTHORITY_INVALID                 │
  └──────────────────────────────────────────────────────────────┘
                    │ ✅
  ┌─ 3. NOT EXPIRED? ────────────────────────────────────────────┐
  │  Check Valid From / Valid Until dates                        │
  │  ❌ Expired → NET::ERR_CERT_DATE_INVALID                     │
  └──────────────────────────────────────────────────────────────┘
                    │ ✅
  ┌─ 4. DOMAIN MATCHES? ─────────────────────────────────────────┐
  │  Subject / SAN must match the URL hostname                   │
  │  ❌ Mismatch → NET::ERR_CERT_COMMON_NAME_INVALID             │
  └──────────────────────────────────────────────────────────────┘
                    │ ✅
  ┌─ 5. NOT REVOKED? ────────────────────────────────────────────┐
  │  Check CRL or OCSP for revocation                           │
  │  ❌ Revoked → NET::ERR_CERT_REVOKED                          │
  └──────────────────────────────────────────────────────────────┘
                    │ ✅
            Certificate trusted ✅ → proceed with handshake
```

### 4.5 Certificate Revocation

```
  If a private key is stolen, the certificate must be invalidated early.

  ┌──────────────────────────────────────────────────────────┐
  │  Method 1: CRL (Certificate Revocation List)             │
  │  CA publishes a periodic list of revoked serial numbers. │
  │  ⚠️  Downside: can be stale, large to download           │
  ├──────────────────────────────────────────────────────────┤
  │  Method 2: OCSP (Online Certificate Status Protocol)     │
  │  Browser queries CA in real time: "Is this cert valid?"  │
  │  CA responds: Good / Revoked / Unknown                   │
  ├──────────────────────────────────────────────────────────┤
  │  Method 3: OCSP Stapling (best)                          │
  │  Server queries OCSP itself, attaches signed response    │
  │  to the TLS handshake. No extra round trip for client.   │
  └──────────────────────────────────────────────────────────┘
```

---

## 5. Key Exchange — From RSA to ECDHE

### 5.1 RSA Key Exchange (TLS 1.2 and earlier — removed in TLS 1.3)

**How it works:** The client generates a random secret, encrypts it with the server's public key from the certificate, and sends it over. The server decrypts it with its private key. Both sides now have the same secret and derive session keys from it.

```
  Client                                        Server
    │                                             │
    │◄──────────── Certificate ───────────────────│
    │              (contains server public key)   │
    │                                             │
    │  Generate random Pre-Master Secret (PMS)    │
    │  Encrypt PMS with server's public key       │
    │─────────── Encrypted PMS ──────────────────►│
    │                                             │
    │              Server decrypts PMS            │
    │              with private key               │
    │                                             │
    │  Both derive session keys from PMS          │
    │════════════ Encrypted Data ════════════════►│

  ❌ FATAL FLAW: No Forward Secrecy
  If attacker records traffic today, then later steals the server's
  private key → they can decrypt ALL past sessions.
  (private key → decrypt PMS → derive session key → read everything)
```

**Why this is dangerous — "record now, decrypt later":**

The server's RSA private key is long-lived — it stays the same for months or years (tied to the certificate). This means:

1. An attacker (e.g. a state-level actor, ISP, rogue Wi-Fi) passively records all your HTTPS traffic today
2. Years later, they obtain the server's private key (breach, court order, key expiry mismanagement)
3. They go back and decrypt **every past session** — because every session's PMS was encrypted with that same key

This is the "harvest now, decrypt later" attack. The entire security of the session rests on one static key, forever.

**Why RSA key exchange was removed in TLS 1.3:**
- The private key is the single point of failure for all past and future sessions
- Session keys are not independent — compromise one key → compromise everything
- ECDHE (next section) solves this by generating a fresh, throwaway key pair per session

### 5.2 Diffie-Hellman Key Exchange (DH / ECDHE)

DH allows two parties to agree on a shared secret **over a public channel** without ever transmitting it.

```
  ╔══════════════════════════════════════════════════════════════╗
  ║  ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)             ║
  ╠══════════════════════════════════════════════════════════════╣
  ║                                                              ║
  ║  G = agreed base point on the elliptic curve (public)       ║
  ║      G is a fixed constant defined in the curve standard     ║
  ║      (e.g. X25519 RFC) — both sides know it the same way    ║
  ║      both sides know what "port 443" means: agreed upon      ║
  ║      before the connection starts, not exchanged on the wire ║
  ║                                                              ║
  ║  CLIENT                              SERVER                  ║
  ║  ──────                              ──────                  ║
  ║  picks random scalar: a              picks random scalar: b  ║
  ║  computes: A = aG                    computes: B = bG        ║
  ║                                                              ║
  ║  ─────── ClientHello (A) ──────────────────────────────►    ║
  ║  ◄────── ServerHello  (B) ──────────────────────────────    ║
  ║                                                              ║
  ║  Client computes: S = aB = a(bG) = abG                      ║
  ║  Server computes: S = bA = b(aG) = abG                      ║
  ║                           ▲                                  ║
  ║                    Both arrive at the same S.                ║
  ║                    S was NEVER sent over the wire.           ║
  ╚══════════════════════════════════════════════════════════════╝
```

### 5.3 Forward Secrecy — Why "Ephemeral" Matters

```
  ┌───────────────────────────┐    ┌───────────────────────────┐
  │  WITHOUT Forward Secrecy  │    │  WITH Forward Secrecy     │
  │  (RSA key exchange)       │    │  (ECDHE)                  │
  ├───────────────────────────┤    ├───────────────────────────┤
  │  Session 1 key derived    │    │  Session 1: temp key A    │
  │  from server private key  │    │  (discarded after session)│
  │                           │    │                           │
  │  Session 2 key derived    │    │  Session 2: temp key B    │
  │  from server private key  │    │  (discarded after session)│
  │                           │    │                           │
  │  Attacker records all.    │    │  Attacker records all.    │
  │  Later steals private key │    │  Later steals private key │
  │  → Decrypts ALL sessions! │    │  → Cannot decrypt any!    │
  │  ❌                       │    │  Temp keys are long gone. │
  └───────────────────────────┘    │  ✅                       │
                                   └───────────────────────────┘
  TLS 1.3 mandates ECDHE — forward secrecy is non-negotiable.
```

---

## 6. TLS 1.2 vs TLS 1.3

```
  ┌──────────────────────┬──────────────────────┬────────────────────────┐
  │  Feature             │  TLS 1.2             │  TLS 1.3               │
  ├──────────────────────┼──────────────────────┼────────────────────────┤
  │  Handshake RTT       │  2 RTT               │  1 RTT (0-RTT option)  │
  │  Forward Secrecy     │  Optional            │  Mandatory             │
  │  Key Exchange        │  RSA or ECDHE        │  ECDHE only            │
  │  Encryption          │  AES-CBC or AES-GCM  │  AEAD only             │
  │  Weak ciphers        │  RC4, 3DES allowed   │  All removed           │
  │  Certificate data    │  Partly plaintext    │  Fully encrypted       │
  │  Downgrade attack    │  Possible            │  Prevented             │
  │  Cipher suite count  │  Hundreds            │  5 (all strong)        │
  └──────────────────────┴──────────────────────┴────────────────────────┘
```

**What TLS 1.3 removed:** RSA key exchange, static DH, MD5, SHA-1, RC4, 3DES, AES-CBC, compression, renegotiation.

---

## 7. TLS 1.3 Handshake — Full Detailed Flow

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                        TLS 1.3 HANDSHAKE                                    ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║   CLIENT                                                    SERVER           ║
║   ──────                                                    ──────           ║
║                                                                              ║
║  ── TCP HANDSHAKE (SYN / SYN-ACK / ACK) ────────────────────────────────   ║
║                                                                              ║
║ ┌─── STEP 1 ────────────────────────────────────────────────────────────┐  ║
║ │                                                                        │  ║
║ │  ClientHello ──────────────────────────────────────────────────────►  │  ║
║ │  [PLAINTEXT]                                                           │  ║
║ │  · TLS version: 1.3                                                    │  ║
║ │  · client_random (32 bytes)                                            │  ║
║ │  · cipher suites: TLS_AES_128_GCM_SHA256, TLS_CHACHA20_POLY1305...    │  ║
║ │  · key_share: client ECDHE public key (X25519)                         │  ║
║ │  · SNI extension: "example.com"                                        │  ║
║ │  · supported_groups: x25519, P-256                                     │  ║
║ │                                                                        │  ║
║ └────────────────────────────────────────────────────────────────────────┘  ║
║                                                                              ║
║ ┌─── STEP 2 ────────────────────────────────────────────────────────────┐  ║
║ │                                                                        │  ║
║ │  ◄──────────────────────────────────────────────────  ServerHello     │  ║
║ │  [PLAINTEXT]                                                           │  ║
║ │  · server_random (32 bytes)                                            │  ║
║ │  · key_share: server ECDHE public key (X25519)                         │  ║
║ │  · cipher suite chosen: TLS_AES_256_GCM_SHA384                         │  ║
║ │                                                                        │  ║
║ └────────────────────────────────────────────────────────────────────────┘  ║
║                                                                              ║
║ ┌─── BOTH SIDES COMPUTE SHARED SECRET ──────────────────────────────────┐  ║
║ │  Client: SharedSecret = a × B                                          │  ║
║ │  Server: SharedSecret = b × A   →  Both get same abG                  │  ║
║ │  HKDF derives Handshake Keys from SharedSecret                         │  ║
║ │  All remaining handshake messages are NOW ENCRYPTED ──────────────────►│  ║
║ └────────────────────────────────────────────────────────────────────────┘  ║
║                                                                              ║
║ ┌─── STEP 3 ────────────────────────────────────────────────────────────┐  ║
║ │                                                                        │  ║
║ │  ◄────────────────────────────── [ENCRYPTED] EncryptedExtensions      │  ║
║ │  · ALPN: h2 (HTTP/2)                                                   │  ║
║ │  · other server-side parameters                                        │  ║
║ │                                                                        │  ║
║ │  ◄────────────────────────────── [ENCRYPTED] Certificate              │  ║
║ │  · server's cert chain (example.com → Intermediate CA → Root CA)      │  ║
║ │                                                                        │  ║
║ │  ◄────────────────────────────── [ENCRYPTED] CertificateVerify        │  ║
║ │  · signature over entire handshake transcript                          │  ║
║ │  · proves server holds the private key matching the certificate        │  ║
║ │                                                                        │  ║
║ │  ◄────────────────────────────── [ENCRYPTED] Finished                 │  ║
║ │  · HMAC of entire handshake transcript                                 │  ║
║ │  · proves handshake was not tampered with                              │  ║
║ │                                                                        │  ║
║ └────────────────────────────────────────────────────────────────────────┘  ║
║                                                                              ║
║ ┌─── CLIENT VERIFICATION ───────────────────────────────────────────────┐  ║
║ │  ✓ cert chain traces to trusted root CA                                │  ║
║ │  ✓ cert not expired                                                    │  ║
║ │  ✓ SNI "example.com" matches cert SAN                                  │  ║
║ │  ✓ CertificateVerify signature valid                                   │  ║
║ │  ✓ Finished HMAC valid                                                 │  ║
║ └────────────────────────────────────────────────────────────────────────┘  ║
║                                                                              ║
║ ┌─── HKDF DERIVES APPLICATION KEYS ─────────────────────────────────────┐  ║
║ │  Master Secret → Client Write Key + Server Write Key + IVs            │  ║
║ └────────────────────────────────────────────────────────────────────────┘  ║
║                                                                              ║
║ ┌─── STEP 4 ────────────────────────────────────────────────────────────┐  ║
║ │                                                                        │  ║
║ │  Finished ──────────────────────────────────────────────────────────► │  ║
║ │  [ENCRYPTED]                                                           │  ║
║ │  · client's HMAC of handshake transcript                               │  ║
║ │                                                                        │  ║
║ └────────────────────────────────────────────────────────────────────────┘  ║
║                                                                              ║
║ ─── APPLICATION DATA PHASE ──────────────────────────────────────────────   ║
║                                                                              ║
║  GET /api HTTP/1.1 ────────────────────── [AES-256-GCM ENCRYPTED] ──────►  ║
║  ◄──────────────────────────────────────────────── HTTP 200 OK (encrypted) ║
║                                                                              ║
╠══════════════════════════════════════════════════════════════════════════════╣
║  Total: 1 RTT before data flows  │  Certificate hidden from observers       ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## 8. TLS 1.2 vs TLS 1.3 Handshake Side-by-Side

```
  TLS 1.2 (2 RTT)                         TLS 1.3 (1 RTT)
  ────────────────────────────────         ──────────────────────────────
  Client                Server             Client                Server
    │                     │                  │                     │
    │── ClientHello ──────►│                  │── ClientHello ──────►│
    │                     │                  │   + ECDHE key share  │
    │◄─ ServerHello ───────│                  │                     │
    │◄─ Certificate ───────│                  │◄─ ServerHello ───────│
    │◄─ ServerKeyExchange ─│                  │◄─ EncryptedExt ──────│  [ENCRYPTED]
    │◄─ ServerHelloDone ───│                  │◄─ Certificate ───────│  [ENCRYPTED]
    │                     │                  │◄─ CertVerify ────────│  [ENCRYPTED]
    │── ClientKeyExchange ►│                  │◄─ Finished ──────────│  [ENCRYPTED]
    │── ChangeCipherSpec ─►│                  │                     │
    │── Finished ─────────►│                  │── Finished ─────────►│  [ENCRYPTED]
    │◄─ ChangeCipherSpec ──│                  │                     │
    │◄─ Finished ──────────│                  │══ Application Data ══│
    │══ Application Data ══│                  │                     │
    │                     │
  2 round trips required               1 round trip required
  Certificate sent in plaintext        Certificate fully encrypted
```

---

## 9. Key Derivation Chain (HKDF)

After ECDHE produces the shared secret, both sides independently derive the same keys using HKDF (HMAC-based Key Derivation Function). Neither side transmits these keys.

```
  ╔══════════════════════════════════════════════════════════════╗
  ║           TLS 1.3 KEY SCHEDULE (HKDF)                        ║
  ╠══════════════════════════════════════════════════════════════╣
  ║                                                              ║
  ║   PSK (or 0 if none)                                         ║
  ║          │                                                   ║
  ║          ▼                                                   ║
  ║   ┌─────────────────┐                                        ║
  ║   │  Early Secret   │  ← used for 0-RTT session resumption  ║
  ║   └────────┬────────┘                                        ║
  ║            │ + ECDHE Shared Secret                           ║
  ║            ▼                                                  ║
  ║   ┌─────────────────────┐                                    ║
  ║   │  Handshake Secret   │  ← encrypts handshake messages    ║
  ║   └──────────┬──────────┘                                    ║
  ║              ├──► Client Handshake Traffic Key               ║
  ║              └──► Server Handshake Traffic Key               ║
  ║              │                                               ║
  ║              ▼                                               ║
  ║   ┌─────────────────┐                                        ║
  ║   │  Master Secret  │  ← encrypts application data          ║
  ║   └────────┬────────┘                                        ║
  ║            ├──► Client Application Write Key                 ║
  ║            ├──► Server Application Write Key                 ║
  ║            ├──► Client Write IV (nonce)                      ║
  ║            └──► Server Write IV (nonce)                      ║
  ║                                                              ║
  ║  Keys are symmetric — both sides independently compute them. ║
  ║  Discarded after session ends → Forward Secrecy ✅           ║
  ╚══════════════════════════════════════════════════════════════╝
```

### 9.1 What Each Derived Key Is Used For

TLS derives **multiple separate keys** — one key per direction, per phase. This is called **key separation**: using the same key for multiple purposes is a security risk.

---

**Early Secret** — Input: PSK from previous session, or `0` if none.
- Powers **0-RTT resumption**: client encrypts early application data before the handshake completes, saving 1 RTT on reconnect
- Powers **session tickets**: server encrypts a resumption ticket so the client can skip the full handshake later
- ⚠️ 0-RTT data has **no forward secrecy** and is **replay-vulnerable** — only safe for idempotent requests (e.g. GET, not POST)

---

**Handshake Secret** — Input: Early Secret + ECDHE Shared Secret.

Splits into two directional keys:

- **Client Handshake Traffic Key** (Client → Server): encrypts the client's `Finished` message
- **Server Handshake Traffic Key** (Server → Client): encrypts `EncryptedExtensions`, `Certificate`, `CertificateVerify`, `Finished`

> In TLS 1.2, the Certificate and CertificateVerify were sent in plaintext. TLS 1.3 encrypts them — an attacker on the wire cannot see which certificate the server is using.

---

**Master Secret** — Input: Handshake Secret (after both Finished messages pass).

Splits into four values:

- **Client Application Write Key**: encrypts all HTTP requests, passwords, cookies (Client → Server)
- **Server Application Write Key**: encrypts all HTTP responses, tokens, API data (Server → Client)
- **Client Write IV**: nonce counter for AEAD — XORed with sequence number (0, 1, 2…) to guarantee uniqueness per record
- **Server Write IV**: same purpose for server direction

> Reusing a nonce with the same key catastrophically breaks AEAD — XORing two ciphertexts made with the same key+nonce leaks the XOR of their plaintexts. TLS 1.3 prevents this by design.

---

### 9.2 Key Summary Table

| Derived Key | Direction | Protects | Phase |
|---|---|---|---|
| Client Handshake Traffic Key | Client → Server | `Finished` | Handshake |
| Server Handshake Traffic Key | Server → Client | `EncryptedExtensions`, `Certificate`, `CertificateVerify`, `Finished` | Handshake |
| Client Application Write Key | Client → Server | All HTTP requests, credentials | Application |
| Server Application Write Key | Server → Client | All HTTP responses, tokens | Application |
| Client Write IV | Client → Server | AEAD nonce counter | Application |
| Server Write IV | Server → Client | AEAD nonce counter | Application |

**Security properties of all derived keys:**
- Never transmitted — both sides compute the same keys independently via HKDF
- Separate keys per direction — prevents cross-direction forgery
- Separate keys per phase — handshake keys expire before application keys activate
- All discarded after session ends → **Forward Secrecy**: stealing the server's private key later cannot decrypt old captured sessions

---

## 10. Record Protocol & AEAD

After the handshake, every byte of application data travels as a **TLS Record**. The Record Protocol is the "worker" — it runs continuously for the entire session, protecting every HTTP request and response.

### 10.1 The Two Phases of TLS

```
  ╔══════════════════════════════════════════════════════════════════╗
  ║  HANDSHAKE PROTOCOL         │  RECORD PROTOCOL                  ║
  ╠══════════════════════════════════════════════════════════════════╣
  ║  Runs once at connection     │  Runs for every byte of data      ║
  ║  start                       │  sent or received                 ║
  ╠══════════════════════════════════════════════════════════════════╣
  ║  Uses asymmetric crypto      │  Uses symmetric crypto            ║
  ║  (ECDHE, certificates)       │  (AES-GCM, ChaCha20)              ║
  ╠══════════════════════════════════════════════════════════════════╣
  ║  Computationally heavy       │  Computationally light            ║
  ║  (elliptic curve math)       │  (optimized for speed)            ║
  ╠══════════════════════════════════════════════════════════════════╣
  ║  Goal: establish trust       │  Goal: protect data in transit    ║
  ║  and agree on keys           │  using those agreed keys          ║
  ╚══════════════════════════════════════════════════════════════════╝
```

### 10.2 TLS Record Structure

Every chunk of data is wrapped in a TLS Record before being sent over TCP:

```
  ╔══════════════════════════════════════════════════════════════════╗
  ║  TLS RECORD (wire format)                                        ║
  ╠══════════╦══════════════════════════════════════════════════════╣
  ║  HEADER  ║  Content Type  (1 byte)  — handshake / alert / data  ║
  ║  5 bytes ║  TLS Version   (2 bytes) — 0x0303 (TLS 1.2 compat)  ║
  ║          ║  Length        (2 bytes) — length of payload below   ║
  ╠══════════╬══════════════════════════════════════════════════════╣
  ║ PAYLOAD  ║  Ciphertext    (variable) — encrypted application    ║
  ║          ║                             data + content type byte ║
  ╠══════════╬══════════════════════════════════════════════════════╣
  ║  AUTH    ║  Auth Tag      (16 bytes) — AEAD integrity proof     ║
  ║  TAG     ║                             covers header + payload  ║
  ╚══════════╩══════════════════════════════════════════════════════╝

  Max record size: 16,384 bytes (16 KB)
  Large data (e.g. file download) is split into multiple records.
```

> In TLS 1.3 the real content type (application_data, handshake, alert) is **hidden inside the encrypted payload**. The header always shows `0x17` (application_data) to prevent traffic analysis of what type of data is flowing.

### 10.3 AEAD — Authenticated Encryption with Associated Data

AEAD combines **encryption** and **integrity** in a single operation. You get both confidentiality and tamper detection from one algorithm — no separate HMAC step needed.

```
  ╔══════════════════════════════════════════════════════════════════╗
  ║  SENDING (AEAD Encrypt)                                          ║
  ╠══════════════════════════════════════════════════════════════════╣
  ║                                                                  ║
  ║   Plaintext ──────────────────────────────────────┐             ║
  ║   Session Key (Application Write Key) ────────────┤             ║
  ║   Nonce (Write IV XOR sequence number) ───────────┤             ║
  ║   Associated Data (TLS record header) ────────────┤             ║
  ║                                                   ▼             ║
  ║                                           ┌──────────────┐      ║
  ║                                           │  AEAD Cipher │      ║
  ║                                           │  (AES-GCM or │      ║
  ║                                           │  ChaCha20)   │      ║
  ║                                           └──────┬───────┘      ║
  ║                                                  │              ║
  ║                                    ┌─────────────┴──────────┐   ║
  ║                                    │ Ciphertext  │ Auth Tag  │   ║
  ║                                    │  (payload)  │ (16 bytes)│   ║
  ║                                    └─────────────────────────┘   ║
  ║                                    ← sent over the wire ────►   ║
  ╚══════════════════════════════════════════════════════════════════╝

  ╔══════════════════════════════════════════════════════════════════╗
  ║  RECEIVING (AEAD Verify + Decrypt)                               ║
  ╠══════════════════════════════════════════════════════════════════╣
  ║                                                                  ║
  ║   Ciphertext + Auth Tag ──────────────────────────┐             ║
  ║   Session Key ────────────────────────────────────┤             ║
  ║   Nonce ──────────────────────────────────────────┤             ║
  ║   Associated Data ────────────────────────────────┤             ║
  ║                                                   ▼             ║
  ║                                           ┌──────────────┐      ║
  ║                                           │  Recompute   │      ║
  ║                                           │  Auth Tag    │      ║
  ║                                           └──────┬───────┘      ║
  ║                                                  │              ║
  ║                            ┌─────────────────────┴──────────┐   ║
  ║                            │  Tags match? ✅ → Plaintext    │   ║
  ║                            │  Tags differ? ❌ → DROP record │   ║
  ║                            │                    close conn  │   ║
  ║                            └────────────────────────────────┘   ║
  ╚══════════════════════════════════════════════════════════════════╝
```

### 10.4 Why Auth Tag Is Essential

Encryption alone does **not** prevent modification. An attacker who cannot read the ciphertext can still flip bits and change the plaintext after decryption.

```
  WITHOUT AUTH TAG (broken):

  Plaintext:   "Transfer $100 to Alice"
                      │
               [Encrypt]
                      │
  Ciphertext:  a8 f1 3d 92 c4 07 ...
                      │
  Attacker flips bit in ciphertext (no auth check)
                      │
  Receiver decrypts:  "Transfer $900 to Alice"  ← silently corrupted ❌


  WITH AUTH TAG (AEAD):

  Plaintext:   "Transfer $100 to Alice"
                      │
               [AEAD Encrypt]
                      │
  Ciphertext + Auth Tag: a8 f1 3d 92 ...  |  8c 2e 9f 41 ...
                      │
  Attacker flips bit in ciphertext
                      │
  Receiver recomputes tag → does NOT match stored tag
  → Record dropped, connection closed             ✅ tamper detected
```

### 10.5 Associated Data — What Gets Authenticated

The "Associated Data" (AD) in AEAD is the TLS record **header** — content type, version, length. It is **authenticated but not encrypted**.

This means:
- An attacker cannot modify the header silently (e.g. change the length field to cause parsing errors)
- The header stays readable for routing and parsing purposes
- Any modification to the header breaks the auth tag → connection dropped

```
  ┌──────────────────────────────────────────────────┐
  │  Authenticated (not encrypted) → header          │
  │  · content type                                  │
  │  · TLS version                                   │
  │  · record length                                 │
  ├──────────────────────────────────────────────────┤
  │  Authenticated + Encrypted → payload             │
  │  · actual application data                       │
  │  · true content type (hidden in TLS 1.3)         │
  └──────────────────────────────────────────────────┘
```

### 10.6 Nonce — Why It Cannot Repeat

The nonce must be **unique for every record** using the same key. If a nonce is reused:

```
  Record A: Encrypt("GET /login",  key K, nonce N)  →  C_A
  Record B: Encrypt("password=x", key K, nonce N)  →  C_B

  Attacker computes: C_A XOR C_B = "GET /login" XOR "password=x"
  → With known-plaintext, attacker recovers both messages ❌
```

TLS 1.3 prevents this by deriving a unique nonce per record:

```
  Nonce = Write IV  XOR  (64-bit sequence number)

  Record 0:  Nonce = IV XOR 0x0000000000000000
  Record 1:  Nonce = IV XOR 0x0000000000000001
  Record 2:  Nonce = IV XOR 0x0000000000000002
  ...
  Always unique as long as fewer than 2^64 records per session.
```

### 10.7 AES-GCM vs ChaCha20-Poly1305

TLS 1.3 allows two AEAD ciphers:

```
  ╔═══════════════════╦═══════════════════════╦═══════════════════════╗
  ║                   ║  AES-GCM              ║  ChaCha20-Poly1305    ║
  ╠═══════════════════╬═══════════════════════╬═══════════════════════╣
  ║  Speed            ║  Very fast with       ║  Fast in software     ║
  ║                   ║  AES-NI hardware      ║  (no hardware needed) ║
  ╠═══════════════════╬═══════════════════════╬═══════════════════════╣
  ║  Best for         ║  Servers, desktops    ║  Mobile, embedded,    ║
  ║                   ║  with AES-NI CPU      ║  IoT devices          ║
  ╠═══════════════════╬═══════════════════════╬═══════════════════════╣
  ║  Auth component   ║  GCM (Galois/Counter) ║  Poly1305 MAC         ║
  ╠═══════════════════╬═══════════════════════╬═══════════════════════╣
  ║  Key size         ║  128-bit or 256-bit   ║  256-bit              ║
  ╠═══════════════════╬═══════════════════════╬═══════════════════════╣
  ║  Security         ║  ✅ Strong            ║  ✅ Strong            ║
  ╚═══════════════════╩═══════════════════════╩═══════════════════════╝

  TLS 1.3 negotiates which one to use during the handshake (cipher suite).
  The client and server pick the best one both support.
```

---

## 11. Cipher Suites

A cipher suite is a **named combination of algorithms** used for a TLS session.

### TLS 1.2 Cipher Suite Anatomy

```
  TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
   │    │     │        │    │   │   │
   │    │     │        │    │   │   └── Hash for PRF (SHA-384)
   │    │     │        │    │   └────── Mode of operation (GCM)
   │    │     │        │    └────────── Key size (256-bit)
   │    │     │        └─────────────── Bulk cipher (AES)
   │    │     └──────────────────────── Authentication (RSA cert)
   │    └────────────────────────────── Key Exchange (ECDHE)
   └─────────────────────────────────── Protocol (TLS)
```

### TLS 1.3 Cipher Suite Anatomy

```
  TLS_AES_128_GCM_SHA256
   │    │    │   │   │
   │    │    │   │   └── Hash for key derivation (SHA-256)
   │    │    │   └─────── Mode of operation / auth (GCM)
   │    │    └─────────── Key size (128-bit)
   │    └──────────────── Bulk cipher (AES)
   └───────────────────── Protocol (TLS)

  TLS_CHACHA20_POLY1305_SHA256
   │    │        │       │
   │    │        │       └── Hash for key derivation (SHA-256)
   │    │        └─────────── Auth tag algorithm (Poly1305)
   │    └──────────────────── Bulk cipher (ChaCha20)
   └───────────────────────── Protocol (TLS)
```

Notice what is **missing** compared to TLS 1.2:
- No key exchange algorithm (ECDHE is always used — mandatory)
- No authentication algorithm (certificate type is negotiated separately)
- Far fewer moving parts → less room for misconfiguration

```
  ┌───────────────────────────────────┬────────────────────────────────────┐
  │  TLS_AES_128_GCM_SHA256           │  Default, fast, widely supported   │
  │  TLS_AES_256_GCM_SHA384           │  Stronger security                 │
  │  TLS_CHACHA20_POLY1305_SHA256     │  No hardware AES needed (mobile)   │
  └───────────────────────────────────┴────────────────────────────────────┘
```

### Encryption Modes Compared

```
  ┌──────────────────┬────────────────────────────────────────────────────┐
  │  AES-CBC         │  Old. Encryption and integrity separate.           │
  │  (removed 1.3)   │  Padding required → POODLE, padding oracle attacks │
  ├──────────────────┼────────────────────────────────────────────────────┤
  │  AES-GCM         │  Modern AEAD. Encryption + auth in one operation.  │
  │  ✅ TLS 1.3      │  Hardware-accelerated (AES-NI). Parallelisable.    │
  ├──────────────────┼────────────────────────────────────────────────────┤
  │  ChaCha20-       │  Modern AEAD. No hardware needed.                  │
  │  Poly1305        │  Preferred on mobile/IoT/ARM without AES-NI.       │
  │  ✅ TLS 1.3      │  Used in WireGuard, Signal protocol.               │
  └──────────────────┴────────────────────────────────────────────────────┘
```

---

## 12. SNI — Server Name Indication

One IP address often hosts many domains. SNI tells the server which certificate to send.

```
  Single IP: 203.0.113.50 hosts:
  ├── example.com
  ├── api.example.com
  └── shop.example.com

  ┌──────────────────────────────────────────────────────────┐
  │  ClientHello includes SNI extension (PLAINTEXT):         │
  │                                                          │
  │  · cipher suites: [...]                                  │
  │  · client_random: [...]                                  │
  │  · Extensions:                                           │
  │      SNI: "shop.example.com"  ◄── visible to observers! │
  │      ALPN: h2, http/1.1                                  │
  │      key_share: [ECDHE public key]                       │
  └──────────────────────────────────────────────────────────┘

  Server reads SNI → picks correct certificate → proceeds.

  ⚠️  SNI leaks the hostname to anyone observing the connection.
  🔒  ECH (Encrypted Client Hello) in TLS 1.3 extensions addresses this.
```

---

## 13. 0-RTT Session Resumption

Returning clients can send data with the very first message using a session ticket from a previous connection.

```
  First connection (normal 1-RTT):
  Client ──► ClientHello ──────────────────────► Server
  Client ◄── ServerHello + ... + SessionTicket ── Server
  Client ──► Finished ──────────────────────────► Server

  Resumed connection (0-RTT):
  Client ──► ClientHello + EarlyData ──────────► Server
             (data sent immediately, no wait)
  Client ◄── ServerHello + ... ─────────────────── Server

  ┌──────────────────────────────────────────────────────────┐
  │  ✅  Benefit: Zero latency for returning clients          │
  │  ⚠️  Risk:   0-RTT data is vulnerable to replay attacks  │
  │              (attacker re-sends the first flight)        │
  │  ✅  Mitigation: Only use 0-RTT for idempotent requests  │
  │                 (GET, not POST/payment)                   │
  └──────────────────────────────────────────────────────────┘
```

---

## 14. Full HTTPS Connection — End to End

```
  Browser visits https://example.com

  ╔═══════════════════════════════════════════════════════════════╗
  ║  STEP 1 — DNS                                                 ║
  ║  Resolve example.com → 93.184.216.34                          ║
  ╚═══════════════════════════════════════════════════════════════╝
                       │
  ╔═══════════════════════════════════════════════════════════════╗
  ║  STEP 2 — TCP Handshake                                       ║
  ║  SYN ──► SYN-ACK ──► ACK           (1 RTT)                   ║
  ╚═══════════════════════════════════════════════════════════════╝
                       │
  ╔═══════════════════════════════════════════════════════════════╗
  ║  STEP 3 — TLS 1.3 Handshake                                   ║
  ║                                                               ║
  ║  ClientHello (SNI=example.com + ECDHE key share) ──────────► ║
  ║  ◄───── ServerHello (ECDHE key share)                         ║
  ║  ◄───── [ENCRYPTED] Certificate chain                         ║
  ║  ◄───── [ENCRYPTED] CertificateVerify                         ║
  ║  ◄───── [ENCRYPTED] Finished                                  ║
  ║                                                               ║
  ║  Client verifies cert, computes shared secret, derives keys   ║
  ║  Finished ────────────────────────────────────────────────►  ║
  ║                                               (1 RTT)         ║
  ╚═══════════════════════════════════════════════╪═══════════════╝
                       │           RTT = 100ms → Total so far: ~200ms
  ╔═══════════════════════════════════════════════╪═══════════════╗
  ║  STEP 4 — Encrypted HTTP Request              │               ║
  ║  GET / HTTP/1.1  [AES-256-GCM] ──────────────►               ║
  ╚═══════════════════════════════════════════════════════════════╝
                       │
  ╔═══════════════════════════════════════════════════════════════╗
  ║  STEP 5 — Encrypted HTTP Response                             ║
  ║  ◄──── HTTP/1.1 200 OK + HTML  [AES-256-GCM]                 ║
  ╚═══════════════════════════════════════════════════════════════╝
```

---

## 15. Common TLS Attacks & Mitigations

| Attack | What It Does | Mitigation in TLS 1.3 |
|--------|-------------|----------------------|
| **MITM** | Intercept with rogue cert | Certificate chain validation |
| **BEAST** | CBC IV predictability | CBC removed entirely |
| **POODLE** | Downgrade to SSL 3.0/CBC | SSL 3.0 + CBC removed |
| **HEARTBLEED** | Read server memory via OpenSSL bug | Patch; unrelated to TLS version |
| **SWEET32** | Birthday attack on 3DES | 3DES removed |
| **Downgrade** | Force weaker TLS version | TLS_FALLBACK_SCSV; 1.3 rejects old versions |
| **Replay (0-RTT)** | Re-send first flight of data | Avoid 0-RTT for non-idempotent ops |
| **Cert spoofing** | Forge trusted cert | CAA DNS records, Certificate Transparency |

---

## 16. TLS Errors Reference

TLS errors occur when any step of certificate validation or the handshake fails. Each error maps directly to one of the validation checks.

### Certificate Errors (shown in browser)

```
  Error                              Cause
  +--------------------------------+------------------------------------------------+
  | NET::ERR_CERT_AUTHORITY_INVALID | Certificate signed by an unknown or           |
  | "Your connection is not         | untrusted CA. Common causes:                  |
  |  private"                       | - Self-signed cert (no CA at all)             |
  |                                 | - Private CA root not installed on client     |
  |                                 | - Incomplete certificate chain sent by server |
  +--------------------------------+------------------------------------------------+
  | NET::ERR_CERT_DATE_INVALID      | Certificate is expired or not yet valid.      |
  | "Certificate has expired"       | Most common cause: admin forgot to renew.     |
  |                                 | Let's Encrypt certs expire every 90 days.     |
  +--------------------------------+------------------------------------------------+
  | NET::ERR_CERT_COMMON_NAME_      | Domain in URL does not match Subject / SAN    |
  | INVALID                         | in certificate. Common causes:                |
  | "Certificate name mismatch"     | - Visiting https://192.168.1.1 but cert       |
  |                                 |   says router.local                           |
  |                                 | - www.example.com cert used for api.example   |
  |                                 | - Wildcard *.example.com used for example.com |
  |                                 |   (root not covered by wildcard)              |
  +--------------------------------+------------------------------------------------+
  | NET::ERR_CERT_REVOKED           | Certificate was revoked by the CA before      |
  |                                 | its expiry date (private key compromised,     |
  |                                 | mis-issue, or company dissolved).             |
  +--------------------------------+------------------------------------------------+
  | NET::ERR_CERT_WEAK_SIGNATURE_   | Certificate signed with weak algorithm        |
  | ALGORITHM                       | (MD5 or SHA-1) — no longer trusted by         |
  |                                 | modern browsers.                              |
  +--------------------------------+------------------------------------------------+
  | HSTS error (hard block,         | Site previously declared HTTPS-only via       |
  | no bypass button)               | Strict-Transport-Security header. Browser     |
  |                                 | refuses HTTP or invalid HTTPS absolutely.     |
  +--------------------------------+------------------------------------------------+
```

### Handshake Errors

```
  Error                              Cause
  +--------------------------------+------------------------------------------------+
  | SSL_ERROR_HANDSHAKE_FAILURE_    | Client and server share no common cipher       |
  | ALERT                          | suite or TLS version. Example: server          |
  | "SSL handshake failed"          | supports only TLS 1.3, client is very old.    |
  +--------------------------------+------------------------------------------------+
  | ERR_SSL_VERSION_OR_CIPHER_      | No overlapping TLS version between client     |
  | MISMATCH                        | and server. Server may have disabled TLS 1.2  |
  |                                 | but client doesn't support TLS 1.3.           |
  +--------------------------------+------------------------------------------------+
  | SSL_ERROR_BAD_CERT_DOMAIN       | SNI hostname sent by client does not match    |
  |                                 | any certificate the server holds.             |
  +--------------------------------+------------------------------------------------+
  | ERR_SSL_PROTOCOL_ERROR          | General handshake failure — server sent       |
  |                                 | malformed data, version mismatch, or          |
  |                                 | middlebox (proxy/firewall) interfered.        |
  +--------------------------------+------------------------------------------------+
  | certificate_unknown (TLS alert) | Server sent a certificate the client could    |
  |                                 | not process or validate.                      |
  +--------------------------------+------------------------------------------------+
  | bad_certificate (TLS alert)     | mTLS: client certificate was rejected by      |
  |                                 | server (wrong CA, revoked, malformed).        |
  +--------------------------------+------------------------------------------------+
```

### Error Flow — What Happens When Validation Fails

```
  Browser connects to https://example.com
         |
         v
  Receives certificate
         |
         v
  Check 1: Chain traces to trusted root?
         |          |
        YES         NO -----> NET::ERR_CERT_AUTHORITY_INVALID
         |                    (self-signed or unknown CA)
         v
  Check 2: Not expired?
         |          |
        YES         NO -----> NET::ERR_CERT_DATE_INVALID
         |
         v
  Check 3: Domain matches?
         |          |
        YES         NO -----> NET::ERR_CERT_COMMON_NAME_INVALID
         |
         v
  Check 4: Not revoked?
         |          |
        YES         NO -----> NET::ERR_CERT_REVOKED
         |
         v
  Check 5: Cipher suite / version agreed?
         |          |
        YES         NO -----> ERR_SSL_VERSION_OR_CIPHER_MISMATCH
         |
         v
  Handshake completes --> Encrypted connection established
```

---

## 17. Quick Mental Model

```
  TLS 1.3 in five steps:

  ┌─────┬──────────────────────────────────────────────────────────┐
  │  1  │  Agree on encryption  →  ClientHello / ServerHello        │
  │  2  │  Generate shared secret  →  ECDHE (never transmitted)     │
  │  3  │  Derive all keys  →  HKDF chain                           │
  │  4  │  Authenticate  →  Certificate + CertVerify + Finished     │
  │  5  │  Exchange encrypted data  →  AES-GCM / ChaCha20-Poly1305  │
  └─────┴──────────────────────────────────────────────────────────┘
```

---

## 18. Common TLS Vulnerabilities in Automotive Systems

Modern vehicles are software-defined systems — ECUs, telematics units, OTA update servers, mobile apps, and V2X infrastructure all rely on TLS. But automotive environments have unique constraints (long lifetimes, resource-limited ECUs, offline operation, safety-critical payloads) that create vulnerabilities not common in web applications.

---

### 18.1 Certificate Management Failures

```
  ╔══════════════════════════════════════════════════════════════════════╗
  ║  AUTOMOTIVE CERTIFICATE PROBLEMS                                      ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                      ║
  ║  Problem 1: Hardcoded / Shared Certificates                          ║
  ║  ─────────────────────────────────────────────────────────────────  ║
  ║  Manufacturer ships all vehicles with the same TLS certificate       ║
  ║  and private key embedded in firmware.                               ║
  ║                                                                      ║
  ║  Attack:  One vehicle compromised → private key extracted from       ║
  ║           flash memory → attacker can impersonate backend server     ║
  ║           to ALL vehicles of that model                              ║
  ║                                                                      ║
  ║  Problem 2: Expired Certificates, No Update Mechanism               ║
  ║  ─────────────────────────────────────────────────────────────────  ║
  ║  Vehicle cert expires after 1–3 years. No automated renewal.        ║
  ║  Vehicle goes offline for months (stored, fleet, infrequent use).   ║
  ║                                                                      ║
  ║  Attack:  TLS connection rejected → OTA update fails → vehicle      ║
  ║           stuck on old firmware with known vulnerabilities           ║
  ║           OR developer disables cert validation to "fix" it ❌       ║
  ║                                                                      ║
  ║  Problem 3: Self-Signed Certificates with No Revocation              ║
  ║  ─────────────────────────────────────────────────────────────────  ║
  ║  ECU uses self-signed cert, no PKI, no CRL/OCSP endpoint.           ║
  ║  If private key is stolen → no way to invalidate it.                ║
  ║  Attacker can impersonate the ECU indefinitely.                      ║
  ║                                                                      ║
  ╚══════════════════════════════════════════════════════════════════════╝
```

---

### 18.2 Disabled or Bypassed Certificate Validation

The most common automotive TLS vulnerability — and the most dangerous.

```
  WHAT DEVELOPERS DO (and why):

  // "Fix" for cert expired / self-signed / domain mismatch errors:
  sslContext.setHostnameVerifier((hostname, session) -> true);  // Java
  curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 0L);          // C/libcurl

  Result: TLS still encrypts data, but provides ZERO authentication.
  Any server can present any certificate → accepted blindly.
```

**Impact in automotive:**

```
  ╔══════════════════════════════════════════════════════════════════════╗
  ║  Vehicle with disabled cert validation                               ║
  ║                                                                      ║
  ║  Vehicle ──── "Connect to OTA server" ────► Rogue Wi-Fi AP         ║
  ║               Attacker intercepts, presents fake cert               ║
  ║               Vehicle accepts (no validation) ❌                    ║
  ║                                                                      ║
  ║  Attacker now sends:                                                 ║
  ║   · Malicious firmware update → code execution on ECU               ║
  ║   · Fake speed limit data (V2X) → unsafe vehicle behavior           ║
  ║   · Forged diagnostic commands → disable airbags, brakes            ║
  ║                                                                      ║
  ╚══════════════════════════════════════════════════════════════════════╝
```

**Why it happens:**
- Dev cert expired in the test environment → developer adds `acceptAllCerts()` as a temporary fix → ships to production
- ECU uses an internal IP address, not a domain name → cert domain validation always fails → developers disable it
- Time and resource constraints → "we'll fix it later"

---

### 18.3 Weak or Legacy TLS Versions and Cipher Suites

Automotive ECUs often run for 10–15 years. Hardware chosen in 2015 may not support TLS 1.3.

```
  ╔═══════════════════╦════════════════════════════════════════════════╗
  ║  Vulnerable Config ║  Risk                                          ║
  ╠═══════════════════╬════════════════════════════════════════════════╣
  ║  TLS 1.0 / 1.1    ║  BEAST, POODLE, protocol downgrade attacks     ║
  ╠═══════════════════╬════════════════════════════════════════════════╣
  ║  TLS_RSA_* suites ║  No forward secrecy — one key leak exposes     ║
  ║  (RSA key exch.)  ║  all past and future OTA sessions              ║
  ╠═══════════════════╬════════════════════════════════════════════════╣
  ║  AES-CBC suites   ║  Padding oracle attacks (POODLE, Lucky13)      ║
  ╠═══════════════════╬════════════════════════════════════════════════╣
  ║  RC4               ║  Broken — statistical biases allow plaintext  ║
  ║                   ║  recovery of sensitive data                    ║
  ╠═══════════════════╬════════════════════════════════════════════════╣
  ║  MD5 / SHA-1 certs║  Collision attacks — forged certificates       ║
  ╚═══════════════════╩════════════════════════════════════════════════╝
```

**Automotive-specific problem — downgrade attacks:**

```
  Attacker-controlled Wi-Fi or cellular MITM:

  Vehicle: "I support TLS 1.3, 1.2, 1.1, 1.0"
  Attacker: (strips 1.3 and 1.2 from ClientHello)
  Server: "OK, TLS 1.1 it is" → weak session established ❌

  Fix: Server must refuse TLS < 1.2 regardless of what client advertises.
       TLS 1.3 adds downgrade sentinel values in ServerHello random field
       to detect this tampering.
```

---

### 18.4 OTA Firmware Update — TLS Misconfiguration

Over-The-Air updates are the highest-value target. A compromised OTA path gives an attacker persistent, fleet-wide code execution.

```
  ╔══════════════════════════════════════════════════════════════════════╗
  ║  SECURE OTA FLOW (correct)                                           ║
  ║                                                                      ║
  ║  OTA Server ──[TLS 1.3 + cert pinning]──► Vehicle                   ║
  ║                                           │                          ║
  ║                                           ▼                          ║
  ║                                 Verify firmware signature            ║
  ║                                 (separate from TLS)                  ║
  ║                                           │                          ║
  ║                                 ✅ Valid → apply update              ║
  ║                                                                      ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║  INSECURE OTA FLOW (common mistakes)                                 ║
  ║                                                                      ║
  ║  Mistake 1: TLS used but no firmware signature verification          ║
  ║  → Attacker does MITM, serves modified firmware over valid TLS conn  ║
  ║    (if cert validation is weak or cert is stolen)                    ║
  ║                                                                      ║
  ║  Mistake 2: HTTP used for download URL, only HTTPS for auth         ║
  ║  → Auth happens over HTTPS, but firmware binary downloaded over HTTP ║
  ║  → Attacker intercepts HTTP response, swaps firmware file            ║
  ║                                                                      ║
  ║  Mistake 3: TLS cert not pinned — any valid CA cert accepted        ║
  ║  → Attacker obtains cert from any trusted CA for a similar domain    ║
  ║  → Vehicle accepts it as legitimate OTA server                       ║
  ║                                                                      ║
  ╚══════════════════════════════════════════════════════════════════════╝
```

---

### 18.5 Certificate Pinning — Right and Wrong

**Certificate pinning** means the vehicle only accepts one specific certificate (or public key), ignoring the normal CA chain of trust.

```
  Normal TLS validation:
  Any cert signed by any trusted Root CA → accepted

  Certificate pinning:
  Only this specific cert (or public key hash) → accepted
  Everything else → rejected, even if CA-signed
```

**When pinning goes wrong in automotive:**

```
  ┌──────────────────────────────────────────────────────────────────┐
  │  Problem: Pinned cert expires                                     │
  │  Result:  Vehicle cannot connect to OTA server at all            │
  │           Requires physical reflash to update the pinned cert    │
  │           Fleet of 100,000 vehicles now unmanageable ❌          │
  ├──────────────────────────────────────────────────────────────────┤
  │  Problem: Pinned cert private key compromised                     │
  │  Result:  Cannot revoke — vehicle will still accept the cert     │
  │           Attacker can MITM all vehicles with stolen key         │
  ├──────────────────────────────────────────────────────────────────┤
  │  Best practice: Pin the PUBLIC KEY HASH (not the cert)           │
  │  → Allows cert renewal with same key pair without reflashing     │
  │  → Pin a backup key as well for rotation                         │
  └──────────────────────────────────────────────────────────────────┘
```

---

### 18.6 V2X (Vehicle-to-Everything) TLS Challenges

V2X connects vehicles to other vehicles (V2V), infrastructure (V2I), and cloud (V2C). TLS is used for cloud endpoints but V2V/V2I uses its own PKI (IEEE 1609.2 / ETSI ITS).

```
  ╔══════════════════════════════════════════════════════════════════════╗
  ║  V2X SECURITY CHALLENGES                                             ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                      ║
  ║  Latency constraint: V2V safety messages must arrive < 100 ms        ║
  ║  → Full TLS 1.3 handshake (1 RTT) may be too slow for real-time     ║
  ║  → Certificates are pre-loaded, not negotiated per session           ║
  ║                                                                      ║
  ║  Privacy: V2X broadcasts vehicle identity to nearby vehicles         ║
  ║  → Pseudonym certificates used — rotate frequently                  ║
  ║  → Prevents tracking a specific vehicle across geography             ║
  ║                                                                      ║
  ║  Attacks specific to V2X:                                            ║
  ║  · Replay attack: Capture "road clear" message, replay later        ║
  ║  · Sybil attack: One attacker broadcasts as many fake vehicles      ║
  ║  · False data injection: Broadcast fake hazard/speed limit data     ║
  ║                                                                      ║
  ╚══════════════════════════════════════════════════════════════════════╝
```

---

### 18.7 Summary — Automotive TLS Vulnerability Map

```
  ╔═════════════════════════╦═════════════════════════════╦════════════════════════════╗
  ║  Vulnerability          ║  Root Cause                 ║  Impact                    ║
  ╠═════════════════════════╬═════════════════════════════╬════════════════════════════╣
  ║  Disabled cert          ║  Dev shortcut / expired     ║  Full MITM — malicious     ║
  ║  validation             ║  cert, domain mismatch      ║  firmware, data injection  ║
  ╠═════════════════════════╬═════════════════════════════╬════════════════════════════╣
  ║  Hardcoded shared       ║  No per-device PKI          ║  One breach → entire       ║
  ║  private key            ║  provisioning               ║  fleet compromised         ║
  ╠═════════════════════════╬═════════════════════════════╬════════════════════════════╣
  ║  Legacy TLS / weak      ║  Old ECU hardware, no       ║  Downgrade, decryption,    ║
  ║  cipher suites          ║  firmware update path       ║  no forward secrecy        ║
  ╠═════════════════════════╬═════════════════════════════╬════════════════════════════╣
  ║  OTA over HTTP          ║  Split HTTPS auth +         ║  Firmware swap mid-        ║
  ║  (mixed content)        ║  HTTP download              ║  download, RCE on ECU      ║
  ╠═════════════════════════╬═════════════════════════════╬════════════════════════════╣
  ║  Expired cert, no       ║  Long vehicle lifetime vs   ║  OTA blocked → stuck on    ║
  ║  renewal mechanism      ║  short cert validity        ║  vulnerable firmware       ║
  ╠═════════════════════════╬═════════════════════════════╬════════════════════════════╣
  ║  No firmware signature  ║  Relying solely on TLS      ║  MITM replaces firmware    ║
  ║  verification           ║  for integrity              ║  even over valid TLS conn  ║
  ╠═════════════════════════╬═════════════════════════════╬════════════════════════════╣
  ║  Pinned cert expiry     ║  No key rotation plan       ║  Fleet loses OTA access,   ║
  ║  (no rotation plan)     ║                             ║  requires physical recall  ║
  ╚═════════════════════════╩═════════════════════════════╩════════════════════════════╝
```

---

## 19. Key Takeaways

- **TLS 1.3 = 1 RTT** — faster than TLS 1.2's 2 RTT
- **ECDHE only** — forward secrecy is mandatory; no RSA key exchange
- **AEAD only** — AES-GCM or ChaCha20-Poly1305; no CBC, no separate MAC
- **Certificate fully encrypted** — observer sees SNI but not cert details
- **5 cipher suites** — all strong; no weak legacy options
- **Compromise of long-term key ≠ compromise of past sessions** — forward secrecy
- **Auth tag on every record** — tampering is detected and connection dropped

---
*Sources: `tls-certificates-ciphers.md` + `tls-gpt-full.md`*
