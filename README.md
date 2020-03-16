"# referenceARMtemplates" 

Reference infrastructure formatted for use from Azure CLI.

## Preparation

Install the Azure CLI
https://docs.microsoft.com/en-gb/cli/azure/install-azure-cli

Log in to Azure with the following command.
az login

Provide your atlascity.io login credentials

## Installing infrastructure
Open the script file in the Scripts directory for your target environment.

deploy_PRD.azcli - Production environment script
deploy_CI.azcli  - Continuous installation environment script
deploy_TST.azcli - Test environment script

You can create other environments by creating a new script .azcli file, copying the content from one of the other scripts and editing the target environment letters.

For the remainder of this document we will reference the deploy_PRD.azcli production script.

Within this script are the series of commands to issue to stand up the environment.
Copy the first line to your command prompt and execute it.

az group create -n "ATLAS-PRD-VNET" -l "NorthEurope" 
This line creates a new resource group in NorthEurope with the name ATLAS-PRD-VNET

az deployment group create -g "ATLAS-PRD-VNET" --template-file ".\Templates\atlas-vnet.deploy.json" --parameters @".\Parameters\PRD\VNET.params.json" --verbose
This line runs the template atlas-vnet.deploy.json with the configuration parameters stored in VNET.params.json

## Template descriptions

#VNET
Deploys a network with subnets for partitioning the infrastructure. Each VM set is within its own subnet with its own Network Security Group (NSG) which allows us to tightly control access between each zone of the network.

If creating a new type of server that needs its own NSG then be sure to edit the atlas-vnet.deploy.json template to support it.

#DC
Deploys a server pair to be configured to act as a Domain Controller (DC).
This server pair is only required if the intention is to set up Active Directory for convenient management of the whole virtualised server estate.

#MGMT
A management jump box into the environment.
This machine is shut down when not required. When someone needs to RDP into the infrastructure then this box is started and kept live only for the duration of the administration work. Importantly, the NSG needs to have tight security rules to only permit RDP from trusted IP's or should be placed behind a firewall to further restrict access to the machine when running.

#EXWA
Web servers intended to be accessed from the Internet.
These servers are load balanced. However, a better configuration would be not load balance these with the standard Azure load balancers but from a Barracuda WAF which not only load balances but provides advanced security rules. You should only have one set of loadbalancers for each zone and so if using Barracuda's then change the script file to use the non lb template.

#BITCOIN
#ETHEREUM
#CATALYST
These are VM's designed to run nodes for different blockchain networks.

There needs to be some further work here since blockchain nodes have some unusual properties.
TODO:
* Change to the lb template to make nodes load balanced when making RPC calls from within the infrastructure. External access should be through Barracuda's and so the load balancing would come from the WAF rather than the standard Azure load balancers. This mix of load balancers will need testing to find the right configuration.

* Change the data disks to managed disks to allow snapshots to be made of the blockchain history.
Consider changing all the templates to use managed disks by default.

#BCD
A load balanced SQL server pair.

# Further notes
The above ARM template script stands up a mix of VM's within a secure infrastructure running its own secure private network, partitioned with NSG's with different security rules at each boundary.

The intention is to use this as a reference for future projects.

For example, we may want to just run blockchain node sets in a non-load balanced configuration behind Barracuda WAF's that act as public facing secure load balanced nodes. This would provide us with trusted and stable nodes for use in our own projects such as the wallet.

Alternatively, we may want to do run nodes like this but block RPC calls from anywhere except from our office or our Azure services, depending on the particular use case.

Where possible to reduce maintenance we should strive to use PaaS in the cloud but there may be instances where we want to construct some secure infrastructure and so these templates can be used at these times.
