---
title: Uncovering LLMNR and NBT-NS Poisoning Attacks and Abusing WebDAV with Responder; From Exploitation to Defense

published: true
---


_In modern networks, protocols like LLMNR (Link-Local Multicast Name Resolution), NBT-NS (NetBIOS Name Service), and mDNS (Multicast DNS) are often used for local name resolution. While they help devices communicate in environments without DNS, these protocols also present significant security risks. Attackers can leverage tools like `Responder` to perform poisoning attacks, capturing sensitive credentials such as NTLM hashes_.

In this blog, we’ll demonstrate how these poisoning attacks work using Responder, discuss how WebDAV played a role in capturing an NTLMv2 hash, and analyze the captured traffic. We’ll also provide detection and prevention strategies to secure your network.




## [](#header-3) Understanding LLMNR, NBT-NS, and mDNS

`LLMNR` enables name resolution in local networks when a DNS query fails. It allows devices to broadcast requests and receive responses from other devices within the same network segment.

 `NBT-NS` is used to resolve NetBIOS names to IP addresses, mainly in legacy systems or networks where NetBIOS is enabled.

  `mDNS` is similar to LLMNR but primarily used by non-Windows devices such as macOS and Linux for local name resolution and service discovery.
  
The common flaw across LLMNR, NBT-NS, and mDNS is their inherent trust in responses from any network device. This lack of source validation allows attackers to execute MITM attacks, impersonate critical network resources, or capture NTLM authentication hashes. Once a machine authenticates to the attacker’s spoofed resource, it often sends NTLM credentials that can be cracked offline, granting the attacker unauthorized access to systems. 



### [](#header-3) Responder and Its Role in Poisoning Attacks:

`Responder` is an open-source tool that performs LLMNR, NBT-NS, and mDNS poisoning attacks by listening for name resolution requests on the local network and sending spoofed responses. This tricks victim machines into sending sensitive information, such as NTLMv2 hashes, which can be captured and used for further exploitation, such as cracking the hashes offline or leveraging them in Pass-the-Hash attacks to gain unauthorized access.

How Does Responder Work?
*   Listening for Queries: Responder starts by listening for LLMNR, NBT-NS, or mDNS queries on the local network.

*   Poisoning Responses: When a victim machine sends out a query, Responder responds as if it is the requested host.
  
*   Capturing Credentials: The victim machine attempts to authenticate with the attacker’s machine, sending its NTLMv2 hash.

Responder can al so perform other attacks, such as intercepting WPAD (Web Proxy Auto-Discovery Protocol) requests, DHCP, and DNS spoofing.


### [](#header-3) Running **Responder**

<img width="600" alt="responder1" src="https://github.com/user-attachments/assets/0bfe6da1-86b8-420e-9097-6945fdc8ee0d">

-I eth0: Specifies the network interface to listen on.
-w: Enables WPAD poisoning, which captures credentials from automatic proxy configuration.
-d: Enables DHCP poisoning, allowing Responder to offer a fake IP to devices requesting an IP address.

After running this command, Responder will listen for LLMNR, NBT-NS, mDNS, and WPAD queries. When a victim machine queries for a non-existent network share _\\red-teamDC_, Responder will send a poisoned response, causing the victim to authenticate with Responder.

<img width="800" alt="responder2" src="https://github.com/user-attachments/assets/236772c7-5c8c-4c2a-9c4c-17dcc934267d">

The provided Responder output demonstrates a successful network poisoning attack where the attacker used LLMNR, NBT-NS, and mDNS poisoning to capture an NTLMv2 hash via WebDAV.
WebDAV is a protocol used for remote file management over HTTP(S).

Responder captures the NTLMv2 hash and logs it in a file 

<img width="600" alt="responder1" src="https://github.com/user-attachments/assets/a439d61e-3a3d-4f06-899e-57b7ea0b6b39">

The file WebDAV-NTLMv2-*.txt contains the captured NTLMv2 hash.

### [](#header-3) Cracking the captured NTLMv2 with **Hashcat**

Save the hash from the log file (WebDAV-NTLMv2-*.txt).

Run Hashcat with the following command:

hashcat -m 5600 -a 0 WebDAV-NTLMv2-*.txt mywords.txt
-m 5600: Specifies the NTLMv2 hash type.
-a 0: Specifies a dictionary attack.

<img width="830" alt="hashcat" src="https://github.com/user-attachments/assets/a1d32fed-0fac-4494-a3e8-f9adffdc68b9">

If successful, Hashcat will display the cracked plaintext password, which can be used for further attacks.


### [](#header-3) How to detect **Responder**

As I dive into the Wireshark logs, I see multiple queries and responses that indicate the poisoning attack was successful.

![nbt-wireshark](https://github.com/user-attachments/assets/6680837c-672f-4667-acbd-863b29fc2965)

*   The victim, 192.168.50.208, is sending a Name query for RED-TEAMDC. The attacker’s IP x.x.x.251 responds, claiming to be RED-TEAMDC.

  
![llmnr](https://github.com/user-attachments/assets/572236cb-3322-4ef2-a3a2-255cc77645f8)
*   The victim is broadcasting LLMNR queries because it couldn’t resolve the hostname using DNS. The attacker IP x.x.x.251 sends a fake response, directing the victim to connect to it instead of the legitimate host.



### ![4](https://github.com/user-attachments/assets/d99c087e-210a-40ae-8d3d-a9894bf31c44)
*   Windows Defender flagged and blocked psexec.py due to the file-based activity and new service creation.

### ![5](https://github.com/user-attachments/assets/82193ee5-46cb-4f57-abcc-42b72e9bca74)




---



### [](#header-3) What is **wmiexec.py**


  ![6](https://github.com/user-attachments/assets/ea775741-5f4a-48c5-8a32-1803f1e55fcd)
*   Defender did not detect `wmiexec.py` due to its fileless operation, making it stealthier than `psexec.py`

### [](#header-3) Why Was `psexec.py` Blocked but `wmiexec.py` Wasn't

`psexec.py` triggers antivirus/EDR alerts by creating files and services. Conversely, `wmiexec.py` avoids file creation, making it less detectable by traditional AV solutions. However, it can still be detected via WMI logs and unusual port traffic.


### [](#header-4) Detectability and Mitigation


To detect `wmiexec.py`, monitor network traffic on ports 135 RPC and 445 SMB for unusual activity, check for suspicious command lines involving wmic.exe or powershell.exe with WMI arguments, and watch for commands targeting local admin shares. Additionally, track Windows login events (Event ID 4624) for unexpected or high-privilege logins, as these can indicate potential remote command execution.

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


