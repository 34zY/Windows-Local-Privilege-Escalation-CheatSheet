# SeImpersonatePrivilege

[Back to Main](https://github.com/nickvourd/Windows-Local-Privilege-Escalation-Cookbook?tab=readme-ov-file#vulnerabilities)

## Table of Contents

- [SeImpersonatePrivilege](#SeImpersonatePrivilege)
  - [Table of Contents](#table-of-contents)
  - [Description](#description)
  - [Lab Setup](#lab-setup)
    - [Manual Lab Setup](#manual-lab-setup)
    - [PowerShell Script Lab Setup](#powershell-script-lab-setup)
  - [Enumeration](#enumeration)
    - [Manual Enumeration](#manual-enumeration)
    - [Tool Enumeration](#tool-enumeration)
  - [Exploitation](#exploitation)
  - [Mitigation](#mitigation)
  - [References](#references)

## Description

The SeImpersonatePrivilege is a Windows privilege that grants a user or process the ability to impersonate the security context of another user or account. This privilege allows a process to assume the identity of a different user, enabling it to perform actions or access resources as if it were that user.

However, if not properly managed or granted to unauthorized users or processes, the SeImpersonatePrivilege can pose a significant security risk. The SeImpersonatePrivilege vulnerability can be exploited by malicious actors to conduct various attacks and gain unauthorized access on a system.

## Lab Setup

### Manual Lab Setup

:warning: <b>For this scenario, it is recommended to use Windows Server 2019 (Build 17763) rather than Windows 10/11.</b>

1) Authenticate to the server as the local Administrator.

2) Open the "Server Manager" and select the option named "Add roles and features":

![SeImpersonatePrivilege-Labsetup-Manual-1](/Pictures/SeImpersonatePrivilege-Labsetup-Manual-1.png)

3) Press the "Next" button until you reach the "Server Roles" section:

![SeImpersonatePrivilege-Labsetup-Manual-2](/Pictures/SeImpersonatePrivilege-Labsetup-Manual-2.png)

4) Add the role named "Web Server (IIS)" and then press the button named "Add Features":

![SeImpersonatePrivilege-Labsetup-Manual-3](/Pictures/SeImpersonatePrivilege-Labsetup-Manual-3.png)

5) Press "Next", and in section named "Features" add the followings:
   - ASP.NET 4.7
   - WCF Services (1 of 5 installed)
     - HTTP Activation
    
Outcome:

![SeImpersonatePrivilege-Labsetup-Manual-4](/Pictures/SeImpersonatePrivilege-Labsetup-Manual-4.png)

6) Press "Next" until you reach the "Confirmation" section. In this section, check the checkbox for "Automatically restart":

![SeImpersonatePrivilege-Labsetup-Manual-5](/Pictures/SeImpersonatePrivilege-Labsetup-Manual-5.png)

7) Finally, press the "Install" button.

![SeImpersonatePrivilege-Labsetup-Manual-6](/Pictures/SeImpersonatePrivilege-Labsetup-Manual-6.png)

8) Verify that the IIS Web site is working:

![SeImpersonatePrivilege-Labsetup-Manual-7](/Pictures/SeImpersonatePrivilege-Labsetup-Manual-7.png)

9) Delete all files from "C:\inetpub\wwwroot" and add the [index.html](/Lab-Setup-Source-Code/index.html) and [cmdasp.aspx](/Lab-Setup-Source-Code/cmdasp.aspx):

![SeImpersonatePrivilege-Labsetup-Manual-8](/Pictures/SeImpersonatePrivilege-Labsetup-Manual-8.png)

### PowerShell Script Lab Setup

:warning: <b>For this scenario, it is recommended to use Windows Server 2019 (Build 17763) rather than Windows 10/11.</b>

To set up the lab with the 'SeImpersonatePrivilege' vulnerability is by using the custom PowerShell script named [SeImpersonatePrivilege.ps1](/Lab-Setup-Scripts/SeImpersonatePrivilege.ps1).

1) Authenticate to the server as the local Administrator.

2) Open a PowerShelll and run the script:

```
.\SeImpersonatePrivilege.ps1
```

Outcome:

![SeImpersonatePrivilege-Labsetup-Script](/Pictures/SeImpersonatePrivilege-Labsetup-Script.png)

## Enumeration

### Manual Enumeration

After initial access via Web application as `iis apppool\defaultapppool` user, to perform manual enumeration, you can use the following command to enumerate the current privileges of the user:

```
whoami /priv
```

Outcome:

![SeImpersonatePrivilege-Manual-Enum](/Pictures/SeImpersonatePrivilege-Manual-Enum.png)

### Tool Enumeration

To run the SharpUp tool and perform an enumeration of the `SeImpersonatePrivilege` vulnerability, you can execute the following command with appropriate arguments:

```
SharpUp.exe audit TokenPrivileges
```

Outcome:

![SeImpersonatePrivilege-Tool-Enum](/Pictures/SeImpersonatePrivilege-Tool-Enum1.png)

## Exploitation

To abuse this vulnerability you should use the [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) tool:

```
PrintSpoofer64.exe -i -c cmd
```

Outcome:

![SeImpersonatePrivilege-Exploitation](/Pictures/SeImpersonatePrivilege-Exploitation.png)

## Mitigation

Follow the steps below to remove the SeImpersonatePrivilege from a user:

1) Press Win + R to open the Run dialog, type secpol.msc, and hit Enter. This will open the Local Security Policy editor.
  
2) In the Local Security Policy editor, navigate to Local Policies > User Rights Assignment.

3) Look for the Impersonate a client after authentication policy (which corresponds to SeImpersonatePrivilege).

4) Double-click the policy, and a properties window will appear.

5) In the properties window, you can remove the user or group from the list to revoke the privilege. Click Apply and then OK to save the changes.

## References

- [Special privileges assigned to new logon Microsoft](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672)
- [Windows Potatoes by jlajara](https://jlajara.gitlab.io/Potatoes_Windows_Privesc)
- [LocalPotato by decoder-it](https://github.com/decoder-it/LocalPotato)
- [decoder-it blog](https://decoder.cloud)
