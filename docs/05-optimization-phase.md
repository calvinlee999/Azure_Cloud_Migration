# Optimization Phase

## Objective
Fine-tune Azure resources for optimal performance, cost-efficiency, security, and reliability after migration completion.

## Cost Optimization

### 1. Right-Sizing Resources

#### Identify Oversized Resources
```bash
# Azure CLI to list VM sizes and utilization
az vm list --query "[].{Name:name, Size:hardwareProfile.vmSize}" -o table

# Use Azure Advisor for recommendations
az advisor recommendation list --category Cost
```

**Actions**:
- Review Azure Advisor recommendations
- Analyze CPU/Memory utilization (target 40-60%)
- Downsize underutilized VMs
- Delete unused resources (NICs, disks, IPs)

### 2. Azure Reserved Instances (RIs)

**Savings**: Up to 72% compared to pay-as-you-go

**Best For**:
- Predictable workloads
- Production VMs running 24/7
- Long-term commitments (1 or 3 years)

**How to Purchase**:
1. Analyze usage patterns (minimum 30 days)
2. Calculate potential savings
3. Purchase RIs via Azure Portal > Reservations
4. Apply to appropriate resource groups

### 3. Azure Hybrid Benefit (AHB)

**Savings**: Up to 85% (when combined with RIs)

**Eligible Licenses**:
- Windows Server with Software Assurance
- SQL Server with Software Assurance
- Red Hat and SUSE Linux subscriptions

**Activation**:
```bash
# Enable Azure Hybrid Benefit for a VM
az vm update --resource-group <rg> --name <vm-name> \
  --license-type Windows_Server
```

### 4. Storage Optimization

#### Blob Storage Tiers
- **Hot**: Frequently accessed data
- **Cool**: Infrequently accessed (>30 days)
- **Archive**: Rarely accessed (>180 days)

#### Managed Disk Optimization
- Use Standard SSD for non-critical workloads
- Use Premium SSD only for high-IOPS requirements
- Enable disk bursting for intermittent workloads

#### Lifecycle Management
```json
{
  "rules": [
    {
      "name": "moveToArchive",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 180
            }
          }
        }
      }
    }
  ]
}
```

### 5. Auto-Shutdown for Non-Production

**Configure Auto-Shutdown**:
1. Navigate to VM > Operations > Auto-shutdown
2. Set shutdown time (e.g., 7 PM daily)
3. Configure timezone
4. Enable notification before shutdown

**Expected Savings**: 50-65% for dev/test environments

### 6. Cost Management Tools

#### Set up Budgets and Alerts
```bash
# Create a budget
az consumption budget create \
  --budget-name "Monthly-Budget" \
  --amount 5000 \
  --category Cost \
  --time-grain Monthly \
  --start-date 2025-01-01 \
  --end-date 2025-12-31
```

#### Enable Cost Alerts
- Set alert at 80%, 90%, 100% of budget
- Configure action groups for notifications
- Review Cost Analysis regularly

## Performance Optimization

### 1. VM Performance

#### Enable Accelerated Networking
- Reduces latency and CPU utilization
- Available on most VM sizes (Dv3, Ev3, FSv2, etc.)

```bash
# Enable accelerated networking
az network nic update \
  --resource-group <rg> \
  --name <nic-name> \
  --accelerated-networking true
```

#### Optimize Disk Configuration
- Use Premium SSD for database servers
- Enable disk caching (ReadOnly for data disks)
- Use larger disks for higher IOPS limits
- Consider Ultra Disks for extreme performance needs

### 2. Database Performance

#### Azure SQL Database
- Choose appropriate service tier (Basic, Standard, Premium, Hyperscale)
- Enable auto-tuning
- Use Query Performance Insight
- Implement indexes based on recommendations

```sql
-- Enable automatic tuning
ALTER DATABASE [DatabaseName] 
SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON);
```

