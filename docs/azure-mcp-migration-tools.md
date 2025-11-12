# Azure MCP Tools for Cloud Migration

## Overview
This document provides specific Azure MCP (Model Context Protocol) tool usage for each phase of cloud migration. Azure MCP tools are activated and ready to use for your migration project.

## Activated Azure MCP Tools

The following Azure MCP tools are now available:

1. **azure_auth-get_auth_context** - Get current Azure authentication context
2. **azure_auth-set_auth_context** - Configure Azure authentication
3. **azure_bicep-get_azure_verified_module** - Get Bicep IaC modules
4. **azure_development-recommend_custom_modes** - Get Azure-specific recommendations
5. **azure_dotnet_templates-get_tags** - List .NET templates
6. **azure_dotnet_templates-get_templates_for_tag** - Get specific .NET templates
7. **azure_resources-query_azure_resource_graph** - Query Azure resources

## Phase-Specific Tool Usage

### Discovery Phase Tools

#### Query Existing Azure Resources
Use Azure Resource Graph to discover existing resources:

**Example: List all VMs in subscription**
```bash
# Query for all VMs
az graph query -q "Resources | where type == 'microsoft.compute/virtualmachines' | project name, location, properties.hardwareProfile.vmSize"
```

**Example: Discover SQL Servers**
```bash
# Query for SQL servers
az graph query -q "Resources | where type == 'microsoft.sql/servers' | project name, location, properties"
```

**Example: Find all storage accounts**
```bash
# Query storage accounts
az graph query -q "Resources | where type == 'microsoft.storage/storageaccounts' | project name, location, sku.name"
```

### Assessment Phase Tools

#### Cost Estimation
Use Azure Pricing Calculator and Azure Cost Management:

```bash
# Get current month costs by resource group
az consumption usage list \
  --start-date $(date -u -d "1 month ago" +%Y-%m-%d) \
  --end-date $(date -u +%Y-%m-%d) \
  --query "[].{ResourceGroup:resourceGroup, Cost:pretaxCost, Currency:currency}" \
  --output table
```

#### Azure Advisor Recommendations
Get optimization recommendations:

```bash
# Get all recommendations
az advisor recommendation list --output table

# Get cost recommendations only
az advisor recommendation list --category Cost --output table

# Get high availability recommendations
az advisor recommendation list --category HighAvailability --output table
```

### Planning Phase Tools

#### Infrastructure as Code (Bicep)
Generate Bicep templates for target infrastructure:

**Get Azure Verified Modules:**
- Use `azure_bicep-get_azure_verified_module` for pre-built Bicep modules
- Available for common resources: VMs, SQL, Storage, Networking

**Example Bicep for VM:**
```bicep
param location string = resourceGroup().location
param vmName string
param vmSize string = 'Standard_D2s_v3'
param adminUsername string
@secure()
param adminPassword string

resource vm 'Microsoft.Compute/virtualMachines@2023-03-01' = {
  name: vmName
  location: location
  properties: {
    hardwareProfile: {
      vmSize: vmSize
    }
    osProfile: {
      computerName: vmName
      adminUsername: adminUsername
      adminPassword: adminPassword
    }
    // ... additional configuration
  }
}
```

#### Network Planning
Query and plan network resources:

```bash
# List all virtual networks
az network vnet list --output table

# Get available IP ranges
az network vnet list --query "[].{Name:name, AddressSpace:addressSpace.addressPrefixes}" --output table

# Check subnet availability
az network vnet subnet list --vnet-name <vnet-name> --resource-group <rg> --output table
```

### Execution Phase Tools

#### Migration Monitoring
Monitor migration progress using Azure Resource Graph:

```bash
# Check replication status
az migrate replication list --project-name <project> --resource-group <rg> \
  --query "[].{Machine:properties.machineName, Status:properties.replicationStatus}" \
  --output table

# Monitor VM creation
az vm list --resource-group <target-rg> \
  --query "[].{Name:name, State:provisioningState, Location:location}" \
  --output table
```

#### Resource Deployment
Deploy resources using Bicep or ARM templates:

```bash
# Deploy Bicep template
az deployment group create \
  --resource-group <rg> \
  --template-file main.bicep \
  --parameters @parameters.json

# Validate deployment
az deployment group validate \
  --resource-group <rg> \
  --template-file main.bicep
```

### Optimization Phase Tools

#### Right-Sizing Recommendations
Get VM sizing recommendations:

```bash
# Get Azure Advisor recommendations for VMs
az advisor recommendation list --category Cost \
  --query "[?contains(resourceMetadata.resourceId, 'virtualMachines')]" \
  --output table
```

