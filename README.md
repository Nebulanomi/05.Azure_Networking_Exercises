# Azure Networking labs

## Introduction

This set of Azure Networking labs are simplified to demonstrate a single concept in each lab, using Azure portal or Azure CLI.

Each lab has a diagram that provides information on the lab setup.

Most labs build on each other so prior setup is expected.

	Note: Some alterations were made on on my lab completion because there are some errors in the files.

## Table of Lab Content

#### [01. Virtual Network:](https://github.com/binals/azurenetworking/blob/master/Lab%2001%20Virtual%20Network.pdf)

    1. Created a vNET (10.1.0.0/16).
    2. Created 2 subnets (10.1.1.0/24 & 10.1.2.0/24).
    3. Added a VM on each subnet ("Mgmt" & "Web").
    4. Installed "Apache" on the "Web" VM in subnet 2.

#### [02. Network Security Groups:](https://github.com/binals/azurenetworking/blob/master/Lab%2002%20Network%20Security%20Groups.pdf)

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
    9. Added the following inbound security rules to the NSG:

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

            az network vnet create \
            -g $ResourceGroup \
            -n $VnetName \
            --address-prefix $VnetPrefix \
            --subnet-name $SubnetName \
            --subnet-prefix $SubnetPrefix \
            -l $Location

    3. Created a NSG and added an inbound security rule.

        Variables:

            ResourceGroup=Azure_Net_Lab_RG
            NSG=NSG-Hub
            NSGRuleName=Vnet-Hub-Allow-SSH
            Location=westeurope
            DestinationAddressPrefix=10.0.1.0/24
            DestinationPortRange=22
        
        Commands:

            az network nsg create \
            --name $NSG \
            --resource-group $ResourceGroup \
            --location $Location
            
            az network nsg rule create \
            -g $ResourceGroup \
            --nsg-name $NSG \
            --name $NSGRuleName \
            --direction inbound \
            --destination-address-prefix $DestinationAddressPrefix \
            --destination-port-range $DestinationPortRange \
            --access allow \
            --priority 100
    
    4. Attached the NSG to the new Subnet.
    
        Variable:
        
            NSG=NSG-Hub
        
        Command:
        
            az network vnet subnet update \
            -g $ResourceGroup \
            -n $SubnetName \
            --vnet-name $VnetName \
            --network-security-group $NSG

    5. Created a VM.

        Variables:

            VmName=Vnet-Hub-VM1
            SubnetName=Vnet-Hub-Subnet1
            AdminUser=hubuser
            AdminPassword="A secure password"
            
        Command:

            az vm create \
            --resource-group $ResourceGroup \
            --name $VmName \
            --image UbuntuLTS \
            --vnet-name $VnetName \
            --subnet $SubnetName \
            --admin-username $AdminUser \
            --admin-password $AdminPassword

    6. Listed the created subnet.

        az network vnet subnet list \
        -g $ResourceGroup \
        --vnet-name $VnetName \
        -o table

![plot](./04.%20Images/image.png)

