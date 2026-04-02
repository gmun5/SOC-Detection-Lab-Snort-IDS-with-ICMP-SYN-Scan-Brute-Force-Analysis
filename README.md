# SOC Detection Lab: Snort IDS with Brute Force, ICMP, and SYN Scan Analysis

# Overview


This Snort Intrusion Detection Lab was built with two isolated virtual machines: an Ubuntu VM running Snort, and a Windows VM used to generate network traffic. The lab was designed to simulate three realistic SOC-relevant scenarios: a controlled SSH brute-force simulation, a benign ICMP baseline, and a TCP SYN reconnaissance scan. 

The purpose of this lab was not just to "run Snort", but to practice the workflow a SOC analyst would actually follow. To simulate the workflow, I had to:
- Establish a normal baseline
- Generate suspicious activity
- Observe alerts and packet behavior
- Identify source/destination and protocol details

Because the host machine was a 2022 MacBook Pro with 8 GB of RAM, the lab was intentionally kept lightweight by using only two VMs at a time.

## Lab Objectives
The main goals of this lab were to:
1. Build an isolated virtual network for safe traffic generation
2. Configure Snort on Ubuntu to monitor live traffic
3. Simulate realistic SOC scenarios
4. Monitor and analyze live data traffic
5. Understand the difference between normal and suspicious network activity

## Environment
### Host System
- 2022 Macbook Pro
- 8 GB RAM
- VMware Fusion

### Virtual Machines
- Ubuntu Server (CLI) VM (*Snort sensor*)
- Windows VM (*traffic generator*)

### Network Design
Both virtual machines were configured to use a private host-only network in VMware Fusion so the generated traffic stayed inside the lab and did not touch the host network.

### Key Lab IP Information
- Ubuntu VM (running Snort IDS): `172.16.14.129`
- Lab network (subnet): `172.16.14.0/24`
- Snort monitoring interface: `enp2s0`

## Tools Used
- Snort
- Nmap on Windows VM
- OpenSSH Client
- VMware Fusion
- Ubuntu Server (CLI)
- Windows Command Prompt

## Skills Demonstrated
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

### Screenshots
01-vm-network-settings.png

VMware Fusion settings page showing Private to my Mac selected for the Ubuntu VM
Repeat for Windows if possible, or show one screenshot and mention both VMs were configured the same way


## Step 2: Install Snort on Ubuntu

The second step in the lab was installing Snort on the Ubuntu VM. This ensured the IDS was available before configuring networking and generating traffic.

### Screenshot needed

03-snort-install-or-version.png

terminal showing snort --version

### Installation
I ran `sudo apt-get update` to download the latest package information for Ubuntu. I then ran `sudo apt-get install snort -y` to install Snort. Lastly, I typed `snort --version` to verify the installation.


## Step 3: Verify Network Interface and IP

The next step was to identify the active network interface, assigned IP address, and subnet information. This information was required to; specify the correct interface for Snort, correctly configure `HOME_NET` and target the Ubuntu VM from the Windows VM. 

### Screenshots needed

02-ubuntu-ip-interface.png

Ubuntu terminal showing ip a with interface and IP

### Verification

I began by running `ip a` to display the IP addresses and status of all the network interfaces on the Ubuntu server. From there, I identified the interface name `enp2s0`, Ubuntu IP address `172.16.14.129` and subnet `172.16.14.0/24`.


## Step 4: Configure HOME_NET in Snort

The fourth step in this lab was configuring `HOME_NET`. Snort requires a defined internal network range to properly distinguish between trusted internal traffic and potentially suspicious activity. In this lab, both the Ubuntu VM and Windows VM were assigned IP addresses within the `172.16.14.0/24` subnet. 

Leaving `HOME_NET` as `any` makes alerts less meaningful. To ensure accurate monitoring and detection, the `HOME_NET` variable in the Snort configuration was updated to match the `172.16.14.0/24` subnet.

