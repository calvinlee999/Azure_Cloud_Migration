# Migration Planning Phase

## Objective
Create a detailed migration plan with prioritized waves, risk mitigation strategies, and rollback procedures to ensure a successful migration with minimal business disruption.

## Migration Wave Planning

### What is a Migration Wave?
A migration wave is a group of servers/applications migrated together in a coordinated effort. Waves help manage complexity and reduce risk.

### Wave Prioritization Criteria

#### 1. **Pilot Wave** (Wave 0)
**Characteristics**:
- Low business criticality
- Simple architecture (minimal dependencies)
- Low user impact
- Good candidate for learning

**Examples**:
- Development/test environments
- Internal tools
- Non-critical applications

**Purpose**: Validate migration process and tooling

#### 2. **Early Waves** (Waves 1-2)
**Characteristics**:
- Medium criticality
- Manageable dependencies
- Willing business stakeholders
- Clear success metrics

**Purpose**: Build confidence and refine processes

#### 3. **Later Waves** (Waves 3+)
**Characteristics**:
- Business-critical applications
- Complex dependencies
- Requires detailed planning
- High availability requirements

**Purpose**: Migrate production workloads with proven processes

## Migration Strategies (The 7 Rs)

### 1. **Rehost** (Lift and Shift)
- Migrate as-is to Azure VMs or Azure VMware Solution
- Minimal changes to application
- **Best for**: Quick migration, legacy apps, limited time

### 2. **Refactor** (Re-platform)
- Minor modifications to leverage Azure PaaS
- Example: Move SQL to Azure SQL Database
- **Best for**: Modernization without full rewrite

### 3. **Rearchitect**
- Significant modifications for cloud-native features
- Example: Break monolith into microservices
- **Best for**: Long-term cloud optimization

### 4. **Rebuild**
- Completely rewrite application using cloud-native services
- **Best for**: Legacy apps needing modernization

### 5. **Replace**
- Replace with SaaS solution
- Example: Move to Microsoft 365, Dynamics 365
- **Best for**: Commodity applications

### 6. **Retire**
- Decommission unused applications
- **Best for**: Reducing technical debt

### 7. **Retain**
- Keep on-premises for now
- **Best for**: Apps with compliance restrictions or recently upgraded

## Migration Methods

### 1. Private Connection (ExpressRoute)
**Characteristics**:
- Dedicated private connection to Azure
- Higher bandwidth (50 Mbps - 100 Gbps)
- More secure than public internet
- Higher cost

**Best for**:
- Large data volumes
- Ongoing hybrid scenarios
- Security requirements

### 2. VPN over Internet
**Characteristics**:
- Encrypted connection over public internet
- Lower bandwidth
- Lower cost
- Quick to set up

**Best for**:
- Smaller migrations
- Test migrations
- Budget constraints

### 3. Offline Transfer (Azure Data Box)
**Characteristics**:
- Physical device ships to your location
- Transfer large datasets offline
- Multiple device sizes (8TB - 1PB)

**Best for**:
- Initial bulk data transfer (>10TB)
- Limited bandwidth
- Compliance with data sovereignty

## Migration Timeline Planning

### Sample Timeline Template

#### Week 1-2: Pre-Migration
- [ ] Finalize wave assignments
- [ ] Set up Azure landing zone
- [ ] Configure networking (VPN/ExpressRoute)
- [ ] Set up Azure Migrate project
- [ ] Prepare target resource groups
- [ ] Review and approve migration plan

#### Week 3-4: Pilot Migration (Wave 0)
- [ ] Migrate pilot servers
- [ ] Validate functionality
- [ ] Performance testing
- [ ] Document lessons learned
- [ ] Refine migration runbook

#### Week 5+: Production Waves
- [ ] Execute migration waves per schedule
- [ ] Monitor and validate each wave
- [ ] Update documentation
- [ ] Decommission on-premises after validation

## Risk Management

### Common Migration Risks

| Risk | Impact | Mitigation Strategy |
|------|--------|---------------------|
| Data loss during migration | High | Test migrations, use replication, backup verification |
| Application downtime | High | Plan maintenance windows, use minimal-downtime methods |
| Performance issues post-migration | Medium | Performance testing, proper sizing, monitoring |
| Dependency failures | High | Dependency mapping, coordinated cutover |
| License compliance | Medium | Review licensing, Azure Hybrid Benefit |
| Skills gap | Medium | Training, partner engagement, documentation |
| Cost overruns | Medium | Budget monitoring, cost alerts, reserved instances |

### Rollback Planning

**Requirements for Safe Rollback**:
1. Keep on-premises environment intact until validation complete
2. Document rollback procedures for each wave
3. Define rollback decision criteria
4. Test rollback procedures during pilot
5. Set rollback time windows

**Rollback Triggers**:
- Critical functionality broken
- Performance degradation >50%
- Data integrity issues
- Security breach
- Business stakeholder veto

## Pre-Migration Checklist

### Azure Environment Setup
- [ ] Azure subscription created
- [ ] Resource groups created per wave
- [ ] Networking configured (VNet, subnets, NSGs)
- [ ] VPN or ExpressRoute established
- [ ] DNS planning completed
- [ ] Azure Backup configured
- [ ] Monitoring and alerting set up
- [ ] RBAC and security policies applied

### On-Premises Preparation
- [ ] Final discovery completed
- [ ] Dependencies validated
- [ ] Application owners notified
- [ ] Change management approved
- [ ] Maintenance windows scheduled
- [ ] Backup verification completed
- [ ] Rollback procedures documented

### Team Readiness
- [ ] Roles and responsibilities assigned
- [ ] Migration runbooks created
- [ ] Communication plan established
- [ ] Support procedures defined
- [ ] Training completed

## Communication Plan

### Stakeholder Communication

**Before Migration**:
- Timeline and schedule
- Expected downtime
- Testing requirements
- Success criteria

**During Migration**:
- Status updates (hourly/daily)
- Issue escalation
- Decision points

**After Migration**:
- Validation results
- Performance metrics
- Lessons learned
- Next steps

## Documentation Requirements

### Migration Runbook Template
For each wave, document:
1. **Scope**: Servers and applications in wave
2. **Dependencies**: Required services and data
3. **Pre-migration steps**: Backups, snapshots, validation
4. **Migration steps**: Detailed procedure with commands
5. **Validation steps**: Health checks and tests
6. **Rollback procedure**: Steps to revert if needed
7. **Post-migration tasks**: Monitoring, optimization

## Next Phase
After planning is complete, proceed to [Migration Execution](./04-migration-execution.md)

## Templates & Tools
- Wave planning spreadsheet: `../templates/migration-wave-template.xlsx`
- Risk register: `../templates/risk-register.xlsx`
- Migration runbook: `../templates/migration-runbook-template.md`
- Cutover checklist: `../templates/cutover-checklist.md`