#### Cost Analysis
Analyze costs by service:

```bash
# Cost by resource type
az consumption usage list \
  --start-date $(date -u -d "30 days ago" +%Y-%m-%d) \
  --end-date $(date -u +%Y-%m-%d) \
  --query "[].{Type:meterCategory, Cost:pretaxCost}" \
  --output table | awk '{arr[$1]+=$2} END {for (i in arr) print i, arr[i]}'

# Set up budget alerts
az consumption budget create \
  --budget-name "Monthly-Budget" \
  --amount 5000 \
  --category Cost \
  --time-grain Monthly
```

## Azure SQL MCP Tools

For database migration, use Azure SQL MCP tools:

### List SQL Resources
```bash
# List SQL servers in resource group
# Using MCP: sql_server_list
az sql server list --resource-group <rg> --output table

# List databases on server
# Using MCP: sql_db_list  
az sql db list --server <server-name> --resource-group <rg> --output table
```

### Database Migration
```bash
# Get database details
# Using MCP: sql_db_show
az sql db show --name <db-name> --server <server-name> --resource-group <rg>

# Update database SKU (scale)
# Using MCP: sql_db_update
az sql db update --name <db-name> --server <server-name> \
  --resource-group <rg> --service-objective S1
```

### Firewall Configuration
```bash
# List firewall rules
# Using MCP: sql_server_firewall-rule_list
az sql server firewall-rule list --server <server-name> --resource-group <rg>

# Create firewall rule
# Using MCP: sql_server_firewall-rule_create
az sql server firewall-rule create \
  --server <server-name> \
  --resource-group <rg> \
  --name AllowMigrationServer \
  --start-ip-address 10.0.1.10 \
  --end-ip-address 10.0.1.10
```

## Automation Scripts Using Azure MCP

### Complete Server Migration Script
```bash
#!/bin/bash
# migrate-server.sh - Automated server migration using Azure MCP

set -e

PROJECT_NAME="migration-project"
RESOURCE_GROUP="rg-migration"
SOURCE_MACHINE="web-server-01"
TARGET_RG="rg-azure-production"
TARGET_VNET="vnet-production"

echo "=== Azure Cloud Migration Automation ==="

# 1. Authenticate
echo "Authenticating with Azure..."
az login --use-device-code

# 2. Verify machine discovered
echo "Verifying machine discovered..."
MACHINE_ID=$(az migrate machine list \
  --project-name $PROJECT_NAME \
  --resource-group $RESOURCE_GROUP \
  --query "[?name=='$SOURCE_MACHINE'].id" -o tsv)

if [ -z "$MACHINE_ID" ]; then
  echo "ERROR: Machine $SOURCE_MACHINE not found in discovery"
  exit 1
fi

echo "Machine found: $MACHINE_ID"

# 3. Enable replication
echo "Enabling replication..."
az migrate machine replicate \
  --project-name $PROJECT_NAME \
  --resource-group $RESOURCE_GROUP \
  --source-machine-id $MACHINE_ID \
  --target-resource-group $TARGET_RG \
  --target-network $TARGET_VNET

# 4. Monitor replication
echo "Monitoring replication status..."
while true; do
  STATUS=$(az migrate replication show \
    --project-name $PROJECT_NAME \
    --resource-group $RESOURCE_GROUP \
    --machine-name $SOURCE_MACHINE \
    --query "properties.replicationStatus" -o tsv)
  
  echo "Replication status: $STATUS"
  
  if [ "$STATUS" == "Synced" ]; then
    echo "Replication complete!"
    break
  fi
  
  sleep 60
done

# 5. Test migration
echo "Performing test migration..."
az migrate replication test-migrate \
  --project-name $PROJECT_NAME \
  --resource-group $RESOURCE_GROUP \
  --machine-name $SOURCE_MACHINE

echo "Test migration initiated. Please validate the VM in Azure Portal."
echo "After validation, run cleanup and final migration."

echo "=== Migration automation complete ==="
```