### Screenshot

03-snort-home-net-config.png

show the line: ipvar HOME_NET 172.16.14.0/24

### Configuration

Firstly, I ran `sudo nano /etc/snort/snort.conf` to open the Snort configuration file so i'd be able to edit it. From there, I located `ipvar HOME_NET any` and changed it to `ipvar HOME_NET 172.16.14.0/24`. 


## Step 5: Validate the Snort Configuration

Before running Snort, I had to validate the configuration. This prevents wasting time troubleshooting live traffic if anything goes wrong, and makes sure everything is configured how its supposed to. 

### Screenshot needed

04-snort-config-validation.png

terminal showing the validation command and the success message at the bottom

### Validation

I ran `sudo snort -T -c /etc/snort/snort.conf -i enp2s0` in Ubuntu. This validates that the Snort configuration, rules, and network interface are correctly set up and ready for execution. The command prints a long validation report, and at the end of it writes "Snort successfully validated the configuration".


## Step 6: Create Custom Snort Detection Rules

Before running Snort, I also had to create custom detection rules. For this step, I ran `sudo nano /etc/snort/rules/local.rules` to add custom detection rules to the `local.rules` file. Adding custom detection rules defines what types of traffic Snort should identify and alert on within the lab environment. These custom rules were used to tell Snort what suspicious or notable activity to look for, rather than relying only on default behavior.

### Screenshot needed

10-snort-local-rules.png

local.rules file showing custom rules

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

Next, I installed Nmap on the Windows VM to prepare for generating network traffic that would later be analyzed by Snort. This step ensured that a reliable tool was available to generate controlled network traffic. Installing and validating Nmap allowed for the simulation of real-world activity that could be detected and analyzed by Snort.

### Screenshot needed

07-nmap-download-install.png

Nmap website and/or installation progress window
07-nmap-version-check.png

Command prompt showing nmap --version output

### Installation

The official Nmap website was accessed from the Windows VM to download the latest version of Nmap. The installer was executed, and Nmap was installed using the default configuration settings. 

After installation, the `nmap --version` command was ran in Command Prompt to verify that Nmap was successfully installed without any errors and was available on the system.

## Step 8: Verify Connectivity Between Virtual Machines

Before generating more advanced traffic, connectivity between the Windows VM and the Ubuntu VM was verified using a basic ICMP test. Doing so ensured that the lab environment was properly set up before generating additional traffic. Verifying connectivity is a critical prerequisite, as all subsequent activity (scans, authentication attempts, and monitoring in Snort) depends on successful communication between systems.

### Screenshot needed

06-windows-ping-command.png

Windows command prompt showing successful ping replies from 172.16.14.129

### Verification

From the Windows VM, I pinged the Ubuntu VM's IP address by running `ping 172.16.14.129` in the Command Prompt. The Windows VM sent ICMP echo requests to the Ubuntu VM IP address (172.16.14.129) to confirm that both systems could successfully communicate across the configured host-only network. 

The network configuration between both VMs was functioning correctly. The connection was successfully established, as indicated by reply messages from the Ubuntu VM (e.g., Reply from 172.16.14.129). No packet loss occurred (0% loss) confirming stable communication. The consistent response times indicated reliable connectivity.

## Step 9: Run Snort in Live Console Mode

To start the lab, Snort was ran in console alert mode. This mode was used because the view is cleaner and more practical than dumping every raw packet.

### Screenshot needed

05-snort-running-console.png

terminal showing Snort running and waiting for traffic

### Execution
To begin, I ran `sudo snort -i enp2s0 -A console -q -c /etc/snort/snort.conf`. This command starts Snort in console mode, instructing it to monitor traffic on the specified network interface, apply the configured rules and settings from the configuration file, and display any detected alerts in real time.


# Lab Scenarios


## Scenario 1: ICMP Echo Baseline

Establish a baseline of benign network traffic before introducing suspicious activity.

