# 3-tier Architecture (Project 1)

## Scenario
Design and configure a highly available 3-tier Architecture on Azure
* Tier 1 - User/Presentation Tier
* Tier 2 - Application Tier
* Tier 3 - Data Tier

---

## Step by Step
### Create Virtual Network and Subnets
- Create a Virtual Network with address space (e.g., 10.0.0.0/16)
- Create 4 subnets:
  * Public Subnet (e.g., 10.0.1.0/24) - for Bastion and Web Servers
  * Private App Subnet 1 (e.g., 10.0.2.0/24) - Application Tier AZ 1
  * Private App Subnet 2 (e.g., 10.0.3.0/24) - Application Tier AZ 2
  * Private Data Subnet (e.g., 10.0.4.0/24) - Database Tier
  <img width="1366" height="497" alt="image" src="https://github.com/user-attachments/assets/ce6c8559-8f97-4728-9c83-459419a854da" />

### Configure Network Security
- Create Network Security Groups:
  * Bastion-NSG: Allow SSH (port 22) from your IP only
  * Web-NSG: Allow HTTP (80), HTTPS (443) from the internet, SSH from Bastion-NSG
  * App-NSG: Allow traffic from Web-NSG on required ports, SSH from Bastion-NSG
  * DB-NSG: Allow MySQL/MariaDB (3306) from App-NSG only
  <img width="1366" height="466" alt="image" src="https://github.com/user-attachments/assets/efcc8c65-3370-4114-90b0-c8ab78dcb176" />

### Configure NAT Gateway 
- Allocate a Public IP address
- Configure NAT Gateway for outbound internet access from private subnets
  <img width="1366" height="383" alt="image" src="https://github.com/user-attachments/assets/a7dfc3ec-e771-4e18-98cb-df4e14f30c34" />

### Create Bastion Host (Azure Bastion Service)
- Deploy Azure Bastion service in the public subnet
- Or create a jumpbox VM:
  * OS: Ubuntu 20.04 LTS or CentOS
  * Size: Standard_B1s
  * Virtual Network: Your VNet, Public Subnet
  * NSG: Bastion-NSG
  * Public IP: Enabled
  <img width="1364" height="614" alt="image" src="https://github.com/user-attachments/assets/281ab6f3-e155-4737-896f-25112d909977" />

### Create Web Servers
- Create a Virtual Machine Scale Set or multiple VMs in an Availability Set
  * OS: Ubuntu 20.04 LTS or CentOS
  * Size: Standard_B1s
  * Virtual Network: Your VNet, Public Subnet
  * NSG: Web-NSG
  * Custom data (cloud-init for Ubuntu):
    ```bash
    #!/bin/bash
    sudo apt-get update
    sudo apt-get install -y apache2
    sudo systemctl start apache2
    sudo systemctl enable apache2
    ```
  * or Custom data (cloud-init for CentOS):
    ```bash
    #!/bin/bash
    sudo yum update -y
    sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
    sudo yum install -y httpd
    sudo systemctl start httpd
    sudo systemctl enable httpd
    ```
  <img width="1366" height="615" alt="image" src="https://github.com/user-attachments/assets/de95956a-00f5-4552-845c-23e775bad2a5" />

### Create Application Servers
- Create a Virtual Machine Scale Set or multiple VMs in an Availability Set
  * OS: Ubuntu 20.04 LTS or CentOS
  * Size: Standard_B1s
  * Virtual Network: Your VNet, Private App Subnets (spread across AZs)
  * NSG: App-NSG
  * Custom data (cloud-init for Ubuntu):
    ```bash
    #!/bin/bash
    apt-get update
    apt-get install -y mariadb-server
    systemctl start mariadb
    systemctl enable mariadb
    ```
  * Custom data (cloud-init for CentOS):
    ```bash
    #!/bin/bash
    sudo yum install -y mariadb-server
    Sudo service mariadb start
    ```
  <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/cf4b5b5b-3efb-48ce-8950-4e3bcd641a04" />

### Create Database Tier
- Create an Azure Database for MySQL server
  * Server name: [your-db-server-name]
  * Version: MySQL 8.0
  * Workload type: Dev/Test
  * High availability: Disabled (for lab purposes)
  * Admin credentials:
    - Username: AzureUsr
    - Password: Re:Start!9
  * Connectivity method: Private access (VNet Integration)
  * Virtual Network: Configure VNet integration with private data subnet
    <img width="1366" height="615" alt="image" src="https://github.com/user-attachments/assets/d7933641-ac98-4968-b06a-497c1997394d" />

---

## Test Setup and Connection
### Upload SSH keys to Bastion Host
- Windows users
  * Go to your command prompt and type out
    - Pscp -scp -P 22 -i '.\Downloads\Lab-SSH-Key.pem' '.\Downloads\Lab-SSH-Key.pem' azureuser@bastion-host-public-ip:/home/azureuser/
    - Replace `Lab-SSH-Key.pem` with what you named your keys
    - Replace `bastion-host-public-ip` with your bastion host public IP address
- Mac and Linux Users
  * Go to your terminal and type out
    - Chmod 400 Lab-SSH-Key.pem
    - scp -i ~/Downloads/Lab-SSH-Key.pem ~/Downloads/Lab-SSH-Key.pem azureuser@bastion-host-public-ip:/home/azureuser/
    - Replace `Lab-SSH-Key.pem` with what you named your keys
    - Replace `bastion-host-public-ip` with your bastion host public IP address

### Test to make sure the key was uploaded into the Bastion Host
- SSH into Bastion Host
- Type ls
- Lab-SSH-Key.pem should show up
  <img width="1366" height="596" alt="image" src="https://github.com/user-attachments/assets/d927d107-c853-443a-be0b-13a28f9b077e" />

### Connect to App Server
- From SSH inside the Bastion Host, use the terminal
- Test out pinging the app server by typing out `ping -c5 <app-server-private-ip>`
  <img width="1366" height="228" alt="image" src="https://github.com/user-attachments/assets/0a2a7bfa-b5d6-4828-9540-dc66dec3b5fa" />
- Type out `chmod 400 Lab-SSH-Key.pem` to change file permissions
- Type out `ssh -i Lab-SSH-Key.pem azureuser@<app-server-private-ip>`
- Replace `Lab-SSH-Key.pem` with the name of your key
- Replace `app-server-private-ip` with your app server’s private IP address
- Test out connecting to the database by typing out `mysql --host=<database-server-endpoint> --port=3306 --user=AzureUsr --password='Re:Start!9'`
- Replace `database-server-endpoint` with the database server endpoint
- Type `show databases;` to see your database from the app server
  <img width="1366" height="229" alt="image" src="https://github.com/user-attachments/assets/f13e0b56-f7e3-4174-823d-0bf74de8134a" />

---

## Important Notes
- Azure Bastion provides secure RDP/SSH connectivity without public IPs on VMs
- Use Availability Sets or Availability Zones for high availability
- Azure Database services include built-in high availability options
- Network Security Groups in Azure are stateful by default
- Consider using Azure Application Gateway for layer 7 load balancing with a web application firewall
