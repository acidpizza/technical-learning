---
title: "Storage"
weight: 2
---

## Protocols for IO requests

Small Computer System Interface (**SCSI**) Protocol:
- Communication between server to internal hard disk.

Serially Attached SCSI (**SAS**) Protocol:
- Parallel SCSI are poor in accuracy and have issues at high speeds
- SAS performs better than Serial Advanced Technology Attachment (**SATA**) for server workloads
- SAS is full duplex

**Fibre Channel** Protocol:
- Enable faster rates
- Does not need to run on optical fibre medium
- Not aware of data being transferred inside the fibre channel frame. Can manage both networking and IO communications over the same adapter.
- Requires Host Bus Adapter (HBA) with Fibre Channel ports.
- SCSI cannot be used directly as there is a limit in number of supported hosts.

Internet Small Computer System Interface (**iSCSI**) Protocol:
- IP-based standard for interconnecting hosts and storage arrays.
- SCSI-over-IP
- Simplest and cheap solution, but not good performance.


## Storage Connectivity

Directly Attached Storage (**DAS**):
- Appears to server as part of itself
- Disadvantages:
  - Server can only access local data
  - If server fails, data access is lost
  - Physical limit to scaling storage

Network Attached Storage (**NAS**):
- Appears to server as a shared folder
- File level storage: Can share with many devices
- **NFS** and **CIFS** are the main IO protocols
- Has an IP address and its own computing power
- Simple, cheap, slow, high latency

Storage Area Network (**SAN**)
- Two or more devices communicating via a serial SCSI protocol such as Fibre Channel or iSCSI
- Appears to server as a shared device
- Block level storage: Can only share with 1 device
- **SCSI** is main IO protocol
- Good performance


## Cabling Hardware

- Fibre chanel
- Ethernet