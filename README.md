# Azure Networking labs

## Introduction

This set of Azure Networking labs are simplified to demonstrate a single concept in each lab, using Azure portal or Azure CLI.

Each lab has a diagram that provides information on the lab setup.

Most labs build on each other so prior setup is expected.

## Table of Lab Content

#### [01. Virtual Network:](https://github.com/binals/azurenetworking/blob/master/Lab%2001%20Virtual%20Network.pdf)

    Quick resume:

    1. Created a vNET (10.1.0.0/16).
    2. Created 2 subnets (10.1.1.0/24 & 10.1.2.0/24).
    3. Added a VM on each subnet ("Mgmt" & "Web").
    4. Installed "Apache" on the "Web" VM in subnet 2.

#### [02. Network Security Groups:](https://github.com/binals/azurenetworking/blob/master/Lab%2002%20Network%20Security%20Groups.pdf)

    Quick resume:

    1. Created a NSG (Network Security Group) and associated to each subnet.
    2. Removed the NSGs created with the VM NICs.
    3. Created an ASG (Application Security Group) for each VM ("Mgmt" & "Web").
        Note: The ASGs have the same name as the VMs.

    4. Added the following inbound security rules to the NSG:
        Port: "22", "3389"; Protocol: "TCP"; Source: "Any"; Destination: "Mgmt"; Action: "Allow".
        Port: "80", "443"; Protocol: "TCP"; Source: "Any"; Destination: "Web"; Action: "Allow".
            Note: Port "3389" was added so that I could connect to Windows VMs in the future.

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

    10. Tested connectivity from my local computer to the VMs public & private IPs.

#### [03. Azure CLI:](https://github.com/binals/azurenetworking/blob/master/Lab%2003%20CLI.pdf)

    1. Entered Azure CLI (Bash Shell).
        Note: Created a Storage account and File Share for it.

    2. Created a vNET (10.0.0.0/16) & Subnet (10.0.1.0/24).

       Variables:

            ResourceGroup=Azure_Net_Lab_RG
            VnetName=vnet-hub
            VnetPrefix=10.0.0.0/16
            SubnetName=Vnet-Hub-Subnet1
            SubnetPrefix=10.0.1.0/24
            Location=westeurope
        
        Command:

            az network vnet create -g $ResourceGroup -n $VnetName --address-prefix $VnetPrefix --subnet-name $SubnetName --subnet-prefix $SubnetPrefix -l $Location

    3. Created a NSG and added an inbound security rule.

        Variables:

            ResourceGroup=Azure_Net_Lab_RG
            NSG=NSG-Hub
            NSGRuleName=Vnet-Hub-Allow-SSH
            Location=westeurope
            DestinationAddressPrefix=10.0.1.0/24
            DestinationPortRange=22
        
        Commands:

            az network nsg create --name $NSG --resource-group $ResourceGroup --location $Location
            az network nsg rule create -g $ResourceGroup --nsg-name $NSG --name $NSGRuleName --direction inbound --destination-address-prefix $DestinationAddressPrefix --destination-port-range $DestinationPortRange --access allow --priority 100
    
    4. Attached the NSG to the new Subnet.
    
        Variable:
        
            NSG=NSG-Hub
        
        Command:
        
            az network vnet subnet update -g $ResourceGroup -n $SubnetName --vnet-name $VnetName --network-security-group $NSG

    5. Created a VM.

        Variables:

            VmName=Vnet-Hub-VM1
            SubnetName=Vnet-Hub-Subnet1
            AdminUser=hubuser
            AdminPassword="A secure password"
            
        Command:

            az vm create --resource-group $ResourceGroup --name $VmName --image UbuntuLTS --vnet-name $VnetName --subnet $SubnetName --admin-username $AdminUser --admin-password $AdminPassword

    6. Listed the created subnet.

        az network vnet subnet list -g $ResourceGroup --vnet-name $VnetName -o table

            AddressPrefix: 10.0.1.0/24
            Name: Vnet-Hub-Subnet1
            PrivateEndpointNetworkPolicies: Disabled
            PrivateLinkServiceNetworkPolicies: Enabled
            ProvisioningState: Succeeded
            ResourceGroup: Azure_Net_Lab_RG

#### [04. Virtual Network Peering:](https://github.com/binals/azurenetworking/blob/master/Lab%2004%20Virtual%20Network%20Peering.pdf)

    1. Checked connectivity between vNETs ("Hub" & "Vnet1").
    2. Copied the public IPs of the VMs in the "Hub" vNET & "Mgmt" subnet.
    3. Connected to the "Mgmt" VM with SSH.
    4. Pinged the private IP of the VM in the "Hub" vNET.
    5. Pings didn't succeed.
    6. Created a Peer vNET in "Vnet1" (It automatically creates one on the remote vNET).
    7. Allowed access and traffic forwarding between vNETs.
        Note: Traffic forwarding only works if there is a Firewall or NVA on the Hub.

    8. Verified the "Effective Routes" on the NIC of each VM for "Peered vNET".
    9. Accessed the "Mgmt" VM with its public IP and pinged the "Hub" VM's private IP.
        Note: The "AllowVnetInBound" is what allows communication between resources in connected vNETs.
        Note: If a new rule is created that disables that, we won't be able to ping.

