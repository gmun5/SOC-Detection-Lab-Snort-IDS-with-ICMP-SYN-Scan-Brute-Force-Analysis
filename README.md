# SOC Detection Lab: Snort IDS Traffic Analysis (ICMP, SYN Scan, and SSH Brute Force)

# Overview


This Snort Intrusion Detection Lab was built with two isolated virtual machines: an Ubuntu VM running Snort, and a Windows VM used to generate network traffic. The lab was designed to simulate three realistic SOC-relevant scenarios: an SSH brute-force simulation, a benign ICMP baseline, and a TCP SYN reconnaissance scan. 

The purpose of this lab was not just to "run Snort", but to practice the workflow a SOC analyst would actually follow. To simulate the workflow, I had to:
- Establish a normal baseline
- Generate suspicious activity
- Observe alerts and packet behavior
- Identify source/destination IP and protocol details

Because the host machine was a 2022 MacBook Pro with 8 GB of RAM, the lab was intentionally kept lightweight by using only two VMs at a time.

# Lab Objectives
The main goals of this lab were to:
1. Build an isolated virtual network for safe traffic generation
2. Configure Snort on Ubuntu to monitor live traffic
3. Simulate realistic SOC scenarios
4. Monitor and analyze live data traffic
5. Understand the difference between normal and suspicious network activity
6. Use Snort operational modes to analyze alert-based detection and packet-level data

# Environment

## Host System
- 2022 Macbook Pro
- 8 GB RAM
- VMware Fusion

## Virtual Machines
- Ubuntu Server (CLI) VM (*Snort sensor*)
- Windows VM (*traffic generator*)

## Network Design
- Both virtual machines were configured to use a host-only network in VMware Fusion

## Key Lab IP Information
- Ubuntu VM: `172.16.14.129`
- Windows VM: `172.16.14.128`
- Lab network (subnet): `172.16.14.0/24`
- Snort monitoring interface: `enp2s0`

# Tools Used
- Snort
- Nmap on Windows VM
- OpenSSH Client
- VMware Fusion
- Ubuntu Server (CLI)
- Windows Command Prompt

# Skills Demonstrated
- IDS setup and validation
- Network segmentation and VM isolation
- Snort traffic inspection and alert monitoring
- ICMP traffic analysis
- TCP SYN reconnaissance detection
- Repeated authentication attempt analysis
- Traffic pattern interpretation

# Setup


## Step 1: Configure the Virtual Network

The first requirement for this lab was isolation. In VMware Fusion, both the Ubuntu VM and the Windows VM were configured to use Private to my Mac. This creates a host-only network so the traffic remains contained inside the lab. If the VMs were left on NAT or bridged networking, scan traffic could mix with the real host network. That would make the lab less controlled and less professional.

### Screenshot(s)

<img width="1920" height="1080" alt="Screenshot 2026-04-01 at 9 15 04 PM" src="https://github.com/user-attachments/assets/bed70478-e3ca-4e3c-9145-1855797ae6ff" />


<img width="1920" height="1080" alt="Screenshot 2026-04-01 at 9 15 54 PM" src="https://github.com/user-attachments/assets/efcf3247-13f3-40a4-be6a-c7c6472a2907" />


## Step 2: Install Snort on Ubuntu

The second step in the lab was installing Snort on the Ubuntu VM. This ensured the IDS was available before configuring networking and generating traffic.

### Screenshot(s)

<img width="1920" height="1080" alt="Screenshot 2026-03-31 at 10 22 55 PM" src="https://github.com/user-attachments/assets/39eaaaf4-8a53-4499-9fcc-f1a93e36d224" />

<img width="1920" height="1080" alt="Screenshot 2026-03-31 at 10 30 29 PM" src="https://github.com/user-attachments/assets/310ce1e8-03c0-45f5-bd2c-ba1a406c9c83" />


### Installation
I ran `sudo apt-get update` to download the latest package information for Ubuntu. I then ran `sudo apt-get install snort -y` to install Snort. Lastly, I typed `snort --version` to verify the installation.


## Step 3: Verify Network Interface and IP

The next step was to identify the active network interface, assigned IP address, and subnet information. This information was required to; specify the correct interface for Snort, correctly configure `HOME_NET` and target the Ubuntu VM from the Windows VM. 

### Screenshot

<img width="1920" height="1080" alt="Screenshot 2026-04-01 at 1 35 59 AM" src="https://github.com/user-attachments/assets/3fbf5981-bfe2-4217-929e-78f2616ffa35" />


### Verification

I began by running `ip a` to display the IP addresses and status of the network interface on the Ubuntu server. From there, I identified the interface name `enp2s0`, Ubuntu IP address `172.16.14.129` and subnet `172.16.14.0/24`.