#### SQL Server on Azure VM
- Enable SQL Server best practices assessment
- Optimize tempdb configuration
- Configure max server memory
- Enable instant file initialization
- Use separate disks for data, logs, tempdb

### 3. Network Performance

#### Use Azure Front Door or Traffic Manager
- Global load balancing
- Reduced latency via nearest endpoint
- Improved availability

#### Implement Azure CDN
- Cache static content closer to users
- Reduce origin server load
- Improve page load times

### 4. Application Performance

#### Enable Application Insights
```bash
# Add Application Insights to web app
az webapp config appsettings set \
  --resource-group <rg> \
  --name <app-name> \
  --settings APPINSIGHTS_INSTRUMENTATIONKEY=<key>
```

**Monitor**:
- Request response times
- Dependency calls
- Exceptions and failures
- Custom metrics

#### Optimize Code
- Use async/await patterns
- Implement caching (Redis Cache)
- Minimize database calls
- Use connection pooling

## Security Optimization

### 1. Network Security

#### Network Security Groups (NSGs)
- Review and minimize open ports
- Implement least-privilege rules
- Use Application Security Groups for organization

```bash
# Create NSG rule to allow HTTPS only
az network nsg rule create \
  --resource-group <rg> \
  --nsg-name <nsg-name> \
  --name AllowHTTPS \
  --priority 100 \
  --source-address-prefixes '*' \
  --destination-port-ranges 443 \
  --protocol Tcp \
  --access Allow
```

#### Azure Firewall
- Centralized network security
- Application and network-level filtering
- Threat intelligence integration

### 2. Identity and Access Management

#### Implement Azure AD Authentication
- Replace SQL authentication with Azure AD
- Enable Managed Identities for applications
- Use conditional access policies

#### Role-Based Access Control (RBAC)
```bash
# Assign least-privilege role
az role assignment create \
  --assignee <user@domain.com> \
  --role "Virtual Machine Contributor" \
  --resource-group <rg>
```

### 3. Data Protection

#### Enable Azure Backup
```bash
# Enable backup for VM
az backup protection enable-for-vm \
  --resource-group <rg> \
  --vault-name <vault-name> \
  --vm <vm-name> \
  --policy-name DefaultPolicy
```

#### Encryption
- Enable Azure Disk Encryption for VMs
- Use Transparent Data Encryption (TDE) for databases
- Enable encryption at rest for storage accounts
- Use SSL/TLS for data in transit

### 4. Security Monitoring

#### Enable Azure Security Center (Defender for Cloud)
- Continuous security assessment
- Threat detection
- Security recommendations
- Compliance monitoring

#### Configure Log Analytics
```bash
# Create Log Analytics workspace
az monitor log-analytics workspace create \
  --resource-group <rg> \
  --workspace-name <workspace-name>
```

**Collect Logs**:
- Security events
- System logs
- Application logs
- Performance counters

## Reliability Optimization

### 1. High Availability

#### Availability Sets
- Protect against single point of failure
- 99.95% SLA for VMs in Availability Set

#### Availability Zones
- Protection against datacenter failure
- 99.99% SLA for VMs in Availability Zones
- Deploy across multiple zones for critical workloads

```bash
# Create VM in availability zone
az vm create \
  --resource-group <rg> \
  --name <vm-name> \
  --image UbuntuLTS \
  --zone 1
```

### 2. Disaster Recovery

#### Azure Site Recovery (ASR)
- Replicate VMs to another Azure region
- Automated failover and failback
- Regular DR drills

**RPO/RTO Targets**:
- Recovery Point Objective (RPO): <15 minutes
- Recovery Time Objective (RTO): 2-4 hours

#### Backup Strategy
- Configure appropriate retention policies
- Test restore procedures regularly
- Implement geo-redundant backup storage

### 3. Load Balancing

#### Azure Load Balancer
- Distribute traffic across VMs
- Health probes for automatic failover
- Zone-redundant for higher availability

