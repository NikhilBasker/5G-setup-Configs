# 5G Standalone Network Setup using free5GC and UERANSIM (VM-based)

This repository contains configuration files and setup details for deploying a **5G Standalone (SA) core network** using **free5GC** and **UERANSIM**, fully virtualized using **Ubuntu Virtual Machines on VirtualBox**.

The setup supports:
- UE registration and authentication
- PDU session establishment
- End-to-end internet connectivity
- Multiple UEs
- Network slicing and slice differentiation

This repository focuses on **configuration-level reproducibility**, making it easy to recreate the same 5G environment on any compatible system.

---

## 📐 Architecture Overview

The simulation uses **three virtual machines**:

| VM | Role | Description |
|----|------|------------|
| VM1 | free5GC Control Plane | AMF, SMF, AUSF, NRF, NSSF |
| VM2 | free5GC User Plane | UPF and data network |
| VM3 | UERANSIM | gNB |
| VM4-6 | UERANSIM | UEs |

All VMs run **Ubuntu 20.04** and communicate using **host-only networking** for N2/N3 interfaces, with **NAT** used for external internet access.

---

## 📁 Repository Structure

```text
.
├── amfcfg.yaml          # AMF configuration
├── ausfcfg.yaml         # AUSF configuration
├── nrfcfg.yaml          # NRF configuration
├── nssfcfg.yaml         # NSSF configuration (network slicing)
├── smfcfg.yaml          # SMF configuration (slices, DNNs, UPF mapping)
├── upfcfg.yaml          # UPF configuration
├── free5gc-gnb.yaml     # UERANSIM gNB configuration
├── free5gc-ue1.yaml     # UE 1 configuration
├── free5gc-ue2.yaml     # UE 2 configuration
├── free5gc-ue3.yaml     # UE 3 configuration
├── free5gc-ue4.yaml     # UE 4 configuration
└── README.md
```


---

## ⚙️ free5GC Setup (Control Plane & User Plane)

The 5G Core is implemented using **:contentReference[oaicite:0]{index=0}**, split into Control Plane and User Plane functions across separate virtual machines.

---

### 1️⃣ Prerequisites (VM1 & VM2)

All free5GC VMs require the following dependencies:

```bash
sudo apt update && sudo apt upgrade
sudo apt install -y git gcc g++ cmake autoconf libtool pkg-config \
libmnl-dev libyaml-dev curl mongodb

```
Install Go (required for free5GC)
```bash
wget https://dl.google.com/go/go1.17.8.linux-amd64.tar.gz
sudo tar -C /usr/local -zxvf go1.17.8.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc

```
2️⃣ Build free5GC
```bash
git clone --recursive https://github.com/free5gc/free5gc.git
cd free5gc
make
```

### 3️⃣ Control Plane Configuration (VM1)

The free5GC Control Plane behavior is defined using the following configuration files:

- **amfcfg.yaml**
  - PLMN configuration (MCC, MNC)
  - N2 interface IP address
  - Supported S-NSSAI (network slices)
  - Supported DNN list

- **ausfcfg.yaml**
  - UE authentication parameters
  - PLMN configuration

- **nrfcfg.yaml**
  - Network Function discovery
  - Registration of core network functions

- **nssfcfg.yaml**
  - Network slicing definitions
  - Slice parameters (SST and SD)

- **smfcfg.yaml**
  - Slice-to-DNN mapping
  - UPF selection logic
  - User-plane topology information

All IP addresses and PLMN values in these configuration files must match the statically assigned values of the corresponding virtual machines to ensure proper communication between core network functions.


### 4️⃣ User Plane Configuration (VM2)

The User Plane Function (UPF) is configured using the following file:

- **upfcfg.yaml**
  - PFCP interface IP configuration
  - UE tunnel interface definitions
  - Data Network (DN) routing information

To allow UE traffic to reach external networks, IP forwarding must be enabled on the UPF VM.

Enable IP forwarding:
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```
### 5️⃣ Starting free5GC

Start the free5GC components in the following order.

#### VM2 – User Plane (UPF)

```bash
cd free5gc
sudo ./bin/upf

```
### VM1 – Control Plane
```bash
cd free5gc
./run.sh
```
## 📡 UERANSIM Setup (gNB & UEs)

The Radio Access Network (RAN) and User Equipment (UE) are simulated using UERANSIM.

---

### 6️⃣ Prerequisites (VM3–VM6)

Install the required dependencies on all UERANSIM VMs:

```bash
sudo apt update
sudo apt install -y make git g++ libsctp-dev lksctp-tools iproute2
```
### 7️⃣ Build UERANSIM

Clone and build UERANSIM:
```bash
git clone https://github.com/aligungr/UERANSIM.git
cd UERANSIM
make
```
### 8️⃣ gNB Configuration (VM3)

The gNB is configured using the following file:

- **free5gc-gnb.yaml**
  - PLMN configuration (MCC, MNC)
  - gNB N2 interface IP address
  - gNB N3 interface IP address
  - AMF IP address
  - Supported S-NSSAI (network slices)

This configuration enables proper NGAP signaling and GTP tunnel establishment between the gNB and the free5GC core network.


### 9️⃣ UE Configuration (VM4–VM6)

Multiple UE configuration files are provided:

- **free5gc-ue1.yaml**
- **free5gc-ue2.yaml**
- **free5gc-ue3.yaml**
- **free5gc-ue4.yaml**

Each UE configuration file defines:
- SUPI (IMSI)
- Authentication credentials (K, OP/OPc)
- Default and configured NSSAI
- PDU session parameters

Each UE must also be registered in the free5GC WebUI using the same IMSI and authentication credentials to allow successful registration and PDU session establishment.


## ▶️ Running the Network

---

### 🔟 Start gNB (VM3)

Start the gNB using the following command:

```bash
cd UERANSIM/build
./nr-gnb -c ../config/free5gc-gnb.yaml
```
### 1️⃣1️⃣ Start UEs (VM4–VM6)

Start each UE in a separate terminal window:

```bash
sudo ./nr-ue -c ../config/free5gc-ue1.yaml
sudo ./nr-ue -c ../config/free5gc-ue2.yaml
sudo ./nr-ue -c ../config/free5gc-ue3.yaml
sudo ./nr-ue -c ../config/free5gc-ue4.yaml
```