### Cost Monitoring Script
```bash
#!/bin/bash
# monitor-costs.sh - Daily cost monitoring

# Get yesterday's costs
YESTERDAY=$(date -u -d "1 day ago" +%Y-%m-%d)
TODAY=$(date -u +%Y-%m-%d)

echo "=== Azure Cost Report for $YESTERDAY ==="

# Total cost
TOTAL=$(az consumption usage list \
  --start-date $YESTERDAY \
  --end-date $TODAY \
  --query "sum([].pretaxCost)" -o tsv)

echo "Total Cost: $$TOTAL"

# Cost by resource group
echo ""
echo "Cost by Resource Group:"
az consumption usage list \
  --start-date $YESTERDAY \
  --end-date $TODAY \
  --query "[].{ResourceGroup:resourceGroup, Cost:pretaxCost}" \
  --output table

# Budget check
BUDGET=5000
MONTHLY_SPEND=$(az consumption usage list \
  --start-date $(date -u -d "30 days ago" +%Y-%m-%d) \
  --end-date $TODAY \
  --query "sum([].pretaxCost)" -o tsv)

echo ""
echo "Monthly Budget: $$BUDGET"
echo "Monthly Spend: $$MONTHLY_SPEND"

PERCENT=$(echo "scale=2; $MONTHLY_SPEND / $BUDGET * 100" | bc)
echo "Budget Used: $PERCENT%"

if (( $(echo "$PERCENT > 80" | bc -l) )); then
  echo "WARNING: Budget threshold exceeded!"
fi
```

## Best Practices for Azure MCP Tools

### Authentication Best Practices
1. **Use Service Principal for automation:**
   ```bash
   az login --service-principal \
     --username $AZURE_CLIENT_ID \
     --password $AZURE_CLIENT_SECRET \
     --tenant $AZURE_TENANT_ID
   ```

2. **Use Managed Identity in Azure VMs:**
   ```bash
   az login --identity
   ```

3. **Store credentials securely:**
   - Use Azure Key Vault for secrets
   - Never commit credentials to Git
   - Use environment variables

### Query Optimization
1. **Use filters to reduce results:**
   ```bash
   az graph query -q "Resources | where type == 'microsoft.compute/virtualmachines' and location == 'eastus'"
   ```

2. **Project only needed fields:**
   ```bash
   az graph query -q "Resources | project name, type, location"
   ```

3. **Use subscriptions scope when needed:**
   ```bash
   az graph query -q "Resources | where type =~ 'microsoft.storage/storageaccounts'" \
     --subscriptions <subscription-id>
   ```

### Error Handling
```bash
# Check command success
if az vm show --name <vm> --resource-group <rg> &>/dev/null; then
  echo "VM exists"
else
  echo "VM not found"
fi

# Capture errors
ERROR_MSG=$(az vm create ... 2>&1)
if [ $? -ne 0 ]; then
  echo "ERROR: $ERROR_MSG"
  # Send alert or log
fi
```

## Integration with GitHub

### Automated Reporting
Set up GitHub Actions to run Azure MCP queries:

```yaml
name: Azure Cost Report

on:
  schedule:
    - cron: '0 9 * * *'  # Daily at 9 AM

jobs:
  cost-report:
    runs-on: ubuntu-latest
    steps:
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Generate Cost Report
        run: |
          az consumption usage list \
            --start-date $(date -u -d "1 day ago" +%Y-%m-%d) \
            --end-date $(date -u +%Y-%m-%d) \
            --output table > cost-report.txt
      
      - name: Create Issue
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: 'Daily Azure Cost Report',
              body: require('fs').readFileSync('cost-report.txt', 'utf8')
            })
```

## Troubleshooting Azure MCP Tools

### Common Issues

**Issue: Authentication Failed**
```bash
# Solution: Re-authenticate
az account clear
az login
az account show
```

**Issue: Insufficient Permissions**
```bash
# Check current permissions
az role assignment list --assignee <user-id> --output table

# Request required role
# Contributor: Full resource management
# Reader: Read-only access
```

**Issue: Resource Not Found**
```bash
# Verify subscription context
az account show

# Switch subscription
az account set --subscription <subscription-id>

# List all accessible subscriptions
az account list --output table
```

## Next Steps

1. **Authenticate with Azure:**
   ```bash
   az login
   az account show
   ```

2. **Test Azure MCP queries:**
   ```bash
   az graph query -q "Resources | summarize count() by type"
   ```

3. **Review available tools:**
   - Azure Resource Graph for queries
   - Azure SQL MCP for database operations
   - Azure Advisor for recommendations
   - Bicep for Infrastructure as Code

4. **Create automation scripts:**
   - Discovery automation
   - Assessment reporting
   - Migration monitoring
   - Cost tracking

5. **Integrate with GitHub:**
   - Set up GitHub Actions
   - Automate reporting
   - Track migration progress

## Resources

- [Azure CLI Reference](https://learn.microsoft.com/cli/azure/)
- [Azure Resource Graph Query Language](https://learn.microsoft.com/azure/governance/resource-graph/concepts/query-language)
- [Azure Bicep Documentation](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
- [Azure MCP GitHub](https://github.com/Azure/azure-mcp)
