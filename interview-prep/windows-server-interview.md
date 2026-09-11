# Windows Server Administration — Interview Preparation

This document contains interview questions and sample answers based on the hands-on Windows Server, networking, Active Directory, and DNS work completed in the Northstar Solutions enterprise IT homelab.

---

# 1. Windows Server and Network Configuration

## 1. Why would you assign a static IP address to a server?

A server providing core infrastructure services should generally have predictable network addressing so clients and other systems can reliably locate its services.

In my Northstar homelab, `NS-DC01` originally received `192.168.252.129` through VMware DHCP.

Before deploying Active Directory and DNS, I inspected the VMware subnet and DHCP allocation range and configured the server with:

`192.168.252.10`

The address is outside the VMware DHCP allocation range used by the lab.

In production, I would follow the organization's IP address management and network design standards rather than independently selecting an address.

## 2. What is the difference between DHCP and static addressing?

DHCP automatically provides network configuration such as an IP address, subnet mask, gateway, and DNS server information.

A manually configured static address remains predictable unless an administrator changes it.

In my lab, I changed `NS-DC01` from DHCP to static addressing before making it a Domain Controller and DNS Server.

## 3. What does a default gateway do?

A default gateway provides a path toward destinations outside the host's local subnet.

`NS-DC01` is connected to:

`192.168.252.0/24`

and uses the VMware NAT gateway:

`192.168.252.2`

I verified gateway connectivity after configuring the server's static address.

## 4. How would you distinguish a DNS problem from general network connectivity?

I would test the functions separately.

For example, I can verify the system's IP configuration, test the local gateway, and then test an external IP address.

If IP connectivity works but hostname resolution fails, I would investigate DNS rather than assuming that general network connectivity is unavailable.

I observed this directly in my lab. Before DNS Server was installed, `NS-DC01` successfully reached `8.8.8.8`, but a DNS query to its configured DNS address failed because the server was pointing to itself before the DNS service existed.

After DNS Server deployment, external DNS resolution succeeded.

---

# 2. Windows Server Roles and Active Directory

## 1. What is the difference between a Windows Server role and a feature?

A server role represents a primary responsibility performed by the server, such as Active Directory Domain Services, DNS Server, or DHCP Server.

A feature provides additional operating-system functionality or supports installed roles.

In my lab, I installed the Active Directory Domain Services and DNS Server roles on `NS-DC01`.

## 2. What is Active Directory Domain Services?

Active Directory Domain Services provides centralized directory, identity, authentication, and management capabilities for Windows domain environments.

It can centrally manage objects such as users, computers, groups, and Organizational Units and supports technologies such as Group Policy.

In my homelab, I installed AD DS on Windows Server 2025 and promoted `NS-DC01` as the first Domain Controller.

## 3. What is the difference between a forest and a domain?

A forest is the top-level logical structure of an Active Directory environment and can contain one or more domains.

A domain is a logical administrative and security structure containing directory objects such as users, computers, and groups.

My Northstar environment currently uses a single-domain forest:

`ad.northstarsolutions.com`

## 4. What is a Domain Controller?

A Domain Controller is a Windows Server running Active Directory Domain Services and providing directory services for an Active Directory domain.

Domain Controllers participate in authentication, directory queries, Group Policy processing, and Active Directory service discovery.

My first Northstar Domain Controller is:

`NS-DC01.ad.northstarsolutions.com`

## 5. Does installing the AD DS role automatically make a server a Domain Controller?

No.

Installing the Active Directory Domain Services role installs the required server components, but the server must still be promoted to a Domain Controller.

In my lab, I first installed AD DS and DNS Server and then promoted `NS-DC01` as the first Domain Controller in a new forest.

## 6. What happens when you promote the first Domain Controller in a new forest?

In my lab, promotion created the new Active Directory forest and root domain:

`ad.northstarsolutions.com`

It configured the NetBIOS domain name:

`NORTHSTAR`

and configured `NS-DC01` as a Domain Controller, DNS Server, and Global Catalog.

After promotion, the server restarted and I validated the resulting domain and forest configuration.

---

# 3. Active Directory and DNS

## 1. Why is DNS important to Active Directory?

Active Directory depends heavily on DNS.

Domain clients use DNS not only for normal hostname resolution but also to locate Domain Controllers and Active Directory services.

Because of that, incorrect DNS configuration can cause domain joins, authentication, Group Policy, and other Active Directory operations to fail.

## 2. Why does your Domain Controller use itself as its DNS server?

`NS-DC01` hosts DNS for the Northstar Active Directory environment, so its preferred DNS server is configured as:

`192.168.252.10`

which is its own address.

This allows the Domain Controller to use the Active Directory-aware internal DNS infrastructure.

Domain clients will also need to use the Northstar internal DNS service when they are integrated into the domain.

