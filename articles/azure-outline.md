# Azure outline #

## Content ##

- [Foundational Concepts](#foundational-concepts)
  - [Why cloud?](#why-cloud)
  - [Elements of the Cloud Computing Model](#elements-of-the-cloud-computing-model)
  - [Cloud Infrastructure Models](#cloud-infrastructure-models)
  - [Cloud Deliver Models](#cloud-deliver-models)
- [Virtual Machines](#virtual-machines)
  - [Which Model is Best For Your Workload?](#which-model-is-best-for-your-workload)
  - [Things to Keep in Mind Regarding Azure VMs](#things-to-keep-in-mind-regarding-azure-vms)
  - [Virtual Machine Resources](#virtual-machine-resources)
  - [Azure VMs base types](#azure-vms-base-types)
  - [Connect to Windows VMs](#connect-to-windows-vms)
  - [Connect to Linux VMs](#connect-to-linux-vms)
- [Storage](#storage)
  - [Things to Keep in Mind Regarding Azure Storage for IaaS VMs](#things-to-keep-in-mind-regarding-azure-storage-for-iaas-vms)
  - [Azure Storage Types](#azure-storage-types)
  - [Replication Options](#replication-options)
  - [Azure VM Disk Types](#azure-vm-disk-types)
  - [Azure Storage Account Types](#azure-storage-account-types)
  - [Virtual Network Service Endpoints](#virtual-network-service-endpoints)
  - [Host Caching](#host-caching)
- [VM Configuration](#vm-configuration)
  - [Things to Keep in Mind Regarding Azure VMs Configuration](#things-to-keep-in-mind-regarding-azure-vms-configuration)
- [Scaling and High Availability](#scaling-and-high-availability)
  - [Things to Keep in Mind Regarding Scaling and High Availability](#things-to-keep-in-mind-regarding-scaling-and-high-availability)
  - [Azure Maintenance Events](#azure-maintenance-events)
  - [Fault and Update Domains](#fault-and-update-domains)
  - [VM scale set](#vm-scale-set)
  - [Scale set use cases](#scale-set-use-cases)
  - [Load balancer](#load-balancer)
  - [Application gateway](#application-gateway)
  - [Traffic manager](#traffic-manager)
- [Networking](#networking)
  - [When multiple VNets make sense](#when-multiple-vnets-make-sense)
  - [VNet-to-VNet VPN](#vnet-to-vnet-vpn)
  - [VNet-to-VNet VPN Points to Ponder](#vnet-to-vnet-vpn-points-to-ponder)
  - [Inter-VNet troubleshooting tips](#inter-vnet-troubleshooting-tips)
  - [Troubleshooting](#troubleshooting)
  - [Things to Keep in Mind Regarding Networking](#things-to-keep-in-mind-regarding-networking)
  - [Public UP address SKUs](#public-up-address-skus)
  - [Azure VPN SKUs](#azure-vpn-skus)
  - [Name Resolution for Azure VNets](#name-resolution-for-azure-vnets)
  - [Azure VNet Design Best Practices](#azure-vnet-design-best-practices)
  - [Network Security Groups](#network-security-groups)
  - [VM IP addressing best practices](#vm-ip-addressing-best-practices)
- [Security](#security)
  - [Terminology](#terminology)
  - [Access Concepts](#access-concepts)
  - [Things to Keep in Mind Regarding Security](#things-to-keep-in-mind-regarding-security)
  - [Protect the Storage Subsystem](#protect-the-storage-subsystem)
  - [Roles](#roles)
  - [Azure Resource Manager Policies](#azure-resource-manager-policies)
  - [Data Plane Security](#data-plane-security)
  - [Encryption in Transit](#encryption-in-transit)
  - [Encryption at Rest](#encryption-at-rest)
  - [Storage Analytics](#storage-analytics)
  - [Firewall](#firewall)
  - [Antimalware](#antimalware)
  - [Azure Security Center](#azure-security-center)
  - [ASC policy component](#asc-policy-component)
  - [Azure activity log](#azure-activity-log)
  - [Patch Management for Azure VMs](#patch-management-for-azure-vms)
- [Monitoring](#monitoring)
  - [Three monitoring tiers](#three-monitoring-tiers)
  - [Ways to check Azure service status](#ways-to-check-azure-service-status)
  - [Things to Keep in Mind Regarding Azure VM Monitoring](#things-to-keep-in-mind-regarding-azure-vm-monitoring)
  - [Tools](#tools)
  - [Monitoring - A layer Approach](#monitoring-a-layer-approach)
  - [Monitoring and Diagnostics Pipeline](#monitoring-and-diagnostics-pipeline)
  - [Azure Monitor Components](#azure-monitor-components)
  - [Azure diagnostics data](#azure-diagnostics-data)
  - [Azure resource log](#azure-resource-log)
  - [Monitoring - Questions of scale and intelligence](#monitoring-questions-of-scale-and-intelligence)
  - [Log Analytics](#log-analytics)
  - [What can you do with Webhooks](#what-can-you-do-with-webhooks)
- [Troubleshooting and support](#troubleshooting-and-support)
  - [Azure VM troubleshooting checklist](#azure-vm-troubleshooting-checklist)
- [Cost management](#cost-management)
  - [How to buy azure services](#how-to-buy-azure-services)
  - [Azure SLA](#azure-sla)
  - [Azure support plans](#azure-support-plans)
- [Managed disks](#managed-disks)
  - [Storage account limits](#storage-account-limits)
  - [Value proposition](#value-proposition)
  - [Managed disks facts](#managed-disks-facts)
- [Help and support](#help-and-support)
  - [Azure support issue types](#azure-support-issue-types)

## Foundational Concepts ##

### Why cloud? ###

- CapEx vs. OpEx
  - [CapEx](https://www.investopedia.com/terms/c/capitalexpenditure.asp) - Capital expenditure or capital expense (capex or CAPEX) is the money a company spends to buy, maintain, or improve its fixed assets, such as buildings, vehicles, equipment, or land.
  - [OpEx](https://en.wikipedia.org/wiki/Operating_expense) - An operating expense, operating expenditure, operational expense, operational expenditure or OpEx is an ongoing cost for running a product, business, or system
- How old is your technology?
- How flexible is your infrastructure?
- How much [redundancy](https://en.wikipedia.org/wiki/Redundancy_(engineering)) do you have?
- How well do you [scale](https://en.wikipedia.org/wiki/Scalability)?

### Elements of the Cloud Computing Model ###

- [Elasticity](https://sdqweb.ipd.kit.edu/publications/pdfs/HeKoRe2013-ICAC-Elasticity.pdf)
  - Dynamically adjust to service demand
  - Short-term strategy
- [Scalability](https://en.wikipedia.org/wiki/Scalability)
  - Vertical and horizontal
  - Long-term strategy
- [Pooling](https://blogs.technet.microsoft.com/yungchou/2013/08/08/resource-pooling-virtualization-fabric-and-cloud/)
  - Compute, storage, network
  - Services "on tap"
  - Pay only for what you use
- [Provisioning](https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/configure-automatic-user-provisioning-portal)
  - Self-service
  - RBAC - Role Based Access Control
  
  ### Cloud Infrastructure Models ###
  
- [Public](https://azure.microsoft.com/en-us/overview/what-is-a-public-cloud/)
  - All services exist in the internet
  - Multi-tenancy
  - Azure
- [Private](https://azure.microsoft.com/en-us/overview/what-is-a-private-cloud/)
  - All services exists in the private network complex
  - System center
  - Azure Stack
- [Hybrid](https://azure.microsoft.com/en-in/overview/what-is-hybrid-cloud-computing/)
  - Secure, private connection between public and private clouds
  - VPN or ExpressRoute
  
  ### Cloud Deliver Models ###
  
 - [SaaS](https://azure.microsoft.com/en-us/overview/what-is-saas/)
   - Target: Office 365
 - [PaaS](https://azure.microsoft.com/en-us/overview/what-is-paas/)
   - Target:
     - Developer
     - Azure App Service
- [IaaS](https://azure.microsoft.com/en-us/overview/what-is-iaas/)
  - Target:
    - Sysadmin
    - Azure Virtual Machines

** [Availability Zone](https://docs.microsoft.com/en-us/azure/availability-zones/az-overview) - zone of availability inside the region
A region is a set of Azure data centers in a named geographic location. Every Azure resource, including virtual machines, is assigned a region

## Virtual Machines ##

### Which Model is Best For Your Workload? ###

- PaaS
  - Higher agility
  - Higher ease of management
  - Lower degree of control
  - Lower support for legacy apps
- IaaS (VM's)
  - Higher degree of control
  - Higher support for legacy apps
  - Lower ease of management
  - Lower agility
  
### Things to Keep in Mind Regarding Azure VMs ###
  
- Unless the VM is deallocated, it still incurs charge
- Deleting the VM doesn't delete the managed disk
- Related assets are charged separately
- You can't connect without an NSG (network security group) rule
- You can't connect to VMs in other virtual networks
- DNS names require creativity and should be standardized

### Virtual Machine Resources ###

- SRP (storage resource provider)
  - Disk (blob)
  - Storage account
- CRP (compute resource provider)
  - Virtual machine
  - Availability set
- NRP (network resource provider)
  - VM IP address
  - Virtual network
  - NSG
  - Load balancer
  
Scale refers to adding network bandwidth, memory, storage, or compute power to achieve better performance.

Scaling up, or vertical scaling, means to increase the memory, storage, or compute power on an existing virtual machine. For example, you can add additional memory to a web or database server to make it run faster.

Scaling out, or horizontal scaling, means to add extra virtual machines to power your application. For example, you might create many virtual machines configured in exactly the same way and use a load balancer to distribute work across them.

Azure Advisor and Azure Cost Management are two services that help you optimize cloud spend. You can use these services to identify where you're using more than you need, and then scale back to the capacity you're actually using.

### Azure VMs base types ###

Type | Sizes | Description
--- | --- | ---
General purpose | Dsv3, Dv3, DSv2, Dv2, DS, D, Av2, A0-7 | Balanced CPU-to-memory. Ideal for dev/test and small to medium applications and data solutions.
Compute optimized | Fs, F | High CPU-to-memory. Good for medium-traffic applications, network appliances, and batch processes.
Memory optimized | Esv3, Ev3, M, GS, G, DSv2, DS, Dv2, D | High memory-to-core. Great for relational databases, medium to large caches, and in-memory analytics.
Storage optimized | Ls | High disk throughput and IO. Ideal for big data, SQL, and NoSQL databases.
GPU optimized | NV, NC | Specialized VMs targeted for heavy graphic rendering and video editing.
High performance | H, A8-11 | Our most powerful CPU VMs with optional high-throughput network interfaces (RDMA).

### Connect to Windows VMs ###

- RDP
  - TCP 3389
  - relies upon the NSG
- PowerShell
  - TCP 5985/5986

### Connect to Linux VMs ###

- Secure Shell (SSH)
  - Industry standard, secure remote access protocol
- Authentication
  - Password or keys
- RDP
  - Install a desktop environment, RDP server, and create an NSG rule

## Storage ##

### Things to Keep in Mind Regarding Azure Storage for IaaS VMs ###

- Standard disks cost per transaction and per GB
- Premium disks aren't charged by transaction
- Azure holds an infinite lease on page blobs
- Blob storage namespace is flat
- Premium storage supports only LRS
- 99.9% read/write availability SLA

### Azure Storage Types ###

- Blob - unstructured file data.
  - A collection of binary data stored as a single entity in a storage system. Blobs are typically images, audio or other multimedia objects, or binary executable code. VM VHDs are stored as page blobs. 
- Table - NoSQL semi-structured data
- Queue - pub\sub messaging data
- File storage - SMB file shares

### Replication Options ###

- LRS Locally redundant storage
  - 3 copies withn single data center in single region
  - For premium storage only
- ZRS Zore redundant storage
  - 3 copies across 2-3 data centers in single region
  - block blobs only; available only during SA creation
  - available only at VM creation
- GRS Geo redundant storage
  - 3 copies in primary region
  - 3 copies in secondary region
- RA-GRS Read-Access geo redundant storage
  - RO access to secondary region data

### Azure VM Disk Types ###

- OS Disks
  - Fixed disk VHD files stored as page blobs
  - Generation 1 (VHD) // Generation 2 is VHDx format
  - Registered as SATA
  - Maximum size 1023 GB (1TB)
- Data Disks
  - D: is a temporary disk that holds pagefile/swapfile
  - Attached VHD
  - Registered as SCSI
  - Maximum size 1023 GB (1TB)

### Azure Storage Account Types ###

- Standard
  - HDD
  - Default for some instances sizes; others use SSD
  - IOPS values represent maximum values (IOPS input-ouput per second)
  - Can use any redundancy option ()Premium is LRS only)
- Premium
  - SSD
  - OIPS alues represent expected performance levels (not maximum)
  - Great for I/O intensive workloads like Dynamics, Exchange Server, SQL server
  - Not available in all Azure regions
  - Need a DS-, DSv2-, GS-, or FS-series VM
  - Available in 3 sizes (128 GB, 512 GB, or 1024 GB)

### Virtual Network Service Endpoints ###

Associate a storage account with a VNet
Limit access to storage account from VNet
NAT rules to support on-premises connection (NAT network address rules)

### Host Caching ###

Host - is hardware host, HyperV VM

- None
  - Disable on IaaS DCs
  - Good for random I/O
- Read only
  - Write-through
  - Stored on disk and RAM of host OS
- Read\write
  - Write-back
  - Stored in memory of host OS

## VM Configuration ##

### Things to Keep in Mind Regarding Azure VMs Configuration ###

- NSG rules
- Need availability sets for 99.95% SLA
- Premium storage offers 99.9% for single VMs
- Shared responsibility model
- Combine Azure Load Balancer with Avail. Sets
- Use separate storage accounts for each VM
	
** to connect to an Azure-based VM that isn't in your Active Directory domain add an entry to workstation's Trusted Hosts list.

## Scaling and High Availability ##

### Things to Keep in Mind Regarding Scaling and High Availability ###

- Group related VMs into availability sets
- Combine a load balancer with availabiilty sets
- Use separate storage accounts for each AS
- Consider Scale Sets for bigger compute jobs
- Know that AS and SS both incur runtime charges
- Premium storage supports single-instance SLA

### Azure Maintenance Events ###

- Planned - the azure team gives you advances notification
- Unplanned - rack - or datacenter-level failures

### Fault and Update Domains ###

- Fault domains are VMs that share the same power source and switch
  - 3 fault domains available
- Update domains are VMs that share the same hardware host
  - 5-20 update domain available
- Place VMs of each app tier into their own availability sets

** Availability set can be configured only when creating a VM workaround can be done via PS

### VM scale set ###

- Platform-independent PaaS
  - Azure App Service is known for elastic auto scaling
- Method for:
  - Deploying and managing Azure VMs as a set
  - Scalable compute platform
- Integrated with:
  - Azure load balancer
  - Azure auto scale

** scale set 

### Scale set use cases ###

- Hyperscale workloads
- Stateless web frontends
- Container orchestration
- Microservices clusters

### Load balancer ###

- Entity which watches to the internet and distributes traffic among recipients
- A secure entry point to VNet
- Works at OSI layer 4 (TCP/UDP)
- Uses 5-tuple has algorithm
  - Source and destination IP
  - Source and destination port
  - Protocol

### Application gateway ###

- Works OSI layer 7 (HTTP/HTTPS)
- SSL offload

### Traffic manager ###

- Global load balancer
- Uses DNS
- Geo-distributed services

## Networking ##

*Virtual Network (VNet)* - is a communications and security boundary that enables Azure resources (virtual machines, storage accounts, App Services apps, Azure SQL Database instances) to communicate with each other securely.

- 50-100 VNets allowed per subscription
- Provides network isolation and segmentation
- Name resolution (Azure-provided DNS; Azure DNS service)
- Traffic filtering (NSGs, network virtual appliances)

### When multiple VNets make sense ###

- Saving money
  - Share a network virtual appliance among several VNets (service chaining)
- Segmentation workloads
  - NSGs and UDRs give you routing and traffic control
- Securing traffic
  - Private connectivity that uses the Microsoft backbone network

*Virtual Private Network (VPN)* - is a secure connection over an unsecure medium. Azure site-to-site VPNs use IPSec/IKE tunnels

### VNet-to-VNet VPN ###

- Create isolation or administrative boundaries
- Provide cross-region geo-redundancy and replication securely
- No traffic crosses the public internet
- Private IP addressing
	
### VNet-to-VNet VPN Points to Ponder ###

- Different pricing model from VNet peering
- Make sure your VNet address spaces do not overlap
- Global VNet peering means you no longer need a VPN gateway for cross-region connectivity

** for P2S and VNet-to-VNet VPN deploy route-based Azure VPN gateway

*Network Peering* - is a seamless connection between two Azure virtual networks. The peered networks appear as one, or connectivity purposes.

### Inter-VNet troubleshooting tips ###

- Azure blocks ICMP (Internet Communication Messaging Protocol (uses for ping)) between VNets and the internet
- Simplify your NSGs as much as possible to reduce troubleshooting friction
- Azure portal diagnose and solve problems/Resource Health blade is useful
- Network watcher and network performance monitor make troubleshooting much easier

### Troubleshooting ###

- tcping
- Network watcher
  - Enable for your azure region(s)
  - Install VM extension
- Network performance monitor (NPM)
  - Part of Insight & Analytics azure management solution
  - Lag analytics workspace
  - Microsoft monitoring agent

### Things to Keep in Mind Regarding Networking ###

- Most regions support dual-stack IP
- Reserved public IPs can be chargeable
- Vnet-to-Vnet requires peering or VPN
- Remember NSG and Windows Firewall
- Configuring a local VPN gateway has complexity
- Azure DNS questions

### Public UP address SKUs ###

- Basic
  - The original public IP address
  - Static or dynamic allocation
  - Assigned to any public IP-addressable resource
  - Can be assigned to a specific availability zone
  - Not zone redundant
- Standard
  - Different price point
  - Static allocation only
  - Assigned to vNICs or standard internet-facing load balancers
  - Zone redundant by default

### Azure VPN SKUs ###

**basic, standard, high performance

- Basic
  - 80-100 Mbps, 99.9% SLA
- Dev/test
- New SKUs all have 99.95% SLA and are faster
  - VpnGw1
    - 650 Mbps
  - VpnGw2
    - 1 Gbps
  - VpnGw3
    - 1.25 Gbps

### Name Resolution for Azure VNets ###

- Azure-provided name resolution
  - No configuration required
  - All VMs within a VNet can resolve each others' host names
  - Issue: cross-VNet name resolution
  - Issue: No custom DNS suffix
  - You can add custom DNS server IP addresses
  - You can host your own DNS server(s)
- Azure DNS
  - Host your public DNS domain in Azure
    - Use Azure geo-distributed name servers
  - Create private DNS zones
    - Linked to VNets
    - Registration VNet
    - Resolution VNet

### Azure VNet Design Best Practices ###

- Create subnets based on workloads
- Bind network security groups (NSGs) at the subnet level
- Deploy a network virtual appliance (NVA) and user-defined routes (UDRs) to further customize traffic
- Implement site-to-site or point-to-site VPN tunnels with on-premises environment
	
### Network Security Groups ###
	
  **(NSGs)
  **Software firewall object

- Stateful firewall for inbound and outbound traffic
- 5-tuple hash src+dst IP and ports; protocol
- Default rules
- Augmented rules
- Service tags and ASGs
- Bound to vNIC or subnet
	
### VM IP addressing best practices ###

- If a VM doesn't need a public IP address (PIP), then don't assign one and use an Azure load balancer instead
- Plan your VNet private address space to avoid overlap
- Never configure networking from within the VM

## Security ##

### Terminology ###

*Asset* - people, property, or information (databases, software, code, company records)

*Threat* - person or process that can exploit a vulnerability (intentionally or accidentally) to obtain, damage or destroy an asset

*Vulnerability* - weakness or gap in a security program that can be exploited by threats to gain unauthorized access to an asset

*Risk* - the potential for loss, damage, or destruction of an asset as a result of a threat exploiting a vulnerability

### Access Concepts ###

- Least privilege
  - Limiting resource access to what is necessary and no more (scope permissions)
- RBAC
  - Regulating resource access based on job role

### Things to Keep in Mind Regarding Security ###

- Azure conforms with many compliance certs
- Leverage defence in depth
- You can get Microsoft's permissions to run pen tests
- Consider virtual network security appliances
- Control routing behaviour
- Enable azure security center

- Logically segment subnets
  - Use NSGs for inter-subnet traffic
- Control routing
  - Consider virtual network appliances
- Enable forced tunneling
  - Constrain outbound VM traffic
- Disable RDP/SSH
  - Employ VPNs and jump box
- Web Application Firewall
  - Centralised web application protection from common exploits and vulnerabilities
- DDoS Protection
  - Always-on traffic monitoring and real-time DDoS mitigation
- Azure Security Center
  - Unify security management and enable advanced threat protection


### Protect the Storage Subsystem ###

- Protecting the storage account itself
  - Managed disk storage vs. traditional storage account
- RBAC
  - Simplify it AMAP
- Storage account keys (use for traditional storage accounts)
  - 512-bit string created by Azure
  - Key rollover
  - Azure Key Vault
    - Secret protection store

#### Role based access control (RBAC) ####
regulating resource access based on job role. It is a granular, role-based resource access to users, groups, and applications.

### Roles ###

- Owner
  - Full access
- Contributor
  - Full resource access but no grant/revoke privileges
- Reader
  - Read-only resource access
- Custom
  - Defined programmatically with JSON, PS, CLI or REST API

### Azure Resource Manager Policies ###

- RBAC focuses on user actions at different scopes
- ARM policies focus on resource properties during deployment
  - Require tags for cost center management
  - Constrain resources to particular regions
- Create policy definitions with JSON
  - New-AzureRmPolicyDefinition
  - New-AzureRmPolicyAssignment
		
### Data Plane Security ###

- Protecting storage account services
  - Blobs, files, queues, tables
- Shared access signatures (SAS)
  - Time-limited URIs
  - CRUD
- Stored Access Policies
  - Standardised, reusable SAS rules

### Encryption in Transit ###

- Enable HTTPS
  - App Services apps
  - IaaS web apps
  - Blob service URIs
- SMB 3.0 encryption in Azure File Share service
- Client-side encryption
  - Protects user app data before it's uploaded to Azure
  - Accessed through the Azure SDK libraries

### Encryption at Rest ###

- Storage Service Encryption (SSE)
  - On-the-fly storage account encryption
- Azure Disk Encryption
  - BitLocker for Windows Server VM OS and data disks
  - DM-Crypt for Linux VM disks

### Storage Analytics ###

- Storage account logging and metrics data
  - Storage Analytics metrics are available for the Blob, Queue, Table, and File services
  - Storage Analytics logging is available for the Blob, Queue, and Table services. However, premium storage account is not supported
  - Storage accounts with a replication type of Zone-Redundant Storage (ZRS) do not have the metrics or logging capability enabled at this time
- Requests
  - Successful and failed access
  - SAS requests
  - Request for analytics data
- Accessible (as always) through REST API

### Firewall ###

- NSG
  - Can be attached to vNIC or vNet subnet
- Software Firewall
  - May or may not be necessary
  - More restrictive rules than NSG (cant overwrite NGS rules)
- Virtual Appliances
  - VM images from leading third-party OEMs

### Antimalware ###

- VM Extensions
  - Microsoft Monitoring Agent
  - Third-party partners
- Antimalware Assessment
  - Part of Security & Compliance management solution
  - [Azure security center](https://docs.microsoft.com/en-us/azure/security-center/security-center-partner-integration)
			Azure Security Center makes it easy to enable integrated third-party partner security solutions. Benefits include:
    - Simplified deployment: Security Center offers streamlined provisioning of integrated partner solutions. For solutions like antimalware and vulnerability assessment, Security Center can provision the needed agent on your virtual machines. For firewall appliances, Security Center can take care of much of the network configuration required.
    - Integrated detections: Security events from partner solutions are automatically collected, aggregated, and displayed as part of Security Center alerts and incidents. These events also are correlated with detections from other sources to provide advanced threat detection capabilities.
    - Unified health monitoring and management: Integrated health events provide monitoring of all partner solutions at a glance. Basic management is available, with easy access to advanced setup using the partner solution.
    - Currently, integrated security solutions include:
      - Endpoint protection such as Trend Micro, Symantec, System Center Endpoint Protection (SCEP), and Windows Defender
      - Web application firewalls (WAF) such as Azure Application Gateway, Barracuda, F5, Fortinet, and Imperva
      - Next-generation firewalls (NGFW) such as Barracuda, Check Point, Cisco, Fortinet, and Palo Alto Networks
      - Vulnerability assessment such as Qualys
- Log analytics

### Azure Security Center ###

Provides:
- Unified security management
- Advanced threat protection
  - Machine learning and behavioral analytics
- Protection for hybrid cloud workloads
  - Onboard and protect Azure and on-premises resources

It monitors:
- Windows Server and Linux VMs
  - In Azure and on-premises
- vNets
- Azure SQL
- Azure storage accounts
- Azure web apps in ASE
- Partner solutions (WAFs)

Has kind of alerts:
- Unencrypted disks
- Missing or out-of-date antimalware
- Attacks:
  - Brute force attacks against VMs
  - Alerts from antimalware/firewall software
  - Compromised VMs communicating with known malicious IP addresses
  - SQL injections against Azure SQL and IaaS SQL Server databases

Notes:
- ASC provides 30-days trial period.
- For monitoring it uses Microsoft Monitoring Agent available for Windows and Linux.
- It stores its data in a traditional Azure storage account.
- Forced tunneling requires selected Azure VMs to access the Internet through the on-premises network.
- Azure uses Dm-Crypt to encrypt Linux VM OS and data disks.

### ASC policy component ###

Consists of:
- Pricing tier
- Webhooks
- Prevention policy

### Azure activity log ###

- Formerly known as “audit logs”  or “operational logs”
- Separate and distinct from diagnostic logs
- Report control-plane events on your subscription
- Great flexibility in query, report and export

** set up security center installs Microsoft Monitoring Agent to the VMs
 
### Patch Management for Azure VMs ###

- WSUS (Windows Server Update Services)
  - Local network
  - Virtual network
- System Center
  - Local network
  - Virtual network
- Update Management Solution
  - Part of Azure Automation service
  - Manage updates for Windows server and Linux endpoint

## Monitoring ##

### Three monitoring tiers ###

- Resource health
- Monitor Center
- Log Analytics and Operations Management suite (OMS)

### Ways to check Azure service status ###

- [Twitter](https://twitter.com/azurestatus)
- [Azure site](https://azure.microsoft.com/en-us/status/)
- Service health on the Azure portal

### Things to Keep in Mind Regarding Azure VM Monitoring ###

- You're free to use native tools (PerfMon, ps)
- With hybrid cloud, you can use System Center
- Take Azure into account when evaluating mon. solution
- OMS (operation management suit) is a tightly integrated solution
- Learn about webhooks, REST, and SQL
- Consider streaming data via Event Hub

** Log analytics sets inside an OMS workspace which has a free tier option

### Tools ###

#### Infrastructure provides ####

- Application logs and metrics
- Diagnostic logs
- Metrics
- Activity log

#### Analysing tools ####

- Visualize
  - Graphics
  - Portal views
  - Dashboards
  - Charts
- Archive
  - Store in blob storage
  - Audit trail
  - Security analysis
- Query
  - REST API
  - CLI
  - PowerShell
- Route
  - Streaming
  - Notification service
- Automate
  - Webhooks
  - Autoscale
  - Azure automation
  - Azure runbooks
		

### Monitoring - A layer Approach ###

- Resource level
  - Internal tools
  - Metrics, alert rules
- Resource group level
  - Monitor blade
- Enterprise level
  - Hybrid nodes
  - Log analytics
  - Azure automation

*Telemetry* - is an automated communications process by which measurements and other data are collected at remote or inaccessible points and transmitted to receiving equipment for monitoring.

### Monitoring and Diagnostics Pipeline ###

<img src="https://docs.microsoft.com/en-us/azure/architecture/best-practices/images/monitoring/pipeline.png"/>

### Azure Monitor Components ###

- Activity log
  - On a Resource Group scale
- Metrics
  - Plot data points to a custom, sharable dashboard
- Diagnostic logs
  - Aggregated data can be analyzed with Power BI
- Alerts
  - Get notified when metrics exceed threshold values
  - Automatic remediation (Azure automation runbook, webhook)

### Azure diagnostics data ###

- Event tracing
  - Configured by app developers
- Performance counters
  - The 4 traditional subsystems
- Event logs
  - ETW
- Application logs
  - Configured by app developers
- Azure diagnostics logs
  - Table and/or blob storage

### Azure resource log ###

- Activity log (audit log)
  - Records operational details
  - "Who did what, and when" in Azure
- Diagnostic log
  - Performance counters
  - Event logs (system, application, security)
  - IIS logs
  - Crash dump data
  - Custom error log

### Monitoring - Questions of scale and intelligence ###

- Log flood
  - So many data sources
  - How to quickly extract value?
- Predictive analytics
  - How to know what to look for?
- Automation
  - Immediate remediation

### Log Analytics ###

#### Provides ####

- Monitoring
- Analysis
- Alerting
- Remediation

#### Data sources ####

- Counters
- Logs
- App Insights
- Agent devices (cloud and on-prem)
- SCOM management groups

[Log Analytics query syntax](https://docs.microsoft.com/en-us/azure/azure-monitor/log-query/query-language)

#### The need for an alerting solution ####

- Avoiding surprises
- Email, SMS
- Third-party security and event management (SIEM)
- Azure integration is eminently possible

*Webhook* - is a way for an app to provide other applications with real-time information. Also called a web callback or an HTTP push API. The payload is ordinarily JSON.

### What can you do with Webhooks ###

- Third-party integrations
  - PagerDuty, OpsGenie, VictorOps, Slack, HipChat, Campfire, etc.…
- Script execution
  - Azure automation runbooks
- SMS text
  - Twilio API
 
## Troubleshooting and support ##

### Azure VM troubleshooting checklist ###

- Reset
  - Admin credentials
  - RDP configuration
- Verify
  - NSG rules
- Review
  - Console logs
- Check VM resource health
- Restart
- Redeploy
  - Migrates VM to a new hardware host
  - VM will be down during the migration
- Delete
  - Only the config, not the NIC or storage
  - Attack disk to working VM
  - Tweak registry
  - Recreate

## Cost management ##

### How to buy azure services ###

- Pay-as-You-Go
  - No minimum spend
  - Pay for what you use
  - Cancel anytime
- Enterprise agreement (EA)
  - Advantage: forecasting
  - Azure enterprise portal
		

Trial is not automatically converted
Analyze resource usage
Set billing alerts
Consider spending limits


### Azure SLA ###

*Service-Level Agreement (SLA)* - is an official commitment between a service provider and a customer.

### Azure support plans ###

- Included
  - Free
- Developer
  - 29$/month
- Standard
  - 300$/month
- Professional direct
  - 1,000$/month
- Premier
  - Contact Microsoft

## Managed disks ##

### Storage account limits ###

- Number of storage account per subscription is 200
- TB per storage account is 500 TB

### Value proposition ###

- Azure manages VHDs
- Node direct public access
- Disk-level RBAC

- Scaling that's free of storage account limitations
- VM scale set support

- Availability set storage isolation

### Managed disks facts ###

- LRS only
- Fixed sizes:
  - Standard: 32 Gb, 64 Gb, 128 Gb, 512 Gb, 1 TB
  - Premium: 128 Gb, 512 Gb, 1 TB
- Can create empty managed disk

## Help and support ##

### Azure support issue types ###

- Technical
- Quota
- Billing
