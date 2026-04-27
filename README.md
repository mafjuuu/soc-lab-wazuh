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

## Unexpected alert and quick investigation

Before I started any network scans or security tests, I noticed a Level 7 alert in my Wazuh dashboard. The system claimed it found a trojan in the /usr/bin/diff file on my Ubuntu machine. This was very strange because I was using fresh, clean virtual machines. I didn't want to just ignore it, so I decided to check what was actually going on.

Instead of just deleting the alert, I did a manual check. I used the sha256sum command to get the hash of the "suspicious" file on both of my Ubuntu machines.

<img width="1046" height="702" alt="image" src="https://github.com/user-attachments/assets/1356a8ba-1fc0-4351-b151-28565b754795" />

The hashes on both systems were exactly the same. This was 100% proof that the files were clean and had not been changed by any malware. It was just a False Positive (a mistake by the SIEM). To stop this alert from popping up again and "polluting" my dashboard, I went into the agent.conf file on the Wazuh Manager and added a rule to ignore that specific file.

<img width="465" height="129" alt="image" src="https://github.com/user-attachments/assets/99609058-f91a-4d06-85d3-659b2f2edcf8" />

By fixing this, I cleared the noise from my logs and could move on to more interesting tests, knowing that my environment was actually secure.

## Nmap scanning

After fixing the false alerts, I performed a network scan using Nmap to see if my SIEM would detect it. To my surprise, the dashboard showed no alerts at all. Wazuh didn't see the scan because the default Windows and Linux logs don't always track silent network probes.

<img width="1201" height="328" alt="image" src="https://github.com/user-attachments/assets/76a22055-7882-43eb-a576-975472266d71" />


I realized that I needed more data. My next step was to install Sysmon (System Monitor) on the Windows agent. Sysmon is a powerful tool that tracks much more than standard Windows Event Logs - it monitors network connections, process creations, and changes to file times.

After installing sysmon and configuration, instead of a blank screen, I could finally see the detailed activity of my system. The SIEM started catching things like suspicious PowerShell execution and discovery techniques. This proved that while Wazuh is the "brain," it needs good "eyes" like Sysmon to actually see the threats.

<img width="789" height="507" alt="image" src="https://github.com/user-attachments/assets/6f93d2fc-e6b6-4ecd-a341-e6af4ac2e346" />


<img width="1650" height="440" alt="image" src="https://github.com/user-attachments/assets/edf2929c-2b7d-4bab-92b9-4577f0b4d306" />

## Password guessing

After nmap scanning, I wanted to see how the system reacts to basic human activity. I tried to log in to the Linux machine by manually guessing passwords.

Wazuh caught this activity immediately. On the dashboard, I saw a Level 10 alert (which is a high priority). The description was very clear:

"User missed the password more than one time"

<img width="1065" height="173" alt="image" src="https://github.com/user-attachments/assets/4821a5b7-bae7-4531-8721-2e2238a55ad6" />


As you can see in the screenshot, the system also linked this event to the MITRE ATT&CK framework under the technique T1110 (Brute Force). This simple test proved that my SIEM was working correctly and could alert me in real-time if someone tried to break into my accounts.
Manual guessing was easily detected, but I wanted to see how the system handles a much faster, automated attack. This led me to my next test: SSH Brute Force with Hydra.

## Hydra attack

After testing manual login attempts, I wanted to see how Wazuh handles a much faster, automated attack. I used Hydra to perform an SSH brute force attack against my Linux server.

As you can see in the terminal, Hydra was attempting thousands of logins in a very short time.

<img width="1019" height="107" alt="image" src="https://github.com/user-attachments/assets/c05d3e89-79b0-4df3-9a9d-4e7bc3048ae4" />


The Results:
The SIEM reacted perfectly. The dashboard immediately showed a massive spike in alerts. Because Hydra sends so many requests, Wazuh aggregated these events and triggered high-level alerts for Multiple authentication failures.
<img width="1038" height="429" alt="image" src="https://github.com/user-attachments/assets/c709fad5-5f2f-4d4e-9cca-854f05e7837e" />
<img width="1501" height="248" alt="image" src="https://github.com/user-attachments/assets/41210365-8c65-416e-9740-53220be4696f" />
<img width="1534" height="360" alt="image" src="https://github.com/user-attachments/assets/6826bc39-56f8-4623-8d82-33b201ce5273" />



This test was the most important part of the project. It proved that my home lab can successfully detect real-world hacking tools and provide a clear warning to a security analyst.

## Monitoring file integrity (FIM)

Another important test I conducted was monitoring specific directories for any unauthorized changes. I wanted to see if the SIEM could detect when a sensitive file is modified.

I created a directory called /home/important/ and a file named dontopen.txt. I configured the Wazuh Syscheck module to monitor this folder in real-time.
<img width="732" height="84" alt="image" src="https://github.com/user-attachments/assets/872946c2-f003-47e2-9aa4-2f6eb828cf73" />


I originally wrote "1234567890" in the file. I then modified the file by adding the line "55555".

<img width="629" height="135" alt="image" src="https://github.com/user-attachments/assets/bde9d933-827e-4587-8175-f32c60d19ece" />


Wazuh detected the change immediately with a Level 7 alert. What’s most impressive is the level of detail: the system showed exactly what changed, including the file size and the old vs. new hashes (md5, sha1, and sha256). This proves that even if a hacker changes a single character in a sensitive file, the system will catch it and provide evidence for investigation.

## Project summary

This project was a hands-on experiment in building and managing a SOC environment. Using Wazuh as the central brain, I learned how to collect logs, tune security rules, and bridge "visibility gaps" with tools like Sysmon. The lab taught me that security is not just about seeing alerts, but about understanding the data behind them. By running real-world attacks like Nmap and Hydra, I could see exactly how a SIEM detects suspicious behavior and how an analyst must respond to keep the network safe.












