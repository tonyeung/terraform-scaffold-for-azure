# Terraform scaffold for Azure

This repo contains everything to get started with Terraform on Azure.

## What you will get

After executing the below steps you will get:

* a service principal used to run Terraform on behalf
* a Storage Container used to store the Terraform state file
* a Key Vault containing all secrets to allow easy and secure access

## Requirements

This project requires the following:

* a Unix-Shell or Powershell  (you can use [Azure Cloud Shell](http://shell.azure.com/))
* Azure CLI (authenticated)
* the executing user needs Subscription owner access (to allow Terraform to add future Service Principals as contributors) as well as the Application Developer role in AAD (to create the Service Principal)

## Get started with Unix-Shell

Execute the following steps to get started:

1. Authenticate against Azure by executing `az login`
2. Optional: Export your Tenant (`tenantId`) and Subscription ID (`subscriptionId`) if you don't like to deploy with your `az` defaults.
3. Customize `.env` based on your needs and naming conventions (Make sure you met all [Azure naming rules and restrictions](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-name-rules)).
4. Execute `up.sh` to deploy everything needed
5. Grant admin consent for the created app registrations (Terraform will then be allowed to create app registrations and groups in Azure AD). This needs Azure Active Directory global admin access. Find more details on how to grant consent [here](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/grant-admin-consent).

## Get started with Powershell

Execute the following steps to get started:

1. Authenticate against Azure by executing `az login`
2. Optional: Create environment variables for Tenant (`tenantId`) and Subscription ID (`subscriptionId`) if you don't like to deploy with your `az` defaults.
3. Customize `config.json` based on your needs and naming conventions (Make sure you met all [Azure naming rules and restrictions](https://docs.microsoft.com/azure/azure-resource-manager/management/resource-name-rules)).
4. Execute `up.ps1` to deploy everything needed
5. Grant admin consent for the created app registrations (Terraform will then be allowed to create app registrations and groups in Azure AD). This needs Azure Active Directory global admin access. Find more details on how to grant consent [here](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/grant-admin-consent).


## Scaffold a Terraform project

You will need to tell Terraform where to store its state file. To do so, you need to customize your `main.tf` file based on the below example:

```
provider "azurerm" {
  version = "~> 2.66"
  features {}
}

terraform {
    backend "azurerm" {
        key = "azure.tfstate"
    }
}
```

We do not recommend to store any secrets and credentials in code. Therefore everything needed will be requested from Key Vault as needed. To init you project run the following script:

``` Bash
#!/bin/bash

# customize your subscription id and resource group name
export subscriptionId="00000000-0000-0000-0000-000000000000"
export rg="my-rg"

# sets subscription;
az account set --subscription $subscriptionId

# get vault
export vaultName=$(az keyvault list --subscription=$subscriptionId -g $rg --query '[0].{name:name}' -o tsv)

## extracts and exports secrets
export saKey=$(az keyvault secret show --subscription=$subscriptionId --vault-name="$vaultName" --name sa-key --query value -o tsv)
export saName=$(az keyvault secret show --subscription=$subscriptionId --vault-name="$vaultName" --name sa-name --query value -o tsv)
export scName=$(az keyvault secret show --subscription=$subscriptionId --vault-name="$vaultName" --name sc-name --query value -o tsv)
export spSecret=$(az keyvault secret show --subscription=$subscriptionId --vault-name="$vaultName" --name sp-secret --query value -o tsv)
export spId=$(az keyvault secret show --subscription=$subscriptionId --vault-name="$vaultName" --name sp-id --query value -o tsv)

# exports secrets
export ARM_SUBSCRIPTION_ID=$subscriptionId
export ARM_TENANT_ID=$tenantId
export ARM_CLIENT_ID=$spId
export ARM_CLIENT_SECRET=$spSecret

# runs Terraform init
terraform init -input=false \
  -backend-config="access_key=$saKey" \
  -backend-config="storage_account_name=$saName" \
  -backend-config="container_name=$scName"
```

## Azuread provider configuration

```
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~>1.6"
    }
  }
}
provider "azuread" {
  use_microsoft_graph = true
}
```

## Disclaimer

The `up.sh` script ask you whether you would like to map our Partner ID to the created Service Principal. Feel free to opt-out or remove the marked lines if you don't like to support us.

> THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
