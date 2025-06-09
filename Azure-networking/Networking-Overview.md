### **Azure Networking Overview**
Azure networking enables secure communication between different Azure resources, on-premises environments, and the internet. It provides tools for managing connectivity, security, and performance across a globally distributed infrastructure.

Components of Azure networking:
- **Virtual Networks (VNets):** The foundation of Azure networking, allowing Azure resources to communicate securely.
- **Subnets:** Segments within a VNet that help organize and control traffic.
- **Network Security Groups (NSGs):** Act like firewalls, defining inbound/outbound rules for traffic control.
- **Azure Load Balancer:** Distributes traffic across multiple resources for better performance and availability.
- **Azure Firewall:** Cloud-based security service that protects networks from threats.
- **ExpressRoute:** Private connections to Azure from on-premises environments.
- **VPN Gateway:** Secure connectivity for hybrid environments via encrypted tunnels.
- **Azure DNS:** Provides name resolution for resources in Azure.

### **Azure Virtual Networks (VNets)**
VNets form the backbone of Azure networking. They allow Azure resources like virtual machines, databases, and applications to communicate securely while offering flexibility in managing IP addressing, subnets, and routing.

Key features of VNets:
- **Isolation:** Each VNet is a private network within Azure, ensuring security and segmentation.
- **Subnetting:** VNets can be divided into subnets to organize workloads efficiently.
- **Peering:** VNets can connect to each other seamlessly using **VNet Peering**, allowing communication across different VNets without extra latency.
- **Connectivity to On-Premises Networks:** VNets support VPN and ExpressRoute connections for hybrid setups.
- **Traffic Filtering:** Security tools like **NSGs** and **Azure Firewall** control inbound/outbound traffic.
- **Integration with Azure Services:** Many Azure services, including **App Service**, **Azure Kubernetes Service (AKS)**, and **Azure SQL Database**, can connect to a VNet.

### **Why VNets Matter in Azure Networking**
VNets provide a secure and scalable network foundation, ensuring applications and workloads can communicate efficiently while maintaining security and compliance. They are essential for managing enterprise-grade applications, hybrid cloud setups, and multi-tier architectures.

### Network Security Groups (NSGs) and Firewalls

**NSGs** in Azure are essential for controlling inbound and outbound traffic to resources within a **VNet**, acting like cloud-based firewalls, enabling security filtering based on rules.

### **Key Features of NSGs**
- **Rule-Based Traffic Filtering:** NSGs operate with security rules that define whether traffic should be **allowed** or **denied**.
- **Stateful Inspection:** Once a rule allows traffic in one direction, the response traffic is automatically permitted.
- **Application to Resources:** NSGs can be linked to:
  - **Subnets** (to apply rules to multiple VMs or services)
  - **Network interfaces (NICs)** (to apply rules at the individual VM level)
- **Prioritized Rules:** Rules are processed in order, starting from the lowest-numbered rule.
- **Default Security Rules:** Predefined rules ensure baseline security by restricting unwanted internet access while allowing necessary internal traffic.

### **How NSG Rules Work**
NSGs consist of **Inbound** and **Outbound** security rules. Each rule includes:
- **Source & Destination:** Defines where the traffic is coming from and where it's going.
- **Port & Protocol:** Specifies which TCP or UDP ports are involved.
- **Action:** Either **Allow** or **Deny** traffic.
- **Priority:** Determines rule processing order (lower numbers have higher precedence).

For example:
| Priority | Name            | Source    | Destination | Port  | Action |
|----------|---------------|----------|------------|------|--------|
| 100      | AllowWeb      | Any      | VM1        | 443  | Allow  |
| 200      | DenyAll       | Any      | Any        | Any  | Deny   |

### **NSGs vs Azure Firewall**
While NSGs manage **layer 4 filtering (TCP/UDP traffic)** within Azure, **Azure Firewall** offers **layer 7 filtering** with centralized logging and advanced threat protection.

### **Best Practices for NSGs**
- **Apply NSGs at the subnet level** for consistent security.
- **Use least privilege access** to minimize exposure.
- **Regularly audit and refine rules** to prevent unnecessary security vulnerabilities.
- **Combine NSGs with Azure Firewall** for additional security layers.

**Network Security Groups (NSGs)** and **Azure Firewall** are both essential security tools in Azure, but they work at different levels and complement each other.

### **How NSGs and Azure Firewall Work Together**
- **NSGs:** Operate at the **network interface (NIC) or subnet level**, controlling inbound and outbound traffic between resources **within** a Virtual Network (VNet).
- **Azure Firewall:** Provides **centralized security** at the network perimeter, filtering traffic **between VNets** or **from external sources** based on advanced rules.

### **Key Differences Between NSGs & Azure Firewall**
| Feature               | NSGs                        | Azure Firewall              |
|-----------------------|---------------------------|-----------------------------|
| **Scope**            | Subnet or NIC-level rules  | VNet-wide or cross-VNet control |
| **Traffic Type**      | Layer 4 (TCP/UDP) filtering | Layer 7 (application filtering) |
| **Logging**         | Basic logging via Azure Monitor | Advanced logging & threat analytics |
| **Threat Protection** | No built-in threat intelligence | Uses Microsoft Threat Intelligence |
| **NAT & DNAT**       | Not supported              | Supported for inbound/outbound rules |

### **Example: How They Work Together**
Consider a scenario where you have:
1. **Web App VNet** with multiple subnets.
2. **Azure Firewall** deployed to **filter malicious external traffic**.
3. **NSGs** on **subnets** to restrict access to individual resources.

