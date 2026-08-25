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