## Step 4: Configure HOME_NET in Snort

The fourth step in this lab was configuring `HOME_NET`. Snort requires a defined internal network range to properly distinguish between trusted internal traffic and potentially suspicious activity. In this lab, both the Ubuntu VM and Windows VM were assigned IP addresses within the `172.16.14.0/24` subnet. 

Leaving `HOME_NET` as `any` makes alerts less meaningful. To ensure accurate monitoring and detection, the `HOME_NET` variable in the Snort configuration was updated to match the `172.16.14.0/24` subnet.

### Screenshot

<img width="1920" height="1080" alt="Screenshot 2026-03-31 at 11 54 06 PM" src="https://github.com/user-attachments/assets/f7bf8586-326b-4b25-a11a-d1c2bbfd28c1" />


### Configuration

Firstly, I ran `sudo nano /etc/snort/snort.conf` to open the Snort configuration file so i'd be able to edit it. From there, I located `ipvar HOME_NET any` and changed it to `ipvar HOME_NET 172.16.14.0/24`. 


## Step 5: Validate the Snort Configuration

Before running Snort, I had to validate the configuration. This prevents wasting time troubleshooting live traffic if anything goes wrong, and makes sure everything is configured how its supposed to. 

### Screenshot

<img width="1920" height="1080" alt="Screenshot 2026-04-01 at 12 26 13 AM" src="https://github.com/user-attachments/assets/cb7fa47d-1bd6-47e1-8a87-ef9686ca81c4" />


### Validation

I ran `sudo snort -T -c /etc/snort/snort.conf -i enp2s0` in Ubuntu. This validates that the Snort configuration, rules, and network interface are correctly set up and ready for execution. The command prints a long validation report, and at the end of it writes "**Snort successfully validated the configuration!**".


## Step 6: Create Custom Snort Detection Rules

Before running Snort, I also had to create custom detection rules. For this step, I ran `sudo nano /etc/snort/rules/local.rules` to add custom detection rules to the `local.rules` file. Adding custom detection rules defines what types of traffic Snort should identify and alert on within the lab environment. These custom rules were used to tell Snort what suspicious or notable activity to look for, rather than relying only on default behavior.

### Screenshot

<img width="1914" height="1033" alt="Screenshot 2026-04-01 at 3 03 07 AM" src="https://github.com/user-attachments/assets/6e79b416-4d40-4b7f-9535-a42dea5bae95" />


### Rules Implemented

<ins>SSH Brute Force Detection:</ins>

`alert tcp any any -> $HOME_NET 22 (msg:"SSH Brute Force Attempt"; flags:S; threshold:type threshold, track by_src, count 2, seconds 60; sid:1000003; rev:1;)`

**What It Means:**

Alert when 2 or more connection attempts to port 22 are detected from the same source within a 60 second time frame. This simulates brute force behavior by identifying repeated authentication attempts.

<ins>ICMP Rule:</ins>

`alert icmp any any -> $HOME_NET any (msg:"ICMP Test Detected"; sid:1000001; rev:1;)`

**What It Means:**

Alert whenever any system sends ICMP (ping) traffic to the internal network.

<ins>SYN Scan:</ins>

`alert tcp any any -> $HOME_NET any (flags:S; msg:"SYN Port Scan Detected"; sid:1000002; rev:1;)`

**What It Means:**

Alert when any system attempts to initiate TCP connections to the internal network using SYN packets, which may indicate port scanning or reconnaissance activity.


## Step 7: Nmap Installation

Next, I installed Nmap on the Windows VM to prepare for generating network traffic that would later be analyzed by Snort. This step ensured that a reliable tool was available to generate controlled network traffic.

### Screenshot(s)

<img width="1920" height="1080" alt="Screenshot 2026-04-01 at 2 10 24 AM" src="https://github.com/user-attachments/assets/6f355e07-bbd8-4ba0-b9a4-ca10f135b296" />

<img width="1920" height="1080" alt="Screenshot 2026-04-01 at 2 14 21 AM" src="https://github.com/user-attachments/assets/f4e45e08-7a01-41e1-ad3f-7bcaa9c97a0a" />


### Installation

The official Nmap website was accessed from the Windows VM to download the latest version of Nmap. The installer was executed, and Nmap was installed using the default configuration settings. 

After installation, the `nmap --version` command was ran in Command Prompt to verify that Nmap was successfully installed without any errors and was available on the system.

## Step 8: Verify Connectivity Between Virtual Machines