### Actitivy Execution

To generate baseline traffic, I initiated ICMP requests from the Windows VM to the Ubuntu VM by using the `ping` command targeting the Ubuntu IP address `172.16.14.129`. This was done to simulate normal, benign network communication between hosts within the lab environment. While the ping command was running, Snort was actively monitoring traffic on the Ubuntu VM. 

### Observed Behavior

As a result, multiple alerts were generated in real time, indicating that ICMP traffic was being detected and matched against the custom rule defined in the `local.rules` file.

The Snort console output displayed repeated alerts labeled **"ICMP Test Detected"**, along with timestamps, source IP (`172.16.14.1`), and destination IP (`172.16.14.129`). The repeated alerts confirmed that Snort was successfully capturing and analyzing live network traffic based on the configured detection rules.


### Screenshot description

12-icmp-snort-alerts.png

Snort console output
multiple “ICMP Test Detected” alerts
showing source → destination IP

### Analysis

This scenario established a benign baseline by generating normal ICMP echo traffic from the Windows VM to the Ubuntu Snort sensor. The traffic pattern was expected and consistent, making it useful as a reference point for later suspicious scenarios.

## Scenario 2: TCP SYN Scan (Reconnaissance)

Simulate early-stage attacker reconnaissance by identifying open or responsive TCP ports without fully establishing connections.

### Activity Execution

To simulate reconnaissance activity, I initiated a SYN scan from the Windows VM targeting the Ubuntu Snort sensor using the following command: `nmap -sS 172.16.14.129`. This command performs a TCP SYN scan, which sends connection requests to multiple ports on the target system without completing the full TCP handshake. The purpose of this scan was to identify open or responsive ports on the Ubuntu VM, simulating early-stage attacker behavior.

### Observed Behavior

After executing the scan, Snort generated multiple real-time alerts indicating that SYN packets were being sent from the Windows VM to the Ubuntu VM across various destination ports.

Each alert included:

- Source IP: `172.16.14.128` (Windows VM)
- Destination IP: `172.16.14.129` (Ubuntu VM)
- Different destination ports (e.g., 80, 443, 22, etc.)
- Alert message: **“SYN Port Scan Detected”**

The high volume of alerts in a short time frame confirmed that multiple connection attempts were being made rapidly across different ports, which is consistent with port scanning behavior.

### Screenshot(s) Needed

13-nmap-syn-scan-command.png

Windows Command Prompt showing:
nmap -sS 172.16.14.129

14-snort-syn-alerts.png
Snort console output displaying multiple
“SYN Port Scan Detected” alerts

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

Each alert included:

- Source IP: `172.16.14.128` (Windows VM)
- Destination IP: `172.16.14.129` (Ubuntu VM)
- Destination Port: `22` (SSH)

Additionally, the Snort output also showed **"SYN Port Scan Detected"** alerts, which occurred because each SSH attempt begins with a TCP connection request, triggering both the SYN scan rule and the brute force detection rule.

### Screenshot(s) Needed

15-ssh-bruteforce-windows.png

Windows Command Prompt showing:
ssh analyst@172.16.14.129
multiple “Permission denied” messages

16-snort-bruteforce-alerts.png

Snort console output showing:
“SSH Brute Force Attempt” alerts
source → destination IP
port 22 activity

### Analysis

This scenario simulated a brute force attack by generating repeated SSH login attempts against the Ubuntu system. The behavior was characterized by multiple failed authentication attempts within a short period, which is a common method attackers use to gain unauthorized access.

The Snort brute force detection rule successfully identified this pattern by tracking repeated connection attempts from the same source IP to port 22. Because the threshold condition was met (multiple attempts within a defined time window), Snort generated alerts indicating potential brute force activity.

The presence of both brute force and SYN alerts demonstrates how different detection rules can overlap, providing additional context during analysis.

# Snort Operational Modes Analysis

