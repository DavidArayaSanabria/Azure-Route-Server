# Azure Route Server

Azure Route Server enables network appliances to exchange route information with Azure virtual networks dynamically. Configure your network appliances and Azure ExpressRoute and VPN gateways to automatically take the latest route information from Azure Route Server instead of manually talking to each network.

Azure Route Server simplifies dynamic routing between your network virtual appliance (NVA) and your virtual network. It allows you to exchange routing information directly through Border Gateway Protocol (BGP) routing protocol between any NVA that supports the BGP routing protocol and the Azure Software Defined Network (SDN) in the Azure Virtual Network (VNET) without the need to manually configure or maintain route tables. Azure Route Server is a fully managed service and is configured with high availability.

It requieres at least an /27 subnet with the name "RouteServerSubnet" that subnet will not support UDRs or NSGs

It works with BGP and your NVAs must support eBGP multi-hop

Azure route server is not just a VM, it's a fully managed service designed with high availability. If it's deployed in an Azure region that supports Availability Zones, it will have zone-level redundancy

## Target audience

- Infrastructure Architect
- Security Team
- Network Administrator
- Cloud Solution Architect

# Create an Azure Route Server using a Bicep Template

The [Template.bicep](https://github.com/DavidArayaSanabria/Azure-Route-Server/blob/3a266d47b6247b149f5ec7385d8a4a919769e62a/Deployment%20Templates/Template.bicep) Azure Bicep template will help you automatically deploy an Azure Route Server

# Create an Azure Route Server using Azure CLI

Azure Route Server requires a dedicated subnet named RouteServerSubnet. The subnet size has to be at least /27 or short prefix (such as /26 or /25) or you'll receive an error message when deploying the Route Server. Create a subnet configuration named RouteServerSubnet with az network vnet subnet create:

##Add a dedicated subnet

```
az network vnet subnet create \
    --name RouteServerSubnet \
    --resource-group myRouteServerRG \
    --vnet-name myVirtualNetwork \
    --address-prefix 10.0.0.0/24
```

Azure Route Server requires a dedicated subnet named RouteServerSubnet. The subnet size has to be at least /27 or short prefix (such as /26 or /25) or you'll receive an error message when deploying the Route Server. Create a subnet configuration named RouteServerSubnet with az network vnet subnet create:

```
subnet_id=$(az network vnet subnet show \
    --name RouteServerSubnet \
    --resource-group myRouteServerRG \
    --vnet-name myVirtualNetwork \
    --query id -o tsv) 

echo $subnet_id
```
To ensure connectivity to the backend service that manages Route Server configuration, assigning a public IP address is required. Create a Standard Public IP named RouteServerIP with az network public-ip create:

```
az network public-ip create \
    --name RouteServerIP \
    --resource-group myRouteServerRG \
    --version IPv4 \
    --sku Standard
```
Create the Azure Route Server with az network routeserver create. This example creates an Azure Route Server named myRouteServer. The hosted-subnet is the resource ID of the RouteServerSubnet created in the previous section.

```
az network routeserver create \
    --name myRouteServer \
    --resource-group myRouteServerRG \
    --hosted-subnet $subnet_id \
    --public-ip-address RouteServerIP

```

# Architectures

## How does it work?
The following diagram illustrates how Azure Route Server works with an SDWAN NVA and a security NVA in a virtual network. Once you’ve established the BGP peering, Azure Route Server will receive an on-premises route (10.250.0.0/16) from the SDWAN appliance and a default route (0.0.0.0/0) from the firewall. These routes are then automatically configured on the VMs in the virtual network. As a result, all traffic destined to the on-premises network will be sent to the SDWAN appliance. While all Internet-bound traffic will be sent to the firewall. In the opposite direction, Azure Route Server will send the virtual network address (10.1.0.0/16) to both NVAs. The SDWAN appliance can propagate it further to the on-premises network.

![alt image](https://github.com/DavidArayaSanabria/Azure-Route-Server/blob/0ab522a33d3ef76c12c435ffd82fe08348e828c3/Images/Diagram%201.png)

## Dual-Homed

In a typical hub and spoke architecture, application workloads are deployed in spoke VNets. These spokes are peered with a single hub VNet, which contains shared network resources such as VPN and ExpressRoute gateways. In some situations it can be desirable to peer spokes to more than one hub VNet, for example if multiple VPN or ExpressRoute Gateways are required for any reason. Azure Route Server enables this architecture so that workloads in a spoke VNet can communicate through either of the hub VNets it is connected to.

![alt image](https://github.com/DavidArayaSanabria/Azure-Route-Server/blob/0ab522a33d3ef76c12c435ffd82fe08348e828c3/Images/Diagram%202.png)

How does it work?
In the control plane, the NVA and the Route Server will exchange routes as if they’re deployed in the same virtual network. The NVA will learn about spoke virtual network addresses from the Route Server. The Route Server will learn routes from each of the NVAs. The Route Server will then program all the virtual machines in the spoke virtual network with the routes it learned.

In the data plane, virtual machines in the spoke virtual network will see the security NVA or the VPN NVA in the hub as the next hop. Traffic destined for the Internet-bound traffic or the hybrid cross-premises traffic will now route through the NVAs in the hub virtual network. You can configure both hubs to be either active/active or active/passive. In the case when the active hub fails, the traffic to and from the virtual machines will fail over to the other hub. These failures include but not limited to: NVA failures or service connectivity failures. This set up ensures your network is configured for high availability.

## Integration with ExpressRoute

In the control plane, the NVA in the hub virtual network will learn about on-premises routes from the ExpressRoute gateway through route exchange with the Route Server in the hub. In return, the NVA will send the spoke virtual network addresses to the ExpressRoute gateway using the same Route Server. The Route Server in both the spoke and hub virtual network will then program the on-premises network addresses to the virtual machines in their respective virtual network.
This was previously only possible laveraging Azure Virtual WAN

![alt image](https://github.com/DavidArayaSanabria/Azure-Route-Server/blob/0ab522a33d3ef76c12c435ffd82fe08348e828c3/Images/Diagram%203.png)


## Azure services and related products

- Azure VNET
- NVA
- VNET Gateway
- Express Route
- S2S VPN

## Related references
- https://azure.microsoft.com/en-us/services/route-server/#overview
- https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.network/route-server
- https://docs.microsoft.com/en-us/azure/route-server/about-dual-homed-network


