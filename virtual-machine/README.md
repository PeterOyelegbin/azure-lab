# 🖥️ Azure Virtual Machine Setup in Public and Private Subnets
Here’s a clear, step-by-step guide to setting up a Virtual Machine (VM) using the Microsoft Azure portal (dashboard) — no CLI needed.

## Step 1: Create a Virtual Machine in the Public Subnet
- In the Azure portal, click Create a resource → Virtual Machine.
- Under Basics:
  * Resource group: blog-stagging
  * Name: blog-fe-vm
  * Region: West Europe
  * Image: Choose Ubuntu 24.04 LTS
  * Size: Select any small VM (e.g., Standard_B1s)
  * Authentication type: SSH public key
  <img width="1098" height="616" alt="image" src="https://github.com/user-attachments/assets/0ffd3a10-2519-4752-acc1-86cf11eab4dd" />
- Under Networking:
  * Virtual network: blog-stagging-vnet
  * Subnet: frontend-subnet
  * Public IP: Create new
  * NIC network security group: Select Advanced
  * Select existing NSG: Choose `blog-stagging-nsg-public`
  <img width="1132" height="617" alt="image" src="https://github.com/user-attachments/assets/71a80258-3575-47e2-bcc5-474b5a2a47ed" />
- Click Review + create → Create.

## Step 2: Create a Virtual Machine in the Private Subnet (No Public IP)
- Go back to `Virtual Machines` → Create.
- Under Basics:
  * Resource group: blog-stagging
  * Name: blog-be-vm
  * Region: West Europe
  * Image: Choose Ubuntu 24.04 LTS
  * Size: Select any small VM (e.g., Standard_B1s)
  * Authentication type: SSH public key
  <img width="1135" height="617" alt="image" src="https://github.com/user-attachments/assets/23d16cc3-bd94-4162-9ebe-f499c143c4d4" />
- Under Networking:
  * Virtual network: blog-stagging-vnet
  * Subnet: backend-subnet
  * Public IP: None
  * NIC network security group: None (or create a private NSG if needed)
  <img width="1135" height="614" alt="image" src="https://github.com/user-attachments/assets/bfd7b866-be1e-4f25-918f-6df2b143d859" />
- Click Review + create → Create.

## Step 3: Test the Network
- Connect to the Public VM by SSH using:
  ```
  ssh -i <private-key-file-path> <username>@<public-ip>
  ```
  <img width="962" height="541" alt="image" src="https://github.com/user-attachments/assets/cf70f6db-74fe-4da7-9e75-df34a89a94b9" />
- Connect to the Public VM via the browser using:
  ```
  http://<public-ip>
  ```
  <img width="1362" height="696" alt="image" src="https://github.com/user-attachments/assets/65b37a3f-b0c3-4a59-a423-2e249c33e902" />
- From the Public VM, test private connectivity:
  ```
  ping -c5 <private-ip>
  ```
  (replace with the Private VM’s private IP)
  <img width="1362" height="270" alt="image" src="https://github.com/user-attachments/assets/7933ae39-08b1-4f23-9464-b49818c75794" />
If the ping works, your public and private subnets are communicating correctly.

## ✅ Result
You now have a fully functional Azure Virtual Machine, ready to:
- 🌐 Access the VM in the public subnet via SSH and HTTP
- 🔐 Access the VM in the private subnet via the public VM only
- 👮🚷 No HTTP/web access to the VM in the private subnet
