# Migration Execution Phase

## Objective
Execute the migration plan with minimal downtime, ensuring successful transfer of workloads to Azure while maintaining data integrity and application functionality.

## Migration Tools by Workload Type

### 1. Server Migration (VMs)

#### Azure Migrate: Server Migration
**Use for**: VMware VMs, Hyper-V VMs, physical servers

**Process**:
1. Install replication appliance (if not using agentless)
2. Enable replication for servers
3. Monitor replication status
4. Perform test migration
5. Execute final cutover

**Replication Methods**:
- **Agentless** (VMware only): No agent on VM, uses VMware snapshots
- **Agent-based**: Agent installed on each VM, works for all platforms

#### Azure Site Recovery (ASR)
**Use for**: Disaster recovery and migration

**Features**:
- Continuous replication
- Orchestrated failover
- Minimal downtime
- Supports VMware, Hyper-V, physical servers

### 2. Database Migration

#### Azure Database Migration Service (DMS)

**Supported Sources**:
- SQL Server
- MySQL
- PostgreSQL
- MongoDB
- Oracle (via third-party tools)

**Migration Modes**:
- **Offline**: Application downtime required, full backup/restore
- **Online**: Minimal downtime, continuous sync until cutover

**Process for SQL Server to Azure SQL**:
1. Create DMS instance
2. Create migration project
3. Select source and target
4. Map databases
5. Configure migration settings
6. Run migration
7. Perform cutover

#### Data Migration Assistant (DMA)
**Use for**: SQL Server assessment and schema migration

**Steps**:
1. Install DMA on-premises
2. Create new migration project
3. Connect to source SQL Server
4. Assess compatibility
5. Migrate schema
6. Migrate data (if using offline method)

### 3. Web Application Migration

#### App Service Migration Assistant

**Supported Apps**:
- ASP.NET (Framework and Core)
- PHP
- Java
- Node.js

**Process**:
1. Download and run Migration Assistant
2. Enter website URL or IIS server
3. Review compatibility report
4. Configure App Service settings
5. Migrate application code
6. Migrate application data/database
7. Update DNS

### 4. Large Data Transfers

#### Azure Data Box

**When to Use**:
- Data size >10TB
- Limited bandwidth
- Network transfer time >1 week

**Process**:
1. Order Data Box from Azure Portal
2. Receive device at your location
3. Connect and unlock device
4. Copy data to device
5. Ship device back to Azure
6. Data uploaded to Azure Storage
7. Device securely erased

**Data Box Options**:
- **Data Box Disk**: 8TB per disk, up to 5 disks (40TB)
- **Data Box**: 80TB usable capacity
- **Data Box Heavy**: 770TB usable capacity

## Migration Execution Steps

### Phase 1: Pre-Migration Validation

#### 1. Environment Readiness Check
```bash
# Azure CLI commands to verify setup
az account show
az group list
az network vnet list
az vm list --resource-group <migration-rg>
```

#### 2. Backup Verification
- [ ] Verify all on-premises backups are current
- [ ] Test backup restoration
- [ ] Document backup retention policy
- [ ] Ensure Azure Backup is configured

#### 3. Dependency Check
- [ ] Verify all dependent services are available
- [ ] Test network connectivity
- [ ] Validate DNS resolution
- [ ] Check firewall rules

### Phase 2: Test Migration

#### Purpose
Validate the migration process without affecting production

#### Steps
1. Select subset of servers for test
2. Enable replication to Azure
3. Create test virtual network (isolated)
4. Perform test failover/migration
5. Validate application functionality
6. Test performance
7. Document findings
8. Clean up test resources

#### Test Validation Checklist
- [ ] Application starts successfully
- [ ] Database connectivity works
- [ ] User authentication functions
- [ ] Application features work as expected
- [ ] Performance meets requirements
- [ ] Monitoring and alerts configured

### Phase 3: Production Migration

#### Migration Cutover Process

**1. Pre-Cutover (T-24 hours)**
- [ ] Notify all stakeholders
- [ ] Final backup of source systems
- [ ] Verify Azure target environment ready
- [ ] Review rollback procedures
- [ ] Confirm maintenance window

**2. Cutover Start (T-0)**
- [ ] Place application in maintenance mode
- [ ] Stop application services
- [ ] Perform final data sync
- [ ] Verify data consistency
- [ ] Start Azure resources
- [ ] Update DNS/load balancer
- [ ] Remove maintenance mode

**3. Post-Cutover (T+1 hour)**
- [ ] Validate application functionality
- [ ] Monitor performance metrics
- [ ] Check error logs
- [ ] Verify user access
- [ ] Monitor Azure costs
- [ ] Document any issues

**4. Stabilization (T+24 hours)**
- [ ] Continue monitoring
- [ ] Address any issues
- [ ] Collect user feedback
- [ ] Review performance data
- [ ] Begin optimization

