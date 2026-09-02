# NVIDIA DGX Spark
## User Access & Operations Manual • Standard Operating Procedure

**Classification:** Internal Confidential  
**DGX Host:** 192.168.0.143  
**DGX User:** truviq_domain

---

## 1. Executive Summary

The NVIDIA DGX Spark hosts Truviq's internal AI services, including LibreChat, Qwen3.6-35B, vLLM, SearXNG, and supporting microservices. All AI processing and conversational data remain securely within the corporate network.

### Employee Requirements:
- Windows, macOS, or Linux laptop
- Standard web browser
- Network access to 192.168.0.143
- No local Docker, Python, CUDA/NVIDIA drivers, or model weights required

### Request Flow:
```
Laptop
 ↓ (SSH Tunnel / Office Wi-Fi / NRPT Split DNS)
LibreChat (:3080 / :443)
 ↓
vLLM (:8000) → Qwen3.6-35B
```

---

## 2. Direct DGX Host Connection (SSH)

Connect directly to the DGX environment:

```bash
ssh truviq_domain@192.168.0.143
```

Enter the assigned password when prompted. General users do not need to manage Docker containers directly.

---

## 3. LibreChat Access — SSH Tunnel

Forward port 3080 to your local machine:

```bash
ssh -L 3080:localhost:3080 truviq_domain@192.168.0.143
```

Keep the terminal open, then browse to:

```
http://localhost:3080
```

With Split DNS, `http://aichat.truviq:3080` may also be used.

Select Qwen DGX Spark and model:
```
nvidia/Qwen3.6-35B-A3B-NVFP4
```

---

## 4. LibreChat HTTPS Access from Windows

Direct Office Wi-Fi access does not require an active SSH port-forwarding terminal.

### Certificate Distribution for New Users

> **Important:** A new employee must obtain the HTTPS certificate before opening LibreChat for the first time.

The certificate is maintained on the DGX server at:

```
~/truviq-root-ca.crt
```

However, employees should not be required to SSH into the DGX server just to obtain the certificate.

**The DGX administrator must copy the certificate to an approved internal distribution location** accessible to employees, such as:

- Corporate OneDrive
- SharePoint
- Company shared network folder
- Approved internal software distribution location

The administrator distributes the certificate file to users with the filename:

```
aichat.truviq.crt
```

**New users must download this file and save it in their Windows Downloads folder before proceeding.**

The expected location on the user's Windows laptop is:

```
C:\Users\<WindowsUsername>\Downloads\aichat.truviq.crt
```

> **Administrator responsibility:** Ensure that the certificate distributed to employees is the current certificate used by aichat.truviq.

---

### Method A: PowerShell (Recommended)

#### STEP 1: Add the DGX hostname

Open PowerShell as Administrator:

```powershell
Add-Content -Path "C:\Windows\System32\drivers\etc\hosts" -Value "`r`n192.168.0.143 aichat.truviq"
```

Verify:

```powershell
Get-Content "C:\Windows\System32\drivers\etc\hosts" | Select-String "aichat.truviq"
```

Expected output:
```
192.168.0.143 aichat.truviq
```

#### STEP 2: Flush DNS

```powershell
ipconfig /flushdns
```

#### STEP 3: Test HTTPS connectivity

```powershell
Test-NetConnection aichat.truviq -Port 443
```

Confirm:
```
TcpTestSucceeded : True
```

#### STEP 4: Download the certificate

Download the LibreChat HTTPS certificate from the approved internal certificate distribution location provided by the DGX/IT administrator.

Save the certificate in:

```
C:\Users\<WindowsUsername>\Downloads\
```

The filename must be exactly:

```
aichat.truviq.crt
```

The complete expected path is:

```
C:\Users\<WindowsUsername>\Downloads\aichat.truviq.crt
```

> **Important:** Do not attempt to copy `~/truviq-root-ca.crt` directly unless you have authorized SSH access to the DGX. General LibreChat users should receive the certificate through the approved internal distribution location.

#### STEP 5: Install the certificate

Open PowerShell as Administrator:

```powershell
Import-Certificate -FilePath "$env:USERPROFILE\Downloads\aichat.truviq.crt" -CertStoreLocation Cert:\LocalMachine\Root
```

