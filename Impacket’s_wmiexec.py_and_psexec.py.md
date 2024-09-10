This post explores the use of Impacket’s wmiexec.py and psexec.py, detailing their functions, detection methods, and my practical experiences with them.
Understanding Impacket



Impacket is a collection of Python libraries designed for network protocols like SMB, WMI, and NTLM. It is essential for conducting lateral movement, privilege escalation, and credential harvesting. In this post, I focus on two key Impacket tools: psexec.py and wmiexec.py.


What is psexec.py?



<img src="{{ "assets/images/Picture1.png" | relative_url }}" alt="Screenshot" style="width:700px;">


 
psexec.py is a tool that replicates Microsoft’s PsExec functionality, allowing remote command execution over SMB. Here’s how it operates:
1.	Creates a Remote Service: Uploads a randomly named executable to the hidden ADMIN$ share on the target machine.
2.	Registers the Service: Uses RPC and the Service Control Manager to register and start the service, running with elevated privileges.
3.	Communicates via Named Pipes: After service start, it uses named pipes for data communication.










Detection:

 

 
Service Creation: Logs in Windows Event ID 7045.


 
File Uploads: Monitored via SMB traffic on port 445.


 
Windows Defender flagged and blocked psexec.py due to the file-based activity and new service creation.

What is wmiexec.py?
wmiexec.py uses Windows Management Instrumentation (WMI) for command execution without file uploads. Here’s how it works:
1.	Executes Commands In-Memory: Runs commands directly in memory, avoiding disk writes.
2.	Uses RPC and WMI Service: Relies on port 135 for WMI communications.
3.	No New Files or Services: Avoids detection through file-based and service-based mechanisms.

 

Defender did not detect wmiexec.py due to its fileless operation, making it stealthier than psexec.py.
Why Was psexec.py Blocked but wmiexec.py Wasn’t?
psexec.py triggers antivirus/EDR alerts by creating files and services. Conversely, wmiexec.py avoids file creation, making it less detectable by traditional AV solutions. However, it can still be detected via WMI logs and unusual port traffic.
Detectability and Mitigation
Even though wmiexec.py is stealthier, it can be detected by:


 
Monitoring Port 135 and 445 Traffic: Look for unusual activity.


 
Checking Command-Line Logs: Event logs can reveal suspicious commands.



 
Analyzing Windows Login Activities: Check Event ID 4624 for unusual login patterns that may indicate remote command execution.
Reviewing WMI Event Logs and Windows login: Look for Event ID 4688 to detect remote command executions.
Post-Execution Steps with wmiexec.py
Once access is gained, you can perform various tasks:
•	Dump Credentials: Use tools like Mimikatz or secretsdump.py.
•	Enable Persistence: Create new users or scheduled tasks.
•	Gather Network Information: Use commands like netstat, ipconfig, or arp etc..
•	Run certutil: Interact with the Certificate Authority or download payloads.
Conclusion
wmiexec.py offers a stealthy approach to remote command execution, but it is not undetectable. Effective network and event logging can identify such activities. My experience with both psexec.py and wmiexec.py highlights the importance of understanding detection and evasion techniques, which I will continue to refine in my home lab.