#### Azure Application Gateway
- Layer 7 load balancing
- Web Application Firewall (WAF)
- SSL termination
- URL-based routing

### 4. Auto-Scaling

#### VM Scale Sets
```bash
# Create autoscale profile
az monitor autoscale create \
  --resource-group <rg> \
  --resource <vmss-name> \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --min-count 2 \
  --max-count 10 \
  --count 2
```

#### App Service Auto-Scale
- Scale based on CPU, memory, or custom metrics
- Schedule-based scaling for predictable patterns

## Monitoring and Operations

### 1. Azure Monitor Setup

#### Key Metrics to Monitor
- CPU and memory utilization
- Disk IOPS and throughput
- Network ingress/egress
- Application response times
- Database DTU/vCore usage

#### Create Dashboards
- Consolidate key metrics in single view
- Share with team members
- Pin critical alerts

### 2. Alerting Strategy

#### Priority 1 (Critical)
- VM is down
- Database is unavailable
- Application errors >5%

#### Priority 2 (Warning)
- CPU utilization >80% for 10 minutes
- Memory >90%
- Disk space <10%

#### Priority 3 (Informational)
- Budget threshold reached
- Security recommendations

### 3. Update Management

#### Azure Update Management
- Schedule patching windows
- Approve/test updates before deployment
- Track compliance

```bash
# Enable Update Management for VM
az vm extension set \
  --resource-group <rg> \
  --vm-name <vm-name> \
  --name MicrosoftMonitoringAgent \
  --publisher Microsoft.EnterpriseCloud.Monitoring
```

## Optimization Checklist

### Cost
- [ ] Right-sized all VMs based on actual usage
- [ ] Purchased reserved instances for production workloads
- [ ] Enabled Azure Hybrid Benefit where applicable
- [ ] Implemented storage tiering
- [ ] Configured auto-shutdown for dev/test
- [ ] Set up budgets and cost alerts
- [ ] Deleted unused resources

### Performance
- [ ] Enabled accelerated networking
- [ ] Optimized disk configuration
- [ ] Implemented caching strategies
- [ ] Database tuning completed
- [ ] Application Insights configured
- [ ] CDN implemented for static content

### Security
- [ ] Network Security Groups reviewed and optimized
- [ ] Azure AD authentication enabled
- [ ] Encryption enabled (at rest and in transit)
- [ ] Azure Defender enabled
- [ ] RBAC properly configured
- [ ] Backup policies implemented
- [ ] Security monitoring configured

### Reliability
- [ ] High availability configured (Availability Sets/Zones)
- [ ] Disaster recovery plan implemented
- [ ] Load balancing configured
- [ ] Auto-scaling enabled where appropriate
- [ ] Backup tested and verified
- [ ] Monitoring and alerting operational

## Continuous Improvement

### Monthly Reviews
- Review cost analysis and trends
- Analyze performance metrics
- Review security recommendations
- Check backup success rates
- Update documentation

### Quarterly Activities
- Disaster recovery drill
- Security audit
- Reserved instance analysis
- Architecture review
- Capacity planning

### Tools and Resources
- [Azure Advisor](https://portal.azure.com/#blade/Microsoft_Azure_Expert/AdvisorMenuBlade/overview)
- [Azure Cost Management](https://portal.azure.com/#blade/Microsoft_Azure_CostManagement/Menu/overview)
- [Azure Well-Architected Review](https://learn.microsoft.com/assessments/)
- [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)

## Next Steps
Your Azure migration is now optimized! Continue with regular monitoring and improvement cycles to maintain optimal performance and cost efficiency.

For advanced scenarios, consider:
- [Azure Kubernetes Service (AKS)](./advanced/aks-migration.md) for containerization
- [Azure DevOps](./advanced/devops-setup.md) for CI/CD pipelines
- [Infrastructure as Code](./advanced/iac-terraform-bicep.md) with Terraform or Bicep
