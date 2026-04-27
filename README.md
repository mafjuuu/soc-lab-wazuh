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

Before I started any network scans or security tests, I noticed a Level 7 alert in my Wazuh dashboard. The system claimed it found a trojan in the /usr/bin/diff file on my Ubuntu machine. This was very strange because I was using fresh, clean virtual machines. I didn't want to just ignore it, so I decided to check what was actually going on.

Instead of just deleting the alert, I did a manual check. I used the sha256sum command to get the hash of the "suspicious" file on both of my Ubuntu machines.

<img width="1046" height="702" alt="image" src="https://github.com/user-attachments/assets/1356a8ba-1fc0-4351-b151-28565b754795" />

The hashes on both systems were exactly the same. This was 100% proof that the files were clean and had not been changed by any malware. It was just a False Positive (a mistake by the SIEM). To stop this alert from popping up again and "polluting" my dashboard, I went into the agent.conf file on the Wazuh Manager and added a rule to ignore that specific file.

<img width="465" height="129" alt="image" src="https://github.com/user-attachments/assets/99609058-f91a-4d06-85d3-659b2f2edcf8" />

By fixing this, I cleared the noise from my logs and could move on to more interesting tests, knowing that my environment was actually secure.



After fixing the false alerts, I performed a network scan using Nmap to see if my SIEM would detect it. To my surprise, the dashboard showed no alerts at all. Wazuh didn't see the scan because the default Windows and Linux logs don't always track silent network probes.

<img width="1201" height="328" alt="image" src="https://github.com/user-attachments/assets/76a22055-7882-43eb-a576-975472266d71" />


I realized that I needed more data. My next step was to install Sysmon (System Monitor) on the Windows agent. Sysmon is a powerful tool that tracks much more than standard Windows Event Logs - it monitors network connections, process creations, and changes to file times.

Instead of a blank screen, I could finally see the detailed activity of my system. The SIEM started catching things like suspicious PowerShell execution and discovery techniques. This proved that while Wazuh is the "brain," it needs good "eyes" like Sysmon to actually see the threats.

<img width="1650" height="440" alt="image" src="https://github.com/user-attachments/assets/edf2929c-2b7d-4bab-92b9-4577f0b4d306" />

## Password guessing

After nmap scanning, I wanted to see how the system reacts to basic human activity. I tried to log in to the Linux machine by manually guessing passwords.

Wazuh caught this activity immediately. On the dashboard, I saw a Level 10 alert (which is a high priority). The description was very clear:

"User missed the password more than one time"

<img width="1065" height="173" alt="image" src="https://github.com/user-attachments/assets/4821a5b7-bae7-4531-8721-2e2238a55ad6" />


As you can see in the screenshot, the system also linked this event to the MITRE ATT&CK framework under the technique T1110 (Brute Force). This simple test proved that my SIEM was working correctly and could alert me in real-time if someone tried to break into my accounts.
Manual guessing was easily detected, but I wanted to see how the system handles a much faster, automated attack. This led me to my next test: SSH Brute Force with Hydra.










