# soc-lab-wazuh

In this project, I wanted to step away from theory and see how real-time security monitoring works. I built a home lab to understand how a professional SOC (Security Operations Center) detects and responds to threats.


I set up a Wazuh SIEM/XDR manager to act as the "brain" of my lab. I connected different "endpoints" (Windows and Linux machines) to it using agents. To get even better data from Windows, I installed Sysmon, which gives me much more detail than standard system logs.

The main goal was to test the system in action. I didn't just wait for alerts - I simulated my own attacks using Hydra (brute-force), manual attempts of that, and nmap scans. I also practiced "tuning" the system by writing my own rules to stop false alarms.

I started the project by building a dedicated virtual network using VirtualBox. I created three virtual machines to simulate a real-world server-client environment:

**Wazuh Manager: A Linux Ubuntu server to collect and analyze all security logs.**

**Linux Agent: A second Ubuntu machine for testing Linux security.**

**Windows Agent: A Windows 10 machine for monitoring Windows-specific events and security tests.**

To make my work efficient, I controlled all virtual machines remotely from my main computer using SSH. This allowed me to run commands and manage the systems without switching between VirtualBox windows.

I organized my workspace to see everything at once. I used a split-screen view to monitor the Linux terminals, Windows PowerShell, and the Wazuh Dashboard simultaneously. This allowed me to see how an action on an agent (like a failed login) immediately appeared as an alert in the SIEM.

<img width="1194" height="630" alt="image" src="https://github.com/user-attachments/assets/77b18e45-4c08-493f-b5ae-f6fb059634c9" />

At this stage, the Windows machine was using only default logs. Later in the project, I upgraded this setup with Sysmon to get more detailed information.

## Nmap scanning

Before I started any security tests with Nmap, I spent some time looking at the Wazuh Dashboard. I noticed a strange alert (Level 7) coming from my Linux agent. The rootcheck module was flagging the /usr/bin/diff file as a potential threat.

<img width="1249" height="159" alt="image" src="https://github.com/user-attachments/assets/6d672837-10ba-461f-8cd5-6f5252134647" />


I quickly checked the system and confirmed this was just a False Positive. The file was safe, but the SIEM was creating unnecessary "noise". To fix this, I decided to tune the configuration. I opened the agent.conf file on the Wazuh Manager and added a small piece of XML code to tell the system to ignore this specific file.

<img width="549" height="156" alt="image" src="https://github.com/user-attachments/assets/73e88d6d-7dc2-483b-9566-c2dfe0080500" />





