# Assessment Phase

## Objective
Analyze discovered workloads to determine Azure readiness, identify the optimal target Azure services, estimate costs, and uncover any migration blockers.

## Assessment Types

### 1. Azure VM Assessment
**Purpose**: Determine the right-sized Azure VMs for on-premises servers

**Key Outputs**:
- Azure VM SKU recommendations
- Monthly cost estimates
- Azure readiness status
- Identified issues and remediation

**Assessment Criteria**:
- CPU utilization
- Memory utilization
- Disk IOPS and throughput
- Network requirements
- Storage requirements

### 2. Azure SQL Assessment
**Purpose**: Assess SQL Server instances for migration to Azure SQL

**Target Options**:
- Azure SQL Database
- Azure SQL Managed Instance
- SQL Server on Azure VM

**Key Outputs**:
- Recommended Azure SQL target
- Compatibility issues
- Feature parity analysis
- Cost comparison

### 3. Azure App Service Assessment
**Purpose**: Evaluate web applications for Azure App Service

**Supported Applications**:
- ASP.NET web apps
- Java web applications
- PHP applications

**Assessment Factors**:
- Application compatibility
- Dependencies
- Configuration requirements
- Recommended App Service tier

### 4. Azure VMware Solution (AVS) Assessment
**Purpose**: Assess for lift-and-shift VMware migration

**Use Case**: When you want to migrate VMware VMs without re-platforming

## Performing Assessments

### Step 1: Group Servers
Create logical groups based on:
- Application tiers (web, app, database)
- Business function
- Migration waves
- Dependencies

### Step 2: Configure Assessment Settings

**Sizing Criteria**:
- Performance-based (recommended): Uses historical performance data
- As on-premises: Uses current allocated resources

**Comfort Factor**: 
- Buffer for growth (1.0x to 2.0x)
- Recommended: 1.2x to 1.3x

**Azure Offer**:
- Pay-as-you-go
- Enterprise Agreement
- CSP

**Reserved Instances**:
- None
- 1-year reserved
- 3-year reserved

**Target Location**:
- Select Azure region closest to users
- Consider data residency requirements

### Step 3: Run Assessment
1. Navigate to discovered servers
2. Select servers or groups
3. Click "Assess"
4. Choose assessment type
5. Configure assessment properties
6. Review and create assessment

### Step 4: Review Assessment Results

**Readiness Categories**:
- ✅ **Ready**: No issues, can migrate as-is
- ⚠️ **Ready with conditions**: Minor issues, can migrate with changes
- ❌ **Not ready**: Blockers exist, requires remediation
- ❓ **Readiness unknown**: Insufficient data

**Cost Analysis**:
- Compute costs
- Storage costs
- Network costs
- Total monthly estimate
- Potential savings vs. on-premises

## Assessment Best Practices

### 1. Performance Data Collection
- Collect at least **7-14 days** of performance data
- Include peak usage periods
- Consider seasonal variations

### 2. Right-Sizing Strategy
- Use **95th percentile** for sizing (filters out spikes)
- Apply appropriate comfort factor
- Consider future growth

### 3. Cost Optimization
- Evaluate **Azure Hybrid Benefit** for Windows/SQL licensing
- Consider **Reserved Instances** for predictable workloads
- Use **Dev/Test pricing** where applicable

### 4. Dependency Mapping
- Enable **dependency analysis** for application grouping
- Identify inter-server communication
- Plan migration waves based on dependencies

## Assessment Reports

### Generate Reports
- Azure readiness summary
- Cost estimate comparison
- Recommended SKUs by server
- Issues and blockers list

### Export Options
- Excel export for offline analysis
- CSV for data processing
- Share with stakeholders

## Common Assessment Issues

| Issue | Description | Remediation |
|-------|-------------|-------------|
| Unsupported OS | OS version not supported in Azure | Upgrade OS before migration |
| Boot type | BIOS boot on large disk (>2TB) | Convert to UEFI or resize disk |
| Conditional readiness | Temporary disk warning | Use Azure VM with temp disk |
| Network adapters | Too many NICs | Reduce to supported limit |
| Disk size/count | Exceeds Azure limits | Consolidate or resize disks |

## Decision Matrix

### Migration Strategy Decision

| Scenario | Recommended Approach |
|----------|---------------------|
| VMware VMs, no app changes needed | Azure VMware Solution (AVS) |
| Windows/Linux VMs, willing to re-platform | Azure VMs (IaaS) |
| SQL Server databases | Azure SQL MI or Azure SQL DB |
| Web applications (.NET, Java) | Azure App Service (PaaS) |
| Containerized apps | Azure Kubernetes Service (AKS) |
| Large databases (>10TB) | SQL Server on Azure VM |

## Next Phase
After completing assessments and addressing blockers, proceed to [Migration Planning](./03-migration-planning.md)

## Tools & Resources
- [Data Migration Assistant (DMA)](https://www.microsoft.com/download/details.aspx?id=53595) - SQL assessment tool
- [App Service Migration Assistant](https://appmigration.microsoft.com/) - Web app assessment
- [Azure Total Cost of Ownership (TCO) Calculator](https://azure.microsoft.com/pricing/tco/calculator/)