#### [04. Virtual Network Peering:](https://github.com/binals/azurenetworking/blob/master/Lab%2004%20Virtual%20Network%20Peering.pdf)

    1. Checked connectivity between vNETs ("Hub" & "Vnet1").
    2. Copied the public IPs of the VMs in the "Hub" vNET & "Mgmt" subnet.
    3. Connected to the "Mgmt" VM with SSH.
    4. Pinged the private IP of the VM in the "Hub" vNET and verified that it didnt succeed.
    5. Created a Peer vNET in "Vnet1" (It automatically creates one on the remote vNET).
    6. Allowed access and traffic forwarding between vNETs.
        Note: Traffic forwarding only works if there is a Firewall or NVA on the Hub.

    7. Verified the "Effective Routes" on the NIC of each VM for "Peered vNET".
    8. Accessed the "Mgmt" VM with its public IP and pinged the "Hub" VM's private IP.
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

            az network vnet create \
            -g $ResourceGroup \
            -n $VnetName \
            --address-prefix $VnetPrefix \
            --subnet-name $SubnetName \
            --subnet-prefix $SubnetPrefix \
            -l $Location

    3. Attached the old NSG to the new Subnet (Bash Shell).

        Variable:

            NSG=NSG1

        Command:
            
            az network vnet subnet update \
            -g $ResourceGroup \
            -n $SubnetName \
            --vnet-name $VnetName \
            --network-security-group $NSG

    4. Created a new VM.

        Variables:

            VmName=Vnet2-VM1
            SubnetName=Vnet2-Subnet1
            AdminUser=azureuser
            AdminPassword="A secure password"
            
        Command:
        
            az vm create \
            --resource-group $ResourceGroup \
            --name $VmName \
            --image UbuntuLTS \
            --vnet-name $VnetName \
            --subnet $SubnetName \
            --admin-username $AdminUser \
            --admin-password $AdminPassword

    5. Removed the NSG created from the VMs creation.
    6. Peered the new "Vnet2" vNET with the "Hub" vNET.
    7. Allowed access and traffic forwarding between vNETs.
    8. Verified the "Effective Routes" on the NIC of the new VM for "Peered vNET".
    9. Altered the NSG to allow SSH to the new VMs public IP.
    10. Accessed the "Vnet2" VM with its public IP and pinged the "Hub" VM's private IP.
    11. Accessed the "Vnet2" VM with its public IP and verified that I couldn't ping the "Mgmt" VM's private IP.
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

    1. Created a route table and added it to a new Resource Group.
    2. Created a route in it:

        Address prefix: "10.0.1.0/24"; Next hop type: "VirtualAppliance"; Next hop IP address: "10.1.1.5" (The Routers private IP address on "Vnet1-Subnet1").
    
    3. Associated the route table to subnet "Vnet1-Subnet2".
    4. Deleted the route tables created from the creation of the "Csr" VM.
    5. Enabled IP forwarding on the NVAs NIC attached to "Vnet1-Subnet2".
    6. Altered the NSG to allow ICMP to the new VM.
        Note: All of this forced the "Web" VM on "Vnet1-Subnet2" to route to "Csr1" when it wants to communicate with the IP range 10.0.1.0/24.
    
    7. Accessed the "Web" VM with its public IP and pinged the "Csr1" & "Hub" VM's private IPs.
    8. Did a traceroute from the "Web" VM to the "Hub" VM:
     
        1  csr1.internal.cloudapp.net (10.1.1.5)  3.487 ms  3.458 ms  3.447 ms
        2  10.0.1.4 (10.0.1.4)  4.466 ms *  4.443 ms

