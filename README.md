# example-hub

This is an example Terraform config creating a hub in a hub and spoke topology. As an example repo for learning purposed then you are encouraged to copy any of the Terraform from it, or fork it and make your own changes.

It is a work in progress and may be updated at any point. It will be used for a number of training labs in [Azure Citadel](https://azurecitadel.com).

## tl;dr

One example usage once you've cloned the repo:

* [Optional] Copy the bootstrap output files from the storage account created by [terraform-bootstrap](https://github.com/terraform-azurerm-modules/terraform-bootstrap)
* `mv terraform.tfvars.example terraform.tfvars` and edit
* `terraform init`
* `terraform validate`
* `terraform plan`
* `terraform apply`

The resources will be created in a single resource group called example-hub.

Read the full readme for more information and options.

## azurerm_provider.tf

By default the azurerm_provider.tf will use your Azure CLI token. Ensure you are logged into the correct [context](docs/context.md).

The file also includes comments if using service principals or managed identities.

Alternatively, look at the terraform-bootstrap repo as this will generate a service principal, remote state storage account and a key vault. It is intended for production environments. It will create outputs you can use:

* updated azurerm_provider.tf
* backend.tf
* bootstrap_secrets.tf

## terraform.tfvars

An example terraform.tfvars is included. More defaulted variables can be found in the variables.tf and across some of the other .tf files.

## Modularity

Against the usual Terraform convention, this repo has standalone HCL files for services that contain all the variables, resources and outputs for that specific resource.

Some files can therefore be renamed or removed from of the root module directory to be selective about the resources created whilst working through different labs.

## Networking

The vnet.tf has a predefined subnet structure based on a /24 address space. Feel free to reconfigure your own fork.

Current subnets:

| Name  | Address Prefixes |
| ------------- | ------------- |
| SharedServices | x.x.x.0/26, x.x.x.64/27 |
| Domain Controllers | x.x.x.96/29  |
| AzureFirewallSubnet | x.x.x.128/26  |
| AzureBastionSubnet | x.x.x.192/27  |
| GatewaySubnet | x.x.x.224/27  |

This leaves some unused space in the vNet.

To check your [cidrsubnet](https://www.terraform.io/docs/configuration/functions/cidrsubnet.html) values, use [terraform console](https://www.terraform.io/docs/commands/console.html):

```text
$ terraform console
> cidrsubnet("172.16.0.0/24", 5, 12)
172.16.0.96/29
```

> The azurerm_subnet resources use the new address prefixes attribute so the assignable space is now more flexible.

## VPN Gateway

The VPN Gateway takes over 30 minutes to deploy and is one of the more expensive resources. If you wish to create a VPN Gateway then move the vpngw.tf file up from the waiting_room.

The SKU has been selected as it is a Gen2 and can be scaled up to a SKU with greater bandwidth if required.

The example vpngw.tf supports the option of [P2S with Azure Active Directory](docs/vpngw.md) authentication.

## Bastion

To create the Bastion service, move the bastion.tf file in the waiting_room folder up to the root of the repo.

The required AzureBastion subnet /27 is already in vnet.tf.

## Images

The management VM examples use modules. You don't have to use these and can use native resources or your own modules. This example repo is not prescriptive.

The current VM modules use custom images. If you wish to create your own example custom images and publish them to a Shared Image Gallery then you can use the <https://github.com/richeney/packer>.

## Virtual Machines

Three example VM configs are included in the management-vms.tf.

1. A standalone jumpbox server.
1. An example pair of VMs are created in an availability set to represent the domain controllers.
1. A pair of highly available config management servers fronted by a load balancer.

> Note that all servers are standard Ubuntu images and are illustrative only. (See [futures](#futures).)

## Virtual Machine Scale Sets

An example VMSS has been included to represent a manually scalable set of testbed servers.

The local.defaults object in the management-vms.tf requires the contents of an SSH public key for the default. This example pulls that public key from the key vault.

## Adding an SSH Public Key to the Key Vault

The example terraform.tfvars file will create a key vault secret called ubuntu that uses the default SSH public key name.

```terraform
ssh_public_keys = [
  { username = "ubuntu", ssh_public_key_file = "~/.ssh/id_rsa.pub" }
]
```

This is then used for the ubuntu user on the example VMs.

(The module also supports additional SSH key secrets, if you wanted to centralise a group of named admins.)

Example command to generate a public key pair with default names:

```bash
ssh-keygen -m PEM -t rsa -b 4096
```

If you accept the default name then it will create id_rsa and id_rsa.pub in ~/.ssh.

## Spokes

Once deployed, explore the example spoke repositories:

* [terraform-example-app](https://github.com/terraform-azurerm-modules/terraform-example-app)

## Future

Currently planned:

* Example Azure Firewall
* Route Table for use by the spokes.
* Network Security Group
* Azure Policy Initiatives for automated configuration of VM extensions for monitoring and protection
* Use future updates to the VM and VMSS modules to enable fully configured read only domain controllers

Please use the issues to request enhancements.
