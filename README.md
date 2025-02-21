# EDR Attack and Defend Home Lab

## Project Overview  
This **EDR Attack and Defend Home Lab** simulates real-world cyberattacks and endpoint detection using industry tools. The lab consists of:  

- **Ubuntu Server attacker VM** running **Sliver** as the method of attack.  
- **Windows 11 defender VM** with **LimaCharlie** as the Endpoint Detection and Response (EDR) solution.  

The goal is to execute attacks, monitor activity, and analyze detections to understand both offensive and defensive security techniques.  

This lab will be completed using the guide by Eric Capuano: https://blog.ecapuano.com/p/so-you-want-to-be-a-soc-analyst-intro

## Objectives  
- **Set up the lab** with two virtual machines and proper networking.  
- **Simulate attacks** using Sliver on Ubuntu Server.  
- **Deploy LimaCharlie** on Windows 11 for EDR monitoring.  
- **Analyze detections** and document key findings. 

## Lab Setup
To begin the lab, we'll set up two virtual machines: Ubuntu Server for the attack machine and a Windows 11 machine to act as our endpoint. To ensure it doesn't spoil the fun for our attacker, Microsoft Defender will be disabled on the Windows machine. As for the attack, we'll use **Sliver** on the Ubuntu Server machine to simulate the offensive operations. On the Windows 11 machine, **LimaCharlie** will be installed as the EDR solution, with a sensor linked to the system and Sysmon logs being imported for enhanced detection capabilities.

