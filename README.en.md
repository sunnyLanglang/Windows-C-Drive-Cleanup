[English](README.en.md) | [简体中文](README.md)

# Windows C Drive Deep Cleanup Engineering Log: From Critical Red to Full Recovery

> **Project Background**: Windows 11 (200 GB C Drive)  
> **Initial State**: Only 17.8 GB free (Critical)  
> **Final State**: 62.6 GB free (Healthy)  
> **Core Goal**: Safely free up space and establish a recurrence prevention mechanism.

---

## Before & After Comparison

**🔴 Before: C drive critical, only 17.8 GB free**

<img width="600" alt="Before cleanup - initial state" src="https://github.com/user-attachments/assets/d066fe28-a41c-4198-a6a5-9e6f6b880fa5" />

*The desperate moment before cleanup: C drive in critical red, only 17.8 GB left.*

**🟢 After: WizTree scan confirms 62.6 GB free on C drive**

<img width="600" alt="After cleanup - final state" src="https://github.com/user-attachments/assets/0e35dc43-3ec9-428d-87ec-40e467dc6fdf" />

*The victory moment after cleanup: free space restored to 62.6 GB.*

---

## 1. Introduction: Why Ordinary "Disk Cleanup" Doesn't Work

Half a year ago, I tried to help a friend clean up her C drive. Using only Windows built-in "Disk Cleanup" and uninstalling software, the result was minimal. Today, while troubleshooting my own computer, I finally found the root cause: **the real space killers are not on the surface; they hide in software caches, system hidden files (such as hibernation files and virtual memory), and leftovers from old versions.** This note records the complete troubleshooting logic and practical commands.

## 2. Basic Environment Investigation

<img width="300" alt="Disk Cleanup screenshot" src="https://github.com/user-attachments/assets/aeaa69f1-178e-4d2d-82b4-8d68c29e0fdc" />

*   **Action**: `Win + R`, type `cleanmgr` -> select C drive -> click "Clean up system files" (key step!).
*   **Summary**: This step only cleaned up 39.4 MB, which is "surface cleaning" (because I had already done the same operation once before).

<img width="300" alt="System storage overview" src="https://github.com/user-attachments/assets/a53621fe-e891-4129-90cf-b6facd6f9e97" />

*   **Action**: Settings -> System -> Storage.
*   **Finding**: "Installed apps" took 95.6 GB, "System & reserved" took 61.6 GB. We need to dig deeper.

## 3. Deep Dive: Manual Investigation, PowerShell, and Core File Handling

### 3.1 Manual Investigation of Hidden Files (Finding Battlefield Cache)

<img width="500" alt="Comparison before and after showing hidden items on C drive" src="https://github.com/user-attachments/assets/f4245c5c-5fe5-4d2f-9389-58e01e917e96" />
<img width="500" alt="Battlefield cache under AppData" src="https://github.com/user-attachments/assets/ce60e0ab-ee80-46a1-899f-29f18818746c" />

