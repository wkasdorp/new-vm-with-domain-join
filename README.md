# Create new VMs and join them to an existing AD domain

This template creates one or more new VMs and joins them to an 
existing AD domain. Prerequisites:
* The target OU must exist.
* The VNET and subnet must exist, but may be in another Resource Group.
* The AD domain must have connectivity to the VNET. It could be in the same VNET or remote, that does not matter. 
* The existing DNS definitions on the VNET will inherit to the VMs and _must_ enable the new VMs to resolve the AD domain. 
* The AD DNS must be able to resolve the public internet. 

* The storage account must exist. 

One-click deployment to Azure:

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fwkasdorp%2Fnew-vm-with-domain-join%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

Warning: this template **creates one or more running VMs**. 
Make sure to deallocate it when you are done to avoid incurring costs. 

The existing VNET and Storage Groups can be in different Resource Groups. 
This enables a neat trick: after you are done with the VMs you can remove
 the entire RG and its contents in one shot, 
instead of having to pick the resources from the existing RG one by one. 

The VMs are deliberately created without public NICs. They already
have internal connectivity, so use that to open RDP connections etc.
Your VNET must allow internet access for the VMs. 

All VMs are configured in an automatically named Availability Set. 
Although this is not always needed, it is required for certain cases such
as placing VMs in a load balancer. Each set of VMs has its own
Availability Set, allowing you to deploy multiple sets to the same
Resource Group.

The template, unusually, does not ask for the administrator name 
and password for the new VMs. This is because the VMs will 
be joined to a domain and the local Admin account is not needed.
The password is obfuscated but not random. For increased security
you need to set it yourself. 

Final note: do not deploy to the container CN=Computers,<DN-Domain>. I found experimentally that this always fails. You must use an existing OU. 
