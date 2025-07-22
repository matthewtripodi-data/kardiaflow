# KardiaFlow Infrastructure Deployment Guide

This folder contains the infrastructure-as-code (IaC) scripts for deploying and
tearing down the minimal KardiaFlow development environment in Azure using Bicep
and the Azure CLI.

---

## What It Deploys

- Azure Resource Group (`kardia-rg-dev`)
- Azure Data Lake Storage Gen2 account (`kardiaadlsdemo`) with container (`raw`)
- Azure Databricks Workspace (`kardia-dbx`) with managed resource group

Designed for local development and demos. No NAT Gateway, no Unity Catalog, no VNet injection, no Key Vault.

---

## Deploy Instructions

> **Run these from your local terminal. Make sure you're logged into the correct Azure subscription.**

# 1. Load environment variables
source infra/.env

# 2. Create the resource group
az group create --name "$RG" --location eastus

# 3. Deploy Databricks and ADLS via Bicep
az deployment group create \
  --resource-group "$RG" \
  --template-file infra/deploy.bicep \
  --name "$DEPLOY"

# 4. Generate PAT via Databricks UI.

# 5. Export PAT via local IDE:
export DATABRICKS_PAT=""

# 6. Generate & store SAS in Databricks:
bash infra/gen_sas.sh

# 7. Teardown Instructions. To safely destroy all provisioned resources, run:
./infra/teardown.sh

The teardown script script will:

- Delete the Databricks workspace (which deletes the managed RG too)
- Delete the main resource group (kardia-rg-dev)
- Print a confirmation message
- Resources will disappear over the next 2–5 minutes.

---

# Dry-Run Deployment

To preview what the deployment will do without actually creating resources:

az deployment group what-if \
  --resource-group kardia-rg-dev \
  --template-file automation/infra/deploy.bicep
