# Azure Cloud Migration Reference Architecture

## Overview

This document provides high-level reference architectures for migrating on-premises workloads to Microsoft Azure. These architectures follow Azure best practices and are designed to support secure, scalable, and cost-effective cloud migration.

## Table of Contents

1. [High-Level Migration Architecture](#high-level-migration-architecture)
2. [Hub-and-Spoke Network Topology](#hub-and-spoke-network-topology)
3. [Azure Landing Zone Architecture](#azure-landing-zone-architecture)
4. [Migration Execution Architecture](#migration-execution-architecture)
5. [Data Migration Architecture](#data-migration-architecture)
6. [Security and Governance Architecture](#security-and-governance-architecture)

---

## High-Level Migration Architecture

This architecture shows the complete migration ecosystem including on-premises infrastructure, Azure Migrate components, and target Azure environment.

```mermaid
architecture-beta
    group onprem(server)[On-Premises Environment]
    group azure_migrate(cloud)[Azure Migrate Hub]
    group azure_target(cloud)[Azure Target Environment]
    
    service vmware(server)[VMware vCenter] in onprem
    service hyperv(server)[Hyper-V Hosts] in onprem
    service physical(server)[Physical Servers] in onprem
    service databases(database)[SQL Servers] in onprem
    
    service appliance(server)[Migrate Appliance] in azure_migrate
    service assessment(server)[Assessment Engine] in azure_migrate
    service replication(server)[Replication Service] in azure_migrate
    
    service vms(server)[Azure VMs] in azure_target
    service sql(database)[Azure SQL] in azure_target
    service storage(disk)[Azure Storage] in azure_target
    service network(internet)[Virtual Network] in azure_target
    
    vmware:R --> L:appliance
    hyperv:R --> L:appliance
    physical:R --> L:appliance
    databases:R --> L:appliance
    
    appliance:R --> L:assessment
    assessment:R --> L:replication
    replication:R --> L:vms
    replication:R --> L:sql
    vms:B --> T:storage
    sql:B --> T:network
```

**Key Components:**

- **On-Premises Environment**: Source infrastructure including VMware, Hyper-V, physical servers, and databases
- **Azure Migrate Appliance**: Lightweight discovery and assessment tool
- **Assessment Engine**: Analyzes readiness, sizing, and cost estimates
- **Replication Service**: Handles continuous data replication during migration
- **Azure Target Environment**: Destination cloud infrastructure

---

## Hub-and-Spoke Network Topology

This architecture demonstrates the recommended network topology for enterprise Azure deployments, providing centralized governance and security.

```mermaid
architecture-beta
    group hub(cloud)[Hub VNet - Shared Services]
    group spoke1(cloud)[Spoke VNet - Production]
    group spoke2(cloud)[Spoke VNet - Development]
    group spoke3(cloud)[Spoke VNet - DMZ]
    
    service firewall(server)[Azure Firewall] in hub
    service gateway(internet)[VPN Gateway] in hub
    service bastion(server)[Azure Bastion] in hub
    service dns(server)[Private DNS] in hub
    
    service webapp1(server)[Web Apps] in spoke1
    service db1(database)[Databases] in spoke1
    service webapp2(server)[Dev Apps] in spoke2
    service db2(database)[Dev DB] in spoke2
    service lb(internet)[Load Balancer] in spoke3
    service appgw(internet)[App Gateway] in spoke3
    
    gateway:R --> L:firewall
    firewall:B --> T:dns
    firewall:R --> L:webapp1
    firewall:R --> L:webapp2
    firewall:R --> L:lb
    webapp1:B --> T:db1
    webapp2:B --> T:db2
    lb:R --> L:appgw
```

**Architecture Benefits:**

- **Centralized Security**: Firewall and security controls in hub
- **Isolation**: Workloads separated across spokes
- **Shared Services**: DNS, monitoring, and management in hub
- **Scalability**: Easy to add new spoke VNets
- **Cost Optimization**: Shared resources reduce duplication

---

## Azure Landing Zone Architecture

Azure Landing Zones provide the foundation for secure, compliant, and scalable Azure deployments.

```mermaid
architecture-beta
    group management(cloud)[Management Groups]
    group platform(cloud)[Platform Subscriptions]
    group landing(cloud)[Landing Zone Subscriptions]
    
    service root(server)[Root MG] in management
    service policy(server)[Azure Policy] in management
    service rbac(server)[RBAC] in management
    
    service identity(server)[Identity] in platform
    service connectivity(internet)[Connectivity] in platform
    service monitoring(server)[Management] in platform
    
    service prod(server)[Production] in landing
    service dev(server)[Development] in landing
    service sandbox(server)[Sandbox] in landing
    
    root:R --> L:policy
    policy:R --> L:rbac
    rbac:B --> T:identity
    identity:R --> L:connectivity
    connectivity:R --> L:monitoring
    monitoring:B --> T:prod
    prod:R --> L:dev
    dev:R --> L:sandbox
```

**Core Components:**

- **Management Groups**: Hierarchical organization and governance
- **Azure Policy**: Automated compliance and standards enforcement
- **RBAC**: Role-based access control across subscriptions
- **Identity Subscription**: Centralized identity and access management
- **Connectivity Subscription**: Hub networking and hybrid connectivity
- **Management Subscription**: Monitoring, logging, and security operations
- **Landing Zone Subscriptions**: Application workload environments

---

## Migration Execution Architecture

This architecture shows the technical flow for executing server and application migrations using Azure Site Recovery and Azure Migrate.

```mermaid
architecture-beta
    group source(server)[Source Environment]
    group migration(cloud)[Migration Services]
    group target(cloud)[Target Environment]
    group mgmt(cloud)[Management & Monitoring]
    
    service src_vm(server)[Source VMs] in source
    service src_db(database)[Source DBs] in source
    service src_app(server)[Applications] in source
    
    service asr(server)[Site Recovery] in migration
    service dms(database)[Database Migration] in migration
    service migrate(server)[Azure Migrate] in migration
    
    service tgt_vm(server)[Azure VMs] in target
    service tgt_db(database)[Azure SQL] in target
    service tgt_app(server)[App Service] in target
    service storage(disk)[Managed Disks] in target
    
    service monitor(server)[Azure Monitor] in mgmt
    service backup(disk)[Azure Backup] in mgmt
    service security(server)[Security Center] in mgmt
    
    src_vm:R --> L:asr
    src_db:R --> L:dms
    src_app:R --> L:migrate
    
    asr:R --> L:tgt_vm
    dms:R --> L:tgt_db
    migrate:R --> L:tgt_app
    
    tgt_vm:B --> T:storage
    tgt_vm:R --> L:monitor
    tgt_db:R --> L:backup
    tgt_app:R --> L:security
```

**Migration Services:**

- **Azure Site Recovery (ASR)**: VM replication and migration
- **Database Migration Service (DMS)**: Database migration with minimal downtime
- **Azure Migrate**: Unified migration hub
- **Azure Monitor**: Performance and health monitoring
- **Azure Backup**: Backup and disaster recovery
- **Security Center**: Security posture management

---

## Data Migration Architecture

This architecture illustrates various data migration patterns for different scenarios and data volumes.

```mermaid
architecture-beta
    group onprem_data(server)[On-Premises Data]
    group transfer(cloud)[Transfer Methods]
    group azure_data(cloud)[Azure Data Services]
    
    service files(disk)[File Servers] in onprem_data
    service sql(database)[SQL Databases] in onprem_data
    service apps(server)[App Data] in onprem_data
    
    service databox(disk)[Data Box] in transfer
    service express(internet)[ExpressRoute] in transfer
    service vpn(internet)[VPN Gateway] in transfer
    service dms_svc(database)[DMS] in transfer
    
    service blob(disk)[Blob Storage] in azure_data
    service azure_sql(database)[Azure SQL] in azure_data
    service files_azure(disk)[Azure Files] in azure_data
    service cosmos(database)[Cosmos DB] in azure_data
    
    files:R --> L:databox
    sql:R --> L:dms_svc
    apps:R --> L:express
    apps:R --> L:vpn
    
    databox:R --> L:blob
    dms_svc:R --> L:azure_sql
    express:R --> L:files_azure
    vpn:R --> L:cosmos
```

**Data Migration Methods:**

1. **Azure Data Box**: Offline migration for large datasets (TBs to PBs)
2. **ExpressRoute**: Private, high-bandwidth connection (50 Mbps to 100 Gbps)
3. **VPN Gateway**: Encrypted internet connection for smaller datasets
4. **Database Migration Service**: Online migration with minimal downtime

**Target Data Services:**

- **Blob Storage**: Unstructured data (files, backups, media)
- **Azure SQL Database**: Relational databases
- **Azure Files**: Lift-and-shift file shares
- **Cosmos DB**: Globally distributed NoSQL databases

---

## Security and Governance Architecture

This architecture demonstrates defense-in-depth security and comprehensive governance for migrated workloads.

```mermaid
architecture-beta
    group identity(server)[Identity & Access]
    group network(internet)[Network Security]
    group data(database)[Data Protection]
    group governance(cloud)[Governance & Compliance]
    
    service entra(server)[Entra ID] in identity
    service mfa(server)[MFA] in identity
    service pim(server)[PIM] in identity
    
    service nsg(server)[NSG] in network
    service fw(server)[Firewall] in network
    service ddos(internet)[DDoS Protection] in network
    service waf(internet)[WAF] in network
    
    service encryption(disk)[Encryption] in data
    service keyvault(server)[Key Vault] in data
    service backup_svc(disk)[Backup] in data
    
    service policy_svc(server)[Azure Policy] in governance
    service cost(server)[Cost Management] in governance
    service defender(server)[Defender] in governance
    service sentinel(server)[Sentinel] in governance
    
    entra:R --> L:mfa
    mfa:R --> L:pim
    pim:B --> T:nsg
    
    nsg:R --> L:fw
    fw:R --> L:ddos
    ddos:R --> L:waf
    
    waf:B --> T:encryption
    encryption:R --> L:keyvault
    keyvault:R --> L:backup_svc
    
    backup_svc:B --> T:policy_svc
    policy_svc:R --> L:cost
    cost:R --> L:defender
    defender:R --> L:sentinel
```

**Security Layers:**

1. **Identity & Access Management**
   - Azure Entra ID (formerly Azure AD)
   - Multi-Factor Authentication (MFA)
   - Privileged Identity Management (PIM)
   - Role-Based Access Control (RBAC)

2. **Network Security**
   - Network Security Groups (NSG)
   - Azure Firewall
   - DDoS Protection
   - Web Application Firewall (WAF)

3. **Data Protection**
   - Encryption at rest and in transit
   - Azure Key Vault for secrets management
   - Azure Backup for business continuity

4. **Governance & Compliance**
   - Azure Policy for compliance automation
   - Cost Management + Billing
   - Microsoft Defender for Cloud
   - Microsoft Sentinel (SIEM)

---

## Architecture Principles

### 1. **Security First**
- Zero Trust security model
- Encryption by default
- Least privilege access
- Defense in depth

### 2. **High Availability**
- Multi-region deployment capability
- Availability Zones for redundancy
- Load balancing and auto-scaling
- Disaster recovery planning

### 3. **Scalability**
- Horizontal scaling with VM Scale Sets
- Auto-scaling based on metrics
- Elastic database pools
- CDN for global distribution

### 4. **Cost Optimization**
- Right-sizing VMs based on actual usage
- Reserved Instances for predictable workloads
- Azure Hybrid Benefit for existing licenses
- Storage tiering and lifecycle management

### 5. **Operational Excellence**
- Infrastructure as Code (Terraform, Bicep)
- CI/CD pipelines for deployments
- Centralized monitoring and logging
- Automated backup and patching

### 6. **Performance Efficiency**
- Proximity placement groups
- Premium storage for critical workloads
- Accelerated networking
- Application performance monitoring

---

## Next Steps

1. **Review Architecture Patterns**: Evaluate which architecture best fits your requirements
2. **Design Your Landing Zone**: Implement Azure Landing Zone framework
3. **Plan Network Topology**: Design hub-and-spoke or virtual WAN architecture
4. **Implement Security Controls**: Deploy identity, network, and data security
5. **Set Up Governance**: Configure Azure Policy and management groups
6. **Execute Pilot Migration**: Test with non-critical workload
7. **Monitor and Optimize**: Continuously improve based on metrics

---

## Related Documentation

- [Migration Phases](./01-discovery-phase.md)
- [Security Best Practices](./best-practices.md)
- [Sequence Diagrams](./sequence-diagrams.md)
- [Azure MCP Tools Guide](./azure-mcp-migration-tools.md)

---

**Document Version**: 1.0  
**Last Updated**: November 12, 2025  
**Architecture Standard**: Azure Well-Architected Framework