#### [05. Virtual Network Peering - Transitive behavior:](https://github.com/binals/azurenetworking/blob/master/Lab%2005%20Virtual%20Network%20Peering%20-%20Transitive%20behavior.pdf)

    1. Accessed the Azure CLI (Bash Shell).
    2. Created a new vNET & Subnet.

        Variables:

            ResourceGroup=Azure_Net_Lab_RG
            VnetName=Vnet2
            VnetPrefix=10.2.0.0/16
            SubnetName=Vnet2-Subnet1
            SubnetPrefix=10.2.1.0/24
            Location=westeurope

        Command:

            az network vnet create -g $ResourceGroup -n $VnetName --address-prefix $VnetPrefix --subnet-name $SubnetName --subnet-prefix $SubnetPrefix -l $Location

    3. Attached the old NSG to the new Subnet (Bash Shell).

        Variable:

            NSG=NSG1

        Command:
            
            az network vnet subnet update -g $ResourceGroup -n $SubnetName --vnet-name $VnetName --network-security-group $NSG

    4. Created a new VM.

        Variables:

            VmName=Vnet2-VM1
            SubnetName=Vnet2-Subnet1
            AdminUser=azureuser
            AdminPassword="A secure password"
            
        Command:
        
            az vm create --resource-group $ResourceGroup --name $VmName --image UbuntuLTS --vnet-name $VnetName --subnet $SubnetName --admin-username $AdminUser --admin-password $AdminPassword

    5. Removed the NSG created from the VMs creation.
    6. Peered the new "Vnet2" vNET with the "Hub" vNET.
    7. Allowed access and traffic forwarding between vNETs.
    8. Verified the "Effective Routes" on the NIC of the new VM for "Peered vNET".
    9. Altered the NSG to allow SSH to the new VMs public IP.
    10. Accessed the "Vnet2" VM with its public IP and pinged the "Hub" VM's private IP.
    11. Accessed the "Vnet2" VM with its public IP and could not ping the "Mgmt" VM's private IP.
        Note: It doesnt work because transitive peering is not allowed.

#### [06. NVA CSR1000v:](https://github.com/binals/azurenetworking/blob/master/Lab%2006%20NVA%20CSR1000v.pdf)

    1. Deployed a NVA (Metwork Virtual Appliance) to "Vnet1".
        Note: In this case I deployed the Cisco Cloud Services Router 1000V from the Azure Marketplace.
    
    2. Added it to a new Resource Group ("RG-Csr").
    3. Added 2 NICs to it, each one connected to each subnet in "Vnet1".
    4. Accessed the new Router VM with its public IP.
        Note: Since it is in "Vnet1-Subnet1", it receives its NSG.
        Note: It also created its own NSG (Csr1-SSH-SecurityGroup) which is attached to its 2 NICs.

#### [07. Routing Tables:](https://github.com/binals/azurenetworking/blob/master/Lab%2007%20Routing%20Tables.pdf)

    1. I will force the "Web" VM on "Vnet1-Subnet2" to route to "Csr1" when it wants to communicate with the IP range 10.0.1.0/24.
    2.Created a route table and added it to a new Resource Group.
    3. Created a route in it:
        Address prefix: "10.0.1.0/24"; Next hop type: "VirtualAppliance"; Next hop IP address: "10.1.1.5" (The Routers private IP address on "Vnet1-Subnet1").
    
    4. Associated the route table to subnet "Vnet1-Subnet2".
    5. Deleted the route tables created from the creation of the "Csr" VM.
    6. Enabled IP forwarding on the NVAs NIC attached to "Vnet1-Subnet2".
    7. Altered the NSG to allow ICMP to the new VM.
    8. Accessed the "Web" VM with its public IP and pinged the "Csr1" & "Hub" VM's private IPs.
    9. Did a traceroute from the "Web" VM to the "Hub" VM:
        1  csr1.internal.cloudapp.net (10.1.1.5)  3.487 ms  3.458 ms  3.447 ms
        2  10.0.1.4 (10.0.1.4)  4.466 ms *  4.443 ms

#### [08. Site-to-site VPN:](https://github.com/binals/azurenetworking/blob/master/Lab%2008%20Site-to-site%20VPN.pdf)

#### [09. Virtual WAN:](https://github.com/binals/azurenetworking/blob/master/Lab%2009%20Virtual%20WAN.pdf)

#### [10. Standard Load Balancer:](https://github.com/binals/azurenetworking/blob/master/Lab%2010%20Standard%20Load%20Balancer.pdf)

#### [11. Network Watcher NSG Flow Logs:](https://github.com/binals/azurenetworking/blob/master/Lab%2011%20Network%20Watcher%20NSG%20Flow%20Logs.pdf)

#### [12. Firewall:](https://github.com/binals/azurenetworking/blob/master/Lab%2012%20Firewall.pdf)

#### [13. Firewall-Inbound NAT:](https://github.com/binals/azurenetworking/blob/master/Lab%2013%20Firewall%20-%20Inbound%20NAT.pdf)

#### [14. Firewall - Spoke to spoke communication:](https://github.com/binals/azurenetworking/blob/master/Lab%2014%20Firewall%20-%20Spoke%20to%20spoke%20communication.pdf)