Before generating more advanced traffic, connectivity between the Windows VM and the Ubuntu VM was verified using a basic ICMP test. Doing so ensured that the lab environment was properly set up before generating additional traffic. Verifying connectivity is a critical prerequisite, as all subsequent activity (scans, authentication attempts, and monitoring in Snort) depends on successful communication between systems.

### Screenshot

<img width="1920" height="1080" alt="Screenshot 2026-04-01 at 2 15 02 AM" src="https://github.com/user-attachments/assets/df9fc990-3a21-44e9-bac0-5cdc027599ad" />


### Verification

From the Windows VM, I pinged the Ubuntu VM's IP address by running `ping 172.16.14.129` in the Command Prompt. The Windows VM sent ICMP echo requests to the Ubuntu VM IP address (172.16.14.129) to confirm that both systems could successfully communicate across the configured host-only network. 

The network configuration between both VMs was functioning correctly. The connection was successfully established, as indicated by reply messages from the Ubuntu VM (e.g., Reply from 172.16.14.129). No packet loss occurred (0% loss) confirming stable communication. The consistent response times indicated reliable connectivity.

## Step 9: Run Snort in Console Alert Mode

To start the lab, Snort was ran in console alert mode. This mode was used because the view is cleaner and more practical than dumping every raw packet.

### Screenshot

<img width="1920" height="1080" alt="Screenshot 2026-04-02 at 6 03 29 AM" src="https://github.com/user-attachments/assets/2d72058f-db78-4917-9ab0-e66f6377abd6" />


### Execution
To begin, I ran `sudo snort -i enp2s0 -A console -q -c /etc/snort/snort.conf`. This command starts Snort in console mode, instructing it to monitor traffic on the specified network interface, apply the configured rules and settings from the configuration file, and display any detected alerts in real time.


# Lab Scenarios


## Scenario 1: ICMP Echo Baseline

Establish a baseline of benign network traffic before introducing suspicious activity.

### Actitivy Execution

To generate baseline traffic, I initiated ICMP requests from the Windows VM to the Ubuntu VM by using the `ping` command targeting the Ubuntu IP address `172.16.14.129`. 


### Observed Behavior

As a result, multiple alerts were generated in real time, indicating that ICMP traffic was being detected and matched against the custom rule defined in the `local.rules` file.

The Snort console output displayed repeated alerts labeled **"ICMP Test Detected"**, along with timestamps, source IP (`172.16.14.1`), and destination IP (`172.16.14.129`). The repeated alerts confirmed that Snort was successfully capturing and analyzing live network traffic based on the configured detection rules.


### Screenshot

<img width="1920" height="1080" alt="Screenshot 2026-04-01 at 2 32 37 AM" src="https://github.com/user-attachments/assets/64314f05-2165-44da-89b3-0bc2b36f37b6" />


### Analysis

While the ping command was running, Snort was actively monitoring traffic on the Ubuntu VM. This scenario established a benign baseline by generating normal ICMP echo traffic from the Windows VM to the Ubuntu Snort sensor. The traffic pattern was expected and consistent, making it useful as a reference point for later suspicious scenarios.

## Scenario 2: TCP SYN Scan (Reconnaissance)

Simulate early-stage attacker reconnaissance by identifying open or responsive TCP ports without fully establishing connections.

### Activity Execution

To simulate reconnaissance activity, I initiated a SYN scan from the Windows VM targeting the Ubuntu Snort sensor using the following command: `nmap -sS 172.16.14.129`. This command performs a TCP SYN scan, which sends connection requests to multiple ports on the target system without completing the full TCP handshake. 


### Observed Behavior

After executing the scan, Snort generated multiple real-time alerts indicating that SYN packets were being sent from the Windows VM to the Ubuntu VM across various destination ports.

<ins>Each alert included:</ins>

- Source IP: `172.16.14.128` (Windows VM)
- Destination IP: `172.16.14.129` (Ubuntu VM)
- Different destination ports (e.g., 79, 259, 7443, etc.)
- Alert message: **“SYN Port Scan Detected”**

The high volume of alerts in a short time frame confirmed that multiple connection attempts were being made rapidly across different ports, which is consistent with port scanning behavior.

### Screenshot(s)

<img width="1914" height="112" alt="Screenshot 2026-04-02 at 2 52 10 PM" src="https://github.com/user-attachments/assets/f855d9bb-250a-42f8-ab0a-dada574d71af" />

<img width="1916" height="1080" alt="Screenshot 2026-04-02 at 2 52 35 PM" src="https://github.com/user-attachments/assets/fe8521d1-29ad-488b-9c0d-9e7cc49cce76" />


### Analysis

This scenario simulated early-stage attacker reconnaissance by identifying open or responsive TCP ports without fully establishing connections. The scan behavior was characterized by repeated SYN packets sent to multiple ports, which is commonly used by attackers to map services running on a target system.

