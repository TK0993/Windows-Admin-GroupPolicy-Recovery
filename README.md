# Windows-Admin-GroupPolicy-Recovery
 CompTIA A+ Practical Lab: Cross-Platform Admin Access Restoration via Linux GRUB Root Shell.
# 📑 IT Case Study: Local Group Policy Registry Corruption & Cross-Platform Recovery
**System Specialist:** Tom Kemp  
**Target Hardware:** Acer Predator Helios 300 (Dual-Boot: Windows 11 / Ubuntu Linux)  

---

## 1. Executive Summary & Ticket Details
* **Incident Description:** The user profile lost all functional Administrative rights despite the Windows User Accounts UI (`netplwiz`) displaying the account as a member of the "Administrators" group. 
* **Impact:** Critical. The system suffered from a broken token loop. User Account Control (UAC) prompts were grayed out, preventing any software installation, elevated command usage, or system modifications.

---

## 2. Phase 1: Diagnostics & Troubleshooting (Windows)
The initial diagnostic path focused on verifying the token failure and attempting localized Windows repairs:

* **Symptom Verification:** Attempted to execute everyday administrative tasks (launching applications, opening an elevated Command Prompt). All requests were blocked by an unusable UAC prompt.
* **Security Evaluation:** Checked the User Account Control settings. Attempted to adjust the UAC slider to *"Never Notify"* to force-break the registry loop. The loop remained unbroken.
* **Native Recovery Attempt:** Executed a hard reboot into the **Windows Recovery Environment (WinRE)** via `Shift + Restart`. Opened the recovery command line to manually inject a secondary administrator account. The system rejected the permission requests natively.

---

## 3. Phase 2: Pivoting to Cross-Platform Intervention (Linux)
Because Windows security handles were entirely inaccessible from within the OS, the strategy pivoted to using an isolated secondary operating system previously deployed on the hardware.

* **Roadblock (Lost Credentials):** The backup Ubuntu partition password was forgotten. 
* **Sub-Resolution (GRUB Exploitation):** Intercepted the boot sequence via the **GRUB Bootloader Menu**. Edited the kernel parameter line temporarily appending `init=/bin/bash` to bypass the graphical login screen. Dropped to a root shell prompt, re-mounted the filesystem as read/write (`mount -o remount,rw /`), identified the local username (`ls /home`), and reset the credentials using the `passwd` command.
* **Filesystem Mounting Issues:** Attempted to mount the Windows NTFS drive partition inside Linux. The mount was repeatedly denied due to a Windows **Fast Startup** lock file, causing Linux to read the partition as "dirty" or hibernated.
* **Forced Unlock:** Cleared the Windows hibernate file state from the Linux terminal using the `ntfsfix` utility to successfully force-mount the filesystem.

---

## 4. Phase 3: Risk Mitigation & Data Preservation
Before executing any destructive OS actions, a comprehensive data backup plan was enacted. Due to severe storage constraints on the internal Linux partition and a lack of external media, data had to be selectively targeted:

* **Application Caches vs. User Data:** Heavy game files were explicitly bypassed to prevent a drive overflow crash.
* **Targeted Backup:** Using the terminal, critical files were staged and duplicated from the Windows mount over to the isolated Linux `/home` space, including:
  * Local game save profiles from `AppData` (Steam, EA, and Ubisoft configs).
  * The user's `Documents` library.
  * Critical external documentation (a vital medical sick note located in the `Downloads` directory).

---

## 5. Phase 4: Final Resolution & Post-Install Verification
* **Resolution Action:** Booted back into WinRE and initiated a **"Reset This PC"** operation with the **"Keep my files"** parameter selected. This completely flushed and rebuilt the broken Windows Registry and Local Group Policy databases while keeping default folders intact.
* **Post-Fix Testing:** 
  1. Executed `netplwiz` to confirm the fresh account profile mapped securely to the **Administrators Group**.
  2. Successfully triggered an elevated Command Prompt window displaying the title handle: `Administrator: Command Prompt`.
  3. Verified unrestricted app installation capabilities.
* **Data Restoration:** Logged back into Ubuntu, copied the safely isolated `Downloads` folder and game profiles, and pasted them back into the active Windows directory paths. **Case Closed.**

---

## 📘 CompTIA A+ Exam Objectives Covered:
* **Domain 1.2 (Windows Command Line Tools):** Utilizing `net user` and directory tracking commands.
* **Domain 1.3 (Windows OS Features):** Working deep within the Windows Recovery Environment (WinRE) and Advanced Startup options.
* **Domain 1.6 (Linux OS Fundamentals):** Terminal navigation, file management tools (`cp`, `rm`, `ls`), permission management, and boot sequence configuration via the GRUB interface.
* **Domain 2.4 (User Account Control):** Understanding UAC permissions, rights elevation, and troubleshooting group policy account corruption.
* **Domain 4.1 (Documentation Best Practices):** Writing a structured incident report logging problem symptoms, errors, solutions, and outcomes.
