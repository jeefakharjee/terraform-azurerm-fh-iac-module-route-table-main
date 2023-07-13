# Azure Route Table

## Usage

```hcl

module "rg" {
  source  = "../../../cfn-tf-az-rg"
  location    = "eastus"
  rg_name     = "cfn-rg-demo-01"
  lock_level  = ""
  tags        = {
      "Environment" = "Terraform Demo"
	  "Owner Email" = "cgajjeli@commonwealth.com"
	  "Application Name" = "cfndemo"
  }  
}

module "vnet" {
  source  =  "../../../cfn-tf-az-vnet"
  location       = module.rg.resource_group_location
  vnet_name      = "cfn-vnet-demo-01"  
  resource_group_name = module.rg.resource_group_name
  vnet_cidr   = ["10.10.0.0/16"]
  dns_servers = ["10.0.0.4", "10.0.0.5"] # Can be empty if not used
  tags        = {
      "Environment" = "Terraform Demo"
	  "Owner Email" = "cgajjeli@commonwealth.com"
	  "Application Name" = "cfndemo"
  }  
}

module "rt" {
  source              = "../../../cfn-tf-az-route-table"
  rt_name             = "route-table"
  resource_group_name = module.rg.resource_group_name
  location            = module.rg.resource_group_location
  routes = [
    { name = "route1", address_prefix = "10.0.0.0/16", next_hop_type = "VirtualAppliance", next_hop_in_ip_address = "10.0.0.4" },
    { name = "route2", address_prefix = "10.10.0.0/16", next_hop_type = "VirtualAppliance", next_hop_in_ip_address = "10.0.0.4" },
    { name = "route3", address_prefix = "0.0.0.0/0", next_hop_type = "Internet" },
  ]
  disable_bgp_route_propagation = false
  tags        = {
      "Environment" = "Terraform Demo"
	  "Owner Email" = "cgajjeli@commonwealth.com"
	  "Application Name" = "cfndemo"
  }   
}

resource "azurerm_route" "custom_route" {
  name                = "acceptanceTestCustomRoute1"
  resource_group_name = module.rg.resource_group_name
  route_table_name    = module.rt.name
  address_prefix      = "10.1.0.0/16"
  next_hop_type       = "VnetLocal"
}

```
## Parameters

The following parameters are supported:

| Name                             | Description                                                                           |        Type         | Default | Required |
| -------------------------------- | ------------------------------------------------------------------------------------- | :-----------------: | :-----: | :------: |
| name                             | The name of the route table.                                                          |      `string`       |   n/a   |   yes    |
| resource\_group\_name            | The name of the resource group in which to create the route table.                    |      `string`       |   n/a   |   yes    |
| location                         | The location/region where the route table is created.                                 |      `string`       |   n/a   |   yes    |
| routes                           | List of objects that represent the configuration of each route.                       | `list(map(string))` |  `[]`   |    no    |
| disable\_bgp\_route\_propagation | Boolean flag which controls propagation of routes learned by BGP on that route table. |       `bool`        | `true`  |    no    |
| tags                             | A mapping of tags to assign to the resource.                                          |    `map(string)`    |  `{}`   |    no    |

The `routes` supports the following:

| Name                       | Description                                                                                                                                              |   Type   | Default | Required |
| -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | :------: | :-----: | :------: |
| name                       | The name of the route.                                                                                                                                   | `string` |   n/a   |   yes    |
| address\_prefix            | The destination CIDR to which the route applies.                                                                                                         | `number` |   n/a   |   yes    |
| next\_hop\_type            | The type of Azure hop the packet should be sent to. Possible values are `VirtualNetworkGateway`, `VnetLocal`, `Internet`, `VirtualAppliance` and `None`. | `string` |   n/a   |   yes    |
| next\_hop\_in\_ip\_address | Contains the IP address packets should be forwarded to. Next hop values are only allowed in routes where the next hop type is `VirtualAppliance`.        | `string` | `null`  |    no    |

## Outputs

The following outputs are exported:

| Name                  | Description                                                        |
| --------------------- | ------------------------------------------------------------------ |
| id                    | The route table configuration ID.                                  |
| name                  | The name of the route table.                                       |
| resource\_group\_name | The name of the resource group in which to create the route table. |
| location              | The location/region where the route table is created.              |
| routes                | Blocks containing configuration of each route.                     |
| subnets               | List of the ids of the subnets configured to the route table.      |
| tags                  | The tags assigned to the resource.                                 |
