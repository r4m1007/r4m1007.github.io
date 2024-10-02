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


### [](#header-3) Detection and Mitigation

As I dive into the Wireshark logs, I see multiple queries and responses that indicate the poisoning attack was successful.

![nbt-wireshark](https://github.com/user-attachments/assets/6680837c-672f-4667-acbd-863b29fc2965)

*   The victim, 192.168.50.208, is sending a Name query for RED-TEAMDC. The attacker’s IP x.x.x.251 responds, claiming to be RED-TEAMDC.

  

  
![llmnr](https://github.com/user-attachments/assets/572236cb-3322-4ef2-a3a2-255cc77645f8)
*   The victim is broadcasting LLMNR queries because it couldn’t resolve the hostname using DNS. The attacker IP x.x.x.251 sends a fake response, directing the victim to connect to it instead of the legitimate host.
  


![mdns](https://github.com/user-attachments/assets/dc11729f-4e5b-412c-8b76-4fa9a5c0a4bf)

*   The victim’s IP sends an mDNS query asking for red-teamdc.local. The attacker IP replies with a fake mDNS response, pretending to be the requested host.

### [](#header-3) NTLM Authentication over WebDAV Analysis

This is where things get interesting. I see NTLM authentication attempts captured in the logs over HTTP, specifically using the WebDAV protocol. The victim machine tries to access a resource using WebDAV (PROPFIND method) and ends up sending its NTLMv2 hash.

![ntlmssp](https://github.com/user-attachments/assets/d6e82059-f4e7-4b29-a9b7-2124e33078b0)


*   The NTLM authentication process begins with the client (victim) sending an NTLMSSP_NEGOTIATE message to the server (attacker) to indicate that it wants to use NTLM for authentication. The attacker responds with an NTLMSSP_CHALLENGE message, which includes a unique challenge value. The client uses this challenge and its own credentials to compute a response and sends back an NTLMSSP_AUTH message containing the NTLMv2 hash.

![webdav-ntlmssp](https://github.com/user-attachments/assets/2d0db28f-4176-4fbe-b589-2561932657ba)

*   The 401 Unauthorized message typically indicates that the initial authentication attempt failed, prompting the client to resend its credentials. In an NTLM-based attack, the 401 response from the WebDAV server is expected and part of the process for capturing the NTLM hash.

*  200 OK: This response means the authentication was accepted, which typically only happens when the attacker has a valid hash or password to authenticate successfully.


---

### [](#header-3)Preventing These Attacks


*   **Disable LLMNR and NBT-NS:** Use Group Policy to disable these protocols on all endpoints.
*   **Block mDNS:** Restrict mDNS on Windows machines and non-Windows devices.(can have side effects depending on the environment and use cases)
*   **Disable WebDAV:**  Disable the WebClient service on Windows to prevent WebDAV attacks.
*   **Implement Strong Authentication:** Use Kerberos instead of NTLM and implement multi-factor authentication (MFA).




### [](#header-3) Conclusion

The vulnerabilities in protocols like LLMNR, NBT-NS, and mDNS highlight the importance of understanding how attackers can exploit them to launch MITM attacks and capture sensitive credentials. With tools like Responder, adversaries can easily take advantage of these weak points in the network, redirecting legitimate traffic and gathering NTLM hashes that can be cracked or used in further attacks. Our analysis of the captured traffic using Wireshark reveals how these poisoning attacks unfold, and we see the role that WebDAV can play in NTLM-based authentication scenarios.

To effectively defend against these threats, it’s crucial to disable or limit the use of these legacy protocols, implement strong authentication mechanisms, and continuously monitor network traffic for signs of unusual behavior.


