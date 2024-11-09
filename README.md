# Arc Deployment by GPO
Arc GPO Deployment project contains the necessary files to onboard Non Azure machines to Azure Arc automatically, using a GPO, either using a Service principal with a secret or a certificate.

More information can be found [here](https://learn.microsoft.com/en-us/azure/azure-arc/servers/onboard-group-policy-service-principal-encryption)

## Content

- [DeployGPO.ps1](DeployGPO.ps1): PowerShell script to deploy the GPO in a certain AD domain
- [EnableAzureArc.ps1](EnableAzureArc.ps1): PowerShell script that has to be placed in the network share and will execute the onboarding process.
- [RenewSPSecretDPAPI.ps1](RenewSPSecret.ps1): PowerShell script to renew the secret from the Service Principal used for the onboard of Azure Arc Servers.
- [ParseArcOnboardingPrerequisites.ps1](ParseArcOnboardingPrerequisites.ps1): PowerShell scripts that parses the information of the machines that didn't meet the onboard requirements.
- [ArcGPO](ArcGPO): Folder structure that contains the GPO settings to be imported in AD
- [ARMTemplates](ARMTemplates): Folder with Azure Function Template to monitor Azure Arc Agent version updates.
- [Workbooks](Workbooks): Folder with Azure Workbooks to monitor your Azure Arc onboarding Status and your Azure Arc Servers
- [ScheduledTask](ScheduledTask): Folder with a scheduled task that can, programmatically, upload on-prem XMLs report files to Azure Log Analytics

## Prerequisites

- Create a *Service Principal* and give it Azure Arc onbarding permissions, following this article: [Create a Service Principal for onboarding at scale](https://docs.microsoft.com/en-us/azure/azure-arc/servers/onboard-service-principal#create-a-service-principal-for-onboarding-at-scale)

- Decide if the *Service Principal* should use a client secret or a certificate. The secret is mandatory at the creation of the Service Principal. You can add a certificate afterwards and remove the secret again, if you wish not to use it. The certificate file (PCKS #12 (.PFX) files and ASCII-encoded files (such as .PEM) are supported) should be copied to the *AzureArcOnboard* share that will be created in the next steps. More information to the certificate based option can be found in this article: [CLI Reference azcmagent connect usind a service principal with certificate](https://learn.microsoft.com/en-us/azure/azure-arc/servers/azcmagent-connect#service-principal-with-certificate)
  
- Register *Microsoft.HybridCompute*, *Microsoft.GuestConfiguration* and *Microsoft.HybridConnectivity* as resource providers in your subscription, following this article: [Register Resource Provider](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider)

- Create a *Network Share*,e.g. *AzureArcOnboard* that will be used for deployment and reporting files, with the following permissions:

  *Domain Controllers*, *Domain Computers* and *Domain Admins*: Change Permissions

 
- Copy the *'AzureConnectedMachineAgent.msi'* file to the shared folder.

    You can download it from https://aka.ms/AzureConnectedMachineAgent

## Installation

### Group Policy Deployment

- Copy the project structure to a local folder of a Domain Controller.

- If you want the on-boarding to happen with the the *Service Principal Secret*, execute the deployment script *DeployGPO.ps1*, with the following syntax: 

        .\DeployGPO.ps1 -DomainFQDN contoso.com -ReportServerFQDN Server.contoso.com -ArcRemoteShare AzureArcOnBoard -ServicePrincipalSecret $ServicePrincipalSecret 
       -ServicePrincipalClientId $ServicePrincipalClientId -SubscriptionId $SubscriptionId --ResourceGroup $ResourceGroup -Location $Location -TenantId $TenantId 
       [-AgentProxy $AgentProxy]

  If you would like to use a *Service Principal Certificate* for on-boarding, execute the same script with the following syntax instead:
  
        .\DeployGPO.ps1 -DomainFQDN contoso.com -ReportServerFQDN Server.contoso.com -ArcRemoteShare AzureArcOnBoard -ServicePrincipalCert $ServicePrincipalCert 
       -ServicePrincipalClientId $ServicePrincipalClientId -SubscriptionId $SubscriptionId --ResourceGroup $ResourceGroup -Location $Location -TenantId $TenantId 
       [-AgentProxy $AgentProxy]

    Where:

    - *DomainFQDN* is the fully qualified domain name of the domain
    
    - *ReportServerFQDN* is the Fully Qualified Domain Name of the host where the network share resides.
    
    - *ArcRemoteShare* is the name of the network share you've created
  
    - *ServicePrincipalSecret* is the secret from the Service Principal created previously.

    - *ServicePrincipalCert* is the path to the certificate (*.pem file) from the Service Principal created previously. (e.g. "\\\dc.contoso.local\AzureArcOnBoard\cert.pem")
    
    - *ServicePrincipalClientId* is the client id from the Service Principal created previously

    - *SubscriptionId*, *ResourceGroup*, *Location*, *TenantId* corespond to where your Arc Servers are going to be onboarded

    - *AgentProxy* [optional] is the name of the proxy if used



