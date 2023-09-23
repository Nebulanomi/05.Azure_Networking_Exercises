# Azure Networking labs

## Introduction

This set of Azure Networking labs are simplified to demonstrate a single concept in each lab, using Azure portal or Azure CLI. Each lab has a lab diagram that provides information on the lab setup. Most labs build on each other so prior setup is expected. Please use the lab diagram as guidance if you are doing a specific lab out of order.

## Table of Lab Content

#### [01. Virtual Network](https://github.com/binals/azurenetworking/blob/master/Lab%2001%20Virtual%20Network.pdf)

Quick resume:

1. Created a vNET (10.1.0.0/16).
2. Created 2 subnets (10.1.1.0/24 & 10.1.2.0/24).
3. Added a VM on each subnet ("Mgmt" & "Web").
4. Installed "Apache" on the "Web" VM in subnet 2.

#### [02. Network Security Groups](https://github.com/binals/azurenetworking/blob/master/Lab%2002%20Network%20Security%20Groups.pdf)

Quick resume:

1. Created a NSG (Network Security Group) and associated to each subnet.
2. Removed the NSGs created with the VM NICs.
3. Created an ASG (Application Security Group) for each VM ("Mgmt" & "Web").

    Note: The ASGs have the same name as the VMs.

4. Added the following inbound security rules to the NSG:

    Port: "22", "3389"; Protocol: "TCP"; Source: "Any"; Destination: "Mgmt"; Action: "Allow".
    Port: "80", "443"; Protocol: "TCP"; Source: "Any"; Destination: "Web"; Action: "Allow".

    Note: Port "3389" was added so that I could connect to Windows VMs in case I added them in the future.

5. Tested connectivity from my local computer to the VMs public IPs.
6. Created a new subnet (10.1.3.0/24) and associated it to the NSG.
7. Created a VM on the new subnet ("App") and installed "Apache".
8. Removed the NSGs created with the VM NIC.
9. Added the follwing inbound security rules to the NSG:

    Port: "Any"; Protocol: "ICMP"; Source: "Any"; Destination: "Mgmt", "Web"; Action: "Allow".
    Port: "Any"; Protocol: "ICMP"; Source: "Web"; Destination: "App"; Action: "Allow".
    Port: "Any"; Protocol: "ICMP"; Source: "Mgmt"; Destination: "App", "Web"; Action: "Allow".
    Port: "22", "3389"; Protocol: "TCP"; Source: "Any"; Destination: "Mgmt"; Action: "Allow".
    Port: "80"; Protocol: "TCP"; Source: "Web"; Destination: "App"; Action: "Allow".
    Port: "Any"; Protocol: "Any"; Source: "Any"; Destination: "Any"; Action: "Deny".

#### [03. Azure CLI](https://github.com/binals/azurenetworking/blob/master/Lab%2003%20CLI.pdf)
#### [04. Virtual Network Peering](https://github.com/binals/azurenetworking/blob/master/Lab%2004%20Virtual%20Network%20Peering.pdf)
#### [05. Virtual Network Peering - Transitive behavior](https://github.com/binals/azurenetworking/blob/master/Lab%2005%20Virtual%20Network%20Peering%20-%20Transitive%20behavior.pdf)
#### [06. NVA CSR1000v](https://github.com/binals/azurenetworking/blob/master/Lab%2006%20NVA%20CSR1000v.pdf)
#### [07. Routing Tables](https://github.com/binals/azurenetworking/blob/master/Lab%2007%20Routing%20Tables.pdf)
#### [08. Site-to-site VPN](https://github.com/binals/azurenetworking/blob/master/Lab%2008%20Site-to-site%20VPN.pdf)
#### [09. Virtual WAN](https://github.com/binals/azurenetworking/blob/master/Lab%2009%20Virtual%20WAN.pdf)
#### [10. Standard Load Balancer](https://github.com/binals/azurenetworking/blob/master/Lab%2010%20Standard%20Load%20Balancer.pdf)
#### [11. Network Watcher NSG Flow Logs](https://github.com/binals/azurenetworking/blob/master/Lab%2011%20Network%20Watcher%20NSG%20Flow%20Logs.pdf)
#### [12. Firewall](https://github.com/binals/azurenetworking/blob/master/Lab%2012%20Firewall.pdf)
#### [13. Firewall-Inbound NAT](https://github.com/binals/azurenetworking/blob/master/Lab%2013%20Firewall%20-%20Inbound%20NAT.pdf)
#### [14. Firewall - Spoke to spoke communication](https://github.com/binals/azurenetworking/blob/master/Lab%2014%20Firewall%20-%20Spoke%20to%20spoke%20communication.pdf)
