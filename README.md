# Ubuntu Server Security Hardening: SSH, UFW Firewall, and Service Reduction.

## Objective 

  This lab/project aims to develop practical skills in Linux system hardening using an Ubuntu Server environment. In this lab, we focused on identifying security weaknesses in a baseline system and implementing configurations to reduce the overall attack surface. Through the use of system analysis tools and commands, we examined active services, open ports, and existing firewall settings to understand the system’s initial security posture. We then applied hardening techniques, including securing SSH by disabling root and password-based authentication, implementing key-based access, configuring firewall rules using UFW, and disabling unnecessary services. The final step in this lab involved verifying the effectiveness of these changes by comparing the system’s state before and after hardening, ensuring improved security and reduced exposure to potential threats.

  ### Skills Learned

  This project has provided hands-on experience with the following security solutions/skills:

-   Linux system hardening and baseline security assessment
-   Identification of open ports and active services using command-line tools
-   Secure SSH configuration (disabling root login and password authentication)
-   Service management using systemctl (stopping and disabling unnecessary services)
-   Firewall configuration and rule management using UFW
-   Understanding and reducing system attack surface

  ### Tools Used

-   Ubuntu Server (Non-GUI)
-   UFW Firewall
-   Systemctl
-   Nano Text Editor
-   VirtualBox (Ubuntu VM)
-   Command-Line Tools (ssh, ssh-keygen, ssh-copy-id, ip a, ping, systemctl, etc.)

## Current lab environment Architecture/Setup

Below is a screenshot of my current lab Architecture (IM1). To quickly summarize, I have three VM's within VirtualBox, a Windows VM, and two Linux VM's, Kali Linux and Ubuntu. They are all routed to my pfSense firewall, which has a LAN and WAN interface, in which the WAN interface is used to route my VMs to the internet. 

<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/239fb858-97c8-4c3b-a85f-98510c60460a" />

IM1: Current Lab Architecture
## Steps/Procedure

### Part 0: Pre-Lab Checklist/setup

  To ensure this project would run smoothly and that my lab environment was set, I had to check a few things and make sure everything was correctly configured. 

  1. **UBUNTU SERVER VM INSTALLED.** My Ubuntu VM was already installed and configured a while ago, so that step was easy and out of the way (IM2).
  2. **SSH ACCESS AVAILABLE.** Part of this project was being able to SSH into my Ubuntu VM from my host machine. To ensure this worked, I had to do a few things. First, I had to enable a second adapter (specifically, host only) to make sure my host machine could reach my VM (IM3). One that was configured, I went into my Ubuntu VM and checked my IP configuration to see what IP address was given to the second adapter (IM4). After that was done, I went to see the status of SSH on my VM to see if it was running using the command "sudo systemctl status SSH," and saw that it was enabled and running (IM5). Once that was checked. I attempted to connect to my Ubuntu VM from my host. First, I used the ping command, and that worked. Then, I used the command "ssh username@ip_address" with my information, and it worked, confirming that SSH was working and could be used (IM6).
  3. **ABLE TO USE SUDO PRIVILEGES.** I needed to have elevated privileges to run most of these commands, so I checked if I did by running the command "sudo whoami" and it outputed root, which meant I had full access to all commands and options (IM7).
  4. **CONFIRMING INTERNET ACCESS.** To confirm that my Ubuntu VM had internet access, I simply pinged google.com and it worked, confirming that my Ubuntu VM had internet access (IM8).
  5. **SNAPSHOT OF VM BEFORE LAB.** I took a snapshot of the VM before starting, in case of any crashes or bugs, I could revert to its normal state.

<img width="629" height="605" alt="Screenshot 2026-04-08 175812" src="https://github.com/user-attachments/assets/9c40f788-d345-4a60-a8da-73986cbc6684" />

IM2: Ubuntu VM start-up screen

<img width="549" height="268" alt="Screenshot 2026-04-08 175929" src="https://github.com/user-attachments/assets/fdda7d4b-757d-40d6-827c-435ed3ef9372" />

IM3: Adding a second adapter to the Ubuntu VM so the host machine can access. 

<img width="815" height="316" alt="Screenshot 2026-04-08 180017" src="https://github.com/user-attachments/assets/bfad660c-632e-4207-b95f-d36d95e07de7" />

IM4: Checking IP configuration to see what IP the host adapter received. 

<img width="757" height="323" alt="Screenshot 2026-04-08 180149" src="https://github.com/user-attachments/assets/64322b7f-499f-4ade-9c2f-06b5e730609d" />

IM5: Checking status of SSH

<img width="628" height="515" alt="Screenshot 2026-04-08 180321" src="https://github.com/user-attachments/assets/7b75760a-8ece-49a5-9409-5ab4daba045b" />

IM6: Pinging and Using SSH to try and Reach the Ubuntu VM

<img width="299" height="56" alt="Screenshot 2026-04-08 180534" src="https://github.com/user-attachments/assets/9231520e-4e52-4a40-9f87-724c5b0fcbc6" />

IM7: Confirming that I had Root Privileges