Because the TCP handshake was not completed, the activity appeared as numerous connection attempts rather than full sessions, making it indicative of a half-open (SYN) scan. This type of behavior is frequently used to gather information while minimizing detection.

The Snort alerts confirmed that the custom SYN detection rule successfully identified this pattern of reconnaissance activity.


## Scenario 3: SSH Brute Force Simulation

Simulate repeated failed login attempts that resemble a credential attack.

### Activity Execution

To simulate a brute force attack, I initiated repeated SSH login attempts from the Windows VM to the Ubuntu Snort sensor using the following command: `ssh analyst@172.16.14.129`. After entering the command, multiple incorrect passwords were intentionally entered in succession. This generated repeated authentication failures, mimicking brute force behavior where an attacker attempts to gain unauthorized access by trying multiple credentials within a short period of time.

### Observed Behavior

On the Windows VM, each failed login attempt resulted in repeated “Permission denied” responses, confirming that authentication was unsuccessful.

At the same time, Snort generated alerts labeled **“SSH Brute Force Attempt”**, indicating that multiple connection attempts to port 22 were being detected within a short time frame.

<ins>Each alert included:</ins>

- Source IP: `172.16.14.128` (Windows VM)
- Destination IP: `172.16.14.129` (Ubuntu VM)
- Destination Port: `22` (SSH)

Additionally, the Snort output also showed **"SYN Port Scan Detected"** alerts, which occurred because each SSH attempt begins with a TCP connection request, triggering both the SYN scan rule and the brute force detection rule.

### Screenshot(s)

<img width="1914" height="224" alt="Screenshot 2026-04-01 at 3 08 02 AM" src="https://github.com/user-attachments/assets/3d4ff3c1-59e1-4dd4-842b-28dafdd54e51" />

<img width="1914" height="1080" alt="Screenshot 2026-04-01 at 3 07 40 AM" src="https://github.com/user-attachments/assets/122cacc9-86e3-42b5-b268-168ee87221ec" />


### Analysis

This scenario simulated a brute force attack by generating repeated SSH login attempts against the Ubuntu system. The behavior was characterized by multiple failed authentication attempts within a short period, which is a common method attackers use to gain unauthorized access.

The Snort brute force detection rule successfully identified this pattern by tracking repeated connection attempts from the same source IP to port 22. Because the threshold condition was met (multiple attempts within a defined time window), Snort generated alerts indicating potential brute force activity.

The presence of both brute force and SYN alerts demonstrates how different detection rules can overlap, providing additional context during analysis.

# Snort Operational Modes Analysis

## Mode 1: Console Alert Mode

As stated earlier, to start the lab, `sudo snort -A console -q -c /etc/snort/snort.conf -i enp2s0` was ran in the Snort terminal to initiate Console Alert Mode. 

### Screenshot(s)

<img width="1920" height="1080" alt="Screenshot 2026-04-02 at 6 03 29 AM" src="https://github.com/user-attachments/assets/4577ae1a-8fdf-4f09-8aef-89b92096f59b" />

<img width="1920" height="1080" alt="Screenshot 2026-04-01 at 2 32 37 AM" src="https://github.com/user-attachments/assets/d92f0ad3-8717-4691-9e6c-c219d1e65a2f" />


### What I Learned:

- This mode displays alerts only triggered by rules
- This mode is focused on detection, not raw traffic
- This closely resembles how alerts appear in a SIEM
- It is useful for quick analysis and identifying potential threats

## Mode 2: Verbose Packet Inspection Mode

To gain a deeper understanding of packet-level data, Snort was used in verbose packet inspection (sniffer) mode using `sudo snort -vde -c /etc/snort/snort.conf -i enp2s0`, allowing for a detailed analysis of raw network traffic.

### Screenshot(s)

<img width="1916" height="1080" alt="Screenshot 2026-04-02 at 3 34 21 PM" src="https://github.com/user-attachments/assets/c934666b-af37-4189-9cea-3cca4ccc3629" />

<img width="1916" height="1080" alt="Screenshot 2026-04-02 at 3 39 08 PM" src="https://github.com/user-attachments/assets/98f12618-e413-4705-81e6-4ab4c810c592" />

### What I Learned:

- Gained an understanding of which systems are communicating by analyzing source and destination IP addresses, ports, and protocols
- Gained an understanding on how to examinine packet headers, payload data, and overall packet size to understand how data is transmitted
- Gained an understanding of how to observe how systems interact on the network, including normal background activity (e.g. DNS requests) and system responses (e.g. ICMP “Destination Unreachable” messages)
