# Azure Virtual Network (VNet) Peering (Project 4)
## Scenario
How to Connect Instances in Different Virtual Networks (VNets) Using VNet Peering**

## Instructions
### **1. Create two separate Virtual Networks (VNets)**
*   **VNet A:** `vnet-a` (e.g., Address Space: `10.1.0.0/16`)
*   **VNet B:** `vnet-b` (e.g., Address Space: `10.2.0.0/16`)

**Azure Steps:**
1.  Navigate to the Azure Portal.
2.  Search for and select "Virtual networks".
3.  Click **"+ Create"**.
4.  For **VNet A**:
    *   **Resource group:** `rg-network-project` (Create new)
    *   **Name:** `vnet-a`
    *   **Region:** `(Your preferred region, e.g., East US)`
    *   **IPv4 address space:** `10.1.0.0/16`
5.  Click **"Review + create"**, then **"Create"**.
6.  Repeat steps 3-5 for **VNet B**:
    *   **Resource group:** `rg-network-project` (Use the same one)
    *   **Name:** `vnet-b`
    *   **Region:** `(Must be the same region as VNet A for standard peering)`
    *   **IPv4 address space:** `10.2.0.0/16`

---

### **2. Create Subnets within each VNet**
*   In **VNet A**, create a **Private Subnet**: `subnet-private-a` (e.g., `10.1.1.0/24`)
*   In **VNet B**, create a **Public Subnet**: `subnet-public-b` (e.g., `10.2.1.0/24`)

**Azure Steps:**
1.  Go to your created **VNet A** -> **Subnets** -> **"+ Subnet"**.
    *   **Name:** `subnet-private-a`
    *   **Subnet address range:** `10.1.1.0/24`
    *   Leave all other settings default. Click **"Save"**.
2.  Go to your created **VNet B** -> **Subnets** -> **"+ Subnet"**.
    *   **Name:** `subnet-public-b`
    *   **Subnet address range:** `10.2.1.0/24`
    *   Leave all other settings default. Click **"Save"**.

---

### **3. Create a Virtual Network Gateway (VPN Gateway) and attach to VNet B**
*(In Azure, a Bastion Host is a separate, managed service. For a more direct equivalent to an EC2 Bastion with an IGW, we create a Public IP and a Network Security Group for the VM in VNet B. However, if you specifically need a gateway for other scenarios, you would create a VPN Gateway).*

**For this scenario, we will simply create a Public IP for our Bastion VM later. A Virtual Network Gateway is not required for basic VNet Peering connectivity.**

---

### **4. Create Route Tables**
*   **Route Table A** associated with VNet A's subnet.
*   **Route Table B** associated with VNet B's subnet.

**Azure Steps:**
1.  Search for and select "Route tables".
2.  Click **"+ Create"**.
3.  Create **Route Table A**:
    *   **Resource group:** `rg-network-project`
    *   **Region:** `(Same as your VNets)`
    *   **Name:** `rt-vnet-a`
    *   **Propagate gateway routes:** `Yes` (This is default and fine for our case).
4.  Click **"Review + create"**, then **"Create"**.
5.  Repeat steps 2-4 for **Route Table B**:
    *   **Name:** `rt-vnet-b`

---

### **5. Create Network Security Groups (NSGs)**
*   **NSG A** for resources in VNet A.
*   **NSG B** for the Bastion in VNet B.

**Azure Steps:**
1.  Search for and select "Network security groups".
2.  Click **"+ Create"**.
3.  Create **NSG A**:
    *   **Resource group:** `rg-network-project`
    *   **Name:** `nsg-vnet-a`
    *   **Region:** `(Same as your VNets)`
4.  Click **"Review + create"**, then **"Create"**.
5.  Repeat steps 2-4 for **NSG B**:
    *   **Name:** `nsg-vnet-b`

---

### **6. Launch Virtual Machines (VMs)**
*   **VM A:** A private VM in `vnet-a`, subnet `subnet-private-a`.
*   **VM B (Bastion):** A public VM in `vnet-b`, subnet `subnet-public-b`.

