# Azure MCP Tools Guide

## Overview
This document provides guidance on using Azure MCP (Model Context Protocol) tools throughout the Azure Cloud Migration process.

## What is Azure MCP?
Azure MCP provides a unified interface to interact with Azure services programmatically, enabling:
- Resource discovery and assessment
- Migration planning and execution
- Cost analysis and optimization
- Security and compliance monitoring
- Infrastructure as Code (IaC) deployment

## Available Azure MCP Commands

### Resource Management
```bash
# List all resource groups
az group list --output table

# List resources in a resource group
az resource list --resource-group <rg-name> --output table

# Get resource details
az resource show --ids <resource-id>
```

### Azure Migrate Operations
```bash
# List Azure Migrate projects
az migrate project list

# Get project details
az migrate project show --name <project-name> --resource-group <rg-name>

# List discovered servers
az migrate machine list --project-name <project-name> --resource-group <rg-name>

# Create assessment
az migrate assessment create --project-name <project-name> \
  --group-name <group-name> \
  --assessment-name <assessment-name> \
  --resource-group <rg-name>
```

### VM Operations
```bash
# List VMs
az vm list --output table

# Start VM
az vm start --name <vm-name> --resource-group <rg-name>

# Stop VM
az vm stop --name <vm-name> --resource-group <rg-name>

# Deallocate VM (stop billing)
az vm deallocate --name <vm-name> --resource-group <rg-name>

# Resize VM
az vm resize --name <vm-name> --resource-group <rg-name> --size Standard_DS3_v2
```

### Database Operations
```bash
# List SQL servers
az sql server list --output table

# Create SQL database
az sql db create --name <db-name> --server <server-name> \
  --resource-group <rg-name> --service-objective S0

# List databases
az sql db list --server <server-name> --resource-group <rg-name>
```

### Networking
```bash
# List virtual networks
az network vnet list --output table

# Create VPN gateway
az network vnet-gateway create --name <gateway-name> \
  --resource-group <rg-name> \
  --vnet <vnet-name> \
  --gateway-type Vpn \
  --sku VpnGw1 \
  --vpn-type RouteBased

# List network security groups
az network nsg list --output table
```

### Cost Management
```bash
# Get cost analysis
az consumption usage list --start-date 2025-01-01 --end-date 2025-01-31

# List budgets
az consumption budget list --resource-group <rg-name>

# Create budget
az consumption budget create --budget-name <budget-name> \
  --amount 5000 \
  --category Cost \
  --time-grain Monthly
```

### Monitoring and Diagnostics
```bash
# List metrics
az monitor metrics list --resource <resource-id> \
  --metric-names "Percentage CPU"

# Create alert rule
az monitor metrics alert create --name <alert-name> \
  --resource-group <rg-name> \
  --scopes <resource-id> \
  --condition "avg Percentage CPU > 80"

# List activity logs
az monitor activity-log list --resource-group <rg-name>
```

## Using Azure MCP in Migration Phases

### Discovery Phase
```bash
# Deploy discovery appliance
# (Note: This is typically done through Azure Portal, but configuration can be automated)

# List discovered servers
az migrate machine list --project-name <project> --resource-group <rg>

# Export discovery data
az migrate machine list --project-name <project> --resource-group <rg> \
  --query "[].{Name:name, OS:operatingSystem, CPUs:numberOfCores, Memory:memoryInMB}" \
  --output table
```

### Assessment Phase
```bash
# Create server group for assessment
az migrate group create --project-name <project> \
  --name <group-name> \
  --resource-group <rg>

# Add machines to group
az migrate group update --project-name <project> \
  --name <group-name> \
  --resource-group <rg> \
  --machines <machine-ids>

# Run assessment
az migrate assessment create --project-name <project> \
  --group-name <group-name> \
  --name <assessment-name> \
  --resource-group <rg>

# View assessment results
az migrate assessment show --project-name <project> \
  --group-name <group-name> \
  --name <assessment-name> \
  --resource-group <rg>
```

