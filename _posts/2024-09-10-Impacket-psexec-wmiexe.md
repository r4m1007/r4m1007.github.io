---
title: Impacket’s wmiexec.py and psexec.py

published: true
---


_This post explores the use of Impacket’s wmiexec.py and psexec.py, detailing their functions, detection methods, and my practical experiences with them_




## [](#header-3) Understanding Impacket

Impacket is a collection of Python libraries designed for network protocols like SMB, WMI, and NTLM. It is essential for conducting lateral movement, privilege escalation, and credential harvesting. In this post, I focus on two key Impacket tools: `psexec.py` and `wmiexec.py`




### [](#header-3) What is **psexec.py**


<img width="549" alt="1" src="https://github.com/user-attachments/assets/85147365-283d-4509-ba69-a8e1e7f0750e">


`psexec.py` is a tool that replicates Microsoft’s PsExec functionality, allowing remote command execution over SMB. Here’s how it operates:

1.	**Creates a Remote Service:** Uploads a randomly named executable to the hidden ADMIN$ share on the target machine.
2.	**Registers the Service:** Uses RPC and the Service Control Manager to register and start the service, running with elevated privileges.
3.	**Communicates via Named Pipes:** After service start, it uses named pipes for data communication.


### [](#header-3) How to detect **psexec.py**

### ![2](https://github.com/user-attachments/assets/7f54f869-4f54-4160-aee2-721774b8e1ec)


### ![3](https://github.com/user-attachments/assets/28994abe-26f0-41f4-9202-2a725709837e)
*   Service Creation: Logs in Windows Event ID 7045

### ![4](https://github.com/user-attachments/assets/7551432e-96f8-4614-aff1-7cc15fd183dd)
*   Monitored SMB traffic on port 445

### ![5](https://github.com/user-attachments/assets/1cc973b0-396a-48fb-875f-17dac29467a5)
*   Windows Defender flagged and blocked psexec.py due to the file-based activity and new service creation.



---



### [](#header-3) What is **wmiexec.py**




`wmiexec.py` uses Windows Management Instrumentation (WMI) for command execution without file uploads. Here’s how it works:

1.	**Executes Commands In-Memory**: Runs commands directly in memory, avoiding disk writes.
2.	**Uses RPC and WMI Service:** Relies on port 135 for WMI communications.
3.	**No New Files or Services**: Avoids detection through file-based and service-based mechanisms.


 ### ![6](https://github.com/user-attachments/assets/4ea54330-6ad3-49d4-85f4-cf9445318a8d)
*   Defender did not detect `wmiexec.py` due to its fileless operation, making it stealthier than `psexec.py`

### [](#header-3) Why Was `psexec.py` Blocked but `wmiexec.py` Wasn't

`psexec.py` triggers antivirus/EDR alerts by creating files and services. Conversely, `wmiexec.py` avoids file creation, making it less detectable by traditional AV solutions. However, it can still be detected via WMI logs and unusual port traffic.



### [](#header-4) Detectability and Mitigation

Even though wmiexec.py is stealthier, it can be detected by:
![7](https://github.com/user-attachments/assets/ced79dd9-6a4c-421f-8d95-5edce01ad638)
*   **Monitoring Port 135 and 445 Traffic:** Look for unusual activity.

![8](https://github.com/user-attachments/assets/6e983806-a2ab-4061-8f2d-9c7914a08aac)
*   **Checking Command-Line Logs:** Event Logs can reveal suspicious commands.

<img width="455" alt="9" src="https://github.com/user-attachments/assets/7049cec0-388b-4ca8-afec-4ed0215c5c8f">
*   **Analyzing Windows Login Activities:** Check Event ID 4624 for unusual login patterns that may indicate remote command execution.
*   **Reviewing WMI Event Logs:** Look for Event ID 4688 to detect remote command executions.



### [](#header-3)Post-Execution Steps with `wmiexec.py`

Once access is gained, you can perform various tasks:
•	**Dump Credentials:** Use tools like Mimikatz or secretsdump.py.
•	**Enable Persistence:** Create new users or scheduled tasks.
•	**Gather Network Information:** Use commands like netstat, ipconfig, or arp etc..
•	**Run certutil:** Interact with the Certificate Authority or download payloads.



### [](#header-3) Conclusion

`wmiexec.py` offers a stealthy approach to remote command execution, but it is not undetectable. Effective network and event logging can identify such activities. My experience with both psexec.py and `wmiexec.py` highlights the importance of understanding detection and evasion techniques, which I will continue to refine in my home lab.


