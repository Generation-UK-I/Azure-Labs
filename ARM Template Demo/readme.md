# ARM Templates

The following examples outlines the key sections and logic of an ARM template to deploy two webservers behind a load balancer. We also deploy any necessary supporting resources.

## Parameters

The parameters section defines values that users can provide when deploying the template, which can be used to make it reuseable, for example deploy in different regions by changing the parameter. It also makes your script more reliable and consistent, as you can repeatedly reference the parameter, rather than hoping it is typed correctly each time it is needed.

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "uksouth",
      "metadata": {
        "description": "Location for deployment"
      }
    },
    "adminUsername": {
      "type": "string",
      "defaultValue": "FrankieScout1",
      "metadata": {
        "description": "Admin username for VMs"
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Admin password for VMs"
      }
    }
  },
```

- `location`: Defines the Azure region for deployment.
- `adminUsername` and `adminPassword`: Credentials for the virtual machines.
- `securestring` ensures that passwords are stored securely and not exposed in logs.

## Virtual Network and Subnet

Most resources (with the exception of user account, resource groups, etc.) need to be deployed into a VNet and Subnet in order to communicate. 

```JSON
{
  "type": "Microsoft.Network/virtualNetworks",
  "apiVersion": "2021-05-01",
  "name": "myVNet",
  "location": "[parameters('location')]",
  "properties": {
    "addressSpace": {
      "addressPrefixes": ["10.0.0.0/16"]
    },
    "subnets": [
      {
        "name": "mySubnet",
        "properties": {
          "addressPrefix": "10.0.0.0/24"
        }
      }
    ]
  }
}
{
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2021-05-01",
      "name": "myPublicIP",
      "location": "[parameters('location')]",
      "properties": {
        "publicIPAllocationMethod": "Static"
      }
    },
```

- Defines a Virtual Network (VNet) named `myVNet` with an IP range of **10.0.0.0/16**.
- Contains a subnet named `mySubnet` with an IP range of **10.0.0.0/24**.
- The `location` field uses the value from the `parameters('location')` parameter.
- Request a `publicIPAddress` in the working region, name it `myPublicIP`

## Load Balancer Configuration

The Load Balancer block is quite large, but the logic is not too tricky. 
1. Public traffic is routed to the LB, so it needs the public IP 
1. It needs to know the private IPs of the back-end, but since the IPs can change, we define a pool called `myBackendPool`.
1. We give it rules for the type of traffic we want to balance, in this case `httpRule` balances `TCP` traffic on port `80`.
1. In order to manage our VMs we do need to SSH into them, so the LB needs two `inboundNatRules` which use different ports, `50001` and `50002` in this case, to direct SSH traffic to the individual VMs.
1. Finally we have a `probe` which is how the LB monitors the targets, in this case it probes port 80 on the target, and if two consecutives probes fail, it will stop sending traffic to it. 

```JSON
{
      "type": "Microsoft.Network/loadBalancers",
      "apiVersion": "2021-05-01",
      "name": "myLoadBalancer",
      "location": "[parameters('location')]",
      "dependsOn": ["myPublicIP"],
      "properties": {
        "frontendIPConfigurations": [
          {
            "name": "myFrontend",
            "properties": {
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', 'myPublicIP')]"
              }
            }
          }
        ],
        "backendAddressPools": [
          {
            "name": "myBackendPool"
          }
        ],
        "loadBalancingRules": [
          {
            "name": "httpRule",
            "properties": {
              "frontendIPConfiguration": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/frontendIPConfigurations', 'myLoadBalancer', 'myFrontend')]"
              },
              "backendAddressPool": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'myLoadBalancer', 'myBackendPool')]"
              },
              "protocol": "Tcp",
              "frontendPort": 80,
              "backendPort": 80,
              "enableFloatingIP": false,
              "idleTimeoutInMinutes": 4,
              "loadDistribution": "Default",
              "probe": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/probes', 'myLoadBalancer', 'myHealthProbe')]"
              }
            }
          }
        ],
		"inboundNatRules": [
          { "name": "sshVM1",
		  "properties": {
			  "frontendIPConfiguration": {
				  "id": "[resourceId('Microsoft.Network/loadBalancers/frontendIPConfigurations', 'myLoadBalancer', 'myFrontend')]" 
				  },
				  "protocol": "Tcp",
				  "frontendPort": 50001,
				  "backendPort": 22,
				  "enableFloatingIP": false }
				  },
          { 
		  "name": "sshVM2",
		  "properties": {
			  "frontendIPConfiguration": {
				  "id": "[resourceId('Microsoft.Network/loadBalancers/frontendIPConfigurations', 'myLoadBalancer', 'myFrontend')]" 
				  }, 
				  "protocol": "Tcp", 
				  "frontendPort": 50002, 
				  "backendPort": 22, 
				  "enableFloatingIP": false 
				  }
			}
        ],
        "probes": [
          {
            "name": "myHealthProbe",
            "properties": {
              "protocol": "Tcp",
              "port": 80,
              "intervalInSeconds": 15,
              "numberOfProbes": 2
            }
          }
        ]
      }
    },
