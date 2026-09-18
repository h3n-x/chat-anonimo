<div align="center">

# 🔒 Chat Anónimo v2.0
### Ephemeral Messaging with Real End-to-End Encryption and Zero-Knowledge Blind Relay

![Security](https://img.shields.io/badge/Security-AES--256--GCM_%2B_ECDH-brightgreen?style=for-the-badge)
![Zero Knowledge](https://img.shields.io/badge/Architecture-Zero--Knowledge_Relay-blue?style=for-the-badge)
![Zero Persistence](https://img.shields.io/badge/Storage-Zero--Persistence_RAM-orange?style=for-the-badge)
![Tests](https://img.shields.io/badge/Test_Suite-93%25_Coverage-10B981?style=for-the-badge)

**Open-source ephemeral communication platform engineered on the principle of Zero Trust in the Server (*Zero-Knowledge Blind Relay*).**

[![🚀 Live Demo](https://img.shields.io/badge/🚀_Demo-Netlify-success?style=for-the-badge&logo=netlify)](https://write-ghost.netlify.app)
[![📖 Versión en Español](https://img.shields.io/badge/📖_Leer-Español-green?style=for-the-badge)](README.md)
[![Backend Repository](https://img.shields.io/badge/Backend-chat--backend-teal?style=for-the-badge&logo=fastapi)](https://github.com/h3n-x/chat-backend)
[![Frontend Repository](https://img.shields.io/badge/Frontend-chat--frontend-cyan?style=for-the-badge&logo=react)](https://github.com/h3n-x/chat-frontend)

</div>

---

## 📋 Table of Contents
- [🎯 Why v2.0? (Architectural Evolution)](#-why-v20-architectural-evolution)
- [🛡️ Threat Model & Security Boundaries](#️-threat-model--security-boundaries)
- [🔑 E2EE Cryptographic Protocol](#-e2ee-cryptographic-protocol)
- [📁 Zero-Knowledge File Transfer](#-zero-knowledge-file-transfer)
- [🏗️ Ecosystem Breakdown](#️-ecosystem-breakdown)
- [🧪 Verification & Automated Testing](#-verification--automated-testing)
- [🚀 Local Deployment & Execution](#-local-deployment--execution)
- [📜 License](#-license)

---

## 🎯 Why v2.0? (Architectural Evolution)

The legacy v1.0 version (abandoned in mid-2025) contained critical conceptual flaws:
- The backend server generated symmetric keys and decrypted messages in transit to manage rooms.
- The frontend contained insecure fallbacks silently degrading to weak XOR with `Math.random()`.
- The frontend scaffold suffered from over 600 TypeScript errors and dead v0 scaffolding components.

**Chat Anónimo v2.0 is a complete architectural rewrite:**
1. **True Blind Relay:** The server never generates, deduces, or stores keys, and is mathematically incapable of decrypting content.
2. **Strict Native WebCrypto:** Zero XOR fallbacks. If the browser or network lacks `window.crypto.subtle` (e.g. non-secure HTTP), the application safely halts (*Fail-Closed*).
3. **Zero-Knowledge URL Hash Invitations:** Room keys travel in the URL hash fragment (`#room=...&key=...`), which per RFC 3986 standard is never transmitted across the network or sent to the server.

---

## 🛡️ Threat Model & Security Boundaries

### Security Goals
- **End-to-End Confidentiality (E2EE):** No intermediary (ISP, hosting providers, or an attacker with root server access) can inspect messages or files.
- **Integrity and Authenticity (AEAD):** Message tampering or cross-room replay attacks are detected immediately using AES-256-GCM 128-bit authentication tags and AAD (`room:ID`).
- **Real Zero-Persistence:** No databases or persistent storage. Rooms and messages reside strictly in RAM while participants are connected and vanish upon exit.
- **Metadata Privacy:** Original filenames, MIME types, and nicknames travel encrypted inside the AEAD payload.

### Security Limitations (Out of Scope)
- **Compromised Endpoints:** If a client device is infected with OS-level malware or keyloggers, endpoint security cannot be guaranteed.
- **Link Sharing Channel:** Invitation links must be delivered through a trusted channel between participants.

---

## 🔑 E2EE Cryptographic Protocol

```
+---------------+              +--------------------+              +---------------+
|     ALICE     |              |    BLIND RELAY     |              |      BOB      |
+-------+-------+              +---------+----------+              +-------+-------+
        |                                |                                 |
        | [1] Generates RoomKey (AES-GCM)|                                 |
        |     in local browser RAM       |                                 |
        |                                |                                 |
        |=== Method A: Hash Link (#room=XYZ&key=K) =======================>|
        |    (Fragment # is never sent to the server per RFC 3986)         |
        |                                |                                 |
        |=== Method B: Handshake ECDH (P-256) ============================>|
        |                                |<--- KEY_REQUEST {pk_Bob} -------|
        |<--- KEY_REQUEST {pk_Bob} ------|                                 |
        |                                |                                 |
        | [2] ECDH + HKDF -> K_wrap      |                                 |
        |     AES-GCM-Wrap(RoomKey)      |                                 |
        |                                |                                 |
        |---- KEY_DELIVERY {wrapped_k} ->|                                 |
        |                                |---- KEY_DELIVERY {wrapped_k} -->|
        |                                | [3] ECDH + HKDF -> K_wrap       |
        |                                |     AES-GCM-Unwrap -> RoomKey   |
        |                                |                                 |
        |================== Secure E2EE Messaging =========================|
        |                                |                                 |
        |---- WS: e2ee_message --------->|                                 |
        |     {ciphertext, iv, AAD}      |---- WS: e2ee_message ---------->|
        |                                |     (Decrypts & verifies tag)   |
```

- **Primitives:** AES-256-GCM (96-bit IV, 128-bit tag), ECDH P-256, HKDF-SHA256.
- **Anti-MITM Verification (Out-of-Band SAS Fingerprint):**
  - **Cannot be automated:** No browser or protocol primitive can autonomously determine whether an ephemeral public key was replaced by an active in-path adversary. True MITM protection relies on **mandatory out-of-band human verification** (voice call or in person).
  - **Blocking UI Modal:** The v2.0 client features a blocking 4-word SAS verification modal derived from $\text{SHA-256}(\text{RoomKey})$. Message sending is locked until the user explicitly clicks "Words Match — Unlock Chat". If words do not match ("Mismatch — Abort"), the session is immediately aborted, terminating the WebSocket and purging keys from RAM.

---

## 📁 Zero-Knowledge File Transfer

1. **Local Client-Side Encryption:** The file is bundled with metadata and encrypted in memory using `RoomKey` before transmission.
2. **64KB Streaming Chunks:** The FastAPI server streams the body in 64 KB chunks and immediately cuts off connections with `HTTP 413 Content Too Large` if exceeding **15 MB**.
3. **Opaque Storage & Auto-Destruction:** The server stores an opaque blob with a UUID filename (`temp_uploads/{uuid}.enc`). A background worker permanently deletes it after 10 minutes (`600s`).

---

## 🏗️ Ecosystem Breakdown

| Repository | Tech Stack | Role |
|---|---|---|
| **[chat-backend](https://github.com/h3n-x/chat-backend)** | FastAPI, Python 3.12+, WebSockets | Blind relay message router, streaming file server, rate limiter |
| **[chat-frontend](https://github.com/h3n-x/chat-frontend)** | Vite, React 19, TypeScript, Tailwind v4 | SPA client with native WebCrypto API and WCAG 2.2 AA accessibility |
| **[chat-anonimo](https://github.com/h3n-x/chat-anonimo)** | Protocol & Architecture Docs | Cryptographic protocol specification and umbrella orchestrator |

---

## 🧪 Verification & Automated Testing

### Backend (Pytest): 25 integration tests with 93% code coverage
```bash
cd chat-backend
source .venv/bin/activate
pytest --cov=app --cov-report=term-missing
```

### Frontend (Vitest): WebCrypto native unit tests
```bash
cd chat-frontend
npm run test
npm run build
```

---

## 🚀 Local Deployment & Execution

### 1. Launch Backend
```bash
cd chat-backend
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python main.py
```

### 2. Launch Frontend
```bash
cd chat-frontend
npm install
npm run dev
```

---

## 📜 License
Distributed under the MIT License. See `LICENSE` for more information.