This installs the certificate into Trusted Root Certification Authorities.

#### STEP 6: Open LibreChat

Navigate to:

```
https://aichat.truviq
```

Sign in using the corporate OpenID account.

---

### Method B: Manual VS Code Editor (If PowerShell Method Fails)

Use this method if the PowerShell commands in Method A do not produce the expected output or if you encounter permission issues.

#### STEP 1: Open VS Code as Administrator

- Close VS Code if it is already open
- Search for **Visual Studio Code** in the Windows Start menu
- Right-click **Visual Studio Code**
- Select **Run as administrator**

#### STEP 2: Open the Hosts File

In VS Code:

- Click **File** → **Open File**
- Navigate to:
  ```
  C:\Windows\System32\drivers\etc
  ```
- If the `hosts` file is not visible, change the file type filter to **All Files** (click the filter dropdown in the bottom-right)
- Select the file named:
  ```
  hosts
  ```
- Click **Open**

#### STEP 3: Add the LibreChat Hostname

- Go to the **last line** of the file
- Add a new line with:
  ```
  192.168.0.143 aichat.truviq
  ```

> **Important:** The entry must be exactly `192.168.0.143 aichat.truviq` with a single space between the IP and hostname. Do not add `https://` or `:443`.

#### STEP 4: Save the File

- Press **Ctrl + S**

Because VS Code was opened using **Run as administrator**, the file should save successfully.

#### STEP 5: Flush DNS and Test

Open PowerShell as Administrator and run:

```powershell
ipconfig /flushdns
```

Verify the entry:

```powershell
Get-Content "C:\Windows\System32\drivers\etc\hosts" | Select-String "aichat.truviq"
```

Expected output:
```
192.168.0.143 aichat.truviq
```

Test connectivity:

```powershell
Test-NetConnection aichat.truviq -Port 443
```

Confirm:
```
TcpTestSucceeded : True
```

#### STEP 6: Download and Install the Certificate

Download the LibreChat HTTPS certificate from the approved internal certificate distribution location provided by the DGX/IT administrator.

Save the certificate in the Windows Downloads folder.

The filename must be exactly:

```
aichat.truviq.crt
```

The expected location is:

```
C:\Users\<WindowsUsername>\Downloads\aichat.truviq.crt
```

Open PowerShell as Administrator and install it:

```powershell
Import-Certificate -FilePath "$env:USERPROFILE\Downloads\aichat.truviq.crt" -CertStoreLocation Cert:\LocalMachine\Root
```

This installs the certificate into the Local Computer Trusted Root Certification Authorities store.

#### STEP 7: Open LibreChat

Navigate to:

```
https://aichat.truviq
```

Sign in using the corporate OpenID account.

---

### Important Notes

> **The Windows hosts entry must be exactly:** `192.168.0.143 aichat.truviq`
>
> - Do NOT add `:443` or another port to the hosts entry
> - Do NOT add `https://` prefix
> - HTTPS uses port 443 automatically
> - Use Method B if Method A does not produce the expected output
> - The certificate must be downloaded from the approved internal distribution location before attempting HTTPS access
> - The certificate installation command must be run in PowerShell as Administrator

---

## 5. Service Configurations & Integrations

### Qwen DGX Spark
- Custom LibreChat endpoint configured
- Model: `nvidia/Qwen3.6-35B-A3B-NVFP4`
- Context window: 131,072 tokens (128K)
- Duplicate endpoint configuration fixed
- LibreChat startup verified

### SearXNG Web Search
- Dedicated Docker service added
- Host port 28080 used to avoid port conflicts
- Internal Docker endpoint: `searxng:8080`
- Persistent settings configured
- Search API verified with HTTP 200
- LibreChat connects over the Docker network
- SearXNG URL supplied through `.env`
- Unsupported `searchProvider: searxng` configuration removed

### OneDrive
- OneDrive integration configured for LibreChat
- Required configuration and authentication components verified

### Automatic Cleanup
- Scheduled automatic deletion/cleanup configured
- Prevents old data and files from accumulating indefinitely

---

## 6. Direct Qwen API (vLLM)

