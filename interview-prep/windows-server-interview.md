# Windows Server Administration — Interview Preparation

These questions are based on concepts practiced in the Northstar Solutions enterprise IT homelab.

## 1. Why would you assign a static IP address to a server?

A server providing core infrastructure services should generally have predictable network addressing so clients and other systems can reliably locate its services.

In my Northstar homelab, I configured `NS-DC01` with `192.168.252.10` rather than leaving it on a dynamically assigned VMware DHCP address. I first inspected the DHCP allocation range and selected an address outside that range.

In a production environment, I would follow the organization's IP address management and network design standards rather than independently choosing an address.

---

## 2. What is the difference between DHCP and a static IP address?

DHCP automatically provides network configuration to a client, including settings such as its IP address, subnet mask, gateway, and DNS servers.

A manually configured static address remains predictable unless an administrator changes it.

In my lab, `NS-DC01` originally received `192.168.252.129` through VMware DHCP. I later configured `192.168.252.10` manually because the server is being prepared to provide core infrastructure services.

---

## 3. What does a default gateway do?

A default gateway provides a path toward destinations outside the host's local subnet.

In my lab, `NS-DC01` is on `192.168.252.0/24`, and the VMware NAT gateway is `192.168.252.2`.

I verified gateway connectivity using `ping` after configuring the server's static address.

---

## 4. How would you determine whether a problem is Internet connectivity or DNS?

I would test the layers separately instead of assuming that failure to open a website means the Internet connection is down.

For example, I can first verify the local configuration and gateway, then test an external IP address. If an external IP responds but hostname resolution fails, that points toward a DNS problem rather than general IP connectivity.

I observed this directly in my homelab. `NS-DC01` successfully reached `8.8.8.8`, but `nslookup microsoft.com` failed because the server was configured to use itself for DNS before the DNS Server role had been deployed.

---

## 5. Why is DNS important to Active Directory?

Active Directory depends heavily on DNS.

Domain clients use DNS not only for basic hostname resolution but also to locate Domain Controllers and Active Directory services.

Because of this, DNS configuration is an important part of deploying and troubleshooting an Active Directory environment.

---

## 6. What is the difference between a Windows Server role and a feature?

A server role represents a primary responsibility performed by the server, such as Active Directory Domain Services, DNS Server, or DHCP Server.

Features provide additional functionality that can support the operating system or installed roles.

When installing a server role, Windows may automatically require supporting features.

---

## 7. How did you configure your Windows Server homelab?

I deployed Windows Server 2025 in VMware Workstation and configured the server as `NS-DC01`.

Before installing Active Directory, I inspected the VMware network, identified its subnet, gateway, and DHCP allocation range, and converted the server from DHCP to a static address.

I then verified gateway and external IP connectivity and documented the server's pre-Active Directory baseline.

The next phase of the project is deploying and validating Active Directory Domain Services and DNS.