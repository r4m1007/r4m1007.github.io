---
title: Impacket’s wmiexec.py and psexec.py; How They Work and How to Detect

published: true
---


_This post delves into the technical functionalities of Impacket's psexec.py and wmiexec.py, outlining their use cases, detection methods, and my empirical findings during their deployment. The objective is to provide a detailed analysis from a purple team perspective, focusing on detection and evasion strategies._




## [](#header-3) Understanding Impacket

Impacket is a collection of Python libraries designed for working with network protocols such as SMB, WMI, and NTLM. These tools are crucial for various operations, including lateral movement, privilege escalation, and credential harvesting. In this post, I focus on two key Impacket tools: `psexec.py` and `wmiexec.py`

<img width="603" alt="impacket_" src="https://github.com/user-attachments/assets/f463c0e8-60e8-470d-b17c-d062c82ffbb8">



### [](#header-3) What is **psexec.py**

<img width="549" alt="1" src="https://github.com/user-attachments/assets/51468fed-5388-4e4e-8db4-539f0e8a9c31">


`psexec.py` is a tool that replicates Microsoft’s PsExec functionality, allowing remote command execution over SMB. Here’s how it operates:

1.	**Creates a Remote Service:** Uploads a randomly named executable to the hidden _ADMIN$_ share on the target machine.
2.	**Registers the Service:** Uses RPC and the Service Control Manager to register and start the service, running with elevated privileges.
3.	**Communicates via Named Pipes:** After service start, it uses named pipes for data communication.


### [](#header-3) How to detect **psexec.py**

To detect `psexec.py` usage, monitor Windows Event ID 7045 for the installation of randomly named services. This event indicates a new service, often used for remote execution with SYSTEM privileges. Additionally, check for SMB traffic on port 445, look for remote process execution. Use Event IDs 4624 and 4688 to monitor for abnormal logon and process creation events.

### <img width="549" alt="1" src="https://github.com/user-attachments/assets/10a3b9b9-fe20-4ef7-a080-523cb2ac8e30">

### ![3](https://github.com/user-attachments/assets/94ede26b-3520-4f0d-b077-75db732a5418)
*   Service Creation: Logs in Windows Event ID 7045


### ![4](https://github.com/user-attachments/assets/d99c087e-210a-40ae-8d3d-a9894bf31c44)
*   Monitored SMB traffic on port 445

### ![5](https://github.com/user-attachments/assets/82193ee5-46cb-4f57-abcc-42b72e9bca74)
*   Windows Defender flagged and blocked psexec.py due to the file-based activity and new service creation.



---



### [](#header-3) What is **wmiexec.py**




`wmiexec.py` uses Windows Management Instrumentation (WMI) for command execution without file uploads. Here’s how it works:

1.	**Executes Commands In-Memory**: Runs commands directly in memory, avoiding disk writes.
2.	**Uses RPC and WMI Service:** Relies on port 135 for WMI communications.
3.	**No New Files or Services**: Avoids detection through file-based and service-based mechanisms.


  ![6](https://github.com/user-attachments/assets/ea775741-5f4a-48c5-8a32-1803f1e55fcd)
*   Defender did not detect `wmiexec.py` due to its fileless operation, making it stealthier than `psexec.py`

### [](#header-3) Why Was `psexec.py` Blocked but `wmiexec.py` Wasn't

`psexec.py` triggers antivirus/EDR alerts by creating files and services. Conversely, `wmiexec.py` avoids file creation, making it less detectable by traditional AV solutions. However, it can still be detected via WMI logs and unusual port traffic.



### [](#header-4) Detectability and Mitigation


To detect `wmiexec.py`, monitor network traffic on ports 135 (RPC) and 445 (SMB) for unusual activity, check for suspicious command lines involving wmic.exe or powershell.exe with WMI arguments, and watch for commands targeting local admin shares. Additionally, track Windows login events (Event ID 4624) for unexpected or high-privilege logins, as these can indicate potential remote command execution.

Even though wmiexec.py is stealthier, it can be detected by:

![7](https://github.com/user-attachments/assets/8715f3ad-9813-4f2a-bd98-336caf683f5e)

*   **Monitoring Port 135 and 445 Traffic:** Look for unusual activity.

![8](https://github.com/user-attachments/assets/5cd7a100-1ba3-4201-9453-9984daee2f30)
*   **Checking Command-Line Logs:** Event Logs can reveal suspicious commands.

The command `cmd.exe /Q /c cmd.exe whoami 1> \127.0.0.1\ADMINS\_1725853925.0842316 2>&1` executed by wmiexec.py runs the whoami command on the target machine and redirects both its output and errors to a file on a local administrative share (127.0.0.1\ADMINS\). This allows wmiexec.py to execute commands remotely and capture their results through WMI, utilizing administrative shares for data collection and evasion.

<img width="455" alt="9" src="https://github.com/user-attachments/assets/61aa9180-3964-44c7-b74c-b420cee0f0a6">
*   **Analyzing Windows Login Activities:** Check Event ID 4624 for unusual login patterns that may indicate remote command execution.
*   **Reviewing WMI Event Logs:** Look for Event ID 4688 to detect remote command executions.



### [](#header-3)Post-Execution Steps with `wmiexec.py`

Once access is gained, you can perform various tasks:

*   **Enable Persistence:** Create new users or scheduled tasks.
*   **Dump Credentials:** Use tools like Mimikatz or secretsdump.py.
*   **Gather Network Information:** Use commands like netstat, ipconfig, or arp etc..
*   **Run certutil:** Interact with the Certificate Authority or download payloads.




### [](#header-3) Conclusion

`wmiexec.py` offers a stealthy approach to remote command execution by leveraging WMI. It uses WMI’s built-in capabilities to execute commands on target machines without relying on traditional remote access tools, which helps in evading some detection methods. This technique minimizes visibility compared to more overt tools like psexec.py. By redirecting output to administrative shares, wmiexec.py further reduces the likelihood of detection.

Despite its stealthiness, wmiexec.py is not undetectable. Effective detection involves monitoring WMI traffic on ports 135 RPC and 445 SMB for unusual patterns. Analyzing command-line arguments for tools like wmic.exe and powershell.exe and reviewing Windows Event IDs 5861 (WMI Event) and 4688 (Process Creation) can reveal suspicious activities. 