#### **Traffic Flow Example**
- **Inbound request from internet:** Azure Firewall first checks **Layer 7 filtering** (e.g., allowing HTTP/HTTPS but blocking known malicious IPs).
- **Traffic reaching web server subnet:** NSG then applies **Layer 4 rules**, controlling which **VMs** or services within the subnet can respond.
- **Outbound traffic:** Azure Firewall ensures external traffic is secure before leaving Azure, while NSGs enforce local outbound rules.

### **Best Practices**
- Use **Azure Firewall** for **perimeter security**—filter traffic between VNets and external networks.  
- Use **NSGs for granular security** within **individual subnets and VMs**.  
- Combine both **for layered security**, ensuring malicious traffic is blocked at multiple points.

## **Load Balancing in Azure**

### **Types of Load Balancing in Azure**
Azure provides several load-balancing solutions, each tailored for different workloads:

#### **1. Azure Load Balancer (Layer 4 - TCP/UDP)**
- Distributes network traffic among **virtual machines (VMs)** or **Azure services**.
- Works at **Layer 4 (Transport Layer)**, meaning it handles TCP and UDP traffic only.
- Supports **High Availability & Failover** by routing traffic to healthy instances.
- **Use Case:** Load balancing VMs within a **single Azure region**.

#### **2. Azure Application Gateway (Layer 7 - HTTP/HTTPS)**
- An **application-level load balancer** for web applications.
- Provides **SSL termination**, **URL-based routing**, **session affinity**, and **Web Application Firewall (WAF)**.
- Ideal for **handling HTTP/HTTPS traffic**.
- **Use Case:** Securely routing traffic to multiple web applications.

#### **3. Azure Front Door (Global Load Balancer)**
- Distributes traffic across **multiple Azure regions** with intelligent routing.
- Includes **caching, content delivery acceleration, and DDoS protection**.
- Optimizes performance for **web applications** and **APIs** globally.
- **Use Case:** Enhancing availability and performance for global web applications.

#### **4. Traffic Manager (DNS-Based Load Balancing)**
- Works at the **DNS level** to direct user traffic to the best available endpoint.
- Uses routing methods like **performance-based, geographic, and priority routing**.
- **Use Case:** Multi-region application distribution & disaster recovery.

### **Choosing the Right Load Balancer**
| Load Balancer Type       | Layer | Best For                   |
|-------------------------|------|----------------------------|
| **Azure Load Balancer**  | L4   | VMs within a single region |
| **Azure Application Gateway** | L7 | Web applications           |
| **Azure Front Door**     | Global | Global web apps & APIs     |
| **Traffic Manager**      | DNS  | Multi-region traffic routing |

### **Benefits of Load Balancing in Azure**
- **High Availability:** Ensures applications remain accessible even if some instances fail.  
- **Scalability:** Dynamically distributes workloads based on demand.  
- **Security & Optimization:** Enhances security via **SSL termination**, **DDoS protection**, and **web firewall integration**.  
- **Performance Boost:** Minimizes latency with efficient traffic distribution.  

[Click here for a tutorial on deploying two load balanced web servers](Deploy-a-Load-Balancer.md)

## Bastion Server

**Azure Bastion** is a **fully managed service** that provides **secure and seamless Remote Desktop Protocol (RDP) and Secure Shell (SSH) access** to virtual machines (VMs) without needing a public IP address.

Azure Bastion acts as a **jump box (bastion host)** to securely connect to Azure VMs via the **Azure portal** without exposing them to the public internet. It strengthens security by eliminating **direct RDP/SSH access**, reducing attack vectors like **port scanning, brute force attacks**, and **unwanted exposure**.

### **Key Features**
- **Secure RDP/SSH Access** – No need for direct VM exposure via public IP.  
- **Browser-Based Connectivity** – Connect via the Azure portal without third-party clients.  
- **No Public IP Needed** – Improves security by keeping VMs within a private network.  
- **Network Segmentation** – Restricts access via **Network Security Groups (NSGs)**.  
- **Automatic Scaling** – Handles multiple simultaneous sessions efficiently.  
- **Protection Against Attacks** – Reduces risk of **port scanning & brute force attacks**.  

## **How Azure Bastion Works**

### **Typical Connection Flow**

1. **User initiates a connection** via the **Azure portal**.
2. **Azure Bastion provisions a secure session** using the browser.
3. **Traffic remains within the private Azure network**, avoiding public internet exposure.
4. **VM access is granted** without a public IP, maintaining security.  

### **Deployment & Configuration**

---

1. **Create Azure Bastion**:
   - Go to **Azure Portal** > **Search for Bastion** > **Create Bastion**.
   - Select **Resource Group**, Name, and **VNet**.
   - Choose **Subnet Name:** Must be named `AzureBastionSubnet`.
   - Assign a **Static Public IP** for portal access.
   - Click **Create**.

2. **Connect to a VM using Bastion**:
   - Go to **Virtual Machines** > Select a VM.
   - Click **Connect > Bastion**.
   - Enter **username & password**.
   - Access the VM securely **via browser**.

### **When to Use Azure Bastion**
- **Secure Remote VM Management** – Avoid exposing RDP/SSH ports.  
- **Jump Box Alternative** – Eliminates need for manually configuring bastion hosts.  
- **Enterprise Security Compliance** – Helps meet **cloud security best practices**.  
- **Reducing Attack Surface** – Prevents unnecessary public IP exposure.  

---