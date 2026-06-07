# Servos – Offline AI-Powered Digital Forensics & Incident Response (DFIR) Platform

> **"Forensics for the Rest of Us"** — An air-gapped, local-first digital forensics platform that detects suspicious devices, enforces sound preservation, and guides investigators through complex incident response via offline LLMs.

---

## 📌 Problem Statement

During a cyber incident, the first 60 minutes are critical. Organizations and law enforcement units (such as police cyber cells) face several severe bottlenecks when triaging potential compromises:

1. **Sensitive Data & Cloud Risks:** Digital evidence—containing proprietary logs, passwords, database dumps, or sensitive user data—cannot legally or safely be uploaded to cloud-based AI systems (e.g., ChatGPT, Claude) due to chain-of-custody regulations, privacy laws, and corporate confidentiality.
2. **The Forensic Expertise Gap:** Smaller corporate IT teams, educational institutions, and local police units rarely have on-staff certified forensic experts or budget for enterprise forensic suites ($10k+ licensing fees). Triage is often reduced to guesswork.
3. **The Evidence Preservation Trap:** Under stress, incident responders make critical mistakes—plugging in suspect drives directly, altering file timestamps (metadata), opening infected files, and failing to verify cryptographic checksums. This compromises the integrity of the evidence, making it inadmissible in court.
4. **Offline and Field Inoperability:** Field agents investigating crime scenes or operating in high-security, air-gapped data centers do not have reliable internet connections. They need a tool that works 100% locally on standard workstation hardware.

---

## 💡 The Solution: Servos

**Servos** is an offline digital forensics assistant designed to run locally, safely, and autonomously on an investigator's workstation. By combining system event monitoring, low-level forensic tool wrappers, and locally served Large Language Models (LLMs), Servos acts as a virtual DFIR copilot.

### Core Capabilities
* **Proactive Port & Volume Monitoring:** Continuously watches for storage events (e.g., USB connection). Upon connection, it immediately intercepts the device and initiates a secure workspace prompt.
* **Mandatory Backup Gate:** Enforces evidence preservation. Before any analysis, the investigator must create a forensic clone or structured backup of the target volume. The system calculates cryptographic hashes (MD5, SHA-256) on-the-fly, generating a secure **Chain of Custody (LoC)** JSON document.
* **Write-Block Protection:** Prevents the operating system from writing metadata or files back to the suspect device using native platform write-blocking mechanisms (e.g., Windows registry policies or Linux `hdparm` flags).
* **Local LLM Orchestration:** Routes queries and logs to local offline models (run via Ollama, such as Llama 3.1 8B, Qwen 2.5, Mistral, or DeepSeek-R1) to interpret complex log files, analyze YARA signature hits, and recommend triage steps.
* **Offline Retrieval-Augmented Generation (RAG):** Uses an embedded vector database (ChromaDB) containing standard operational guides (NIST SP 800-86, ISO/IEC 27037) and legal statutes (such as the Indian Information Technology Act / IT Act 2000) to ensure investigation steps comply with standard procedures and court evidence rules.
* **Flexible Modes of Operation:**
  * **Full Automation:** Servos scans the device, extracts metadata, identifies malware indicators, parses artifacts, and exports a signed PDF report automatically.
  * **Hybrid Mode:** Servos suggests forensic actions and waits for explicit operator confirmation at key checkpoints.
  * **Manual Mode:** Servos acts as an interactive advisory terminal, giving checklists and commands for the investigator to run manually.

---

## 🛠️ Technical Stack

Servos is built bottom-up to run without external network dependencies, ensuring an air-gapped, zero-leak footprint.

| Layer | Component | Description / Technology Used |
|---|---|---|
| **Language** | Python 3.10+ & TypeScript | Core business logic in Python; UI components in React/TS. |
| **Backend API** | FastAPI / Uvicorn | High-performance local server strictly bound to `127.0.0.1:8747`. |
| **Frontend UI** | Next.js 14 & Tailwind CSS | Cyberpunk dark-themed workbench utilizing advanced interactive animations from **21st.dev** (HalideTopo Hero, Gooey Text Morphing, Bento grid, and Spatial parallax showcase). |
| **Local AI Engine** | Ollama | Interface to local models (`llama3.1:8b`, `qwen2.5:7b`, `deepseek-r1:14b`, `phi3:mini`) running on GPU/CPU. |
| **Knowledge Base** | ChromaDB + SentenceTransformers | Offline RAG vector store populated with NIST SP 800-86, ISO 27037, and the IT Act 2000 (India). |
| **Preservation** | python-magic, hashlib | Dynamic file type identification and secure hashing (MD5, SHA-256). |
| **Analysis Modules** | YARA, psutil, watchdog | Signature scanners, real-time process tracing, networkSentinels, and system watchers. |
| **Forensic Adapters** | pytsk3, volatility3, scapy | Low-level disk image parsing, volatile RAM inspection, and PCAP analysis. |
| **Vault Security** | Cryptography (PyCA) | AES-256-GCM encryption for evidence vault, PBKDF2 key derivation, and HMAC log integrity chains. |
| **Reporting** | ReportLab & Jinja2 | Automates cryptographic PDF, JSON (STIX 2.1-compliant), and CSV forensic reports. |

