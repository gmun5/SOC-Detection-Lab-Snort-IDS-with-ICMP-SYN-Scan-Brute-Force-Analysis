# SOC Detection Lab: Snort IDS with ICMP SYN Scan Brute Force Analysis

## Overview


This Snort Instrusion Decection Lab was built with two isolated virtual machines: an Ubuntu VM running Snort and a Windows VM used to generate network traffic. The lab was designed to simulate three realistic SOC-relevant scenarios on limited hardware: a benign ICMP baseline, a TCP SYN reconnaissance scan, and a controlled SSH brute-force simulation.

The purpose of this project was not just to "run Snort," but to practice the workflow a SOC analyst would actually follow:
- Establish a normal baseline
- Generate suspicious activity
- Observe alerts and packet behavior
- Identify source/destination and protocol details

Because the host machine was a 2022 MacBook Pro with 8 GB of RAM, the lab was intentionally kept lightweight by using only two VMs at a time.

## Lab Objectives
The main goals of this lab were to:
1. Build an isolated virtual network for safe traffic generation
2. Configure Snort on Ubuntu to monitor live traffic
3. Understand the difference between normal and suspicious network activity
4. Simulate realistic SOC scenarios without resource-heavy tooling

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

## Step 1: Configure the Virtual Network

The first requirement for this lab was isolation. In VMware Fusion, both the Ubuntu VM and the Windows VM were configured to use Private to my Mac. This creates a host-only network so the traffic remains contained inside the lab. If the VMs were left on NAT or bridged networking, scan traffic could mix with the real host network. That would make the lab less controlled and less professional.

### Screenshots
01-vm-network-settings.png

VMware Fusion settings page showing Private to my Mac selected for the Ubuntu VM
Repeat for Windows if possible, or show one screenshot and mention both VMs were configured the same way


## Step 2: Install and Verify Snort on Ubuntu

The second step in the lab was installing Snort on the Ubuntu VM. This ensured the IDS was available before configuring networking and generating traffic.

### Screenshot needed

03-snort-install-or-version.png

terminal showing snort --version

### Installation
I executed the `sudo apt-get update` command to download the latest package information for Ubuntu. Then, I executed the `sudo apt-get install snort -y` command to install Snort. Lastly, I executed the `snort --version` command to verify installation.

## Step 3: Configure HOME_NET

The third step in this lab was configuring HOME_NET. Snort requires a defined internal network range to properly distinguish between trusted internal traffic and potentially suspicious activity. In this lab, both the Ubuntu VM and Windows VM were assigned IP addresses within the `172.16.14.0/24` subnet. Leaving `HOME_NET` as `any` makes alerts less meaningful. To ensure accurate monitoring and detection, the `HOME_NET` variable in the Snort configuration was updated to match the `172.16.14.0/24` subnet.

### Screenshot

03-snort-home-net-config.png

show the line: ipvar HOME_NET 172.16.14.0/24

### Installation

Firstly, I executed the `sudo nano /etc/snort/snort.conf` command to open the Snort configuration file to edit it. From there, I changed `ipvar HOME_NET any` to `ipvar HOME_NET 172.16.14.0/24`. 