#### [08. Site-to-site VPN:](https://github.com/binals/azurenetworking/blob/master/Lab%2008%20Site-to-site%20VPN.pdf)

    1. Created the "OnPrem" vNET:

        Variables:

            ResourceGroup=RG-Lab
            VnetName=OnPrem
            VnetPrefix=10.128.0.0/16
            SubnetName=OnPrem-Subnet1
            SubnetPrefix=10.128.1.0/24
            Location=westeurope

        Command:

            az network vnet create \
            -g $ResourceGroup \
            -n $VnetName \
            --address-prefix $VnetPrefix \
            --subnet-name $SubnetName \
            --subnet-prefix $SubnetPrefix \
            -l $Location

    2. Created a Gateway Subnet in the "Hub" (10.0.254.0/27) & "OnPrem" vNET (10.128.254.0/27).
        Note: A VPN GW needs to be deployed in a specific subnet named "GatewaySubnet".

    3. Created a "Gen1" "Route-based" VNG (Virtual Network Gateway) in the "Hub" & "OnPrem" vNET.
        Note: Active-Active mode is "disabled" & BGP is "enabled" with ASN as "65002".

    4. Created a LNG (Local Network Gateway) with the "On-prem" address space & the details of the "OnPrem" VNG.
        Note: LNG refers to the details of the local VNG including its IP address, BGP AASN & peering IP.
        Note Address space refers to the local address range that we want to be reachable from the other VNG.

    5. Created an LNG with the "Hub" address space & the details of the "Hub" VNG.
    6. Added a VPN connection on the "Hub" GW:

        Connection type: "Site-to-Site (IPsec)"; VNG: "Hub GW"; LNG: "OnPrem GW"; Shared key: "A secret key"; IKE protocol: "IKEv2"; 

    7. Added a VPN connection on the "OnPrem" GW but with the "Onprem" VNG & the "Hub" LNG.
    8. A site to Site VPN is created between the "Hub" vNET & "Onprem" vNET.
    9. Verified that the connections are in the "Connected" status.
    10. Created a VM on the "OnPrem" vNET with Azure CLI (Bash Shell).
    
        Variables:

            ResourceGroup=RG-Lab
            VmName=OnPrem-VM1
            VnetName=OnPrem
            SubnetName=OnPrem-Subnet1
            AdminUser=azureuser
            AdminPassword="A secure key"

        Command:

            az vm create \
            --resource-group $ResourceGroup \
            --name $VmName \
            --image UbuntuLTS \
            --vnet-name $VnetName \
            --subnet $SubnetName \
            --admin-username $AdminUser \
            --admin-password $AdminPassword

    11. Verified the VPN connections by accessing the "OnPrem" VM and connecting it to the VM on the "Hub" vNET.
    12. Added a new prefix on the "Hub" LNG with a range that captures the "Vnet1" VMs private IP (10.1.0.0/16).
    13. Connected to the "OnPrem" VM with its public IP and pinged the private IP address of the "Hub" & "Mgmt" VM.
    14. Verified that only the "Hub" VM was reachable.
    15. Enabled "Allow Gateway Transit" on the "Hub" vNET peering with the "Vnet1" vNET.
    16. Enabled "Use Remote Gateways" on the "Vnet1" vNET peering with the "Hub" vNET.
    17. Verified that both VMs were now reachable.

#### [09. Virtual WAN:](https://github.com/binals/azurenetworking/blob/master/Lab%2009%20Virtual%20WAN.pdf)

    1. A "standard" virtual WAN service is created in a new Resource Group.
    2. In it I created a virtual Hub (vnet) is created with the following information:
        1. Basic: The lowest "Virtual Hub Capacity" (Which is 2 Units) and with a private address of 10.64.0.0/16.
        2. Site-to-Site: Enabled and "Gateway scale units" to 1 scale.
            Note: Two instances are deployed when a VPN gateway is provisioned in a virtual Hub. The Aggregate capacity is the bandiwdth of both put together.
        
    3. Created a "VPN site" on the virtual WAN that corresponds to the "OnPrem vNETs VNG":

            Device vendor: "MSN"; Link speed: "50"; Link provider name: "MSN", Link IP address: "OnPrem NVGs Public IP"; Link BGP address: "OnPrem NVGs BGP address"; link ASN: "OnPrem NVGs ASN".

    4. Verified that the "VPN site" has Hub with status "Connection Needed" because it hasnt been connected to the "Hub" that was created earlier.
    5. Accessed the "Hub" VPN GW and connected the "VPN site" that was created:

        PSK: "Secret key"; Protocol: "IPsec"; IPsec: "Default";
        
    6. Verified that the Connection Provisioning status equals "Succeeded" & Connectivity status equals "Not Connected".
    7. Downloaded the "VPN config" on the "Hub" router which contains information of the "vWAM" and the "OnPrem VPN site".
        Note: It creates a storage account in a new resource group.
        
    8. Created a LNG of the "Hub" vNET with information from the downloaded file:

        IP address: "The vWANs Public IP"; Address space: "10.64.0.0/16"; BGP: "Enabled"; ASN: "65515"; BGP Peer ID address: "The vWANs BGP peer IP".
        
    9. Added a new "Site-to-Site" Connection with the "OnPrem" VNG and the LNG of the "Hub" vWAN.
        Note: PSK & IKE Protocol must be the same & BGP peer enabled.
        
    10. Verified that the status of the connection changed to "Connected" on both sides.
    11. Deleted everything related to the "Hub" GW and its connections to the "Vnet1" vNET from the previous lab.
    12. Created a connection between the "vWAN Hub" and "Vnet1" in "Virtual Network Connections" with everything as default.
    13. Verified that the connection is "Succeeded".
    14. Verified that the "topology" of the vWAN is correct.
    15. Verified the "Hub" status as "Succeeded" & shows "1 VPN site connected".
    16. Verified that the "VPN site" has "Site Provisioning Status" as "Provisioned".
    17. Verified that pinging from the "On-Prem" VM to the private IP of the "Mgmt" Vm was successful.
    18. Created a new "Branch2" vnet with IP range 10.129.0.0/16 and subnet 10.129.1.0/24.
    19. Created a "Gateway subnet" with IP range 10.129.0.0/24.
    20. Created a VPN GW:

        Gateway type: "VPN"; VPN type: "Route-based"; SKU: "VpnGW1"; Generation: "Generation1"; virtual network: "Branch2";
        Public IP address: "Create new";
        Enable active-active mode: "Disabled";
        Configure BGP: "Enabled";
        ASN: "65003";
    
    21. Added a new virtual WAN "Hub2" Hub in another region with IP range 10.65.0.0/16.
    22. Created a new VPN GW in in the new Hub.
    23. Created a new "VPN Site":

        Device vendor: "MSN"; Link speed: "50"; Link provider name: "MSN", Link IP address: "Branch NVGs Public IP"; Link BGP address: "Branch NVGs BGP address"; link ASN: "Branch NVGs ASN".

    24. Connected the VPN site to the new Hub:

        PSK: "Secret key"; Protocol: "IPsec"; IPsec: "Default";

    25. Downloaded the configuration and added the necessary information to a new "Local Network Gateway":

        IP address: "The vWANs Public IP"; Address Space: "10.65.0.0/16" BGP: "Enabled"; ASN: "65515"; BGP Peer ID address: "The vWANs BGP peer IP".

    26. Configured a Site-to-Site connection on the Branch2 VPN GW with the LNG to the new Hub.
    27. Verified that connection was established.
    28. Created a VM in the Branch2 vnet and successfuly pinged the VM on the vnet and onprem.

