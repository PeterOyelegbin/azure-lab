# 🗄 Database and Cache Set up
In cloud environments, keeping your databases private (away from the public internet) is a key step in building secure and production-ready architectures.

In this guide, I’ll walk you through setting up PostgreSQL and Redis inside a private database subnet in Azure, using only the Azure Portal (no CLI, Terraform, or PowerShell).
By the end, you’ll have a fully private database layer, accessible only within your virtual network.

## 🧭 Overview
- Create a PostgreSQL Flexible Server in the private subnet
- Create a Redis Cache instance in the same private subnet
- Secure them using Private Endpoints
- Test connectivity from a VM in the public subnet

## 🗒 Pre-requisite
- Create a Virtual Network (VNet) with public, private, DB, and Redis subnets [link](https://github.com/PeterOyelegbin/azure-lab/tree/main/virtual-machine)

---

## 💾 Step 1: Deploy PostgreSQL Flexible Server (Private Access)
- Search `Azure Database for PostgreSQL flexible server` → Create
- Basics Tab:
  * Resource group: same as VNet
  * Server name: blog-stagging-postgres-db
  * Region: same as VNet
  * Workload type: Development or Production
  * Availability zone: any
  * Authentication method: PostgreSQL authentication only
  * Administrator login: Set username (e.g, blogAdmin)
  * Password: Set a strong password (e.g, Password!23)
  <img width="1366" height="615" alt="image" src="https://github.com/user-attachments/assets/c9410635-fc80-42e8-a5a9-11026b7c1744" />
  * Click Next
- Networking Tab:
  * Connectivity method: Private access (VNet Integration)
  * Virtual network: blog-stagging-vnet
  * Subnet: db-subnet
  <img width="1366" height="618" alt="image" src="https://github.com/user-attachments/assets/a374de75-de0b-4f55-9587-67ae4107717d" />
- Click Review + Create → Create

## ⚡ Step 2: Deploy Redis Cache in Private Subnet
- Search `Azure Cache for Redis` → Create
- Basics Tab:
  * Resource group: same as VNet
  * DNS name: blog-stagging-redis-cache
  * Region: same as VNet
  * Pricing tier: Select Basic, Standard, or Premium
  <img width="1366" height="617" alt="image" src="https://github.com/user-attachments/assets/2ae880d1-2909-4519-adf2-035bad4fcb9b" />
  * Click Next
- Networking Tab:
  * Connectivity method: Private Endpoint
  * Click `Add Private Endpoint`
    - Resource group: same as VNet
    - Name: blog-stagging-redis-private-endpoint
    - Virtual network: blog-stagging-vnet
    - Subnet: redis-subnet
    - Integrate with private DNS zone: Yes
    <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/140a7e17-21bd-459e-a93b-04f415e4c2f6" />
  * Click Next
- Advanced Tab:
  * Microsoft Entra Authentication: Enable
  * Access Keys Authentication: Enable
  <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/3106b2ca-f7ca-4241-a20d-3296c183c9bc" />
- Review + Create → Create

## 🛡️ Step 3: Verify Private Access Configuration
After both databases are deployed:
- Go to PostgreSQL → Networking
  * Ensure Private access is enabled and shows your subnet.
  * Public access should be disabled.
- Go to Redis → Private Endpoint connections
  * Confirm `Approved` status.
  * Note the private IP address assigned (e.g., 10.0.2.4).

## 🧩 Step 4: Test Connectivity from a Public Subnet VM
- SSH into the VM in the public subnet:
  ```
  ssh -i <private-key-file-path> <username>@<public_ip_of_vm>
  ```
- Install required tools:
  ```
  sudo apt update
  sudo apt install postgresql-client redis-tools -y
  ```
- Test PostgreSQL connection:
  ```
  psql "host=<endpoint> dbname=postgres user=<username> password=<password> sslmode=require"
  ```
- Test Redis connection:
  ```
  redis-cli -h <host-name> -p 6380 --tls
  ```
  <img width="1366" height="120" alt="image" src="https://github.com/user-attachments/assets/4b8b83a3-8960-4926-b5e4-947a8a63991e" />
✅ If both connect successfully — your setup is working entirely within the private subnet!

---

## 🎯 The Result
You now have a private data layer that’s secure, scalable, and cloud-native — ready for any application tier to connect within your network.
