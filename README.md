# Azure Hub-and-Spoke Architecture with Azure Firewall

## Overview
This project demonstrates the implementation of a Hub-and-Spoke network architecture in Microsoft Azure, with centralized traffic inspection using Azure Firewall. The lab focuses on secure internet access, controlled inbound connectivity, and traffic routing using User Defined Routes (UDRs).

## Architecture Summary
- Hub VNet hosts Azure Firewall
- Spoke VNet hosts a Windows VM
- Internet-bound traffic is forced through Azure Firewall
- DNAT is used to allow RDP access
- Application rules restrict outbound access (L7 traffic)
- Network rules are present but not used for outbound web traffic

## Resources Created

### Resource Group
- `rg-hubspoke-demo`

### Hub Network
- VNet: `VNet-FW` (10.1.0.0/16)
- Subnet: `AzureFirewallSubnet` (10.1.0.0/26)
- Azure Firewall: `FW-demo`
- Firewall Policy: `FW-policy-01`
- Public IP: `FW-PIP`

### Spoke Network
- VNet: `vnet-prod` (10.2.0.0/16)
- Subnet: `SN-vm-prod` (10.2.1.0/24)
- VM: `vm-prod-01` (Windows Server 2019)

## Implementation Steps

### 1. Virtual Network Setup
- Created hub and spoke VNets with non-overlapping address spaces.
- Deployed Azure Firewall in the hub VNet.

### 2. VM Deployment
- Created a Windows Server 2019 VM in the spoke VNet.
- Verified RDP connectivity using the VM’s public IP.

### 3. VNet Peering
- Configured VNet peering between `vnet-prod` and `VNet-FW` to allow inter-VNet communication.

### 4. Traffic Routing with UDR
- Observed unrestricted internet access from the VM.
- Created a route table `Route-FW`.
- Added a route:
  - Destination: `0.0.0.0/0`
  - Next hop type: Virtual appliance
  - Next hop address: Firewall private IP
- Associated the route table with `SN-vm-prod`.
- Verified effective routes showing traffic forced through the firewall.
- Screenshot reference: screenshots/1-udr-0.0.0.0-via-firewall.png

### 5. DNAT for RDP Access
- RDP access failed after UDR enforcement (expected behavior).
- Created a DNAT rule `Allow-RDP-prod` in `FW-policy-01`:
  - TCP port 3389
  - Firewall public IP → VM private IP
- Successfully restored RDP access via firewall.
- Screenshot reference: screenshots/2-firewall-policy-rules.png

### 6. Application Rules for Internet Access
- Created an application rule `allow-prod-browse`:
  - Source: VM private IP
  - Protocol: HTTP/HTTPS (ports 80, 443)
  - Allowed FQDN: `www.google.com`
- Verified via screenshots:
  - Google accessible: screenshots/3-outbound-allowed-google.png
  - Other sites blocked (e.g., Facebook): screenshots/4-outbound-blocked-facebook.png

## Key Learnings
- Importance of UDRs in enforcing centralized traffic inspection
- How DNAT enables secure inbound access through Azure Firewall
- Fine-grained outbound control using application rules
- Real-world troubleshooting of connectivity issues

## Cleanup
All resources were deleted after testing to avoid unnecessary Azure costs.
