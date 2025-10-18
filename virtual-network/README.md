# 🧭 Step-by-Step: Create a Virtual Network in Azure Portal
Here’s a clear, step-by-step guide to setting up a Virtual Network (VNet) using the Microsoft Azure portal (dashboard) — no CLI needed.

## Part 1: Sign in to Azure Portal
- Go to https://portal.azure.com
- Log in with your Azure account credentials.

---

## Part 2: Create a Resource Group
- From the Home dashboard → search for `Resource groups`
- Click Resource groups → then click Create.
- Set the following parameter:
  * Subscription → Choose your Azure subscription.
  * Resource group name → Enter a name for your resource group  (e.g., blog-stagging).
  * Region → Choose the Azure region closest to your users (e.g., East US, West Europe).
  <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/67b9ca31-4d04-44a9-abda-bf38b836f9e1" />
- click Review + Create → then Create

---

## Part 3: Create the Virtual networks
- Search for `Virtual networks`
- Click Virtual networks → then click Create.

### Step 1: Configure the Basics
- Under the Basics tab:
  * Subscription → Choose your Azure subscription.
  * Resource Group → Select an existing group (e.g., blog-stagging).
  * Name → Enter a name for your VNet (e.g., blog-stagging-vnet).
  * Region → Choose the Azure region closest to your users or workloads (e.g., East US, West Europe).
  <img width="1366" height="615" alt="image" src="https://github.com/user-attachments/assets/2574db86-d6ca-48d8-beb6-919ce52c00d6" />
- Click Next: Security.

### Step 2: Configure Security (Optional)
- BastionHost, DDoS protection, and Firewall are optional.
  * Leave them Disabled for a simple setup.
  * You can enable them later if you need secure remote access or network protection.
- Click Next: IP Addresses

### Step 3: Set IP Address Space
- Under the IP Addresses tab:
  * IPv4 address space → Specify a CIDR block, e.g.:
  ```
  10.0.0.0/16
  ```
  * You can add or edit address spaces later if needed.
  <img width="1366" height="617" alt="image" src="https://github.com/user-attachments/assets/75659bac-c91a-4dbd-9cf6-f582c44f72b3" />
- Click Add subnet and configure:
  * Subnet name: e.g., frontend-subnet
  * Subnet address range: e.g., 10.0.1.0/24
- Click Add once done.
  <img width="1366" height="615" alt="image" src="https://github.com/user-attachments/assets/5c4055aa-d49c-4c89-99cf-4e5340ebc664" />
- Click Add subnet to create a private subnet for backend or database
  * Subnet name: e.g., backend-subnet
  * Subnet address range: e.g., 10.0.2.0/24
  * Enable private subnet (no default outbound access): Enable
- Click Add once done.
  <img width="1366" height="618" alt="image" src="https://github.com/user-attachments/assets/d5d4aa33-c9de-400b-9ea8-92f702f58b6e" />
- You can create more subnets later (e.g., db-subnet).
- Click Next: Tags.

### Step 4: Add Tags (Optional)
- Tags help with resource organization, e.g.:
  ```
  Environment: Stagging
  Owner: PeterO
  ```
  <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/03c8c8d6-f6c3-43f3-a357-25157e37a401" />
- Click Next: Review + create.

### Step 5: Review and Create
- Review your configuration summary.
- Ensure the address space and subnet details are correct.
- Click Create to deploy the Virtual Network.
Azure will provision it within seconds.

---

## Part 4: Create Route Tables for each subnet
- Search for `Route tables`
- Click Route tables → then click Create.
  * Subscription → Choose your Azure subscription.
  * Resource Group → Select an existing group (e.g., blog-stagging).
  * Region → Choose the Azure region closest to your users or workloads (e.g., East US, West Europe).
  * Name → Enter a name for your route table (e.g., blog-stagging-public-rtb).
  * Propagate gateway routes → Yes
  <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/129e431f-8c8f-4258-a2e7-ba48407d56f0" />
- Click Review + create → then Create.
- Create another route table for your private subnet (e.g., blog-stagging-private-rtb).

---

## Part 5: Associate the Route table with a subnet
- Search for `Route tables` → Click Route tables
  1. Select `blog-stagging-public-rtb`.
  2. Expand Settings → Click Subnets → Click Associate
     * Virtual network → Select an existing virtual network (e.g., blog-stagging-vnet).
     * Subnet → Select an existing Subnet to associate (e.g., frontend-subnet).
     <img width="1366" height="614" alt="image" src="https://github.com/user-attachments/assets/4c06dc65-929b-40c5-8e0e-a52e86b2c92b" />
  3. Click Ok
  ---
  4. Select `blog-stagging-private-rtb`.
  5. Expand Settings → Click Subnets → Click Associate
     * Virtual network → Select an existing virtual network (e.g., blog-stagging-vnet).
     * Subnet → Select an existing Subnet to associate (e.g., backend-subnet, db-subnet).
     <img width="1366" height="614" alt="image" src="https://github.com/user-attachments/assets/5710388f-2aa1-4ae1-8ab7-2831beff87e0" />
  6. Click Ok

