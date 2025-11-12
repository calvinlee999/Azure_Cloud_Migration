# Discovery Phase

## Objective
Identify and inventory all on-premises servers, applications, databases, and their dependencies to create a comprehensive baseline for migration planning.

## Prerequisites
- Azure subscription with appropriate permissions
- Network access to on-premises environment
- vCenter Server credentials (for VMware environments)
- Read-only account for discovery

## Steps

### 1. Create Azure Migrate Project
1. Navigate to Azure Portal > Azure Migrate
2. Click "Create project"
3. Select subscription and resource group
4. Choose project name and region
5. Note the project key

### 2. Deploy Azure Migrate Appliance
**For VMware vSphere:**
- Download OVA template (12GB)
- Deploy to vCenter Server
- Requirements: 32GB memory, 8 vCPUs, ~80GB storage
- Configure appliance with project key

**Alternative: PowerShell script (500MB)**
- For existing Windows Server 2022 VM
- Execute deployment script

### 3. Configure Discovery
1. Access appliance configuration manager
2. Provide vCenter Server credentials (read-only)
3. Specify scope (which servers to discover)
4. Enable performance data collection
5. Initiate discovery process

### 4. Discovery Options

#### Server Discovery
- Physical servers
- Virtual machines (VMware, Hyper-V)
- Cloud instances (AWS EC2, GCP CE)

#### Discovery Methods
- **CSV Import**: Quick assessment, manual inventory
- **VMware Inventory (RVTools XLSX)**: Automated VMware discovery
- **SAP® Inventory (XLS)**: SAP system discovery

### 5. Wait for Discovery Completion
Discovery timeline varies based on:
- Number of servers to discover
- Network latency
- Performance counter collection

Expected timeline:
- Small environments (<100 servers): Few hours
- Medium environments (100-500): Several hours
- Large environments (>500): 24-48 hours

## Discovery Outputs
- Complete server inventory with specifications
- Application dependencies map
- Performance data baseline
- SQL Server instances
- Web applications
- Installed software inventory

## Best Practices
1. **Use read-only accounts** to minimize security risks
2. **Scope discovery** to target servers only
3. **Collect performance data** for at least 7-14 days for accurate sizing
4. **Document dependencies** for application grouping
5. **Tag servers** by application, owner, or business function

## Common Issues
- **Credentials**: Ensure proper permissions
- **Network**: Verify connectivity between appliance and vCenter
- **Performance**: Check appliance resources if discovery is slow

## Next Phase
After discovery completion, proceed to [Assessment Phase](./02-assessment-phase.md)
