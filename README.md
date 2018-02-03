# Create new VMs and join them to an existing AD domain

### New for V2
* Added support for managed disks.
* Automatically use correct disk SKU (Standard or Premium) depending on VM Size.
* Optional autoshutdown schedule for each VM, set for 11:00 PM, UTC.
* Breaking change: no parameter for a storage account.

### Features

This template creates one or more new VMs and joins them to an 
existing AD domain. Prerequisites:
* The VNET and subnet must exist in the default region, but may be in another Resource Group.
* The AD domain must have connectivity to the VNET. The DC could be in the same VNET or be remote, that does not matter. 
* The existing DNS definitions on the VNET will inherit to the VMs and _must_ enable the new VMs to resolve the AD domain. 
* The DNS service in your AD must be able to resolve the public internet. 
* The target Organizational Unit (OU) in your AD must exist.
* AD credentials that can create a computer account in the given OU.

One-click deployment to Azure:

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fwkasdorp%2Fnew-vm-with-domain-join%2Fmaster%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

Warning: this template **creates one or more running VMs**. 
Make sure to deallocate them when you are done to avoid incurring costs. 

### Background information

The existing VNET and Storage Groups can be in different Resource Groups. 
This enables a neat trick: after you are done with the VMs you can remove
the entire Resource Group and its contents in one shot, 
instead of having to pick the resources from the existing RG one by one. 

The VMs are deliberately created without public NICs. They already
have internal connectivity, so use that to open RDP connections etc.
Your VNET must allow internet access for the VMs. 

The VMs use managed disks, which means that they do not depend on
an existing storage account. Specifically, the template uses
implicit managed disks where the disks are created as a direct
resource of the VM.

All VMs are configured in an automatically named Availability Set. 
Although an Availability Set is not always needed, it is required for certain cases such
as placing VMs in a load balancer. Each set of VMs has its own
Availability Set, allowing you to deploy multiple sets to the same
Resource Group. The managed disks are also in this Availability Set. 

The template, unusually, does not ask for the administrator name 
and password for the new VMs. This is because the VMs will 
be joined to an AD domain, so the local Admin account is not needed.
The password is obfuscated _but is not random_. For increased security
you need to set it yourself. 

The lack of an accessible local admin account also means that troubleshooting is 
harder when the domain join operation fails.
That would happen if the OU does not exist, the credentials are incorrect, etc. However,
I have also seen it fail for no obvious reason on Windows Server 2016. It has 
not failed me for Windows Server 2012 R2. To generate valid local admin credentials,
reset the password using the ARM portal. 

Final note: do not deploy to the AD container CN=Computers,\<DN-Domain\>. 
I found experimentally that this always fails. You must use an existing OU. 