```

- **Frontend Configuration**: Defines a public IP for the load balancer.
- **Backend Address Pool**: Holds the two VMs.
- **Load Balancing Rule** (`httpRule`):
  - Forwards traffic on **port 80** to backend VMs.
  - Uses **TCP** protocol.
  - Tied to the **backend address pool** where VMs are registered.
- **Health Probe** (`myHealthProbe`):
  - The load balancer checks if VMs are alive by probing port 80.
  - If two consecutive probes fail, the VM is removed from rotation.

## Network Security Group (NSG)

NSGs act as a **subnet-level** firewall, allowing you to permit and deny ingress and egress traffic based on protocols, ports, and IP address(es).

```JSON
{
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2021-05-01",
      "name": "myNSG",
      "location": "[parameters('location')]",
      "properties": {
        "securityRules": [
          {
            "name": "AllowHTTP",
            "properties": {
              "priority": 100,
              "protocol": "Tcp",
              "access": "Allow",
              "direction": "Inbound",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "80"
            }
          },
          {
            "name": "AllowSSH",
            "properties": {
              "priority": 110,
              "protocol": "Tcp",
              "access": "Allow",
              "direction": "Inbound",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "22"
            }
          }
        ]
      }
    },
```

- **Security Rules**:
- `AllowHTTP`: Allows HTTP (port 80) traffic.
- `AllowSSH`: Allows SSH (port 22) traffic.
- `priority` values define rule precedence (lower values apply first).

## Virtual Network Interfaces (NICs)

We're deploying two VMs, so we need two NICs to attach to them, to facilitate communications. Notice that the NICs connect to the load balancer, so it's added with `dependsOn` so that the LB will deploy first. In the NICs `ipConfiguration` we tell it what VNet and Subnet it's in, and we also attach the `loadBalancerBackendAddressPools` and a `loadBalancerInboundNatRule` to each NIC.

```JSON
{
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2021-05-01",
      "name": "myNIC1",
      "location": "[parameters('location')]",
      "dependsOn": ["myLoadBalancer"],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipConfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', 'myVNet', 'mySubnet')]"
              },
              "loadBalancerBackendAddressPools": [
                {
                  "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'myLoadBalancer', 'myBackendPool')]"
                }
              ], 
			  "loadBalancerInboundNatRules": [
                {
                  "id": "[resourceId('Microsoft.Network/loadBalancers/inboundNatRules', 'myLoadBalancer', 'sshVM1')]"
                }
              ]
            }
          }
        ]
      }
    },
	{
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2021-05-01",
      "name": "myNIC2",
      "location": "[parameters('location')]",
      "dependsOn": ["myLoadBalancer"],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipConfig2",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', 'myVNet', 'mySubnet')]"
              },
              "loadBalancerBackendAddressPools": [
                {
                  "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'myLoadBalancer', 'myBackendPool')]"
                }
              ],
			  "loadBalancerInboundNatRules": [
                {
                  "id": "[resourceId('Microsoft.Network/loadBalancers/inboundNatRules', 'myLoadBalancer', 'sshVM2')]"
                }
              ]
            }
          }
        ]
      }
    },