*   **Action**: In File Explorer at the root of C drive (`C:\`), open the "View" menu and check "Hidden items". Compare the hidden files and folders that appear before and after.
*   **Further investigation**: Then go to `C:\Users\YourUsername\AppData\Local` and continue looking for large files.

<img width="400" alt="Battlefield cache folder properties" src="https://github.com/user-attachments/assets/a642639a-afa7-4e41-b550-540e3484b4eb" />

*   **Finding**: In this directory, I found leftover Battlefield shader cache folders: `BattlefieldGameData.kin-live.Win32` (1.6 GB) and `BattlefieldGameData.kin-release.Win32` (3.2 GB).
*   **Action**: Since the game itself is not on C drive, I deleted these two folders directly, instantly freeing **4.8 GB**.

<img width="200" alt="Battlefield cache final properties" src="https://github.com/user-attachments/assets/d29f235f-d93a-48f1-8022-0c1804dbf1a5" />
<img width="200" alt="Screenshot 2026-09-10 222253" src="https://github.com/user-attachments/assets/00aa38a6-244a-4b9f-a94b-b1bb0e66e871" />

### 3.2 PowerShell Deep Scan (Finding Hidden System-Level Large Items)

*   **Action**: `Win + X` -> Terminal (Admin). Use `Get-ChildItem` combined commands to traverse real folder sizes. For system-protected hidden folders, add the `-Force` parameter to scan through.
*   **Scan results**:
    *   Measured by a separate command: **`C:\Windows\Installer` takes 15.97 GB** (⚠️ **Fatal warning: this is the system installation cache used for software uninstall and repair. Absolutely do not delete it manually!**).
    *   Normal scan found: under `C:\Windows`, **`WinSxS` takes 13.23 GB**, **`System32` takes 9.18 GB** (system lifelines, do not touch).
    *   After adding `-Force`, it penetrated system protection and found a hidden folder: **`C:\Program Files\WindowsApps` takes 11.69 GB** (Microsoft Store app library).
    *   Root directory scan found: **`C:\$RECYCLE.BIN` (Recycle Bin) takes 5 GB**.

<img width="600" alt="Installer scan screenshot" src="https://github.com/user-attachments/assets/0adb654f-8bae-4fd5-9189-58c40b7f0545" />
<img width="600" alt="Windows scan screenshot" src="https://github.com/user-attachments/assets/f45cc9b1-edd6-45f7-9d7e-81833851ce92" />
<img width="600" alt="WindowsApps scan screenshot" src="https://github.com/user-attachments/assets/1a6ab390-6750-4bd3-a8e8-9e83f5b3eca5" />
<img width="600" alt="Root directory full scan" src="https://github.com/user-attachments/assets/2b65b69b-f453-4fe9-a71d-17ba29185bcd" />

*   **Engineering summary**: PowerShell helped us see the real cards on the C drive table. These system-level hidden folders are huge, but absolutely must not be touched. What can be safely cleaned are software caches and game leftovers.

---

### 💡 Engineering Note: Environment Restrictions and Correct Usage of WizTree

**Background**: Since I am currently in Russia, my network environment is restricted, and I could not directly download WizTree. Therefore, the first two steps (3.1 Manual Investigation of Hidden Files and 3.2 PowerShell Deep Scan) were **backup troubleshooting solutions** when a visualization tool was unavailable. If you can download WizTree smoothly, you can prioritize using it; troubleshooting will be much more intuitive.

#### 📥 WizTree Download and Usage Guide (Highly Recommended)

<img width="600" alt="WizTree official website screenshot" src="https://github.com/user-attachments/assets/3e5b155f-4bd2-406b-a81a-f3200165d06e" />
<img width="600" alt="WizTree extracted folder screenshot" src="https://github.com/user-attachments/assets/46c49267-5cd0-4d44-8b47-7b9bd957110b" />

1. **Download**: Go to the WizTree official website `diskanalyzer.com` and download the portable version (a compressed archive).
2. **Extract**: **Do not extract to C drive.** Create a new folder on D drive (e.g., `D:\WizTree`) and extract the archive there.
3. **Run as administrator**: Go into the extracted folder, find `WizTree64.exe` (for 64-bit systems), **right-click -> Run as administrator** (this is very important, otherwise it cannot scan hidden system files).
4. **Scan C drive**: In the top-left corner of the software, select `C: [Windows]`, then click the `Scan` button. In just 3 seconds, the interface will generate a huge **colorful block map**. The larger the block, the larger the file.
5. **Pinpoint location**: Hover the mouse over a block to see the specific file path.

<img width="600" alt="WizTree colorful block map" src="https://github.com/user-attachments/assets/31acdb5d-6651-4cce-9339-c3df09d8b359" />

*Image: WizTree's colorful block map (at that time it found the 12.6 GB hibernation file, the largest purple block in the lower right)*

### 3.3 Core Operation A: Disable Hibernation File (hiberfil.sys)

**🔴 Before cleanup: hibernation file (hiberfil.sys) took about 12.6 GB**

<img width="200" alt="Hibernation file before cleanup" src="https://github.com/user-attachments/assets/9b802353-a404-4cc7-a8a1-f8c9b0b1dfcb" />

*   **Action**: In an administrator terminal, enter the command `powercfg -h off`.
*   **Principle**: The hibernation file size equals physical memory size. Disabling it instantly frees over ten GB of space (it does not affect normal shutdown or sleep at all).

> **⚠️ Prerequisite**: This operation **is only suitable for users who normally do not use "Hibernate"**.
>
> *   **"Sleep" vs "Hibernate"**:
>     *   **Sleep**: Memory remains powered, data is not written to disk, wake-up is extremely fast, and it is not affected by this operation.
>     *   **Hibernate**: Memory data is written to disk (i.e., `hiberfil.sys`). After completely powering off, the next boot can still restore the original state.
> *   If you normally only use "Shutdown" or "Sleep", you can safely execute `powercfg -h off` to delete the hibernation file and save over ten GB.
> *   If you really need "Hibernate", do not execute this command. Or when needed, enter `powercfg -h on` in an administrator terminal to restore the function at any time.

**🟢 After cleanup: Run WizTree again to scan C drive. The purple hibernation file block has disappeared.**

<img width="600" alt="Hibernation file after cleanup" src="https://github.com/user-attachments/assets/bdacef53-b5dd-4e66-a967-9a7f19220cde" />

### 3.4 Core Operation B: Migrate Virtual Memory (pagefile.sys)

**📖 Why migrate?**
`pagefile.sys` (virtual memory file) is a hidden large file generated by the system at the root of C drive. It usually equals the size of physical memory by default. Because it is a hidden and locked system file, it cannot be deleted directly. So we use a "moving house" approach to move it from C drive to D drive.

**🛠️ Steps:**

1. **Open Performance Options**: Press `Win + R`, type `sysdm.cpl`, and press Enter. In the "System Properties" window, click the "Advanced" tab -> in the "Performance" section, click "Settings" -> then click the "Advanced" tab.
<img width="200" alt="Performance Options dialog" src="https://github.com/user-attachments/assets/35623080-0993-4c44-881a-a53c4932879a" />

*Image: In the Advanced tab of Performance Options, click the "Change..." button in the Virtual memory section.*

2. **Migrate virtual memory**:
   *   In the "Virtual Memory" window, **uncheck** "Automatically manage paging file size for all drives" at the top.
   *   Select **C drive** -> choose **"No paging file"** -> click **"Set"** (click "Yes" on the warning).
   *   Select **D drive** -> choose **"System managed size"** -> click **"Set"**.
<img width="200" alt="Virtual memory settings interface" src="https://github.com/user-attachments/assets/b07ec210-da1d-41a9-824a-8e5ae9b3346b" />

*Image: The completed interface. C drive shows "None", D drive shows "System managed".*

3. **Save and restart**: Click "OK" all the way to close all windows, then **restart your computer**. After restarting, the invisible `pagefile.sys` on C drive will automatically disappear, and the space will truly be freed.

> **⚠️ Pitfall Guide (Experience from stepping in pits)**:
> If you feel that clicking the "Set" button does "nothing", **that is normal**. As long as the interface shows "None" for C drive and "System managed" for D drive, it means the configuration has succeeded. You do not need to click repeatedly. Just click "OK" and restart your computer.

## 4. Main Event: Seamless Migration of Software Caches

### 4.1 WeChat (Smooth Migration)
**Action**: WeChat Settings -> Account and Storage -> Change storage location to `D:\WeChat` (or a new folder you create).
**Result**: Successfully moved. The C drive Documents directory instantly slimmed down.

<img width="400" alt="WeChat account and storage" src="https://github.com/user-attachments/assets/b63a2ea6-f455-4a1a-864d-b298ec1177c7" />

*Image: WeChat storage settings, changing the save path from C drive to D drive.*

### 4.2 QQ (Uninstall, Reinstall, and the OneDrive Trap)
**Action**: Tried to change the path in QQ's "Storage Management", but encountered permission issues.
**Pitfall**: When migrating QQ, it showed "Cloud file provider is not running (Error 0x8007016A)". This is because the original path was in OneDrive, and the files became "cloud placeholders" that could not be migrated locally.
**Handling**: Eventually gave up migration, **completely uninstalled QQ**, and forcibly deleted the `Tencent Files` leftovers on C drive to free up space. (**Note**: Planned to later download QQ from the official website, manually choose to install it on D drive, and after logging in again, set the chat history path on D drive, completely avoiding the OneDrive pit).

<img width="400" alt="QQ file migration error" src="https://github.com/user-attachments/assets/006cfd04-b299-4e6e-8dd8-b96b6eaa72ec" />

*Image: Error occurred while migrating QQ files.*

**Final Solution (Complete Fix)**:
Even after reinstalling QQ, it still repeatedly showed "Failed to open message file". Investigation found that QQ stubbornly tried to write data to `C:\Users\Qin Lang\OneDrive\Documents\Tencent Files`, and was stuck by OneDrive's on-demand download mechanism. So the following forced measures were taken:

1. **Thorough cleanup (remove all old leftovers)**:
   *   Force-end all QQ/Tencent processes.
   *   Delete leftovers such as `QQ`, `QQNT`, `TXSSO`, etc. under `AppData\Roaming\Tencent`.
   *   Delete cache under `AppData\Local\Tencent`.
   *   Force-delete the `Tencent Files` folder under `Documents` and `OneDrive\Documents`.
   *   Delete `C:\Program Files\Tencent\QQ` (the old version previously installed on C drive by mistake).

<img width="200" alt="Screenshot 2026-09-11 202039" src="https://github.com/user-attachments/assets/cb1d7f63-2ae4-49d5-840a-5fd2370f5241" />

*Image: Cleaning up QQ, QQNT, QQNTOpenSDK, QQTempSys, TXSSO folders in Roaming.*

<img width="300" alt="Screenshot 2026-09-11 202305" src="https://github.com/user-attachments/assets/09c9fb44-612d-4471-9c50-7c0cd9d46e2e" />

*Image: Force-deleting Tencent Files under system Documents.*

<img width="200" alt="Screenshot 2026-09-11 203357" src="https://github.com/user-attachments/assets/2d8e0040-da4d-43e5-bbcc-c8834df55acc" />

*Image: Deleting Tencent Files leftovers under OneDrive Documents.*

<img width="300" alt="Screenshot 2026-09-11 203536" src="https://github.com/user-attachments/assets/76aadbaa-1abf-4d59-bfcf-75970ea29480" />

*Image: Deleting the old QQ folder previously installed on C drive by mistake under Program Files.*

2. **Clean install**: Download the latest version from the official website, choose custom installation to a **brand-new path on D drive** (e.g., `D:\TencentQQ`).

3. **Log in and change the path immediately**:
   After installation, scan the QR code to log in. After successfully entering the QQ interface, **immediately** click the three lines in the lower-left corner -> Settings -> **Storage Management**.
   *   At this time, "Default save location for chat messages" still shows OneDrive. Click "Change storage path" and change it to a purely local **`D:\QQFiles`** (absolutely avoid OneDrive).

<img width="400" alt="Screenshot 2026-09-11 204030" src="https://github.com/user-attachments/assets/a8d94465-0a7e-4b86-b7f5-e0f7a750f597" />

*Image: Entering QQ settings to change the default storage path from OneDrive to a purely local path on D drive.*

4. **Perform data migration**: After confirming the path change, QQ will automatically migrate old data to the new path.

<img width="400" alt="Screenshot 2026-09-11 204245" src="https://github.com/user-attachments/assets/dc85adeb-b675-493c-a192-10c1b8b35df1" />

*Image: QQ is migrating historical data to the new path on D drive.*

5. **Verify result**:
   *   After migration is complete, restart QQ and confirm it no longer shows "Failed to open message file".
   *   Go to D drive and check whether `D:\QQFiles\Tencent Files` is generated normally.

<img width="400" alt="Screenshot 2026-09-11 204358" src="https://github.com/user-attachments/assets/549a3501-8df4-442a-8de2-131a1c4dd54d" />

*Image: Finally succeeded in saving chat messages to D drive, completely saying goodbye to the error.*

### 4.3 WPS Office (Cleaned Up a Full 9 GB!)

WPS's cleanup process was a bit twists and turns, but also very classic. It is divided into 5 key screenshots:

1. **Initial state**: WPS once took up as much as **10.9 GB**.
<img width="400" alt="WPS before cleanup" src="https://github.com/user-attachments/assets/8d72cf32-4a1c-468e-9f54-7919712a9fec" />

*Image: App cache 4.9 GB, cloud document cache 4.1 GB.*

2. **Clean app cache**: Click "Clean now" on the right side of "App cache". This freed 4.9 GB.

3. **Encountered cloud cache blockage**: When clicking to clean "Cloud document cache", a prompt appeared: "The remaining 4.1 GB cannot be cleaned for now" because it contained "files that have not been uploaded successfully". To avoid data loss, it must not be forcibly cleaned!
<img width="400" alt="WPS cannot clean prompt" src="https://github.com/user-attachments/assets/fc96d92a-93b6-42cf-b7b2-44147bed42e8" />

*Image: WPS prompts that there are unuploaded files and cannot be cleaned directly.*

4. **Migrate to D drive**: According to the prompt, click "Go to migrate". In the file selection box that pops up, select a newly created folder on D drive (e.g., `D:\WPS\WPS Cloud Document Cache`).

<img width="400" alt="WPS migration path selection" src="https://github.com/user-attachments/assets/14308691-2ab5-4a58-987d-4b5c1c1d60cc" />

*Image: Manually migrating cloud document cache to D drive.*

<img width="400" alt="WPS is migrating" src="https://github.com/user-attachments/assets/43e47291-82df-4792-a829-9834e8276df7" />

*Image: Migrating.*

5. **Final result**: After migration and cleanup, WPS space usage dropped from **10.9 GB to 1.9 GB**!
<img width="400" alt="WPS after cleanup" src="https://github.com/user-attachments/assets/8513a7a4-ae40-439c-82de-36c9d65364dc" />

*Image: Cleanup complete, usage dropped to 1.9 GB.*

### 4.4 Supplement: Microsoft Store Traps and Settings Adjustment

**💡 Important Conclusion: Try not to download common software from Microsoft Store!**

After experiencing QQ login failure and reinstallation, a big pit was discovered: **Microsoft Store apps are forced to install on C drive by default** (in the deeply hidden `C:\Program Files\WindowsApps` folder), and are often limited by system sandbox permissions, making data migration extremely difficult. In addition, the Store version and the official website version of some domestic software (such as QQ) do not share configurations, which can easily cause strange bugs such as "Failed to open message file".

**✅ Correct approach:**
Go to the software's official website, download the `.exe` installer, click "Custom installation" during installation, and change the path to D drive (e.g., `D:\Program Files\QQ`). This is independent and clean, and perfectly avoids the risk of C drive filling up.

**🛠️ "Emergency Settings" for the Built-in Store:**
If you really need to use Microsoft Store to download games (such as Xbox Game Pass), be sure to change the default path in advance.

1. **Open settings**: In Microsoft Store's "Settings", find "Game install options".
<img width="600" alt="Screenshot 2026-09-11 232522" src="https://github.com/user-attachments/assets/dcbfdadf-2d7d-4771-9f73-50596c3911d3" />

*Image: By default, the install drive is C and the install folder is `C:\XboxGames`.*

2. **Change drive**: Click the "Change drive" drop-down menu and select **`D:`**.
<img width="400" alt="Screenshot 2026-09-11 232755" src="https://github.com/user-attachments/assets/1acb3f95-1252-4bb3-bc2a-bb7804464d20" />

*Image: In the drop-down menu that pops up, switch the drive from C to D.*

3. **Change successful**: After switching, the system will automatically change the path to `D:\XboxGames`. Make sure the "Ask me about these options each time I install a game" switch is **On**.
<img width="400" alt="Screenshot 2026-09-11 232812" src="https://github.com/user-attachments/assets/bf98281b-37a2-45b5-9578-510fd5ba6942" />

*Image: Change successful. The install drive shows D and the folder becomes `D:\XboxGames`. Everything is ready.*

> **💡 Tip**: Below the screenshot there is also "Tencent App Store settings". `Tencent App Store` and `MobileAppEngine` (Mobile Application Engine) are mainly used to run Android phone apps and mobile games on a computer. If you normally do not need to run phone apps on your computer, you can uninstall them in "Settings -> Apps". This can free up several more GB of space!

## 5. Ultimate Weapon: WizTree Visual Analysis

<img width="600" alt="WizTree overview screenshot" src="https://github.com/user-attachments/assets/232b75ee-5db4-465f-972a-16f9b07159bb" />

*   **Tool**: WizTree (portable version, run as administrator).
*   **Purpose**: Scan C drive in 3 seconds. Colorful block map shows space distribution.
*   **Warning**: System lifelines such as `Windows` and `WinSxS`, no matter how large they look, absolutely must not be touched!

## 6. Recurrence Prevention Mechanism

| Strategy | Specific Action | Purpose |
| :--- | :--- | :--- |
| **Install path migration** | When installing new software, always manually change `C:\` to `D:\`. | Cut off at the source |
| **Storage Sense** | Settings -> System -> Storage -> Turn on "Storage Sense", run weekly. | Automatically clean junk |
| **Folder redirection** | Right-click Desktop, Downloads, Documents -> Properties -> Location -> Move to D drive. | Avoid C drive filling up |
| **Regular cache scanning** | Check the storage settings of WeChat, QQ, WPS, CapCut, etc. once a quarter. | Prevent cache rebound |

## Conclusion

Actually, cleaning up C drive is not difficult. The hard part is finding the right direction when facing a mess. Half a year ago, when helping a friend clean up, it was just a small fight that removed only a bit of the surface. This time, I spent a whole day following the clues and pulled out Windows' deeply hidden hibernation file, virtual memory, and "cache assassins" like WeChat, QQ, and WPS by the roots.

Every step in this note, I have stepped into the pit for you. The crazy moments such as QQ's bizarre error and WPS cloud cache blockage were all screenshotted and recorded one by one. If you are also a "C drive red-hot" person, please feel free to copy the homework. I have already cleared the roadblocks for you. May everyone's computer never turn red again and stay smooth as before!

---

## Appendix: PowerShell Command Line Troubleshooting Log

Because Windows File Explorer cannot accurately calculate the size of hidden folders, we used `Get-ChildItem` combined commands in the terminal for low-level inspection. Below are all the commands used in the troubleshooting process and selected results.

<details>
<summary>Click to expand: Core troubleshooting commands and execution results (full version)</summary>

### 1. Scan user data directories (AppData)

**Query the Local directory (the main location for software caches):**

```powershell
Get-ChildItem "C:\Users\YourUsername\AppData\Local" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 10
```

Execution results (selected):

```text
Programs                    3.94
Microsoft                   3.28
Kingsoft                    2.94
JianyingPro                 2.04
KOOK                        1.69
```

**Query the Roaming directory:**

```powershell
Get-ChildItem "C:\Users\YourUsername\AppData\Roaming" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 5
```

Execution results (selected): `kingsoft 3.78, Tencent 3.59, WNS 2.05, .minecraft 1.79, baidu 1.41`

### 2. Scan software installation directories (Program Files)

**Normal scan (does not show hidden folders):**

```powershell
Get-ChildItem "C:\Program Files" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 8
```

Execution results (selected): `MobileAppEngine 6.75, Huawei 4.86, Microsoft Office 4.3, Tencent 2.01`

```powershell
Get-ChildItem "C:\Program Files (x86)" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 8
```

Execution results (selected): `Microsoft 5.65, Steam 1.24, LeiGod_Acc 0.88, Tencent 0.29`

**Add the -Force parameter to penetrate hidden directories (find WindowsApps):**

```powershell
Get-ChildItem "C:\Program Files" -Directory -Force | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -Force -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 5
```

Execution results (selected): `WindowsApps 11.69, MobileAppEngine 6.75, Huawei 4.86, Microsoft Office 4.3, Tencent 2.01`

### 3. Scan system-level directories (ProgramData, Windows, Installer)

```powershell
Get-ChildItem "C:\ProgramData" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 8
```

Execution results (selected): `Comms 3.39, Microsoft 1.54, Huawei 0.57`

```powershell
Get-ChildItem "C:\Windows" -Directory | ForEach-Object { [PSCustomObject]@{Name=$_.Name; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 5
```

Execution results (selected): `WinSxS 13.23, System32 9.18, SystemApps 1.32`
*(⚠️ System lifelines, absolutely must not be deleted manually!)*

**Measure the Windows\Installer directory separately (normal scans may miss it):**

```powershell
[math]::Round((Get-ChildItem "C:\Windows\Installer" -Recurse -File -Force -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)
```

Execution result: `15.97` GB
*(⚠️ System installation cache used for software uninstall and repair. Absolutely must not be deleted manually!)*

### 4. Full root directory scan (penetrate hidden directories)

```powershell
Get-ChildItem "C:\" -Directory -Force | ForEach-Object { [PSCustomObject]@{Name=$_.FullName; SizeGB=[math]::Round((Get-ChildItem $_.FullName -Recurse -File -Force -ErrorAction SilentlyContinue | Measure-Object -Property Length -Sum).Sum / 1GB, 2)} } | Sort-Object SizeGB -Descending | Select-Object -First 10
```

Note: Because there are hidden files protected by system permissions at the root of C drive, this command will report an error "The system cannot find the file specified", but it can still output results.
Execution results (selected):

```text
C:\Windows                    50.64
C:\Program Files              32.21
C:\Program Files (x86)         9.03
C:\ProgramData                 6.11
C:\$RECYCLE.BIN                   5
C:\Recovery                    3.17
```

### 5. Execute cleanup operations

**Disable system hibernation and free hibernation file (hiberfil.sys) space:**

```powershell
powercfg -h off
# Instantly frees space approximately equal to physical memory size (e.g., 16 GB RAM frees 16 GB). Does not affect normal shutdown and sleep.
```

</details>
