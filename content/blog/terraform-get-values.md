---
title: "Terraform Get Values"
date: 2020-07-23T13:42:59+01:00
draft: false
tags: ["terraform"]
---

With not wanting to have hard coded values pushed to a project's code repository, and an antiquated way to derive Azure Service Principal credentials, I set about exploring ways on accomplishing this with this in mind using Terraform.


### Attempt 1

Here ins my first attempt, I load all permutations into map variables.  I use an environment variable as an indexer to the appropriate map value:

```tf
provider "azurerm" {  
  version         = "=2.17"
  ...  
  subscription_id = var.azure_subscription_id[var.environment]     
  client_id       = var.azure_client_id[var.environment]   
  ...
}

variable "azure_subscription_id" {
  type = map
  default = {
    "dev" = "********-****-****-****-************"
    "prod"= "********-****-****-****-************"
  }
}

variable "azure_client_id" {
  type = map
  default = {
    "dev" = "********-****-****-****-************"
    "prod"= "********-****-****-****-************"
  }
}
...
```


### Attempt 2

This second and more efficient approach, I used `jsondecode` function to load the entire credentials JSON to access the subscriptionId property:

```tf
provider "azurerm" {  
  version         = "=2.17"
  ...  
  subscription_id   = var.environment == "dev" ? jsondecode(var.azure_sp_dev).subscriptionId : jsondecode(var.azure_sp_prod).subscriptionId
  client_id   = var.environment == "dev" ? jsondecode(var.azure_sp_dev).clientId : jsondecode(var.azure_sp_prod).clientId
  ...
}

variable "azure_sp_dev" {
  type = string
  default = <<EOT
  {
  "clientId": "********-****-****-****-************",
  "clientSecret": "********-****-****-****-************",
  "subscriptionId": "********-****-****-****-************",
  "tenantId": "********-****-****-****-************",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "activeDirectoryGraphResourceId": "https://graph.windows.net/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
  }
  EOT
}

variable "azure_sp_prod" {
  type = string
...
```

Obviously, none of the above deals with not pushing these credentials into the code repository.  

So, in my opinion, there are 2 options available here.  The first option is to use a GitHub Secret and to inject this secret into a script file.  It could even be passed as a parameter to Terraform (e.g. `terrafor apply -var credentials={...}` ).  Or, the second option is to obtain this key using the GitHub `Azure/get-keyvault-secrets@v1.0` Action.  This method will then allow you to obtain the Service Principal credentials from an Azure KeyVault.  This latter approach means that we never need to expose these secrets outside of Azure, which we would have to do if we cut & paste them into a GitHub Secret.