#### [10. Standard Load Balancer:](https://github.com/binals/azurenetworking/blob/master/Lab%2010%20Standard%20Load%20Balancer.pdf)

    1. A "Web2" VM was created in the same location as "Web1" and with the same "web" ASG in Azure CLI (Baash Shell).

        Variables:

            ResourceGroup=Azure_Net_Lab_RG
            VmName=Vnet1-VM-Web2
            VnetName=Vnet1
            SubnetName=Vnet1-Subnet2
            Admin=azureuser
            Password="A secure password"

        Command:
        
            az vm create --resource-group $ResourceGroup \
            --name $VmName \
            --image UbuntuLTS \
            --vnet-name $VnetName \
            --subnet $SubnetName \
            --nsg "" \
            --asgs Web \
            --public-ip-address "" \
            --admin-username $Admin \
            --admin-password $Password

    2. Connected to the "Mgmt" VM with its public IP and accessed the "Web1" VM.
    3. Elevated to root and installed apache and wrote HTML text to the index.html file:

        echo '<!doctype html><html><body><h1>Web Server 1</h1></body></html>' | tee /var/www/html/index.html

    4. Connected to the "Web2" VM with its private IP and did the same as "Web1":

        echo '<!doctype html><html><body><h1>Web Server 2</h1></body></html>' | tee /var/www/html/index.html

    5. A "standard" LB (Load Balancer) is configured on vNET "Vnet1".
        Note: It is a Layer 4 LB (Balances TCP & UDP traffic).
        
    6. Public IP address was created for the Frontend.
    7. A backend pool was added with both "Web" VMs.
        Note: The backend for the LB includes the 2 Web VMs on the vNET "Vnet1".

    8. A Load Balancing Rule was added and is connected to the Frontend IP and Backend pool.
        Note: In it a health probe was added to monitor the status of port "80" and protocol "HTTP".
        
    9. Verified that "Web1" appears when connecting to the LBs public IP & "Web2" after shutting down "Web1" VM.

