# Imersão Azure Expert - Day 01

### Hands-on Lab

## Exercise 1 - Azure Calculator (20 minutes)

1. In a browser, navigate to the [Azure Pricing Calculator](https://azure.microsoft.com/en-us/pricing/calculator/) webpage.

2. Add the inventory data.

   ![Screenshot of the inventory data](/AllFiles/Images/IMG01.png)

**Important Notes**
- Development environment licensing will be purchased by Marketplace
- Include Reserved instances and Azure Hybrid Benefit in the Production Environment 
- Include VPN Gateway, Load Balancers, Public IP Address and 500 GB Outbound Data Transfer.

3. Scroll to the bottom of the Azure Pricing Calculator webpage to view total Estimated monthly cost.

4. Change the currency to Brazilian Real (R$), then select Export to download a copy of the estimate for offline viewing in Microsoft Excel (.xlsx) format.

## Lab 1 - Deploy Virtual Network (15 minutes)

1. In the Azure portal, search for and select **Virtual networks**, and, on the **Virtual networks** blade, click **+ Add**.

1. Create a virtual network with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you will be using in this lab |
    | Resource Group | (create new) **WGVNetRG1** |
    | Name | **WGVNet1** |
    | Region | the name of any Azure region available in the subscription you will use in this lab |
    | IPv4 address space | **10.7.0.0/20** |
    | Subnet name | **Management** |
    | Subnet address range | **10.7.2.0/25** |
     | | |
      
1. Accept the defaults and click **Review and Create**. Let validation occur, and hit **Create** again to submit your deployment.

1. Once the deployment completes browse for **Virtual Networks** in the portal search bar. Within **Virtual networks** blade, click on the newly created virtual network.

1. On the virtual network blade, click **Subnets** and then click **+ Subnet**. 

1. Create a subnet with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **AzureBastionSubnet** |
    | Address range (CIDR block) | **10.7.5.0/24** |
    | Network security group | **None** |
    | Route table | **None** |
    | | |

1. Explore properties to Virtual networks.

## Lab 2 - Deploy Virtual Machine (20 minutes)

1. In the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, click **+ Add**.

1. On the **Basics** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Subscription | the name of the Azure subscription you will be using in this lab |
    | Resource group | **WGVNetRG1** |
    | Virtual machine name | **WGVM1** |
    | Region | select same region the Resouce group | 
    | Availability options | **No** |
    | Image | **Windows Server 2019 Datacenter - Gen 2** |
    | Azure Spot instance | **No** |
    | Size | **Standard_B2s or other** |
    | Authentication type | Password |
    | Username | **demouser** |
    | Password | **demo@pass123** |
    | Public inbound ports | **RDP (3389) and HTTP (80)** |
     | | |

1. Click **Next: Disks >** and, on the **Disks** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | OS disk type | **Standard SSD** |
    | Enable Ultra Disk compatibility | **No** |

1. Click **OK** and, back on the **Networking** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Virtual Network | **WGVNet1** |
    | Subnet | **Management** |
    | Public IP | **WGVM1-PI** |
    | NIC network security group | **Basic** |
    | Accelerated networking | **Off** |
	| Inbound Ports | **RDP (3389) and HTTP (80)** |
    | Place this virtual machine behind an existing load balancing solution? | **No** |
    | | |

1. Click **Next: Management >** and, on the **Management** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Boot diagnostics | **Enable with managed storage account (recommended)** |
    | Enable auto-shutdown | **Off** |   
    
1. On the **Review + Create** blade, click **Create**.

1. Connect Virtual machine, in the session RDP.

1. Install the Web-Server feature in the virtual machine by running the following command in the **Administrator Windows PowerShell** command prompt. You can copy and paste this command.

   ```powershell
   Install-WindowsFeature -name Web-Server -IncludeManagementTools
   Remove-item  C:\inetpub\wwwroot\iisstart.htm
   Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hub Networking is running " + $env:computername)
   ```

1. Within the computer, start Browser and navigate on **Public IP Address** of the **WGVM1**.

## Lab 3 - Configure Azure DNS for internal name resolution for Hub (15 minutes)

1. In the Azure portal, search for and select **Private DNS zones** and, on the **Private DNS zones** blade, click **+ Add**.

1. Create a private DNS zone with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource Group | **WGVNetRG1** |
    | Name | **woodgrove.corp** |

1. Click Review and Create. Let validation occur, and hit Create again to submit your deployment.

    >**Note**: Wait for the private DNS zone to be created. This should take about 2 minutes.

1. Click **Go to resource** to open the **woodgrove.corp** DNS private zone blade.

1. On the **woodgrove.corp** private DNS zone blade, in the **Settings** section, click **Virtual network links**

1. Click **+ Add** to create a virtual network link with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Link name | **WGVNet1VNL** |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Virtual network | **WGVNet1** |
    | Enable auto registration | enabled |

1. Click **OK**.

    >**Note:** Wait for the virtual network link to be created. This should take less than 1 minute.

1. On the **woodgrove.corp** private DNS zone blade, in the sidebar, click **Overview**

1. Verify that the DNS records for **WGVM1** appear in the list of record sets as **Auto registered**.

    >**Note:** You might need to wait a few minutes and refresh the page if the record sets are not listed.

1. Switch to the RDP session to **WGVM1**.

1. In the Terminal console, run the following to test internal name resolution of the **WGVM1** DNS record set in the newly created private DNS zone:

   ```shell
   nslookup WGVM1.woodgrove.corp
   ```

1. Verify that the output of the command includes the private IP address of **WGVM1**.

## Lab 4 - Deploy Virtual Network Spoke for Application (15 minutes)

1. In the Azure portal, search for and select **Virtual networks**, and, on the **Virtual networks** blade, click **+ Add**.

1. Create a virtual network with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you will be using in this lab |
    | Resource Group | (create new) **WGVNetRG2** |
    | Name | **WGVNet2** |
    | Region | the name of any Azure region available in the subscription you will use in this lab |
    | IPv4 address space | **10.8.0.0/20** |
    | Subnet name | **AppSubnet** |
    | Subnet address range | **10.8.0.0/25** |
     | | |
      
1. Accept the defaults and click **Review and Create**. Let validation occur, and hit **Create** again to submit your deployment.

1. Once the deployment completes browse for **Virtual Networks** in the portal search bar. Within **Virtual networks** blade, click on the newly created virtual network.

1. On the virtual network blade, click **Subnets** and then click **+ Subnet**. 

1. Create a subnet with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **DataSubnet** |
    | Address range (CIDR block) | **10.8.1.0/25** |
    | Network security group | **None** |
    | Route table | **None** |
    | | |

## Lab 5 - Deploy Woodgrove Application for Spoke (30 minutes)

1. Deploy the template **woodgrove.json** to a new resource group. This template deploys a two virtual machines running application and database.

    > **Note:** You can deploy the template by selecting the 'Deploy to Azure' button below. You will need to create a new resource group **WGVNetRG2**. You will also need to select a location **East US or location you hava quotas for VMs**. Then choose **Review + create** followed by **Create**. 

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmaiaacademy%2Fworkshop-day-networking-avancado-azure%2Fmain%2FCloudShop.json" target="_blank">![Button to deploy the Woodgrove template to Azure.](/AllFiles/Images/deploy-to-azure.png)</a>

    > **Note:** The template will take around 30 minutes to deploy. Once template deployment is complete, several additional scripts are executed to bootstrap the lab environment. **Allow at least 45 minutes from the start of template deployment for the scripts to run.**

1. Within the computer, start Browser and navigate to **IP Public address** of the **WGWEB1**.

1. Examine the navegate on Application was successful.

## Lab 6 - Network Security groups (30 minutes)

1.  In the Azure portal, select **+ Create a resource**. In the **Search the Marketplace** box, search for and select **Application security group**. Next, on the **Application security group** blade, select **Create**.

2.  On the **Create an application security group** blade, on the **Basics** tab, enter the following information, and select **Review + create**:

    -  Subscription: **Select your subscription**.

    -  Resource group: **WGVNetRG2**

    -  Name: **WebTier**

    -  Region: This must match the location in which you created the **WGVNet2** virtual network.

3.  On the **Create an application security group** blade, on the **Review + Create** tab, ensure the validation passes, and select **Create**. 

4.  Repeat the previous two steps to create an application security group named **DataTier** with the following settings.

    -  Subscription: **Select your subscription**.

    -  Resource group: **WGVNetRG2**

    -  Name: **DataTier**

    -  Region: This must match the location in which you created the **WGVNet2** virtual network.

1.  In the Azure portal, navigate to the **Virtual machines** blade and select **WGWEB1**.

2.  On the **WGWEB1** blade, select **Networking** under **Settings** on the left.

3.  On the **WGWEB1 - Networking** blade, select **Application security groups** and then select **Configure the application security groups**.

4.  On the **Configure the application security groups** blade, in the **Application security groups** drop-down list, select **WebTier**, then **Save**.

7.  Repeat steps for **WGWEB2** and associate **Application security groups**. 

6.  Repeat steps, but this time for **WGSQL1** in order to assign to its network interface the **DataTier** application security group.

1.  In the Azure portal, select **+ Create a resource**. In the **Search the Marketplace** box, **Network security group** and press Enter. Select it and on the **Network security group** blade, select **Create**.

2.  On the **Create network security group** blade, enter the following information, and select **Review + Create** then **Create**:
   
    -  Subscription: **Select your subscription**.

    -  Resource group: **WGVNetRG2**

    -  Name: **WGAppNSG1**

    -  Region: This must match the location in which you created the **WGVNet2** virtual network.

3.  In the Azure Portal, navigate to **All Services**, type **Network security groups** the search box and select **Network security groups**.

4.  On the **Network security groups** blade, select **WGAppNSG1**. 

5.  On the **WGAppNSG1** blade, select **Inbound security rules** under **Settings** on the left and select **Add**.

6.  On the **Add inbound security rule** blade, enter the following information, and select **Add**:

    -  Source: **Application security group**

    -  Source application security group: **WebTier**

    -  Source port ranges: **\***

    -  Destination: **Application security group**

    -  Destination application security group: **DataTier**

    -  Destination port ranges: **1433**

    -  Protocol: **TCP**

    -  Action: **Allow**

    -  Priority: **100**

    -  Name: **AllowDataTierInboundTCP1433**

7.  On the **WGAppNSG1 - Inbound security rules** blade, select **Add**.

8.  On the **Add inbound security rule** blade, enter the following information, and select **Add**:

    -  Source: **Any**

    -  Source port ranges: **\***

    -  Destination: **Application security group**

    -  Destination application security group: **WebTier**

    -  Destination port ranges: **80**

    -  Protocol: **TCP**

    -  Action: **Allow**

    -  Priority: **150**

    -  Name: **AllowAnyWebTierInboundTCP80**

9.  On the **WGAppNSG1 - Inbound security rules** blade, select **Add**.

12. On the **Add inbound security rule** blade, enter the following information, and select **Add**:

    -  Source: **Service Tag**

    -  Source service tag: **VirtualNetwork**

    -  Source port ranges: **\***

    -  Destination: **Application security group**

    -  Destination application security group: **DataTier**

    -  Destination port ranges: **\***

    -  Protocol: **Any**

    -  Action: **Deny**

    -  Priority: **1000**

    -  Name: **DenyVNetDataTierInbound**

13. On the **WGAppNSG1 - Inbound security rules** blade, select **Add**.

14. On the **Add inbound security rule** blade, enter the following information, and select **Add**:

    -  Source: **Service Tag**

    -  Source service tag: **VirtualNetwork**

    -  Source port ranges: **\***

    -  Destination: **Application security group**

    -  Destination application security group: **WebTier**

    -  Destination port ranges: **\***

    -  Protocol: **Any**

    -  Action: **Deny**

    -  Priority: **1050**

    -  Name: **DenyVNetWebTierInbound**

15. On the **WGAppNSG1 - Inbound security rules** blade, select **Subnets** under **Settings** and then select **+ Associate**.

16. On the **Associate subnet** blade, select **WGVNet2** on the **Virtual network** drop down **DataSubnet** and **AppSubnet** on the **Subnet** dropdown.

17. Select **OK** at the bottom of the **Associate subnet** blade.

1. Within the computer, start Browser and navigate to **IP Public address** of the **WGWEB1**.

1. Examine the navegate on Application was successful.

## Lab 7 - Configure Azure VNET Peering (15 minutes)

1. In the Azure portal, search for and select **Virtual networks**.

1. In the list of virtual networks, click **WGVNet2**.

1. On the **WGVNet2** virtual network blade, in the **Settings** section, click **Peerings** and then click **+ Add**.

1. Specify the following settings (leave others with their default values) and click **Add**:

    | Setting | Value|
    | --- | --- |
    | This virtual network: Peering link name | **WGVNet2-to-WGVNet1** |
    | This virtual network: Traffic to remote virtual network | **Allow (default)** |
    | This virtual network: Traffic forwarded from remote virtual network | **Allow (default)** |
    | Virtual network gateway | **None** |
    | Remote virtual network: Peering link name | **WGVNet1-to-WGVNet2** |    
    | Virtual network deployment model | **Resource manager** |
    | I know my resource ID | unselected |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Virtual network | **WGVNet1** |
    | Traffic to remote virtual network | **Allow (default)** |
    | Traffic forwarded from remote virtual network | **Allow (default)** |
    | Virtual network gateway | **None** |

1. In the Azure portal, search for **Private DNS Zones** and select **woodgrove.corp**

1. Click **Add** and create a new **Virtual networking links**, create a new **WGVNet2VNL** and select enable **Auto registration** on the **WGVNet2**.

1. At the top of the Azure portal, enter the name of a **WGVM1** that is in the running state, in the search box. When the name of the VM appears in the search results, select it.

1. Under Settings on the left, select **Networking**, and navigate to the network interface resource by selecting its name. View network interfaces.

1. On the left, select **Effective routes**. The effective routes for a network interface are shown.
    
1. Connect Virtual Machine on the **WGVM1**, in the session RDP.

1. Start **Windows PowerShell** and, in the **Administrator: Windows PowerShell** and run the following to test connectivity to **WGWEB1**.

   ```powershell
   install-windowsfeature telnet-client
   telnet WGWEB1.woodgrove.corp 80
   ```
    >**Note**: The test uses TCP 80 since this is this port is allowed by default by operating system firewall. 

1. Examine the output of the command and verify that the connection was successful.

## Lab 8 - Deploy Azure Bastion (20 minutes)

1.  In the Azure portal, select **+ Create a resource** then select **Bastion**. In the search results, select the Bastion service with Microsoft as the publisher.

2.  On the **Create a Bastion** blade, on the **Basics** tab, enter the following information, and select **Review + Create**:

    -  Subscription: **Select your subscription**.

    -  Resource group: Select **WGVnetRG1**.

    -  Name: **AzureBastion**

    -  Region: This must match the location in which you created the **WGVNet1** virtual network.

    -  Tier: **Basic**

    -  Virtual network: **WGVNet1**

    -  Subnet: **AzureBastionSubnet** Note: After creation, assign (10.7.5.0/24) as the subnet address..

    -  Public IP: **Create New**

    -  Public IP address name: **BastionPublicIP**

3.  On the **Create a Bastion** blade, on the **Review + Create** tab, ensure the validation passes, and select **Create**. The Bastion host will take about 15 minutes to provision.

5.  In the Azure portal, navigate to the Network Security groups, navegate on the **WGAppNSG1**, select **Inbound security rules** under **Settings** on the left and select **Add** rules allow RDP/SSH for source **Virtual Network** on TAG and destionation **WebTier and DataTier** on Application Security groups.

1. In the Azure portal, navigate to the **WGWEB1** VM and initiate a Bastion connection session to the WGWEB1 virtual machine by selecting Connect and Bastion, enter the following information:

    - User name: **demouser**

    - Password **demo@pass123**

## Lab 9 - Deploy Azure Load Balacer with High Availability (20 minutes)

1. In the Azure portal, from the home page navigation, select **Load balancers**, then select **+ Create**.

2. On the **Create load balancer** blade, on the **Basics** tab, enter the following values:

    - Subscription: **Select your subscription**.

    - Resource group: **WGVNetRG2**

    - Name: **WGWEBLB**

    - Region: **Select same region on application**

    - SKU: **Standard**

    - Type: **Public**

    - Tier: **Regional**

    Ensure your **Create load balancer** dialog looks like the following, and select **Next: Frontend IP configuration** then select **Create**.

3. On the **Frontend IP configuration** tile, select **+ Add a frontend IP configuration** and enter the following values:

    - Name: **WGWEBLBFEIP**

    - Public IP address: **Create new | Name: WGWEBLBPublicIP**

    Ensure your **Create load balancer - Frontend IP configuration** dialog looks like the following, and select **Add**.

2. On the **Backend pools**, and select **+Add** at the beginning.

3. Enter **WGWEBLBE** for the pool name. Select **NIC** for **Backend Pool Configuration**. Under **IP Configurations**, select **+ Add**.

4. Under **Virtual machine**, select **+ Add** and choose the **WGWEB1** and **WGWEB2** virtual machines and select **Add**.

    >**Note**: If you do not see WGWEB1 in the Virtual Machine selection list, the public IP address was not created as a Standard SKU.  Locate **webip** and in the **Overview** tile, select the **Upgrade to Standard SKU** banner to change the SKU.  You will need to change the IP to **Static** in the **Configuration** and temporarily **Disassociate** it from **WGWEB1NetworkInterface**. Once upgraded, **Associate** webip with the **Network Interface** for WGWEB1.

5. Select **Save** at the bottom of the **Add backend pool** blade to add the backend pool.

6. Wait to proceed until the Backend pool configuration is finished updating. When the backends are added, they should look like the following image.

7. Next, under **Settings** on the WGWEBLB Load Balancer blade, select **Health Probes**. Select **+ Add**, and use the following information to create a health probe.

    - Name: **HTTP**

    - Protocol: **HTTP**

8. Select **Add**.

9. After the Health probe has been added, select **Load balancing rules** from the left navigation. Select **+ Add** and complete the configuration as shown below followed by selecting **Add**.

    - Name: **HTTP**

    - Frontend IP address: Select the load balancer IP entry with **10.8.0.100**.

    - Backend pool: **LBBE**

    - Port: **80**

    - Backend port: **80**

    - Health probe: **HTTP**

10. Open Microsoft Edge from the Start menu and navigate to <http://LoadBalancerPublicIPaddress>. Ensure that you successfully connect to either one of the two Web servers.

11. Using the portal, disassociate the public IP from the NIC of **WGWEB1** VM. Do this by navigating to the VM and selecting **Networking** under **Settings** on the left. Select the **NIC Public IP** then choose **Dissociate**. Select **Yes** when prompted.

12. Next, return to the **WGWEB1 - Networking** blade and select the **Network Interface**.

13. Select **IP configurations** under **Settings** on the left.

14. Next, select **ipconfig1** shown above.

15. Select and make sure that the **Public IP address settings** is shown as **Dissociate**, and select **Save** if necessary. This should remove the public IP address from the network interface of the VM.

## Project - Infrastructure as Service (IaaS), Networking and High Availability
<br></br>

   ![Screenshot of the Hub-spoke](/AllFiles/Images/Hub-Spoke.png)

**Important Notes**
- Three Virtual Networks;
- Application running on the Spoke Production network;
- VNET Peering connection Hub and Spokes;
- Azure Bastion on the Hub network;
- Azure Firewall on the Hub network;
- Load Balancer on the Spoke Production network;
- Custom Route tables to address prefix "Address space Spokes" and next hop type to virtual applicance IP address Azure Firewall;
- Create Network rule Azure Firewall allow all traffic.

References: [Hub-spoke network topology](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)

## After the Hands-on lab 

1. Delete all Azure resources created in support of this Hands-on lab.

1. End of **Day 1**
