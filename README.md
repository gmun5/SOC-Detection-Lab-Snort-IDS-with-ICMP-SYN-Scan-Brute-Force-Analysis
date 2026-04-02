# SOC Detection Lab: Snort IDS with ICMP SYN Scan Brute Force Analysis

## Overview


This Snort Instrusion Decection Lab was built with two isolated virtual machines: an Ubuntu VM running Snort and a Windows VM used to generate network traffic. The lab was designed to simulate three realistic SOC-relevant scenarios on limited hardware: a benign ICMP baseline, a TCP SYN reconnaissance scan, and a controlled SSH brute-force simulation.

The purpose of this project was not just to "run Snort," but to practice the workflow a SOC analyst would actually follow:
- Establish a normal baseline
- Generate suspicious activity
- Observe alerts and packet behavior
- Identify source/destination and protocol details
- Document findings clearly

Because the host machine was a 2022 MacBook Pro with 8 GB of RAM, the lab was intentionally kept lightweight by using only two VMs at a time.

## Lab Objectives
The main goals of this lab were to:
1. Build an isolated virtual network for safe traffic generation
2. Configure Snort on Ubuntu to monitor live traffic
3. Understand the difference between normal and suspicious network activity
4. Simulate realistic SOC scenarios without resource-heavy tooling
5. Practice documenting findings like an analyst, not just a lab user

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
