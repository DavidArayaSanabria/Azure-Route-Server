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

![alt image](https://github.com/DavidArayaSanabria/Azure-Route-Server/blob/0ab522a33d3ef76c12c435ffd82fe08348e828c3/Images/Diagram%201.png)

![alt image](https://github.com/DavidArayaSanabria/Azure-Route-Server/blob/0ab522a33d3ef76c12c435ffd82fe08348e828c3/Images/Diagram%202.png)

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