<img width="736" height="159" alt="Screenshot 2026-04-08 180640" src="https://github.com/user-attachments/assets/61e9ad8a-0562-4799-be34-402730d68513" />

IM8: Confirming Ubuntu had VM access

### Part 1: Baseline System Assessment (Before Hardening)

  Before performing any hardening tasks, I had to see what my VM was currently like so I could compare it to its state after the hardening was complete and document it. Here is what I did to check the VM's current state:

  1. **IDENTIFY LISTENING PORTS.** I first wanted to check which ports were open on the VM, as this is one of the most common ways for threat actors to gain access into a system. To do this, I ran the command "ss -tuln" to see which ports were currently open and document them (IM9).
  2. **IDENTIFY OPEN SERVICES.** Many services are in use to make an operating system work, but some can be used for malicious purposes. I ran the command "systemctl list-units --type=service --state=running" to list all the services that were currently running on the system (IM10). I documented the total number of services and whether any might have been suspicious.
  3. **FIREWALL STATUS.** Next, I checked to see if the Firewall on the Ubuntu VM was running, as I was going to use it in this project. I ran the command "sudo ufw (uncomplicated firewall) status" to see if it was running, and it wasn't (IM11).
  4. **CHECKING SSH CONFIGURATION FILE.** This is the Linux file for all SSH configurations and settings. I checked to see if everything was normal and the status of root logins (IM12) and documented it.
  5. **DOCUMENT PRE-HARDENING STATUS.** After checking all these things, I wrote a short documentation summarizing my findings from this pre-hardening assessment (IM13).

<img width="1223" height="158" alt="Screenshot 2026-04-08 181030" src="https://github.com/user-attachments/assets/960067ed-19a6-4e58-91cd-6d927f772086" />

IM9: Checking what ports were currently open and listening

<img width="787" height="416" alt="Screenshot 2026-04-08 181635" src="https://github.com/user-attachments/assets/3756da82-76c8-4118-a54e-97ca9f9b52fd" />

IM10: Checking what services were currently running on the VM

<img width="450" height="200" alt="Screenshot 2026-04-08 181959" src="https://github.com/user-attachments/assets/67b11941-e7a9-4706-ad99-2d0adf8da20d" />

IM11: Checking Ubuntu firewall status

<img width="700" height="600" alt="Screenshot 2026-04-08 182300" src="https://github.com/user-attachments/assets/520029df-ac30-4163-a4d1-29445dce20ac" />

IM12: Checking SSH configuration file

<img width="668" height="251" alt="Screenshot 2026-04-08 182702" src="https://github.com/user-attachments/assets/8674db4a-f71b-4892-9f34-f3de8f9fd178" />

IM13: Pre-Hardening assessment documentation

### Part 2: Securing SSH Configuration: 

  Now that I had a baseline and understanding of the current state of the VM, it was time to start hardening the VM, starting with securing SSH, specifically securing the login aspect. 

  1. **EDITING SSH CONFIG FILE.** I first had to change some things in the SSH config file. I changed two things specifically. One, the "PasswordAuthentication" option to no, second, the "PermitRootLogin" to no as well (IM13&14). I did this because anyone could use a brute-force attack or privilege escalation attack to log in using SSH if these settings are not configured. Instead, it is much more secure to log in using key-based logins.
  2. **GENERATING SSH KEY.** To generate the SSH key, I went on my host machine and used the command "ssh-keygen" to generate a key for that session, and it generated me a random key (I used the default parameters) (IM15).
  3. **COPYING KEY TO UBUNTU VM.** Once the key was generated, I copied it to my VM by using the command "ssh-copy-id username@<vm-ip>" with my specific info.
  4. **TESTING KEY-BASED LOGIN.** To test if this was correctly configured, I tried logging in using SSH with the same command as before, and it worked without a password, confirming that the key-based login was correctly configured.

<img width="507" height="47" alt="Screenshot 2026-04-09 192329" src="https://github.com/user-attachments/assets/2b9dbb69-ff07-439a-a855-c3df133501ba" />
<img width="161" height="38" alt="Screenshot 2026-04-09 192352" src="https://github.com/user-attachments/assets/823bdbcd-9cf1-4495-91db-bb58f778a448" />

IM13&14: Editing SSH config file to disable password and root logins

<img width="571" height="382" alt="Screenshot 2026-04-09 193208" src="https://github.com/user-attachments/assets/da88fd75-d248-443c-8159-924f466bc515" />

IM15: Generating SSH key using "ssh-keygen" command in PowerShell

### Part 3: Disabling Unnecessary Services

  The next task was to disable any unnecessary services running on Ubuntu. Here is how I did that:

  1. **CHECKING WAS SERVICES WERE RUNNING AND WHICH TO DISABLE: I ran the command to check which services were running again to see what was running and what I wanted to disable. I decided to disable the Apache2 service. This service was specifically used by an HTTP server, and since my VM didn't have a GUI, there was no reason to keep that service open, as it just broadens the attack surface for threat actors. I first stopped the service with the command "sudo systemctl stop apache2" and then disabled it with the command "sudo systemctl disable apache2." After that was done, I checked the status of the service to ensure it was disabled, and it was (IM16). This is how I disabled an unnecessary service to reduce the attack surface.

