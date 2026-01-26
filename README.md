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