**Azure Steps:**
1.  Search for and select "Virtual machines".
2.  Click **"+ Create"** -> **"Azure virtual machine"**.
3.  Create **VM A**:
    *   **Resource group:** `rg-network-project`
    *   **Virtual machine name:** `vm-private-a`
    *   **Region:** `(Same as your VNets)`
    *   **Image:** `Ubuntu Server 20.04 LTS`
    *   **Size:** `Standard_B1s`
    *   **Administrator account:** SSH public key or password.
    *   **Inbound port rules:** `Allow selected ports` -> **SSH (22)**
    *   **Networking Tab:**
        *   **Virtual network:** `vnet-a`
        *   **Subnet:** `subnet-private-a`
        *   **Public IP:** `None` (This makes it private).
        *   **NIC network security group:** `Advanced`
        *   **Configure network security group:** `nsg-vnet-a`
4.  Click **"Review + create"**, then **"Create"**.
5.  Repeat steps 2-4 for **VM B (Bastion)**:
    *   **Virtual machine name:** `vm-bastion-b`
    *   **Networking Tab:**
        *   **Virtual network:** `vnet-b`
        *   **Subnet:** `subnet-public-b`
        *   **Public IP:** `(Create a new one, e.g., vm-bastion-b-pip)`
        *   **NIC network security group:** `Advanced`
        *   **Configure network security group:** `nsg-vnet-b`

---

### **7. Create VNet Peering between VNet A and VNet B**
You need to create **two peering connections**: one from VNet A to VNet B, and one from VNet B to VNet A.

**Azure Steps:**
1.  Go to **VNet A** -> **Peerings** -> **"+ Add"**.
    *   **Name (Link to remote VNet):** `vnet-a-to-vnet-b`
    *   **Virtual network deployment model:** `Resource manager`
    *   **Subscription:** `(Your subscription)`
    *   **Virtual network:** `vnet-b`
    *   **Name (Link from remote VNet):** `vnet-b-to-vnet-a`
    *   **Allow virtual network access:** `Enabled`
    *   **Allow forwarded traffic:** `Enabled` (Important for routing via the Bastion).
    *   **Allow gateway transit:** `Disabled` (We don't have a gateway).
    *   **Use remote gateways:** `Disabled`
2.  Click **"Add"**. This will automatically create both sides of the peering link.

---

### **8. Update Route Tables**
We need to tell each VNet how to reach the other VNet's address space via the peering link.

**Azure Steps:**
1.  Go to **Route Table A** (`rt-vnet-a`) -> **Routes** -> **"+ Add"**.
    *   **Route name:** `to-vnet-b`
    *   **Address prefix:** `10.2.0.0/16` (The entire address space of VNet B).
    *   **Next hop type:** `Virtual network peering`
    *   **Next hop address:** `vnet-a-to-vnet-b` (This will be auto-populated).
2.  Click **"Add"**.
3.  Now, **associate** this route table to the subnet in VNet A:
    *   Go to **Route Table A** -> **Subnets** -> **"+ Associate"**.
    *   **Virtual network:** `vnet-a`
    *   **Subnet:** `subnet-private-a`
    *   Click **"OK"**.
4.  Repeat for **Route Table B** (`rt-vnet-b`):
    *   Add a route named `to-vnet-a` with prefix `10.1.0.0/16` and next hop `vnet-b-to-vnet-a`.
    *   Associate this route table to the subnet `subnet-public-b` in VNet B.

---

### **9. Testing**
The goal is to SSH into the private VM A (`vm-private-a`) from the public Bastion VM B (`vm-bastion-b`).

**Azure Steps:**
1.  Get the **Private IP address** of `vm-private-a` from its Overview page (e.g., `10.1.1.4`).
2.  Get the **Public IP address** of `vm-bastion-b` from its Overview page.
3.  SSH into your **Bastion Host (VM B)**:
    ```bash
    ssh azureuser@<Public-IP-of-vm-bastion-b>
    ```
4.  From *inside* the Bastion host's SSH session, try to SSH into the **Private VM (VM A)** using its *private* IP:
    ```bash
    ssh azureuser@10.1.1.4
    ```
5.  If the connection is successful, your VNet Peering and routing are correctly configured!
