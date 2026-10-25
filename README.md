5G-Enabled Drone for Temporary Emergency Network Connectivity 🚁📡

📌 Project Overview

During natural disasters (earthquakes, floods, cyclones), ground-based communication infrastructure is frequently destroyed, leaving affected areas isolated and hindering emergency response efforts.

This project aims to design and develop a UAV-based portable 5G network ("5G-in-a-Box") to provide rapid, temporary, and reliable high-speed wireless connectivity in disaster-stricken zones. By combining a lightweight 5G Standalone (SA) core and a Software-Defined Radio (SDR) on a drone, we can establish an aerial base station that provides immediate cellular coverage for search-and-rescue teams, medical triage, and affected civilians.

Institution: St. John College of Engineering and Management, Palghar

Department: Artificial Intelligence and Machine Learning

Project Guide: Dr. Juily Tarade

🏗️ System Architecture

The system consists of three main segments:

The Aerial Platform: A quadcopter UAV with a payload capacity of ~2kg.

The 5G-in-a-Box Payload: A Single Board Computer (SBC) running a virtualized 5G Core and RAN, connected to an SDR.

The User Equipment (UE): Ground devices (smartphones, coordination laptops, IoT sensors) connecting to the aerial network.

Currently, the system is prototyped and virtualized using Open5GS (5GC) and UERANSIM (RAN/UE simulator).

⚙️ Hardware & Software Requirements

Hardware (Prototype)

UAV: High-lift quadcopter capable of carrying a 2kg payload

Compute: Mini-PC or SBC (e.g., Raspberry Pi 4/5, Intel NUC)

Radio: Software-Defined Radio (SDR) for physical RF transmission (Future Phase)

Power: Unified high-capacity LiPo battery system with power distribution

Software

OS: Kali Linux / Ubuntu 22.04 LTS

5G Core: Open5GS

RAN/UE Simulator: UERANSIM

🚀 Installation & Network Routing Setup

Getting the simulated UE (uesimtun0) to route out to the external internet through the 5G UPF (ogstun) requires specific kernel and routing configurations.

1. Enable IP Forwarding & NAT

Ensure the Linux kernel is allowed to route packets, and set up a NAT rule so the 5G subnet (10.45.0.0/16) can reach the internet via the host interface (eth0).

sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -s 10.45.0.0/16 -o eth0 -j MASQUERADE


2. Disable Reverse Path Filtering

To prevent the kernel from dropping packets due to asymmetric routing inside the tunnels, disable strict rp_filter:

for f in /proc/sys/net/ipv4/conf/*/rp_filter; do echo 0 | sudo tee "$f"; done
sudo sysctl -p


3. Override UERANSIM Policy Routing (Fixing 100% Packet Loss)

UERANSIM creates a strict routing table (rt_uesimtun0) bound to the user ID. To allow internet access (e.g., pinging 8.8.8.8), force the packet lookup to the main table:

# Add main route to eth0
sudo ip route add default dev eth0 table main
# Force UE traffic to bypass local isolation
sudo ip rule add from 10.45.0.3 dev uesimtun0 lookup main prio 50


4. Running the Network

Start Open5GS services, then initialize the gNodeB and UE:

# Start gNB
sudo ./build/nr-gnb -c config/open5gs-gnb.yaml
# Start UE
sudo ./build/nr-ue -c config/open5gs-ue.yaml


👥 Contributors

Parth Ganpat Dhumak - UAV platform design & payload stability

Mayur Leelesh Haldankar - 5G Core configuration & network routing

Maan Surti - RAN integration & system architecture

📄 License

This project configuration is open-source. UERANSIM and Open5GS retain their respective MIT and GPLv3 licenses.
