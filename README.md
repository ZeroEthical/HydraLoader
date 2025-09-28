<p align="center">
  <img src="https://i.imgur.com/lS836hA.png" alt="HydraLoader Logo" width="200"/>
</p>
<h1 align="center">
  <br>
  🐍 HydraLoader 🛡️
  <br>
</h1>

<h4 align="center">An Advanced and Resilient PowerShell Execution Framework for Red Teaming & Adversary Simulation.</h4>

<p align="center">
  <a href="https://github.com/ZeroEthical/HydraLoader/blob/main/LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="License">
  </a>
  <a href="#">
    <img src="https://img.shields.io/badge/Language-PowerShell-blue.svg?style=for-the-badge&logo=powershell" alt="Language">
  </a>
    <a href="https://github.com/ZeroEthical">
    <img src="https://img.shields.io/badge/Author-ZeroEthical-purple?style=for-the-badge" alt="Author">
  </a>
  <a href="#">
    <img src="https://img.shields.io/badge/Maintained%3F-Yes-green.svg?style=for-the-badge" alt="Maintained">
  </a>
</p>

<p align="center">
  <a href="#-about-the-project">About</a> •
  <a href="#-key-features">Key Features</a> •
  <a href="#-architecture--flow">Architecture</a> •
  <a href="#-getting-started">Getting Started</a> •
  <a href="#-disclaimer">Disclaimer</a> •
  <a href="#-author">Author</a>
</p>

---

## 📖 About The Project

**HydraLoader** is a highly sophisticated and resilient PowerShell-based payload execution framework, designed for advanced penetration testing and red teaming operations. It employs a multi-layered approach to evasion, persistence, and in-memory execution, aiming to operate undetected in modern, highly monitored environments.

---

## ✨ Key Features

<details>
<summary>🧠 <strong>Evasion & Anti-Analysis</strong></summary>
<br>

-   **In-Memory AMSI Bypass**: Dynamically patches the Antimalware Scan Interface (AMSI) at runtime to neutralize script-based threat detection.
-   **Comprehensive Environment Checks**: Actively detects and evades analysis environments by checking for:
    -   **Debuggers**: Uses native `IsDebuggerPresent()` API calls.
    -   **Sandboxes**: Verifies system RAM, CPU core count, and uptime.
    -   **Analysis Tools**: Scans for common virtualization and analysis processes (e.g., Wireshark, Process Monitor, VMware/VirtualBox tools).
-   **Deep Obfuscation**: The entire script is heavily obfuscated, with critical strings (API functions, DLLs) Base64 encoded and a compacted code structure to deter static analysis.
</details>

<details>
<summary>🐍 <strong>The "Hydra" Persistence Engine</strong></summary>
<br>

HydraLoader employs a dual-headed, self-healing persistence mechanism to ensure long-term access and resilience against removal attempts.

-   **Method 1: Scheduled Task (Elevated Privileges)**: Creates a scheduled task disguised as a legitimate system process (`Microsoft Compatibility Appraiser`) that runs with `SYSTEM` privileges at logon.
-   **Method 2: WMI Event Subscription (Maximum Stealth)**: Establishes a permanent WMI event subscription that triggers on a timer. This method is extremely difficult to detect as it resides in the WMI repository, outside of standard auto-run locations.
-   **Self-Healing Capability**: On each execution, the framework checks if both persistence mechanisms are active. If one has been discovered and removed, the other automatically recreates it, ensuring the "Hydra" survives.
</details>

<details>
<summary>🚀 <strong>Payload Execution</strong></summary>
<br>

-   **"Fileless" In-Memory Operation**: The payload is downloaded directly into a memory buffer, decoded, and executed without ever touching the disk, minimizing the forensic footprint.
-   **AES-256 Decryption Framework**: Includes a function to decrypt payloads using AES-256 (CBC). This allows the payload to be stored and transmitted in an encrypted state, rendering it useless to network inspection tools. *(Note: Requires a pre-encrypted payload)*.
-   **Dynamic API Resolution**: Resolves all necessary Windows API functions dynamically at runtime, avoiding suspicious static import tables.
</details>

---

## ⚙️ Architecture & Flow

1.  **Initialization**: The AMSI bypass is executed instantly.
2.  **Evasion Checks**: The script performs all anti-analysis and anti-sandbox checks. If any fail, it terminates silently.
3.  **Persistence Check & Repair**: The Hydra engine verifies that both the Scheduled Task and WMI Subscription are in place. If not, it creates them.
4.  **Payload Delivery**: The framework downloads the payload from the configured URL directly into memory.
5.  **Decryption & Preparation**: The payload is Base64 decoded. If AES is enabled, it is then decrypted.
6.  **Execution**: The final payload is injected into memory and executed via stealthy Windows API calls.

---

## 🛠️ Getting Started

### 1. Payload Configuration
-   **URL**: Modify the Base64 encoded URL in the `$cfg` hashtable (`$cfg.o`).
-   **Payload Format**: The payload must be in raw shellcode format, which is then Base64 encoded.

### 2. AES Encryption (Optional)
To use the AES decryption capability:
1.  **Encrypt your payload**: Use your preferred tool to encrypt your shellcode with AES-256 (CBC mode, PKCS7 padding).
2.  **Configure Key and IV**:
    -   Base64 encode your 32-byte encryption key and place it in `$cfg.r`.
    -   Base64 encode your 16-byte IV and place it in `$cfg.s`.
3.  **Base64 encode the encrypted payload** and host it at your URL.
4.  **Activate Decryption**: Uncomment the three lines in the payload handling section of the script to activate the decryption routine.

### 3. Deployment
The script is self-contained. It can be executed via any standard PowerShell invocation method. On its first run on a target machine, it will automatically establish its persistence mechanisms.

---

## ⚠️ Disclaimer

This tool is intended for authorized red teaming, security research, and educational purposes **only**. Unauthorized use of this framework against any system is illegal. The author assumes no liability and is not responsible for any misuse or damage caused by this program.

---

## ✍️ Author

-   **ZeroEthical** - [GitHub](https://github.com/ZeroEthical)