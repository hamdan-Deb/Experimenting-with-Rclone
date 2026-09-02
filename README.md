<div align="center">
  
# Rclone Cloud Aggregator: A Case Study
  
**Aggregating Multiple Cloud Storage Platforms into a Single Unified Dashboard**

---

</div>

## 📖 Overview

This repository documents the setup, configuration, and troubleshooting of aggregating multiple cloud storage platforms using **Rclone**. My primary goal was to find a unified aggregator to bridge all my cloud platforms (currently testing 5 different drives) into one accessible hub. 

During this project, I extensively utilized the **Rclone Web GUI**, which proved vastly superior to standard command-line operations for specific UI tasks—most notably, the ability to easily copy shareable links for uploaded files and pictures directly from the dashboard.

## 🛠️ Key Components & Setup

*   **Rclone Core**: The backend sync mechanism.
*   **Rclone Web GUI**: Used for seamless visual management of aggregated drives.
*   **Rclone Config File (`rclone.conf`)**: Centralized secure storage where all cloud credentials and API keys are stored for GUI consumption.
*   **Target Cloud Platform**: `filen.io` (Primary Case Study).

### System Environment Preparation
Before integrating the cloud platform, system variables required modification. I added the Git binaries to the system `Path` to ensure smooth version control and command executions across environments.
> *[Insert Screenshot: Environment Variables showing `C:\Program Files\Git\bin`]*

---

## 🔍 Case Study: Integrating Filen.io

Connecting cloud platforms to Rclone generally requires API configurations. For **Filen.io**, the integration was highly technical and involved interacting with their command-line interface (`filen-cli`) to extract the necessary keys.

### 1. Installation of Filen-CLI
The initial setup required pulling the executable binaries. 

**Attempt 1: Via Bash / cURL**
> *[Insert Screenshot: Downloading Filen CLI via Command Prompt]*

```bash
curl -sL https://filen.io/cli.sh | bash
```
*Observation*: This downloaded the Linux/macOS binaries but resulted in shell profile warnings. Navigating to the `C:\Users\hpc\.filen-cli\bin` directory showed the `filen` file downloaded, but as it lacked a proper executable extension for Windows, it failed to launch.
> *[Insert Screenshot: Folder `.filen-cli/bin` showing the downloaded file]*

**Attempt 2: Windows PowerShell & Command Breakdown**
After attempting to run the invalid executables (`.\filen.exe` and `.\filen_.exe`) resulting in "CommandNotFound" and "Not a valid application" errors, I formulated a strict, multi-part PowerShell command to clear the corrupted files and properly fetch the Windows x64 binary.
> *[Insert Screenshot: PowerShell complex command execution]*

```powershell
Remove-Item -Force .\filen*, .\filen_*; [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; Invoke-WebRequest -Uri "https://github.com/FilenCloudDienste/filen-cli/releases/download/v0.0.36/filen-cli-v0.0.36-win-x64.exe" -OutFile "filen.exe"
```

**🧠 Breakdown of this complex command:**
1. `Remove-Item -Force .\filen*, .\filen_*`: This cleans the directory by forcefully deleting the previously downloaded broken/incorrect files.
2. `;`: This semicolon acts as a separator, allowing multiple commands to run sequentially in one line.
3. `[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12`: This explicitly forces Windows PowerShell to use the TLS 1.2 security protocol. GitHub requires this to download files safely, otherwise the connection drops.
4. `Invoke-WebRequest -Uri "..." -OutFile "filen.exe"`: This safely downloads the correct `.exe` file directly from Filen's official GitHub releases and saves it locally as `filen.exe`.

### 2. Extracting Cloud API Credentials
With the CLI working, I needed to extract the specific API keys required to bridge Filen to the Rclone aggregator. I utilized the Filen CLI to generate these metrics:

*   `statfs`: Displayed my account limits (**Used: 0 B | Max: 30 GiB**).
*   `whoami`: Confirmed the active account (`kevinknife4@gmail.com`).
*   `links`: Checked for public links.
*   `mount`: Attempted to mount the network drive.
*   `export-api-key`: Printed the exact API key needed to connect the cloud to the aggregator. 

> *[Insert Screenshot: Terminal showing statfs (30 GiB), whoami, and exported API key]*

The full authentication config (a massive Base64 token block) was saved locally at `C:\Users\hpc\.filen-cli\.filen-cli-auth-config`.
> *[Insert Screenshot: The `auth-config` file containing the giant Base64 string]*

### 3. Rclone Web GUI Configuration
Using the data gathered from the CLI, I proceeded to configure the **Rclone Web GUI** under the remote settings. I successfully mapped the crucial parameters:
*   `master_keys`
*   `private_key`
*   `public_key`
*   `auth_version`
*   `base_folder_uuid`

> *[Insert Screenshot: Rclone GUI remote settings showing master/public/private key fields]*

### 4. Troubleshooting & Error Handling
Despite correct configuration, bridging `filen.io` with `Rclone` yielded strict API errors, highlighted during testing.

#### ❌ Error 1: API Obfuscation & Base64 Failure
When attempting to list the file system (`rclone lsf FileN.io:`), Rclone threw a Base64 string decode failure.

```text
CRITICAL: Failed to create file system for "FileN.io:": failed to reveal api key: base64 decode failed when revealing password - is it obscured?: illegal base64 data at input byte 64
```
**Resolution**: The API key initially parsed into the Rclone config file was formatted incorrectly and required unobfuscating/re-entry.

#### ❌ Error 2: Invalid Authorization Header (Yellow Highlight)
After resolving the Base64 error, the API pinged the Filen gateway but failed at the network level:

```text
Cannot send request (Post "https://gateway.filen.io/v3/user/masterKeys": net/http: invalid header field value for "Authorization")
```
> *[Insert Screenshot: PowerShell displaying the highlighted yellow Authorization error]*

**Current Status**: A support ticket has been raised with the cloud provider's customer service to address how their V3 API handles Authorization headers via Rclone requests. Awaiting final patching instructions.

---

## 🚀 How to Run the Web GUI

To replicate this environment and spin up the Rclone Web Dashboard, use the following command:

```bash
rclone rcd --rc-web-gui
```

Once executed, navigate to `http://127.0.0.1:5572/` in your browser.

## 📌 Conclusion
The integration of Rclone as a cloud aggregator provides immense organizational value. While API handshakes (like the `filen.io` case study) can throw strict authentication header errors, the centralization of encrypted credentials, paired with the sheer utility of the Web GUI for fetching asset links, makes it a highly profound workflow.