```
## Virtual Machines

Finally we can launch our VMs into the infrastructure that has been built 

```JSON
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2021-03-01",
  "name": "myVM1",
  "location": "[parameters('location')]",
  "dependsOn": ["myNIC1"],
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_B1s"
    },
    "osProfile": {
      "computerName": "myVM1",
      "adminUsername": "[parameters('adminUsername')]",
      "adminPassword": "[parameters('adminPassword')]"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "Canonical",
        "offer": "UbuntuServer",
        "sku": "18.04-LTS",
        "version": "latest"
      },
      "osDisk": {
        "createOption": "FromImage"
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "[resourceId('Microsoft.Network/networkInterfaces', 'myNIC1')]"
        }
      ]
    }
  }
},
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2021-03-01",
  "name": "myVM2",
  "location": "[parameters('location')]",
  "dependsOn": ["myNIC2"],
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_B1s"
    },
    "osProfile": {
      "computerName": "myVM2",
      "adminUsername": "[parameters('adminUsername')]",
      "adminPassword": "[parameters('adminPassword')]"
    },
    "storageProfile": {
      "imageReference": {
        "publisher": "Canonical",
        "offer": "UbuntuServer",
        "sku": "18.04-LTS",
        "version": "latest"
      },
      "osDisk": {
        "createOption": "FromImage"
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "[resourceId('Microsoft.Network/networkInterfaces', 'myNIC2')]"
        }
      ]
    }
  }
}
```

- Creates a VM named `myVM1`.
- `dependsOn` ensures the VM is deployed after the **Load Balancer** and **Network Security Group**.
- Uses the **Ubuntu 18.04 LTS** image.
- `networkProfile` connects the VM to a **Network Interface** (NIC) that is linked to the **Load Balancer Backend Pool**.



## Summary

1. Create Virtual Network (VNet) & Subnet to house the VMs.
1. Deploy a Load Balancer with:
   - Public IP (for external access).
   - Backend Pool (to distribute traffic to VMs).
   - Load Balancing Rule (routes HTTP traffic).
   - Inbound NAT rules (forward SSH traffic).
   - Health Probe (ensures VMs are running).
1. Deploy two Virtual Machines (VMs):
   - Each with its own NIC.
   - Connected to the Load Balancer Backend Pool.
   - Uses Ubuntu 18.04 LTS as OS.
1. Deploy a Network Security Group (NSG):
   - Allows HTTP (80) and SSH (22) traffic.  

## Deploying the Template

1. Save the JSON file as `template.json` on your local machine.
1. Upload and validate your template using Azure Portal/PowerShell/CLI
1. Deploy resources
1. Test deployment


## All-In-One

```JSON
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "uksouth",
      "metadata": {
        "description": "Location for deployment"
      }
    },
    "adminUsername": {
      "type": "string",
      "defaultValue": "FrankieScout1",
      "metadata": {
        "description": "Admin username for VMs"
      }
    },
    "adminPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Admin password for VMs"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2021-05-01",
      "name": "myVNet",
      "location": "[parameters('location')]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": ["10.0.0.0/16"]
        },
        "subnets": [
          {
            "name": "mySubnet",
            "properties": {
              "addressPrefix": "10.0.0.0/24"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2021-05-01",
      "name": "myPublicIP",
      "location": "[parameters('location')]",
      "properties": {
        "publicIPAllocationMethod": "Static"
      }
    },
    {
      "type": "Microsoft.Network/loadBalancers",
      "apiVersion": "2021-05-01",
      "name": "myLoadBalancer",
      "location": "[parameters('location')]",
      "dependsOn": ["myPublicIP"],
      "properties": {
        "frontendIPConfigurations": [
          {
            "name": "myFrontend",
            "properties": {
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', 'myPublicIP')]"
              }
            }
          }
        ],
        "backendAddressPools": [
          {
            "name": "myBackendPool"
          }
        ],
        "loadBalancingRules": [
          {
            "name": "httpRule",
            "properties": {
              "frontendIPConfiguration": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/frontendIPConfigurations', 'myLoadBalancer', 'myFrontend')]"
              },
              "backendAddressPool": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'myLoadBalancer', 'myBackendPool')]"
              },
              "protocol": "Tcp",
              "frontendPort": 80,
              "backendPort": 80,
              "enableFloatingIP": false,
              "idleTimeoutInMinutes": 4,
              "loadDistribution": "Default",
              "probe": {
                "id": "[resourceId('Microsoft.Network/loadBalancers/probes', 'myLoadBalancer', 'myHealthProbe')]"
              }
            }
          }
        ],
		"inboundNatRules": [
          { "name": "sshVM1",
		  "properties": {
			  "frontendIPConfiguration": {
				  "id": "[resourceId('Microsoft.Network/loadBalancers/frontendIPConfigurations', 'myLoadBalancer', 'myFrontend')]" 
				  },
				  "protocol": "Tcp",
				  "frontendPort": 50001,
				  "backendPort": 22,
				  "enableFloatingIP": false }
				  },
          { 
		  "name": "sshVM2",
		  "properties": {
			  "frontendIPConfiguration": {
				  "id": "[resourceId('Microsoft.Network/loadBalancers/frontendIPConfigurations', 'myLoadBalancer', 'myFrontend')]" 
				  }, 
				  "protocol": "Tcp", 
				  "frontendPort": 50002, 
				  "backendPort": 22, 
				  "enableFloatingIP": false 
				  }
			}
        ],
        "probes": [
          {
            "name": "myHealthProbe",
            "properties": {
              "protocol": "Tcp",
              "port": 80,
              "intervalInSeconds": 15,
              "numberOfProbes": 2
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/networkSecurityGroups",
      "apiVersion": "2021-05-01",
      "name": "myNSG",
      "location": "[parameters('location')]",
      "properties": {
        "securityRules": [
          {
            "name": "AllowHTTP",
            "properties": {
              "priority": 100,
              "protocol": "Tcp",
              "access": "Allow",
              "direction": "Inbound",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "80"
            }
          },
          {
            "name": "AllowSSH",
            "properties": {
              "priority": 110,
              "protocol": "Tcp",
              "access": "Allow",
              "direction": "Inbound",
              "sourceAddressPrefix": "*",
              "sourcePortRange": "*",
              "destinationAddressPrefix": "*",
              "destinationPortRange": "22"
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2021-05-01",
      "name": "myNIC1",
      "location": "[parameters('location')]",
      "dependsOn": ["myLoadBalancer"],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipConfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', 'myVNet', 'mySubnet')]"
              },
              "loadBalancerBackendAddressPools": [
                {
                  "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'myLoadBalancer', 'myBackendPool')]"
                }
              ], 
			  "loadBalancerInboundNatRules": [
                {
                  "id": "[resourceId('Microsoft.Network/loadBalancers/inboundNatRules', 'myLoadBalancer', 'sshVM1')]"
                }
              ]
            }
          }
        ]
      }
    },
	{
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2021-05-01",
      "name": "myNIC2",
      "location": "[parameters('location')]",
      "dependsOn": ["myLoadBalancer"],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipConfig2",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', 'myVNet', 'mySubnet')]"
              },
              "loadBalancerBackendAddressPools": [
                {
                  "id": "[resourceId('Microsoft.Network/loadBalancers/backendAddressPools', 'myLoadBalancer', 'myBackendPool')]"
                }
              ],
			  "loadBalancerInboundNatRules": [
                {
                  "id": "[resourceId('Microsoft.Network/loadBalancers/inboundNatRules', 'myLoadBalancer', 'sshVM2')]"
                }
              ]
            }
          }
        ]
      }
    },
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "myVM1",
      "location": "[parameters('location')]",
      "dependsOn": ["myNIC1"],
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_B1s"
        },
        "osProfile": {
          "computerName": "myVM1",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "18.04-LTS",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage"
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', 'myNIC1')]"
            }
          ]
        }
      }
    },
	{
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2021-03-01",
      "name": "myVM2",
      "location": "[parameters('location')]",
      "dependsOn": ["myNIC2"],
      "properties": {
        "hardwareProfile": {
          "vmSize": "Standard_B1s"
        },
        "osProfile": {
          "computerName": "myVM2",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "18.04-LTS",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage"
          }
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', 'myNIC2')]"
            }
          ]
        }
      }
    }
  ]
}
```
---
*Some content generated with AI*