### Migration Execution
```bash
# Enable replication for VM
az migrate machine replicate --project-name <project> \
  --source-machine-id <machine-id> \
  --target-resource-group <target-rg> \
  --target-network <vnet-id>

# Check replication status
az migrate replication show --project-name <project> \
  --machine-name <machine-name> \
  --resource-group <rg>

# Test migration
az migrate replication test-migrate --project-name <project> \
  --machine-name <machine-name> \
  --resource-group <rg>

# Perform final migration
az migrate replication migrate --project-name <project> \
  --machine-name <machine-name> \
  --resource-group <rg>
```

### Optimization Phase
```bash
# Get VM recommendations from Advisor
az advisor recommendation list --category Cost

# Apply recommendations
az vm resize --name <vm-name> --resource-group <rg> \
  --size <recommended-size>

# Enable Azure Hybrid Benefit
az vm update --resource-group <rg> --name <vm-name> \
  --license-type Windows_Server

# Configure auto-shutdown
az vm auto-shutdown --resource-group <rg> --name <vm-name> \
  --time 1900 --timezone "Pacific Standard Time"
```

## Automation Scripts

### Bulk VM Migration Script
```bash
#!/bin/bash
# migrate-vms.sh - Bulk VM migration script

PROJECT="my-migration-project"
RG="migration-rg"
TARGET_RG="azure-production-rg"

# Get list of VMs to migrate
VMS=$(az migrate machine list --project-name $PROJECT --resource-group $RG \
  --query "[?properties.migrationState=='NotMigrated'].name" -o tsv)

# Enable replication for each VM
for VM in $VMS; do
  echo "Enabling replication for $VM..."
  az migrate machine replicate \
    --project-name $PROJECT \
    --source-machine-id $VM \
    --target-resource-group $TARGET_RG \
    --resource-group $RG
done

echo "Replication enabled for all VMs"
```

### Cost Report Script
```bash
#!/bin/bash
# cost-report.sh - Generate cost report

START_DATE="2025-01-01"
END_DATE="2025-01-31"
OUTPUT_FILE="cost-report-$(date +%Y%m%d).csv"

# Get usage data
az consumption usage list \
  --start-date $START_DATE \
  --end-date $END_DATE \
  --query "[].{Date:usageStart, Resource:instanceName, Cost:pretaxCost, Currency:currency}" \
  --output csv > $OUTPUT_FILE

echo "Cost report saved to $OUTPUT_FILE"
```

## Best Practices for Azure MCP

1. **Authentication**
   - Use Azure CLI login: `az login`
   - Use service principal for automation: `az login --service-principal`
   - Enable MFA for user accounts

2. **Resource Naming**
   - Follow naming conventions consistently
   - Use tags for organization and cost tracking
   - Document naming standards

3. **Error Handling**
   - Always check command exit codes
   - Log errors for troubleshooting
   - Implement retry logic for transient failures

4. **Security**
   - Use least-privilege RBAC roles
   - Store secrets in Azure Key Vault
   - Enable audit logging

5. **Performance**
   - Use parallel execution for bulk operations
   - Filter queries to reduce output
   - Cache frequently accessed data

## Troubleshooting

### Common Issues

**Authentication Errors**
```bash
# Clear cached credentials
az account clear

# Re-login
az login
```

**Resource Not Found**
```bash
# Verify subscription
az account show

# Set correct subscription
az account set --subscription <subscription-id>

# Verify resource group
az group exists --name <rg-name>
```

**Permission Denied**
```bash
# Check current role assignments
az role assignment list --assignee <user@domain.com>

# Request necessary permissions from admin
```

## Additional Resources

- [Azure CLI Documentation](https://learn.microsoft.com/cli/azure/)
- [Azure Migrate REST API](https://learn.microsoft.com/rest/api/migrate/)
- [Azure PowerShell Modules](https://learn.microsoft.com/powershell/azure/)
- [Azure SDK for Python](https://learn.microsoft.com/azure/developer/python/)

## Next Steps

1. Install Azure CLI if not already installed
2. Authenticate with your Azure subscription
3. Explore available commands for your migration phase
4. Create automation scripts for repetitive tasks
5. Integrate with CI/CD pipelines for IaC deployments
