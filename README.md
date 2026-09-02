<div align="center">
  
# ☁️ Rclone Cloud Aggregator: A Case Study
  
**Aggregating Multiple Cloud Storage Platforms into a Single Unified Dashboard**

---

</div>

## 📖 Overview

This repository documents the setup, configuration, and troubleshooting of aggregating multiple cloud storage platforms using **Rclone**. My primary goal was to find a unified aggregator to bridge all my cloud platforms (currently testing 5 different drives) into one accessible hub. 

During this project, I extensively utilized the **Rclone Web GUI**, which proved vastly superior to standard command-line operations for specific UI tasks—most notably, the ability to easily copy shareable links for uploaded files and pictures directly from the dashboard.

## 🛠️ Key Components

*   **Rclone Core**: The backend sync mechanism.
*   **Rclone Web GUI**: Used for seamless visual management of aggregated drives.
*   **Rclone Config File (`rclone.conf`)**: Centralized secure storage where all cloud credentials and API keys are stored for GUI consumption.
*   **Target Cloud Platform**: `filen.io` (Primary Case Study).

---

## 🔍 Case Study: Integrating Filen.io

Connecting cloud platforms to Rclone generally requires API configurations. While some providers offer these details directly on their dashboards, others require you to request access via customer support, or integrate using specific username/password schemas. 

For **Filen.io**, the integration was highly technical and involved interacting with their command-line interface (`filen-cli`).

### 1. Installation of Filen-CLI
The initial setup required pulling the executable binaries and configuring system environments.

**Via Bash / cURL:**

```bash
curl -sL https://filen.io/cli.sh | bash
```

*Note: This downloaded the Linux/macOS binaries but resulted in shell profile warnings.*

**Via Windows PowerShell (Successful Resolution):**
To properly download the Windows x64 binary, I formulated a strict `Invoke-WebRequest` command forcing TLS 1.2 for security protocols:

```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; Invoke-WebRequest -Uri "https://github.com/FilenCloudDienste/filen-cli/releases/download/v0.0.36/filen-cli-v0.0.36-win-x64.exe" -OutFile "filen.exe"
```

### 2. Authentication & Config Generation
Upon successfully installing the CLI, environment variables (`Path`) were updated, and the authentication configurations were generated.
*   **Account Used**: `kevinknife4@gmail.com`
*   **Action**: Exporting the API key allowing full backend access and mounting the network drive.
*   **Config Storage**: The Base64 encoded JSON tokens were successfully generated and stored in `C:\Users\hpc\.filen-cli\.filen-cli-auth-config`.

### 3. Troubleshooting & Error Handling
During the connection bridge between `filen.io` and `Rclone`, several critical errors were observed and systematically debugged.

#### ❌ Error 1: API Obfuscation & Base64 Failure
When attempting to list the file system (`rclone lsf FileN.io:`), Rclone threw a Base64 string decode failure:

```text
CRITICAL: Failed to create file system for "FileN.io:": failed to reveal api key: base64 decode failed when revealing password - is it obscured?: illegal base64 data at input byte 64
```

**Resolution**: The API key initially parsed into the Rclone config file was formatted incorrectly and required unobfuscating/re-entry.

#### ❌ Error 2: Invalid Authorization Header
After resolving the Base64 error, the API pinged the Filen gateway (`https://gateway.filen.io/v3/user/masterKeys`) but failed at the network level:

```text
Cannot send request (Post "https://gateway.filen.io/v3/user/masterKeys": net/http: invalid header field value for "Authorization")
```

**Current Status**: A support ticket has been raised with the cloud provider's customer service to address how their V3 API handles Authorization headers via Rclone requests. Awaiting final patching instructions.

---

## 🚀 How to Run the Web GUI

To replicate this environment and spin up the Rclone Web Dashboard, use the following command:

```bash
rclone rcd --rc-web-gui
```

Once executed, navigate to `http://127.0.0.1:5572/` in your browser.

## 📌 Conclusion
The integration of Rclone as a cloud aggregator provides immense organizational value. While API handshakes (like the `filen.io` case study) can throw strict authentication header errors, the centralization of encrypted credentials in the config file, paired with the sheer utility of the Web GUI for fetching asset links, makes it a highly profound workflow.