### Virtual Machine Configuration
- **Windows 11 Defender VM:**
  - OS: **Windows 11**
  - Security: **Microsoft Defender** Disabled
  - EDR: **LimaCharlie** 

  Windows 11 Settings:![Windows 11 Settings](https://github.com/user-attachments/assets/7c18f950-b61d-4aea-b9fe-f1c36573a80c)

  Windows 11 Group Policy - Defender Disabled:![Windows 11 Group Policy - Defender Disabled](https://github.com/user-attachments/assets/ad09722d-ea3a-4b64-9497-ecd20cadaff0)

  Windows 11 Registry Settings:![Windows 11 Registry Settings](https://github.com/user-attachments/assets/beeba7b8-a42c-43c0-b86b-e4ec54c1c65a)

  LimaCharlie Sensor:![LimaCharlie Sensor](https://github.com/user-attachments/assets/8f897fbd-aeda-4029-a462-e7ba3fbb3e9d)

  LimaCharlie Artifact Collection Rules:![LimaCharlie Artifact Collection Rules](https://github.com/user-attachments/assets/7d82358d-4e1b-45a0-9b25-a7a6ddbab5fe)

- **Ubuntu Attacker VM:**
  - OS: Ubuntu Server
  - Tools: Sliver
  - Note: The Ubuntu Server machine will be controlled via SSH from the host machine for ease of use

   Sliver Install:![Ubuntu Sliver Install](https://github.com/user-attachments/assets/002936ac-71bc-4ce2-8a9f-dd1bd3278f0c)

## The Attack and Defense
Now that we have Sliver installed on our attacking machine, we can generate our payload and implant it on the Windows machine. As a result, we will establish a command and control (C2) session after the malware is executed on the endpoint.

### Malware Staging
- Generate and implant Sliver payload on the Windows machine
- Create a live session between the two mahcines

  Sliver Setup:![Ubuntu Sliver Setup](https://github.com/user-attachments/assets/ce367d92-5530-422a-a848-c0bc58f8af68)

  Payload Implanted:![Windows 11 Payload Implanted 2](https://github.com/user-attachments/assets/731b063b-3c82-47c8-b16e-a56ad0c60157)

  Live Session Established:![Ubuntu Sliver HTTP and Sessions](https://github.com/user-attachments/assets/7ba77e2e-40ff-44b7-9763-6056d99af5f0)

### Information Gathering
Because of the C2 session between the two machines, the attack machine can now gather information about the Windows 11 endpoint including privileges and system processes.

  Host Information and Privileges (the debug privilege will come in handy when we do some shady stuff later on):![Ubuntu Sliver Use and Privs](https://github.com/user-attachments/assets/4cc3ec05-f6ea-473d-b512-dab03b462649)

  System Process Tree; notice how Sliver has identified Sysmon as a defensive tool by highlighting it in red:![Ubuntu Processes Tree](https://github.com/user-attachments/assets/f5405f99-770f-4ed7-abfc-8e63f6146529)

### What LimaCharlie Sees
On our host machine, we can see telemetry from the attacker on the LimaCharlie EDR. We are able to identify the name of the payload and the IP that it's coming from.

Payload Identified in Processes:![LimaCharlie Detects Payload](https://github.com/user-attachments/assets/c21f4cdb-97ab-4334-879a-5184a981b00c)

Payload Network Detection:![LimaCharlie Network Tab](https://github.com/user-attachments/assets/65b5076c-74d8-4263-bdb3-a772f9b95937)

We can also use LimaCharlie to scan the file hash of the payload with VirusTotal. The scan will come back clean since we just created the payload ourselves and detections only come up if the hash has been scanned by VirusTotal in the past.

![LimaCharlie VirusTotal Scan](https://github.com/user-attachments/assets/43d8da6e-f218-4dfa-b1e1-2fead51e712c)

## Creating Rules for Detection
Time to simulate an attack to really test the capabilities of our LimaCharlie EDR. In this case, we'll be executing credential-stealing attack by dumping the LSASS memory. In LimaCharlie, we can observe the telemetry and create rules to detect this kind of attack in the future.

LimaCharlie detects attempts to access the LSASS memory as a "Sensitive Process":![LimaCharlie Sensitive LSASS](https://github.com/user-attachments/assets/62d3d5ff-1481-4060-8511-5c1e99fdb2d1)

Creating a rule in LimaCharlie that identifies the attack and reports it:![LimaCharlie LSASS ACCESSED Rule](https://github.com/user-attachments/assets/dfabd7d9-662c-4db3-96c7-1ab756901507)

LimaCharlie "LSASS ACCESSED" rule in use:![LimaCharlie Rule in Use](https://github.com/user-attachments/assets/79e4e7fe-d9cc-4d9a-a184-3016a7810669)

## Creating Rules for Detection **AND** Response
Now instead of only detection, we can create a rule that will detect **and** block attacks coming from the **Sliver** Server. Using the attacking machine, we can simulate parts of a ransomware attack by attempting to delete the volume shadow copies. We can then create and implement a rule using LimaCharlie that will completely block the attack. We can check the status of our server using the "whoami" command.

Using a shell to attempt to delete the volume shadow copies:![Ubuntu Shell-ShadowCopies](https://github.com/user-attachments/assets/b64bcc49-0548-45da-a9b5-afd509e3ad1a)

Telemetry of our attack on LimaCharlie:![LimaCharlie ShadowCopies Telemetry](https://github.com/user-attachments/assets/1be4a00d-e0fb-41a9-89d2-06438f50e403)

LimaCharlie Volume Shadow Copies Rule Creation:![LimaCharlie VSS Rule](https://github.com/user-attachments/assets/e23db392-c09f-45bd-97bf-ab19e12d4845)

If we try to run the attack again, the shell will hang and fail to return anything from the whoami command due to the termination of the parent process.

![Ubuntu whoami hang](https://github.com/user-attachments/assets/b66bfe50-b0a0-4fe4-a271-ff6b880c933c)

## Key Takewaways
- **Effective EDR Detection**: Getting hand-on experience with a powerful EDR solution like LimaCharlie taught me how to monitor a system in real time and successfully detect and respond to attacks.
- **Attacking Insights**: Using Sliver showed me how attackers use post-exploitation techniques, such as a C2 session, to maintain persistence and escalate privileges on affected systems
- **Importance of Logging and Monitoring**: This project highlighted how important logs and real-time monitoring are in identifying and mitigating threats before they escalate into major incidents.