---

## Part 6: Create a NAT Gateway for private subnet
- In the Azure Portal, search for `NAT Gateway`.
- Click NAT Gateway → then Create.

### Step 1: Configure the Basics
- Under the Basics tab:
  * Resource group: blog-stagging
  * Name: blog-stagging-nat-gw
  * Region: West Europe
  * TCP idle timeout (minutes): 4
  <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/38d1a72e-b3f9-4a36-8ae5-8cf5719bb84e" />
- Click Next: Outbound IP.

### Step 2: Assign a Public IP
You need at least one public IP address for the NAT Gateway (it hides your private VMs behind it).
- Click Create new under `Public IP addresses`.
- Enter:
  * Name: blog-stagging-nat-ip
  * SKU: Standard
  * Assignment: Static
  <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/bed7dff7-9e2d-4a31-b83f-947384adcf6d" />
- Click OK → then Next: Subnet.

### Step 3: Associate the NAT Gateway with Your Private Subnet
- Choose:
  * Virtual network: blog-stagging-vnet
  * Subnet: backend-subnet
  <img width="985" height="614" alt="image" src="https://github.com/user-attachments/assets/f1519298-ce56-4b6a-a446-c36b04c9bf0a" />
- Click Next: Tags → Review + create → Create.

---

## Part 7: Create a Network Security Group (NSG)
An NSG controls inbound and outbound traffic to resources in your subnet or VM.
- Search `Network security groups`.
- click Network security groups → Create.

### Step 1: Create NSG for Public Subnet
- Under Basics:
  * Resource group: blog-stagging
  * Name: blog-stagging-nsg-public
  * Region: West Europe
  <img width="1028" height="617" alt="image" src="https://github.com/user-attachments/assets/716461b6-dde4-4e68-8c0a-cf6c9e4bc99f" />
- Click Review + create → Create.

### Step 2: Add Security Rules to the blog-stagging-nsg-public
- Open the `blog-stagging-nsg-public` resource.
- Go to Inbound security rules → Add.
- Add a rule for SSH (Linux) or RDP (Windows) access:
  * Source: My IP address
  * Source port ranges: *
  * Destination: Any
  * Service: SSH or RDP
  * Destination port ranges: 22 (for SSH) or 3389 (for RDP)
  * Protocol: TCP
  * Action: Allow
  * Priority: 100
  * Name: Allow-SSH or Allow-RDP
  <img width="1366" height="615" alt="image" src="https://github.com/user-attachments/assets/5a29c7d5-8224-4c6c-9f68-78ffc91d5081" />
  * Click Add
- Add another rule for HTTP/HTTPS if needed:
  * Source: Any
  * Service: HTTP or HTTPS
  * Port 80 or 443
  * Action: Allow
  * Priority: 110
  * Name: Allow-HTTP or Allow-HTTPS
  <img width="1366" height="615" alt="image" src="https://github.com/user-attachments/assets/aefd947d-ccaa-4dbf-889d-f0cc9c216882" />
  * Click Add
 
### Step 3: Associate the blog-stagging-nsg-public with the Public Subnet
- In the `blog-stagging-nsg-public` page, select Subnets → Associate.
- Choose:
  * Virtual network: blog-stagging-vnet
  * Subnet: frontend-subnet
  <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/23df101b-ae48-4294-8697-9f550cc4e363" />
- Click OK.

---

✅ Summary: Updated Network Architecture
|  Component	|            Subnet  	      |         Internet Access	        |Notes
|  VM-Public	|        Public-Subnet      |     ✅ Inbound + Outbound  	    |Public IP + NSG
|  VM-Private	|       Private-Subnet      |      ✅ Outbound only	          |Through NAT Gateway
| NAT Gateway |Attached to Private-Subnet	|Provides secure outbound access	|No inbound exposure
|  NSG-Public	|        Public-Subnet	    |      Controls traffic	          |Allows SSH/RDP/HTTP

💬 Bottom line:
- If your private VMs ever need to reach the internet (updates, external APIs, etc.), create a NAT Gateway.
- If not — and they’re isolated database or backend systems — you can skip it.