## 3. Why shouldn't an Active Directory client simply use a public DNS server such as 8.8.8.8?

Public DNS resolvers do not contain the private Active Directory DNS records required for the Northstar domain.

Domain clients need to query the internal DNS infrastructure so they can locate Domain Controllers and other Active Directory services.

The internal DNS server can handle the Active Directory namespace while still providing resolution for external names.

## 4. How did you verify DNS after deployment?

I tested both internal and external DNS resolution.

For the internal environment, I used:

```powershell
Resolve-DnsName NS-DC01.ad.northstarsolutions.com
```

and confirmed that the Domain Controller FQDN resolved to:

`192.168.252.10`

I then tested:

```powershell
Resolve-DnsName microsoft.com
```

and confirmed that external DNS resolution also succeeded.

This allowed me to verify internal and external resolution separately.

## 5. What was the DNS delegation warning you encountered during Domain Controller promotion?

During creation of the new forest, the wizard indicated that a DNS delegation could not be created because an authoritative parent zone could not be found.

In my isolated homelab, there was no existing authoritative parent DNS infrastructure managing the simulated namespace.

I therefore did not create a delegation, and the warning did not prevent successful deployment of the new forest and DNS environment.

---

# 4. Active Directory Validation

## 1. How did you verify that the Domain Controller was working after promotion?

I did not rely only on the installation wizard.

After the restart, I verified the server identity and administrative context using:

```powershell
hostname
whoami
```

I then inspected the domain and forest using:

```powershell
Get-ADDomain
Get-ADForest
```

and checked the core services using:

```powershell
Get-Service NTDS,DNS
```

The results confirmed the correct domain and forest, the Domain Controller, Global Catalog configuration, and that both Active Directory Domain Services and DNS Server were running.

I also tested internal and external DNS resolution.

## 2. What is a Global Catalog?

A Global Catalog is a Domain Controller capability that provides a searchable representation of objects across an Active Directory forest and supports important directory operations.

During my lab, `NS-DC01` was configured as a Global Catalog.

I verified that configuration using `Get-ADForest`.

## 3. What is DSRM?

DSRM stands for Directory Services Restore Mode.

It is a special recovery mode used for Active Directory maintenance and recovery.

I configured a DSRM password during Domain Controller promotion, but I do not store that password in my public project documentation.

## 4. What are forest and domain functional levels?

Forest and domain functional levels determine which Active Directory capabilities are available and which Windows Server versions can participate as Domain Controllers.

My lab uses Windows Server 2025 forest and domain functional levels because the environment is being built using Windows Server 2025 Domain Controller infrastructure.

In production, compatibility with existing infrastructure would need to be evaluated before changing functional levels.

## 5. What are FSMO roles?

FSMO stands for Flexible Single Master Operations.

Active Directory uses several FSMO roles for directory operations that require a single authoritative role holder.

During validation of my new single-Domain-Controller forest, `NS-DC01` appeared as the role holder for the domain and forest roles returned by `Get-ADDomain` and `Get-ADForest`.

The roles included:

- PDC Emulator
- RID Master
- Infrastructure Master
- Schema Master
- Domain Naming Master

I will be studying FSMO role administration in more depth later in the project.

---

# 5. Homelab Design and Production Considerations

## 1. How did you configure your Windows Server homelab?

I deployed Windows Server 2025 in VMware Workstation and standardized the server as `NS-DC01`.

Before deploying Active Directory, I inspected the VMware network, identified the subnet, gateway, and DHCP allocation range, and changed the server from DHCP to a predictable static address.

I verified local gateway and external IP connectivity and established a pre-DNS baseline.

I then installed Active Directory Domain Services and DNS Server and promoted `NS-DC01` as the first Domain Controller in a new forest using `ad.northstarsolutions.com`.

After the restart, I validated the domain, forest, Global Catalog, AD DS and DNS services, and internal and external DNS resolution using Windows Server tools and PowerShell.

The next stage is centralized Active Directory administration, including Organizational Units, domain users, security groups, workstation domain integration, and Group Policy.

## 2. Would you deploy only one Domain Controller in production?

Not necessarily.

My homelab currently uses one Domain Controller because I am working with limited compute resources and the purpose is hands-on learning.

A production environment would evaluate redundancy and availability requirements and would commonly use multiple Domain Controllers and DNS servers where appropriate.

I would not describe my single-DC homelab design as production high availability.

## 3. What production practices are you trying to apply in your homelab?

I am applying practices such as:

- Standardized naming
- Predictable infrastructure addressing
- Configuration baselines
- Pre-change and post-change validation
- Internal Active Directory DNS
- Least privilege
- Evidence-based troubleshooting
- Documentation
- Production-versus-homelab design considerations

I also document limitations rather than presenting a resource-constrained VMware environment as identical to a production enterprise deployment.