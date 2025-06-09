# Configuring an **Azure Load Balancer** step by step.  

## **Objective**  
Deploy a **web server** running on multiple **Virtual Machines (VMs)** in **Azure**, and distribute incoming traffic across them for better performance and availability using an **Azure Load Balancer**.

First create a basic **Azure Virtual Network (VNet)**, deploy **VMs**, install a web server, and prepare them for load balancing.

## **Deploy 2 Web Servers**  
### **Step 1: Create an Azure Virtual Network (VNet)**
1. Sign in to the **Azure Portal**.
2. Search for **Virtual Network** and click **Create**.
3. Fill in the details:
   - **Resource Group:** `WebApp-RG`
   - **Name:** `WebApp-VNet`
   - **Region:** Choose your preferred region.
   - **Subnet Name:** `WebSubnet`
   - **Subnet Address Range:** `10.0.0.0/24`
4. Click **Review + Create** > **Create**.

### **Step 2: Deploy Multiple Virtual Machines (VMs)**
1. Search for **Virtual Machines** in the Azure portal.
2. Click **Create > Virtual Machine**.
3. Fill in VM details:
   - **Resource Group:** `WebApp-RG`
   - **VM Name:** `WebServer-1`
   - **Region:** Same as `WebApp-VNet`
   - **Image:** Choose **Ubuntu**
   - **Size:** `Standard_B1|s`
   - **Authentication Type:** Password-based access.
   - **Public IP:** Assign one for remote access.
   - **VNet & Subnet:** Select `WebApp-VNet` > `WebSubnet`.
   - Click `Next: Storage`, it is recommended that you select standard SSD or HDD to reduce cost.
4. Click **Review + Create** > **Create**.
> Repeat the same steps for `WebServer-2` to deploy multiple VMs.

### **Step 3: Install a Web Server**
1. Connect to the VM via **SSH**.
2. Run the following command:
   ```bash
   sudo apt update
   sudo apt install apache2 -y
   ```
3. Open a browser and visit `http://<VM-Public-IP>`, and the Apache default page should be displayed.

### **Step 4: Configure NSG for Web Traffic**
1. Go to **Networking > Network Security Groups**.
2. Click **Create NSG**.
3. Name it `WebServer-NSG` and associate it with the subnet.
4. Add inbound rule:
   - **Priority:** `100`
   - **Port:** `80` (for HTTP)
   - **Protocol:** `TCP`
   - **Action:** Allow
   - **Source:** Any
5. Click **Save**.

### **Step 5: Verify Web Access**
- Find the **public IP addresses** of your VMs.
- Open a browser and visit `http://<VM-Public-IP>` to see the web server running.
- You now have **multiple VMs** running a web application.

## Deploy a Load Balancer

### **1. Create an Azure Load Balancer**  
1. Sign in to the **Azure Portal**.
2. Search for **Load Balancer** and click **Create**.
3. Fill in the basic details:
   - **Resource Group:** Select or create one (e.g., `WebApp-RG`).
   - **Name:** `WebApp-LB`
   - **Region:** Choose the same region as your VMs.
   - **Type:** Public (for internet-facing) or Internal (for private workloads).
   - **SKU:** Choose `Standard` for advanced features.
4. Click **Review + Create** > **Create**.

### **2. Set Up the Backend Pool**  
1. Go to **WebApp-LB** > **Backend Pools** > **Add**.
2. Name the backend pool (e.g., `WebServers-Pool`).
3. Choose the VM **networking interfaces (NICs)** that will handle traffic.
4. Click **Add** > **Save**.

### **3. Create Health Probe**  
1. Navigate to **Health Probes** > **Add**.
2. Name the probe (e.g., `HTTP-HealthProbe`).
3. Set the parameters:
   - **Protocol:** HTTP or TCP.
   - **Port:** 80 (for web apps) or a custom app port.
   - **Interval:** 5 seconds (how often Azure checks VM health).
   - **Unhealthy Threshold:** 2 failures before marking a VM down.
4. Click **Add** > **Save**.

### **4. Configure Load Balancer Rule**  
1. Go to **Load Balancer Rules** > **Add**.
2. Set the rule parameters:
   - **Name:** `WebTrafficRule`
   - **Frontend IP:** Select the load balancer’s public IP.
   - **Port:** 80 (or another application port).
   - **Backend Pool:** Select `WebServers-Pool`.
   - **Health Probe:** Choose `HTTP-HealthProbe`.
   - **Session Persistence:** Disabled (or `Client IP` for sticky sessions).
   > Sticky sessions allows you to direct returning visitors to the same target host each time they visit.
3. Click **Add** > **Save**.

### **5. Assign a Public IP (If Internet-Facing)**  
1. Go to **Frontend IP Configuration** > **Add**.
2. Set **Public IP Type** to `Static` (recommended for stability).
3. Click **Create New** > Assign a custom name.
4. Click **OK** > **Save**.

### **6. Test the Load Balancer**  
Once deployed:
- Get the **Load Balancer's Public IP** from the portal.
- Try accessing your application via `http://<LoadBalancerIP>`.
- Azure should automatically distribute requests across backend VMs.

## **Best Practices**  
- Use **Standard Load Balancer** for enhanced security & availability.  
- Ensure **backend VMs** are in the same **VNet** and **availability zone**.  
- Fine-tune **Health Probes** to monitor real-time server health.  
- Enable **Diagnostics & Logs** for performance tracking.  

---
