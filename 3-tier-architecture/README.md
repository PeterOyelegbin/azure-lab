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
 
### Configure Network Security
- Create Network Security Groups:
  * Bastion-NSG: Allow SSH (port 22) from your IP only
  * Web-NSG: Allow HTTP (80), HTTPS (443) from the internet, SSH from Bastion-NSG
  * App-NSG: Allow traffic from Web-NSG on required ports, SSH from Bastion-NSG
  * DB-NSG: Allow MySQL/MariaDB (3306) from App-NSG only

### Create Public IP and Load Balancer
- Allocate a Public IP address
- Create Application Gateway (or Load Balancer) for web tier
- Configure NAT Gateway for outbound internet access from private subnets

### Create Bastion Host (Azure Bastion Service)
- Deploy Azure Bastion service in the public subnet
- Or create a jumpbox VM:
  * OS: Ubuntu 20.04 LTS or CentOS
  * Size: Standard_B1s
  * Virtual Network: Your VNet, Public Subnet
  * NSG: Bastion-NSG
  * Public IP: Enabled

### Create Web Servers
- Create a Virtual Machine Scale Set or multiple VMs in an Availability Set
  * OS: Ubuntu 20.04 LTS or CentOS
  * Size: Standard_B1s
  * Virtual Network: Your VNet, Public Subnet
  * NSG: Web-NSG
  * Custom data (cloud-init for Ubuntu):
  ```bash
  #!/bin/bash
  apt-get update
  apt-get install -y apache2
  systemctl start apache2
  systemctl enable apache2
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

### Create Database Tier
- Create an Azure Database for a MariaDB server
  * Server name: [your-db-server-name]
  * Version: MariaDB 10.3
  * Compute: Basic tier, 1 vCore
  * Storage: 5GB
  * Backup: Disabled (for lab purposes)
  * Virtual Network: Configure VNet integration with private data subnet
  * Connection security: Allow access from Azure services
  * Admin credentials:
    - Username: root
    - Password: Re:Start!9
  * Database name: mydb

### Configure Routing and Connectivity
- Create a Route Table for private subnets with a NAT Gateway route
- Associate route tables with appropriate subnets
- Configure Application Gateway to route traffic to web servers
- Set up proper peering or service endpoints if needed

### High Availability Configuration
- Deploy web and app servers across multiple Availability Zones
- Configure load balancing between tiers
- Set up a database with high availability options if using the premium tier

### Upload SSH keys to Bastion Host
- Windows users
  * Go to your command prompt and type out
    - Pscp -scp -P 22 -i ’.\Downloads\labsuser.ppk’ -l user ec2-user‘.\Downloads\labsuser.pem’ ec2-user@bastion-host-public-ip:/home/ec2-user
    - Replace labsuser.ppk and labsuser.pem with what you named your keys
    - Replace bastion-host-public-ip with your bastion host public IP address
- Mac and Linux Users
  * Go to your terminal and type out
    - Chmod 400 labuser.pem
    - scp -i ’.\Downloads\labsuser.pem’ -l user ec2-user’.\Downloads\labsuser.pem’ ec2-user@bastion-host-public-ip:/home/ec2-user
    - Replace labsuser.ppk and labsuser.pem with what you named your keys
    - Replace bastion-host-public-ip with your bastion host public IP address

### Test to make sure the key was uploaded into the Bastion Host
- SSH into Bastion Host
- Type ls
- Labsuser.pem should show up

### Connect to App Server
- From SSH inside the Bastion Host, use the terminal
- Type out chmod 400 labsuser.pem to change file permissions
- Type out ssh -i my-key-pair.pem ec2-user@app-server-private-ip
- Replace my-key-pair with the name of your key
- Replace app-server-private-ip with your app server’s private IP address
- Test out pinging the web server by typing out ping and the private IP address
- Test out connecting to the database by typing out mysql –user=root -password='Re:Start!9' –host=database-server-endpoint
- Replace database-server-endpoint with the database server endpoint
- Type show databases; to see your database from the app server

---

## Important Notes
- Azure Bastion provides secure RDP/SSH connectivity without public IPs on VMs
- Use Availability Sets or Availability Zones for high availability
- Azure Database services include built-in high availability options
- Network Security Groups in Azure are stateful by default
- Consider using Azure Application Gateway for layer 7 load balancing with web application firewall