### Phase 4: Validation and Testing

#### Application Validation
- **Smoke Tests**: Basic functionality check
- **Integration Tests**: Verify all components communicate
- **Performance Tests**: Validate response times
- **Security Tests**: Verify access controls
- **User Acceptance Testing (UAT)**: End-user validation

#### Data Validation
```sql
-- Example SQL validation queries
-- Row count comparison
SELECT COUNT(*) FROM [Table] -- Run on source
SELECT COUNT(*) FROM [Table] -- Run on target

-- Checksum comparison for critical tables
SELECT CHECKSUM_AGG(CHECKSUM(*)) FROM [Table]
```

#### Performance Validation
- Compare response times: on-premises vs. Azure
- Monitor CPU, memory, disk usage
- Check network latency
- Validate under load

## Minimal Downtime Strategies

### Database Migration with Minimal Downtime

#### Using DMS Online Migration
1. Set up continuous replication
2. Monitor replication lag
3. When lag is minimal (<5 minutes):
   - Stop writes to source
   - Wait for final sync
   - Cutover to Azure
4. Update connection strings
5. Resume application

### Application Migration Strategies

#### Blue-Green Deployment
1. Deploy new version (green) alongside existing (blue)
2. Test green environment
3. Switch traffic from blue to green
4. Monitor for issues
5. Keep blue as rollback option

#### Traffic Splitting
1. Deploy Azure version
2. Route small percentage of traffic to Azure (10%)
3. Monitor and validate
4. Gradually increase traffic (25%, 50%, 100%)
5. Decommission on-premises

## Monitoring During Migration

### Key Metrics to Monitor

**Azure Migrate Dashboard**:
- Replication status
- Replication lag
- Data sync progress
- Errors and warnings

**Azure Monitor**:
- VM performance (CPU, memory, disk, network)
- Application response times
- Error rates
- Custom application metrics

**Cost Monitoring**:
- Daily spending
- Resource usage
- Cost anomalies

### Alerting Setup
```bash
# Example: Create CPU alert rule
az monitor metrics alert create \
  --name "High CPU Alert" \
  --resource-group <rg-name> \
  --scopes <vm-resource-id> \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m
```

## Common Migration Issues and Solutions

| Issue | Symptoms | Solution |
|-------|----------|----------|
| Replication failure | Replication not progressing | Check network, firewall rules, credentials |
| Slow replication | High replication lag | Increase bandwidth, reduce change rate |
| Boot failure | VM doesn't start in Azure | Check boot diagnostics, verify OS compatibility |
| Network connectivity | Can't reach Azure VM | Verify NSG rules, VPN/ExpressRoute |
| Performance degradation | Slow application response | Right-size VM, optimize disk configuration |
| Database connection errors | App can't connect to DB | Update connection strings, check firewall |
| License activation | Windows not activated | Ensure proper licensing, check KMS |

## Rollback Procedures

### When to Rollback
- Critical functionality broken beyond quick fix
- Data corruption detected
- Unacceptable performance degradation
- Security breach
- Business stakeholder decision

### Rollback Steps
1. **Immediate**: Redirect traffic back to on-premises
2. **Communication**: Notify all stakeholders of rollback
3. **Data Sync**: If data changed in Azure, sync back to on-premises
4. **Analysis**: Root cause analysis of failure
5. **Planning**: Determine remediation steps
6. **Retry**: Schedule new migration attempt

## Post-Migration Activities

### Immediate (24-48 hours)
- [ ] Monitor continuously
- [ ] Address urgent issues
- [ ] Collect initial feedback
- [ ] Document lessons learned

### Short-term (1-2 weeks)
- [ ] Performance tuning
- [ ] Cost optimization review
- [ ] Security hardening
- [ ] Automation setup
- [ ] Documentation updates

### Medium-term (1-3 months)
- [ ] Decommission on-premises (after validation period)
- [ ] Implement advanced Azure services
- [ ] Optimize reserved instances
- [ ] Regular performance reviews

## Next Phase
After successful migration, proceed to [Optimization Phase](./05-optimization-phase.md)

## Migration Tools Quick Reference

| Tool | Purpose | Download/Access |
|------|---------|-----------------|
| Azure Migrate | Central migration hub | Azure Portal |
| Azure Site Recovery | VM replication & migration | Azure Portal |
| Database Migration Service | Database migration | Azure Portal |
| Data Migration Assistant | SQL assessment | [Download DMA](https://www.microsoft.com/download/details.aspx?id=53595) |
| App Service Migration Assistant | Web app migration | [Download Tool](https://appmigration.microsoft.com/) |
| Azure Data Box | Offline data transfer | Order via Azure Portal |
| Azure Storage Explorer | Manage Azure Storage | [Download](https://azure.microsoft.com/features/storage-explorer/) |