---

## 🏛️ System Architecture & How We Build It

Servos is designed using a modular, decoupled architecture consisting of a Python backend service, an extensible forensic tool orchestrator, and a modern React web application wrapper.

```
                  ┌────────────────────────────────────────┐
                  │          Servos Frontend UI            │
                  │   (Next.js 14 / TypeScript / Tailwind) │
                  └───────────────────┬────────────────────┘
                                      │ REST / WebSockets
                                      ▼
                  ┌────────────────────────────────────────┐
                  │         FastAPI Local Backend          │
                  │        (Bound to 127.0.0.1:8747)       │
                  └───────────────────┬────────────────────┘
                                      │
           ┌──────────────────────────┼──────────────────────────┐
           ▼                          ▼                          ▼
┌──────────────────────┐   ┌──────────────────────┐   ┌──────────────────────┐
│  Preservation Gate   │   │  Forensic Engine     │   │   Offline AI/RAG     │
│  - USB Monitor       │   │  - YARA Scan Manager │   │   - Ollama Client    │
│  - Write Blocker     │   │  - pytsk3 Disk parse │   │   - ChromaDB Vector  │
│  - Backup & Checksum │   │  - Volatility RAM    │   │   - Legal/NIST RAG   │
└──────────┬───────────┘   └──────────┬───────────┘   └──────────┬───────────┘
           │                          │                          │
           └──────────────────────────┼──────────────────────────┘
                                      ▼
                  ┌────────────────────────────────────────┐
                  │        Evidence Vault Database         │
                  │  (SQLite + AES-256 + HMAC Audit Logs)  │
                  └────────────────────────────────────────┘
```

### Core Codebase Layout

* `servos/detection/` (Device Monitor): Monitors the operating system for hardware attachment events (e.g., using `pyudev` on Linux or WMI events on Windows) to automatically register suspect drives.
* `servos/preservation/` (Safety & Encryption): Controls platform-level write-blocking, performs file duplications with integrity checksums, and manages the PBKDF2/AES-encrypted local evidence store.
* `servos/forensics/` (Analysis Tool Adapters): Decoupled adapters wrapping low-level forensic utilities (`pytsk3` for file systems, `volatility3` for memory, `yara` for malware signatures, `pefile` for header classification).
* `servos/llm/` (Reasoning & Context): Manages the local LLM endpoint client, routes different forensic tasks to specialized models, and handles the vector search embeddings pipeline in ChromaDB.
* `servos/orchestration/` (Agent Playbooks): The agent core, directing execution flows through customizable YAML playbooks based on the type of investigation (e.g., ransomware triage vs. USB forensic acquisition).
* `servos/reports/` (Audit & Outputs): Extracts findings, compiles timeline charts (via matplotlib), verifies HMAC audit integrity, and generates STIX-compliant JSON and print-ready PDFs.

---

## 📈 Real-World Impact

* **100% Air-Gapped Security:** Zero telemetry. Critical threat data, PII, and incident details stay completely on-premise, preserving absolute privacy and eliminating leak vectors.
* **Democratizing First-Response Triage:** Empowers entry-level security analysts and field police officers to confidently perform preliminary forensic collections and device analysis without risking evidence damage.
* **Legally Sound Forensics:** Automatically enforces write-blocking and backup gates. All actions are cataloged in an append-only audit trail verified by cryptographic HMAC chains, assuring the defensibility of the evidence in legal trials.
* **Rapid Incident Triage:** Cuts down initial device triage time from hours of manual script configuration to under 3 minutes of guided, automated analysis.

---

## 🚀 Getting Started

### Prerequisites
* Python 3.10 or higher
* Node.js v18+ (for frontend web client)
* [Ollama](https://ollama.com/) (installed and running locally)

### Setup & Installation

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/Sansyuh06/servos-TNPolice_hackathon.git
   cd servos-TNPolice_hackathon
   ```

2. **Backend Setup:**
   ```bash
   # Create and activate virtual environment
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate

   # Install forensic dependencies
   pip install -r requirements.txt
   ```

3. **Frontend UI Setup:**
   ```bash
   cd servos-ui
   npm install
   ```

4. **Pull Required Local LLMs:**
   Make sure Ollama is running, then pull the recommended forensic models:
   ```bash
   ollama pull llama3.1:8b
   ollama pull phi3:mini
   ```

### Running the Application

* **Launch Standalone Desktop App (FastAPI Backend + Dedicated UI Window):**
  ```bash
  # In the root directory
  python launch_app.py
  ```
  *(Or double-click `Servos.bat` on Windows)*

* **Run CLI Commands Directly:**
  ```bash
  python -m servos.main monitor    # Launch active USB monitoring daemon
  python -m servos.main cases      # List local case database records
  python -m servos.main new        # Interactively start a new case
  ```

---

## 👥 The Team: MoMoSapiens
*Built for the CyberHack V4 Hackathon — Advanced Cyber Investigation Track.*
* **Akash Santhnu Sundar**
* **Shanmitha S**
* **Shivani M S**
* **Ajay C**

---
*Disclaimer: Servos is designed to aid digital forensic investigators in collecting, preserving, and analyzing evidence. Ensure compliance with your local laws and organizational policies when executing forensic acquisitions.*