#### [11. Network Watcher NSG Flow Logs:](https://github.com/binals/azurenetworking/blob/master/Lab%2011%20Network%20Watcher%20NSG%20Flow%20Logs.pdf)

    1. Accessed Network Watcher in the search box and verified that there was one from the previous exercises.
        Note: Network Watcher is created in its own Resource Group "NetworkWatcherRG" when a vNET is created and therefore it is also registered in the subscription by default.

    2. Created a "Standard" Azure Storage account.
        Note: NSG flow log needs it to write data.

    3. Created a new "Log Analytics Workspace".
    4. Accessed Network Watcher and created a "NSG flow log" for "NSG1":

        Resource: NSG1; Storage Account: "The one created earlier"; Retention: "2"; Flow Logs Version: 1;
        Traffic Analytics: "Enabled"; Traffic Analytics processing interval: "Every 10 mins"; Log Analytics Workspace: "The one created earlier".
        
        Note: Version 1 logs ingress and egress IP traffic flows for both allowed and denied traffic.
        Note: Traffic Analytics provides rich analytics and visualization derived from flow logs.

    5. Accessed Network Watcher and created a "NSG flow log" for "NSG-Hub".
        Note: Configured with the same information as as "NSG1" but with the different Resource.
    
    6. Accessed the "NSG flow logs" in Network Watcher.
    7. Clicked on the "Storage Account" connected to both.
    8. Went to "Containers" and accessed the "networksecuritygroupflowevent" container.
    9. Navigated through the hierarchy until the "PT1H.json" file is found.
        Note (Naming convention):
            
            https://{storageAccountName}.blob.core.windows.net /  
            insights-logs-networksecuritygroupflowevent / 
            resourceId= / SUBSCRIPTIONS / {SubscriptionID} / 
            RESOURCEGROUPS / {ResourceGroupName} / 
            PROVIDERS / MICROSOFT.NETWORK / 
            NETWORKSECURITYGROUPS / {NSGName} / 
            y={Year} / m={Month} / d={Day} / h={Hour} / m={Minute} / 
            macAddress={MACAddress} /
            PT1H.json

    10. Downloaded the file and verified the "flow log" in it.
        Note: The MAC addresses shown are VM NICs.
        
        Example of a flowTuples information:

            1542110377: The time stamp of when the flow occurred, in UNIX EPOCH format.
                Note: This converts to May 1, 2018 at 2:59:05 PM GMT. 
            
            10.0.0.4: The source IP address that the flow originated from.
            13.67.143.118: The destination IP address that the flow was destined to.
            44931: The source port that the flow originated from.
            443: The destination port that the flow was destined to.
                Note: If the port matches a rule then it will show as the one that processed it.
                
            T: Whether the protocol of the flow was TCP (T) or UDP (U).
            O: Whether the traffic was inbound (I) or outbound (O).
            A: Whether the traffic was allowed (A) or denied (D).
            C: Captures the state of the flow.
                Note: Possible states are:

                    B - When a flow is created (Statistics aren't provided).
                    C - Continuing for an ongoing flow (Statistics are provided at 5-minute intervals).
                    E - When a flow is terminated (Statistics are provided). 
            
            30 Packets sent: The total number of TCP or UDP packets sent from source to destination since last update (Or vice-versa). 
            16978 Bytes sent: The total number of TCP or UDP packet bytes sent from source to destination since last update (Or vice-versa).
                Note: Packet bytes include the packet header and payload.

#### [12. Firewall:](https://github.com/binals/azurenetworking/blob/master/Lab%2012%20Firewall.pdf)

    1. Created a subnet for the Firewall in "vnet-hub" (10.0.251.0/24).
            Note: Azure Firewall requires a dedicated subnet called "AzureFirewallSubnet".

    2. Accessed "Firewalls" in the search box and created one:

        Resource group: "The same as the virtual network";
        Firewall SKU: "Standard"; Firewall SKU: "Premium";
        Virtual Network: "Vnet-Hub"; Public IP: "A new one with Standard SKU".

    3. Created an application rule in the Firewall in "Application rule collection" (Layer 7):
        Note: that allows outbound access to "www.microsoft.com".

        Priority: "200"; Action: "Allow"; Target FQDN source: 10.1.2.0/24;
        Protocol:Port: "http, https"; Target FQDN: "www.microsoft.com".

    4. Created a route table in the same region as the Firewall.
    5. Created a custom route:

        Destination type: "IP Addresses"; Destination IP addresses: "0.0.0.0/0";
        Next hop type: "Virtual appliance"; Next hop address: "The Firewalls private IP address".

    6. Associated the custom route to the spoke "Vnet1" vNET & subnet "Vnet1-Subnet2".
    7. Accessed the "Mgmt" VM through the "Serial Console" and verified that "curl www.microsoft.com" was working.
    8. Did "curl www.google.com" and it didnt work, giving a firewall error.
        Note: Verified that the vNETs are still peered to each other and that "Vnet1" isnt using "Remote GW".

#### [13. Firewall-Inbound NAT:](https://github.com/binals/azurenetworking/blob/master/Lab%2013%20Firewall%20-%20Inbound%20NAT.pdf)

    1. Accessed the Firewall and added a "NAT rule collection" to allow SSH thorugh the Firewall to the "Mgmt" VM:

        Name: "InboundNAT"; Priority: "200";
        Rules:Name: "NatRule1"; Rules:Protocol: "TCP";
        Rules:Source Addresses: *; Rules:Destination Addresses: "Firewalls public IP";
        Rules:Destination ports: "8022"; Translated Address: "Mgmt VMs private IP"; Rules:Translated port: "22".

    2. Verified that SSH was possible with the Firewalls public IP and port "8022".

#### [14. Firewall - Spoke to spoke communication:](https://github.com/binals/azurenetworking/blob/master/Lab%2014%20Firewall%20-%20Spoke%20to%20spoke%20communication.pdf)

    1. Deleted the previous "vNET2" and everything in it.
    2. Created "vNET2" again with Azure CLI (Bash Shell).

        Variables:

            ResourceGroup=Virtual_Network        
            VnetName=vnet2
            VnetPrefix=10.2.0.0/16
            SubnetName=vnet2-subnet1
            SubnetPrefix=10.2.1.0/24
            Location=westeurope

        Command:
        
            az network vnet create \
            -g $ResourceGroup \
            -n $VnetName \
            --address-prefix $VnetPrefix \
            --subnet-name $SubnetName \
            --subnet-prefix $SubnetPrefix \
            -l $Location

    3. Verified that "vNET2" was created:

        az network vnet list -o table

![plot](./04.%20Images/image2.png)

    4. Peered "vNET2" with the "Hub" vNET:

        Variables:

            ResourceGroup=Virtual_Network
            PeeringName=peer-vnet2-vnet-hub 
            VnetName=vnet2 
            RemoteVnet=vnet-hub 
 
        Command:
        
            az network vnet peering create \
            --name $PeeringName \
            --remote-vnet $RemoteVnet \
            --resource-group $ResourceGroup \
            --vnet-name $VnetName \
            --allow-forwarded-traffic \
            --allow-vnet-access
    
    5. Peered the "Hub" vNET with "vNET2":

        Variables:

            ResourceGroup=Virtual_Network
            PeeringName=peer-vnethub-to-vnet2   
            VnetName=vnet-hub
            RemoteVnet=vNET2   
 
        Command:

            az network vnet peering create \
            --name $PeeringName \
            --remote-vnet $RemoteVnet \
            --resource-group $ResourceGroup \
            --vnet-name $VnetName \
            --allow-forwarded-traffic \
            --allow-vnet-access

    6. Made "Allow Forwarded traffic" enabled for "vNET1" peering:

        Variables:

            ResourceGroup=Virtual_Network
            PeeringName=vnet1-to-hub
            VnetName=vnet1  
 
        Command:

            az network vnet peering update \
            -n $PeeringName \
            -g $ResourceGroup \
            --vnet-name $VnetName \
            --set allowForwardedTraffic=true            

        Variables:
        
            PeeringName=hub-to-vnet1  
            VnetName=vnet-hub

        Command:
        
            az network vnet peering update \
            -n $PeeringName \
            -g $ResourceGroup \
            --vnet-name $VnetName \
            --set allowForwardedTraffic=true

    7. Verified peering status between "vNET2" & the "Hub" vnet:

        Variables:

            VnetName=vnet1
            ResourceGroup=Virtual_Network

        Command:
        
            az network vnet peering list \
            -g $ResourceGroup \
            --vnet-name $VnetName \
            -o table

        Variables:
        
            VnetName=vnet2
            ResourceGroup=Virtual_Network

        Command:
        
            az network vnet peering list \
            -g $ResourceGroup \
            --vnet-name $VnetName \
            -o table

![plot](./04.%20Images/image3.png)

    8. Added a NSG to "vNET2".

        Variables:

            Nsg=nsg-hub 
            SubnetName=vnet2-subnet1
            VnetName=vnet2
            ResourceGroup=Virtual_Network

        Command:

            az network vnet subnet update \
            -g $ResourceGroup \
            -n $SubnetName \
            --vnet-name $VnetName \
            --network-security-group $Nsg 

    9. Added a VM to "vNET2".

        Variables:

            ResourceGroup=Virtual_Network
            VmName=vnet2-vm1
            SubnetName=vnet2-subnet1
            VnetName=vnet2
            AdminUser=azureuser
            AdminPassword=Azure123456!

        Command:

            az vm create \
            --resource-group $ResourceGroup \
            --name $VmName \
            --image UbuntuLTS \
            --vnet-name $VnetName \
            --subnet $SubnetName \
            --admin-username $AdminUser \
            --admin-password $AdminPassword \
            --nsg "" 
 
    10. Associated the route table from "vNET1-subnet1" with "vNET2-subnet1".
        Note: This will allow both of them to have a default route to the Firewall.

    11. Added a "Network Rule Collection" to the firewall to allow ICMP traffic between the vNETs:

        Name: "Allow-ICMP"; Priority: "200"; Action: "Allow";
        IP Addresses:Name: "Allow-ICMP"; Protocol: "ICMP"; Source Addresses: "10.0.0.0/8";
        Destination addresses: "10.0.0.0/8"; Destination Ports: "*".

    12. Accessed the "Mgmt" VM and pinged the new VM on "vNET2".
    13. Pinged successfuly between them.

#### [Lab 15 Private Link - Private endpoint:](https://github.com/Nebulanomi/Azure_Networking_Labs/blob/master/01.%20Azure%20Networking%20Labs/Private%20Link%20-%20Private%20endpoint.md)

    1. Created a "standard" "LRS" storage account.
        Note: Left the rest as default.

    2. Created a "Hub" vNET for the Private Endpoint to reside in (10.0.0.0/16) & a subnet for it (10.0.1.0/24).
    3. Created a Private Endpoint:

        Name: "pe-sa1"; Network Interface Name: "pe-sa1-nic";
        Connection method: "Connect to an Azure resource in my directory"; Resource type: "Microsoft.Storage/storageAccounts"; Resource: "The name of the storage account"; Target sub-resource: "blob";
        Virtual network: "The Hub vNET"; Subnet: "The Hub Subnet";
        Integrate with private DNS zone: "Yes"; Resource group: "Our resource group";

    4. Accessed the Private Endpoint and verified that it had a NIC associated to it in the "DNS configuration".
        Note: I also verified that the private IP address is associated to the FQDN of the storage account.

    5. Accessed the Private DNS zone & verified that the "Hub" vNET was linked with the Private DNS Zone.
    6. Created a VM in the same subnet without a "Public IP" & accessed it through the "Serial Console".
    7. Verified that the Private Endpoints IP address appeared after doing an "nslookup" with the Storage Accounts FQDN.
        Note: This shows that the VM was forced to go to the Private Endpoint to reach the Storage Account.

    8. Verified that by doing the same on my local computer that it went directly to the Storage Accounts public IP.