# 📡 Private 5G Network Setup Testbed

A complete end-to-end guide to deploy a standalone private 5G network using open-source software and SDR hardware.

## 🚀 Overview

This repository documents the complete deployment of a Private 5G Standalone (SA) network, integrating an open-source 5G core, radio access network, and SDR hardware. The setup is suitable for academic research, lab experimentation, and private network prototyping.

### Technology Stack

- 5G Core: Open5GS
- RAN (gNB): srsRAN Project
- SDR Hardware: USRP B210
- Programmable SIM: Open-Cells UICC Tools
- Operating System: Ubuntu 22.04 LTS

### 🏗️ System Architecture

The deployment follows a 5G Standalone (SA) architecture:

- srsRAN gNB connects to Open5GS AMF over NGAP (N2)
- User plane traffic flows via GTP-U (N3) to UPF
- Session management via PFCP between SMF and UPF
- Internet breakout using Linux NAT
- UE authentication using programmable SIM credentials

## 📋 Prerequisites

### Hardware Requirements

- Intel i7-class CPU (or equivalent)
- Ubuntu 22.04 LTS (bare metal recommended)
- USRP B210 (USB 3.0)
- Programmable SIM + USB reader

### Software Components

- Open5GS
- srsRAN Project
- MongoDB
- Node.js (WebUI)
- UHD (USRP drivers)

## 🛠️ Installation

### 1️⃣ MongoDB Setup

MongoDB is used as the subscriber database for Open5GS.

```bash
sudo apt update
sudo apt install -y gnupg

curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor
echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list

sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
2️⃣ Open5GS Installation
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install -y open5gs

To verify services:

sudo systemctl status open5gs-*
sudo systemctl status open5gs-* | grep -c "active"

Expected: 17 active services

3️⃣ Open5GS WebUI Setup
sudo apt install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings

curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

NODE_MAJOR=20

echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

sudo apt update
sudo apt install -y nodejs

curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -

WebUI Access

URL: http://localhost:9999
Username: admin
Password: <YOUR_WEBUI_PASSWORD>
4️⃣ Network Configuration
sudo sysctl -w net.ipv4.ip_forward=1

sudo iptables -t nat -A POSTROUTING -o $(ip route show default | awk '/default/ {print $5}') -j MASQUERADE

sudo iptables -I FORWARD 1 -j ACCEPT

sudo systemctl stop ufw

sudo apt install -y iptables-persistent
sudo dpkg-reconfigure iptables-persistent
5️⃣ srsRAN Project Setup
sudo apt update

sudo apt install -y cmake make gcc g++ pkg-config \
libfftw3-dev libmbedtls-dev libsctp-dev \
libyaml-cpp-dev libgtest-dev libuhd-dev uhd-host

git clone https://github.com/srsRAN/srsRAN_Project.git
cd srsRAN_Project
mkdir build && cd build
cmake ../
make -j $(nproc)
make test -j $(nproc)
sudo make install
6️⃣ USRP B210 Setup
sudo apt install -y libuhd-dev uhd-host

sudo python3 /usr/lib/uhd/utils/uhd_images_downloader.py

uhd_find_devices
uhd_usrp_probe
⚙️ Configuration
gNB Configuration
cu_cp:
  amf:
    addr: 127.0.0.5
    port: 38412
    bind_addr: 127.0.0.5
    supported_tracking_areas:
      - tac: 7
        plmn_list:
          - plmn: "99970"

cell_cfg:
  dl_arfcn: 632628
  band: 78
  channel_bandwidth_MHz: 20
  common_scs: 30
  plmn: "99970"
  tac: 7
  pci: 1
Programmable SIM Setup
tar -xvzf uicc-v3.3.tgz
cd uicc-v3.3
make

Program the SIM using your own credentials:

sudo ./program_uicc \
--adm <YOUR_ADMIN_KEY> \
--imsi <YOUR_IMSI> \
--isdn <YOUR_ISDN> \
--acc <YOUR_ACCESS_CONTROL> \
--key <YOUR_KEY> \
--opc <YOUR_OPC> \
--spn "CSE" \
--authenticate \
--noreadafter
🏃 Running the Network
1️⃣ Start Core Network
sudo systemctl restart open5gs-*
sudo systemctl status open5gs-* | grep -c "active"
2️⃣ Add Subscriber (WebUI)
IMSI: <YOUR_IMSI>
Key & OPc: Same as SIM
DNN: IPv4 enabled (dynamic IP)
3️⃣ Start gNB
cd srsRAN_Project/build/apps/gnb/
sudo ./gnb -c gnb_rf_b200_tdd_n78_20mhz.yml
4️⃣ Connecting 5G Phone
Turn on the airplane mode after inserting the SIM.
Turn off the airplane mode once gNB is running.
Set Network mode to 5G / 5G preferred.
Add Access Point Name:
Name: internet
APN: internet
APN type: default
APN protocol and APN roaming protocol: IPv4
Bearer: NR
📊 Testing & Validation
Connectivity
ping 8.8.8.8
Throughput
iperf3 -s
iperf3 -c <server_ip>
Logs
sudo tail -f /var/log/open5gs/amf.log
🔧 Troubleshooting
MongoDB: sudo journalctl -u mongod
Open5GS: sudo journalctl -u open5gs-amf
USRP: Ensure USB 3.0 (lsusb | grep Ettus)
gNB: Verify PLMN, TAC, and AMF IP consistency
🛡️ Security Considerations
Change default WebUI credentials
Restrict WebUI to trusted networks
Backup MongoDB and config files
Harden firewall rules for production use
Do not commit SIM credentials, authentication keys, passwords, or other secrets to GitHub
📝 Regulatory Compliance

⚠️ Ensure compliance with local spectrum regulations:

Verify n78 band authorization
Follow power and antenna limits
Obtain required transmission licenses
🔗 References
Open5GS Documentation
srsRAN Project Documentation
UHD / USRP Documentation
Open-Cells UICC Tools
📄 License

This project is intended for educational and research purposes only. Users are responsible for ensuring regulatory compliance.


**That's the entire README.** Your GitHub page will automatically turn the Markdown into the formatted version.

I have **not included** the GitHub-generated sections like `Skip to content`, `Stars`, `Forks`, `Contrib
