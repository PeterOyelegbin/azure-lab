# ⚖ Load Balancer in Private Subnet (Project 2)
## Scenario
How do I attach backend VMs with private IP addresses to my public-facing load balancer in Azure?

To attach Azure VMs located in a private subnet, you create a Public Load Balancer. The load balancer's frontend has a public IP address, while its backend pool contains the network interface cards (NICs) of the VMs in the private subnet.

---

## Architecture Diagram (Conceptual)

---

## Step-by-Step Implementation
1. Create a Virtual Network (VNet) with private and public subnets (in the same region)
2. Create a Network Security Group (NSG) with ports 80 and 22 open
3. Create a couple of VMs in the private subnet
4. Install the NGINX web server using the custom data (user data) option on both VMs
5. Create a Public Load Balancer and attach both VMs to its backend pool
6. Test

---

## Detailed Azure Steps
### Step 1: Create a Virtual Network with Public and Private Subnets
[Steps to create a Virtual Network](https://github.com/PeterOyelegbin/azure-lab/tree/main/virtual-network) Part 2 to 3.

### Step 2: Create a Network Security Group (NSG)
[Steps to create a Network Security Group](https://github.com/PeterOyelegbin/azure-lab/tree/main/virtual-network) Part 6

### Step 3: Create VMs in the Private Subnet
We will create VMs without public IPs, placing them directly in the private subnet.
1.  In the Azure portal, go to **Virtual machines**
2.  Click **Create** -> **Azure virtual machine**
3.  **Basics Tab:**
    *   **Resource group:** Select existing resources group
    *   **Virtual machine name:** `web-vm-1`
    *   **Region:** Same as your VNet
    *   **Image:** `Ubuntu Server 20.04 LTS`
    *   **Size:** Standard_B1s
    *   **Authentication type:** `SSH public key`
    *   **Username:** `azureuser`
    *   **SSH public key source:** Generate new key pair (or use existing)
4.  **Networking Tab (Crucial):**
    *   **Virtual network:** Select your VNet
    *   **Subnet:** `private-subnet`
    *   **Public IP:** `None`
    *   **NIC network security group:** `Advanced`
    *   **Configure network security group:** Select your NSG created earlier
5.  **Management Tab:**
    *   Under **Custom data**, paste the following script to install NGINX (Azure's equivalent of AWS User Data).
    ```bash
    #!/bin/bash
    sudo apt-get update
    sudo apt-get install -y nginx
    sudo systemctl start nginx
    sudo systemctl enable nginx
    # Create a custom homepage to identify the server
    echo "<h1>Hello from Web VM 1</h1>" | sudo tee /var/www/html/index.html
    ```
6.  Click **Review + create**, then **Create**. Download the private key if you generated a new one.
7.  **Repeat steps 2-6** to create a second VM named `web-vm-2`. In its custom data script, change the echo line to `echo "<h1>Hello from Web VM 2</h1>" | sudo tee /var/www/html/index.html`.

### Step 4: Create a Public Load Balancer and Attach the VMs
This is the Azure equivalent of an Internet-facing ELB.
1.  In the Azure portal, go to **Load balancers**
2.  Click **Create**
3.  **Basics Tab:**
    *   **Resource group:** Select existing resources group
    *   **Name:** `web-lb`
    *   **Region:** Same as your VNet
    *   **SKU:** `Standard`
    *   **Type:** `Public`
    *   **Tier:** `Regional`
4.  **Frontend IP configuration:**
    *   Click **Add a frontend IP configuration**
    *   **Name:** `lb-frontend`
    *   **IP version:** `IPv4`
    *   **IP type:** `IP address`
    *   **Public IP address:** Create new, name it `lb-public-ip`
5.  **Backend pools:**
    *   Click **Add a backend pool**
    *   **Name:** `web-backend-pool`
    *   **Virtual network:** Select your VNet
    *   Under **Virtual machines**, click **Add**
    *   Select both `web-vm-1` and `web-vm-2`
    *   For each, you need to select the correct IP configuration (e.g., `ipconfig1`). Click **Add**
6.  **Health probes:**
    *   Click **Add a health probe**
    *   **Name:** `http-80-probe`
    *   **Protocol:** `TCP`
    *   **Port:** `80`
    *   **Interval:** `5`
7.  **Load balancing rules:**
    *   Click **Add a load balancing rule**
    *   **Name:** `http-rule`
    *   **IP Version:** `IPv4`
    *   **Frontend IP address:** `lb-frontend`
    *   **Backend pool:** `web-backend-pool`
    *   **Protocol:** `TCP`
    *   **Port:** `80`
    *   **Backend port:** `80`
    *   **Health probe:** `http-80-probe`
    *   **Session persistence:** `None`
    *   **Idle timeout:** `4`
8.  Click **Review + create**, then **Create**

### Step 5: Test the Configuration
1.  After the deployment is complete, go to your **Load Balancer** resource `web-lb`
2.  In the **Overview** section, find the **Frontend public IP address** and copy it
3.  Open a web browser and paste the public IP address into the address bar
4.  You should see either "Hello from Web VM 1" or "Hello from Web VM 2"
5.  Refresh the page. You should see the response alternate between the two VMs, demonstrating that the load balancer is distributing traffic
