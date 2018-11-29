# Introduction 
Playbook to deploy an active-active pair of Palo Alto Networks firewalls in Azure using Ansible.

# Manual steps before deployment
- Accept market place terms & conditions
```powershell
Get-AzureRmMarketplaceTerms -Publisher "paloaltonetworks" -Product "vmseries1" -Name "byol" | Set-AzureRmMarketplaceTerms -Accept
```

# Manual steps after deployment
- Firmware upgrade
- Licenses

# Todo:
- Address object & groups
- DNS sink hole
- TCP Settings (Device > Setup > Session > TCP Settings)
- Content ID Settings (Device > Setup > Content-ID > Content-ID Settings)
- Wildfire settingis (Device > setup > WildFire > General Settings)
- Syslog / log forwarding
- Zone protection profile
- Security profiles / Security profile groups
- Automatic upgrade firmware
- Automatic install of licenses

# Information
## This template vs. Palo Alto Networks own
The big differences from what I've seen are:
- Availability zones
- HA ports for internal load balancing
- Managed disks
- Public LB instead of Application Gateway
- Able to easily define what ports to expose on the public LB and how many
- Possibility of linking together the Azure deployment with the Palo Alto configuration using Ansible

## Ansible vs. Panorama
To run Palo Alto Networks VMs in high availability (in Azure) you need to run Active-Active, and the simple way to sync the configuration is to use Panorama.
If you don't have Panorama, you have to do it manually - or write an automation like this. In this case we are able to link together the Azure deployment with the Palo Alto VM configuration making it work without additional licenses.

# Timings
## Clean deploy
Should take about 11 minutes to run

## Existing deploy
Should take about 4 minutes to run

# Roles
## azure-deploy-paloalto
Role to deploy the HA pair to Azure.

## paloalto-base-configuration
Role to configure the both nodes to work with Azure Load Balancing.

# Variables
## azure-prd.yml
Configuration for Azure settings

## paloalto-prd.yml
Only using this to set the environmentShort variable and import the {{ environmentShort }}.yml (prd.yml) from vars.

## prd.yml
Main configuration used by both roles. Configuration for the subnets in Azure, IP-addresses and Palo Alto settings.

## securityRules.yml
What security rules are being used. Should at least keep the first two rules to make sure Azure LB is able to send the health probes.

## staticRoutes.yml
What static routes are being used. We are using two different virtual routers and pointing them to eachother for the different subnets, as well as allowing the interfaces bound to them to use the correct route to Azure LB IP.
What is being used is about the minimum (internal-01/02/03 could be changed without causing issues, but make sure to configure them on both virtual routers).

## natRules.yml
Only configured a Source NAT to allow traffic from trust to untrust.

## staticDnsEntries.yml
Configure static DNS entries for the DNS Proxy.

# How to run
```bash
ansible-playbook -i hosts-prd site.yml -e AZURE_CLIENT_ID="<Client ID>" -e AZURE_SECRET='"<Secret>"' -e AZURE_SUBSCRIPTION_ID="<Subscription ID>" -e AZURE_TENANT="<Tenant ID>" -e adminPassword='"<Palo Alto VM password>"'
```