<img width="1000" height="299" alt="Screenshot 2026-04-10 164904" src="https://github.com/user-attachments/assets/7f2918fb-3f8a-44af-95dd-0cf3418215b3" />

IM16: Stopping and disabling Apache2 in Ubuntu

### Part 4: Configuring UFW Firewall

  The UFW firewall in Ubuntu is used for basic traffic and networking filtering, and I wanted to configure it to better secure the VM. 

  1. **INSTALL UFW.** To install the firewall, I used the command "sudo apt install ufw -y" to install it. It was already installed, so I then checked the status of the firewall, and it was now active and running (IM17).
  2. **SETTING DEFAULT RULES.** I then set the default rules for the firewall. The two rules were "sudo ufw default deny incoming" and "sudo ufw default allow outgoing." These basic rules were set so that, unless some other rule was mentioned specifically, any incoming traffic was blocked by default, and any traffic was okay to leave (IM18).
  3. **ALLOWING SSH ACCESS.** One rule I wanted to make was allowing SSH by using the command "sudo ufw allow ssh" to create the rule.
  4. **VERIFYING FIREWALL STATUS.** The last thing was to verify the status of the firewall and see what rules were enabled. The command "sudo ufw status verbose" Showed the status along with other stats, including what rules are present, and everything was up to speed (IM19).

<img width="723" height="271" alt="Screenshot 2026-04-10 165239" src="https://github.com/user-attachments/assets/43cb6676-838f-44ae-b95c-2f2357b7ede3" />

IM17: Installing UFW firewall

<img width="451" height="94" alt="Screenshot 2026-04-10 165341" src="https://github.com/user-attachments/assets/af5bdc08-a5f5-43cc-900b-c3ae044f9381" />

IM18: Configuring default firewall rules

<img width="498" height="164" alt="Screenshot 2026-04-10 165544" src="https://github.com/user-attachments/assets/76be2c4c-3483-4564-af97-1a6f64a5d0bc" />

IM19: Checking status and overall configuration of firewall

### Part 5: Post-Hardening Verification

  Now that I hardened the Ubuntu VM, I wanted to verify what I did and see what difference it made

  1. **CHECKING OPEN PORTS AGAIN.** I checked what ports were open post-hardening and saw that port 80 was not listening anymore as the Apache web server was disabled, thus disabling port 80 (IM20).
  2. **CHECKING SERVICES AGAIN.** Using the command to check which services were running, I now saw that Apache wasn't there as it was no longer running (IM21).
  3. **CHECKING FIREWALL STATUS.** I checked the status of the firewall, and it was running and active (IM22).

<img width="1201" height="140" alt="Screenshot 2026-04-10 165621" src="https://github.com/user-attachments/assets/0706a073-203c-42bc-8c19-723440d5f19a" />

IM20: Open ports post-hardening 

<img width="786" height="309" alt="Screenshot 2026-04-10 165657" src="https://github.com/user-attachments/assets/789b5859-97d0-41be-a971-88da32de48bd" />

IM21: Running services post-hardening

<img width="418" height="117" alt="Screenshot 2026-04-10 165745" src="https://github.com/user-attachments/assets/682a918d-8e76-4aa8-a656-95e92076f447" />

IM21: Firewall Status post-hardening

### Part 6: Post-Hardening Documentation

  Now that all the hardening on the Ubuntu VM was complete, I did some documentation, highlighting all the changes that were made and how they better protect the system and how it reduced the attack surface (IM22). 

<img width="1421" height="517" alt="Screenshot 2026-04-10 172629" src="https://github.com/user-attachments/assets/d7883865-3c8a-4671-a8d0-a74221199848" />

IM22: Post-Hardening assessment documentation

## Conclusion

This lab was successful in demonstrating the implementation of foundational Linux security hardening techniques on an Ubuntu Server environment, with a focus on SSH configuration, UFW firewall rules, and service reduction. Through this project, key system hardening principles were applied in a practical setting, including securing remote access via SSH configuration adjustments, enforcing network traffic restrictions using UFW, and minimizing the attack surface by disabling unnecessary services.

In addition, several challenges were encountered during the implementation process. These included properly configuring SSH settings without locking out remote access, ensuring UFW rules were applied in the correct order to maintain connectivity while enforcing restrictions, and identifying and disabling non-essential services without impacting system functionality. Troubleshooting these issues reinforced the importance of careful validation when applying security controls at the system level.

Overall, the project strengthened the connection between theoretical cybersecurity principles and their real-world application in Linux system administration. It emphasized core defensive security practices such as access control, network filtering, and system minimization as key components of a hardened server environment.

Some potential areas for further improvement in this lab include implementing advanced SSH protections such as key-based authentication with disabled password login, integrating fail2ban for brute-force protection, expanding firewall rule complexity with zone-based segmentation, and conducting a formal vulnerability assessment of the hardened system.











