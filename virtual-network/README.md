Here’s a clear, step-by-step guide to setting up a Virtual Network (VNet) using the Microsoft Azure portal (dashboard) — no CLI needed.

## 🧭 Step-by-Step: Create a Virtual Network in Azure Portal
### Step 1: Sign in to Azure Portal
- Go to https://portal.azure.com
- Log in with your Azure account credentials.

### Step 2: Create a Resource Group
- From the Home dashboard → search for `Resource groups`
- Click Resource groups → then click Create.
- Set the following parameter;
  ```shell
  Subscription - Select your subscription
  Resource group name - project_name-environment  # e.g blog-stagging
  Region - Select your preferred region
  ```
  <img width="1366" height="616" alt="image" src="https://github.com/user-attachments/assets/67b9ca31-4d04-44a9-abda-bf38b836f9e1" />
- Review + Create → Create

### Step 3: Create the Virtual networks
- Search for `Virtual networks`
- Click Virtual networks → then click Create.
- Set values as shown below;
- 
