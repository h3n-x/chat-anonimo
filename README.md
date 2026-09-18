<div align="center">

# 🔒 Chat Anónimo v2.0
### Mensajería Efímera con Cifrado End-to-End Real y Zero-Knowledge Blind Relay

![Security](https://img.shields.io/badge/Security-AES--256--GCM_%2B_ECDH-brightgreen?style=for-the-badge)
![Zero Knowledge](https://img.shields.io/badge/Architecture-Zero--Knowledge_Relay-blue?style=for-the-badge)
![Zero Persistence](https://img.shields.io/badge/Storage-Zero--Persistence_RAM-orange?style=for-the-badge)
![Tests](https://img.shields.io/badge/Test_Suite-93%25_Coverage-10B981?style=for-the-badge)

**Plataforma de comunicación efímera de código abierto diseñada bajo el principio de Cero Confianza en el Servidor (*Zero-Knowledge Blind Relay*).**

[![🚀 Demo en Vivo](https://img.shields.io/badge/🚀_Demo-Netlify-success?style=for-the-badge&logo=netlify)](https://write-ghost.netlify.app)
[![📖 English Version](https://img.shields.io/badge/📖_Read-English-blue?style=for-the-badge)](README.en.md)
[![Backend Repository](https://img.shields.io/badge/Backend-chat--backend-teal?style=for-the-badge&logo=fastapi)](https://github.com/h3n-x/chat-backend)
[![Frontend Repository](https://img.shields.io/badge/Frontend-chat--frontend-cyan?style=for-the-badge&logo=react)](https://github.com/h3n-x/chat-frontend)

</div>

---

## 📋 Tabla de Contenidos
- [🎯 ¿Por qué v2.0? (Evolución Arquitectónica)](#-por-qué-v20-evolución-arquitectónica)
- [🛡️ Modelo de Amenazas (Threat Model)](#️-modelo-de-amenazas-threat-model)
- [🔑 Protocolo Criptográfico E2EE](#-protocolo-criptográfico-e2ee)
- [📁 Transferencia de Archivos Cero-Conocimiento](#-transferencia-de-archivos-cero-conocimiento)
- [🏗️ Estructura del Ecosistema](#️-estructura-del-ecosistema)
- [🧪 Verificación y Pruebas Automatizadas](#-verificación-y-pruebas-automatizadas)
- [🚀 Despliegue y Ejecución Local](#-despliegue-y-ejecución-local)
- [📜 Licencia](#-licencia)

---

## 🎯 ¿Por qué v2.0? (Evolución Arquitectónica)

La versión v1.0 original de este proyecto (abandonada a mediados de 2025) presentaba fallas conceptuales críticas que contradecían su promesa de seguridad:
- El servidor backend generaba claves simétricas y descifraba el contenido en tránsito para "gestionar el chat".
- El frontend contaba con un fallback inseguro que degradaba silenciosamente a operaciones XOR débiles con `Math.random()`.
- El scaffold del frontend contenía más de 600 errores de TypeScript y componentes muertos de v0.

**Chat Anónimo v2.0 es una reescritura arquitectónica completa:**
1. **Verdadero Blind Relay:** El servidor jamás genera, deduce ni almacena claves privadas o simétricas, y es criptográficamente incapaz de descifrar mensajes o archivos.
2. **WebCrypto Nativo Estricto:** Se erradicó cualquier degradación a XOR. Si el navegador o el contexto de red no soportan la API nativa `window.crypto.subtle`, la aplicación se bloquea de forma segura (*Fail-Closed*).
3. **Invitaciones Zero-Knowledge por URL Hash:** Las claves de sala viajan en el fragmento hash (`#room=...&key=...`), el cual por estándar RFC 3986 nunca viaja por la red ni se envía al servidor.

---

## 🛡️ Modelo de Amenazas (Threat Model)

### Objetivos de Seguridad
- **Confidencialidad Extremo a Extremo (E2EE):** Ningún intermediario en tránsito (ISP, proveedores de hosting como Render o Netlify, ni un atacante con acceso root al servidor backend) puede leer los mensajes ni los archivos.
- **Integridad y Autenticidad (AEAD):** Cualquier manipulación o intento de inyectar un mensaje de una sala a otra es detectado y descartado mediante el tag de 128 bits de AES-256-GCM y AAD (`room:ID`).
- **Zero-Persistence Real:** No existe base de datos ni almacenamiento en disco permanente. Toda sala y mensaje vive únicamente en la RAM mientras haya participantes conectados.
- **Privacidad de Metadatos:** Los nombres de archivo, tipos MIME y apodos viajan cifrados dentro del payload, no en cabeceras legibles para el servidor.

### Límites de Seguridad (Qué NO protege)
- **Compromiso del Endpoint Local:** Si el dispositivo físico del usuario está infectado con malware o keyloggers a nivel de sistema operativo, la seguridad local no puede ser garantizada.
- **Canal de Compartición de Enlaces:** La entrega del enlace de invitación debe realizarse mediante un canal de confianza entre las partes.

---

## 🔑 Protocolo Criptográfico E2EE

```
+---------------+              +--------------------+              +---------------+
|     ALICE     |              |    BLIND RELAY     |              |      BOB      |
+-------+-------+              +---------+----------+              +-------+-------+
        |                                |                                 |
        | [1] Genera RoomKey (AES-GCM)   |                                 |
        |     en memoria local           |                                 |
        |                                |                                 |
        |=== Método A: Enlace Hash (#room=XYZ&key=K) =====================>|
        |    (El fragmento # nunca se envía al servidor por RFC 3986)      |
        |                                |                                 |
        |=== Método B: Handshake ECDH (P-256) ============================>|
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
        |================== Mensajería Segura E2EE ========================|
        |                                |                                 |
        |---- WS: e2ee_message --------->|                                 |
        |     {ciphertext, iv, AAD}      |---- WS: e2ee_message ---------->|
        |                                |     (Descifra y valida tag)     |
```

- **Primitivas:** AES-256-GCM (96-bit IV, 128-bit tag), ECDH P-256, HKDF-SHA256.
- **Verificación Anti-MITM (SAS Fingerprint Out-of-Band):**
  - **No automatizable:** El protocolo o navegador no puede saber de forma autónoma si el par de claves públicas fue manipulado en tránsito por un adversario activo. La garantía contra MITM requiere **verificación humana obligatoria fuera de banda** (voz o presencial).
  - **Modal Bloqueante:** La interfaz presenta un modal de 4 palabras derivado de $\text{SHA-256}(\text{RoomKey})$. El envío de mensajes permanece inhabilitado hasta que el usuario confirma explícitamente la coincidencia de palabras ("Coinciden — Activar Chat"). Si se detecta discrepancia ("No Coinciden — Abortar"), la sesión se destruye de inmediato y se purgan las claves de la memoria.

---

## 📁 Transferencia de Archivos Cero-Conocimiento

1. **Cifrado Local:** El cliente empaqueta el contenido binario junto al nombre del archivo y tipo MIME, cifrándolos en un solo blob con `RoomKey` antes de subirlo.
2. **Streaming en Chunks de 64 KB:** El servidor FastAPI recibe el stream en bloques de 64 KB y corta la conexión inmediatamente con `HTTP 413 Content Too Large` si supera **15 MB**, garantizando memoria acotada.
3. **Almacenamiento Opaco con Auto-Destrucción:** El servidor solo almacena un blob binario con nombre UUID (`temp_uploads/{uuid}.enc`). A los 10 minutos (`600s`), una tarea en segundo plano elimina permanentemente el archivo.

---

## 🏗️ Estructura del Ecosistema

Este proyecto está dividido en componentes especializados desacoplados:

| Repositorio | Rol / Stack | Descripción |
|---|---|---|
| **[chat-backend](https://github.com/h3n-x/chat-backend)** | FastAPI, Python 3.12+, WebSockets | Blind Relay enrutador de mensajes, streaming de archivos y rate limiting |
| **[chat-frontend](https://github.com/h3n-x/chat-frontend)** | Vite, React 19, TypeScript, Tailwind v4 | Cliente SPA con ejecución nativa WebCrypto y accesibilidad WCAG 2.2 AA |
| **[chat-anonimo](https://github.com/h3n-x/chat-anonimo)** | Documentación & Protocolo | Especificación criptográfica, threat model y guías de orquestación |

---

## 🧪 Verificación y Pruebas Automatizadas

El ecosistema cuenta con verificación automatizada integral en ambos extremos:

### Backend (Pytest): 25 pruebas con 93% de cobertura global
```bash
cd chat-backend
source .venv/bin/activate
pytest --cov=app --cov-report=term-missing
```

### Frontend (Vitest): 4 suites criptográficas nativas WebCrypto
```bash
cd chat-frontend
npm run test
npm run build
```

---

## 🚀 Despliegue y Ejecución Local

### 1. Iniciar el Backend
```bash
cd chat-backend
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python main.py
```
Servidor activo en: `http://localhost:8000` (WebSocket en `ws://localhost:8000/ws/{room_id}`).

### 2. Iniciar el Frontend
```bash
cd chat-frontend
npm install
npm run dev
```
Cliente activo en: `http://localhost:5173`.

---

## 📜 Licencia
Distribuido bajo la Licencia MIT. Consulta el archivo `LICENSE` para más información.