Base endpoint:
```
http://localhost:8000/v1
```

### Tunnel:

```bash
ssh -L 8000:localhost:8000 truviq_domain@192.168.0.143
```

### Tests:

```bash
curl http://localhost:8000/v1/models
curl http://localhost:8000/health
```

---

## 7. Windows Split DNS (NRPT)

Keep the Wi-Fi adapter on automatic DHCP. Do not replace global laptop DNS with `192.168.0.143`.

### Baseline:

```powershell
netsh interface ip set dns name="Wi-Fi" source=dhcp
ipconfig /flushdns
ping 192.168.0.143
Test-NetConnection 192.168.0.143 -Port 53
Test-NetConnection 192.168.0.143 -Port 3080
nslookup aichat.truviq 192.168.0.143
```

### Add NRPT in Administrator PowerShell:

```powershell
Add-DnsClientNrptRule -Namespace ".truviq" -NameServers "192.168.0.143"
ipconfig /flushdns
Get-DnsClientNrptRule
```

### Verify:

```bash
nslookup aichat.truviq
nslookup google.com
curl.exe -I https://www.google.com
```

### Remove the rule:

```powershell
Remove-DnsClientNrptRule -Namespace ".truviq" -Force
ipconfig /flushdns
```

---

## 8. Enterprise Microservices & Ports

| SERVICE | PORT | LOCAL ACCESS URL | SSH TUNNEL COMMAND |
|---------|------|------------------|--------------------|
| Temporal UI | 18080 | http://localhost:18080 | `ssh -L 18080:localhost:18080 truviq_domain@192.168.0.143` |
| LibreChat UI | 3080 | http://localhost:3080 | `ssh -L 3080:localhost:3080 truviq_domain@192.168.0.143` |
| LibreChat Admin | 3010 | http://localhost:3010 | `ssh -L 3010:localhost:3010 truviq_domain@192.168.0.143` |
| SearXNG Web Search | 28080 | http://localhost:28080 | `ssh -L 28080:localhost:28080 truviq_domain@192.168.0.143` |
| vLLM Qwen API | 8000 | http://localhost:8000/v1 | `ssh -L 8000:localhost:8000 truviq_domain@192.168.0.143` |
| Langfuse | 3000 | http://localhost:3000 | `ssh -L 3000:localhost:3000 truviq_domain@192.168.0.143` |
| LiteLLM | 4000 | http://localhost:4000 | `ssh -L 4000:localhost:4000 truviq_domain@192.168.0.143` |
| Phoenix | 6006 | http://localhost:6006 | `ssh -L 6006:localhost:6006 truviq_domain@192.168.0.143` |
| Guardrails | 8010 | http://localhost:8010/health | `ssh -L 8010:localhost:8010 truviq_domain@192.168.0.143` |
| Agent Registry Ping | 12121 | http://localhost:12121/v0/ping | `ssh -L 12121:localhost:12121 truviq_domain@192.168.0.143` |
| Agent Registry MCP | 31313 | localhost:31313 | `ssh -L 31313:localhost:31313 truviq_domain@192.168.0.143` |

---

## 9. Quick Troubleshooting

| PROBLEM | CHECK |
|---------|-------|
| SSH timeout/refused | Corporate network and ping 192.168.0.143 |
| LibreChat unavailable | Re-create the SSH tunnel and keep it open |
| Qwen API unavailable | Port 8000 tunnel and /health |
| aichat.truviq fails | Hosts entry, NRPT, DNS, and port 443 |
| SearXNG unavailable | DGX docker ps and container status |

---

## 10. Employee Quick-Start

1. Connect to the company network and ensure 192.168.0.143 is reachable
2. **HTTPS:** Install the certificate and hosts entry, then open https://aichat.truviq
3. **SSH tunnel:** Run `ssh -L 3080:localhost:3080 truviq_domain@192.168.0.143`, then open http://localhost:3080
4. Select Qwen DGX Spark → `nvidia/Qwen3.6-35B-A3B-NVFP4`
5. Use integrated SearXNG Web Search and OneDrive features
6. AI computation runs on the DGX Spark GPU

---

**Truviq Enterprise AI • Internal Confidential**
