

# 🌐 LAB 4 – Azure DNS: Public Zone & Private Zone Auto-registration

This lab is a fundamental exercise for Azure Administrators, demonstrating the configuration of **Public DNS Zones** in Azure and the use of **Private DNS Zones** with automatic VM hostname registration.

## 🧱 Skills Covered

  * Azure Public DNS Zones
  * Understanding **NS** and **SOA** records
  * Azure Private DNS Zones
  * VNet DNS integration and linking
  * Automatic registration of VM hostnames
  * Using `nslookup` for DNS diagnostics

## 📦 Architecture

The lab environment consists of:

  * **1 Public DNS Zone** (e.g., `tutorials-lab.com`)
  * **1 Private DNS Zone** (e.g., `corp.internal`)
  * **2 Virtual Networks** (one linked, one non-linked)
  * **2 Virtual Machines** (to test auto-registration and cross-VNet resolution)

-----

## ⚙️ Step-by-Step Lab Guide

### 🧩 Part A - Public DNS Zone Setup

The public zone is used to host a publicly accessible domain name.

#### 1\. Create a Public DNS Zone

Navigate to **Azure Portal → Create Resource → Networking → DNS zone**.

| Setting | Value |
| :--- | :--- |
| **Name** | `tutorial-lab.com` (or any test domain) |
| **Resource group** | `rg-az104-labs` |
| **Type** | **Public DNS zone** |

#### 2\. Review Default Records (NS & SOA)

After creation, review the default records added by Azure:

| Record Type | Purpose | Key Information |
| :--- | :--- | :--- |
| **NS** (Nameserver) | Specifies the Azure nameservers hosting your zone. | **Critical for domain delegation.** |
| **SOA** (Start-of-Authority) | Identifies the primary DNS server and contains default parameters like TTL. | **Contains primary server details.** |

> 📝 **Note for Production:** In a real-world scenario, you would copy Azure's **NS records** and paste them into your domain registrar's Name Server settings (e.g., GoDaddy). This process delegates control to Azure's DNS service.

-----

### 🧩 Part B - Private DNS Zone + Auto-registration

The private zone is used for internal, non-internet-routable name resolution.

#### 3\. Create Two VNets

| VNet Name | Address Space | Purpose |
| :--- | :--- | :--- |
| **vnet-dns1** | `10.50.0.0/16` | **Non-linked network** (Resolution expected to fail) |
| **vnet-dns2** | `10.60.0.0/16` | **Will support auto-registration** (Resolution expected to pass) |

> Use default subnets. No peering is required for this lab.

#### 4\. Create a Private DNS Zone

Navigate to **Azure Portal → Create Resource → Private DNS zone**.

| Setting | Value |
| :--- | :--- |
| **Name** | `corp.internal` |
| **Resource group** | `rg-az104-labs` |

> This zone is **internal-only** and not visible on the public internet.

#### 5\. Link `vnet-dns2` with Auto-registration Enabled

Go to your Private DNS Zone, select **Virtual network links → Add**.

| Setting | Value |
| :--- | :--- |
| **Link name** | `link-vnet-dns2` |
| **Virtual network** | `vnet-dns2` |
| **Enable auto-registration** | **ON** ✔️ |

> 🔥 **Important:** Only VNets linked with **Auto-registration** enabled will allow new VMs to automatically create A-records for their hostnames.

#### 6\. Create Two Virtual Machines

Use a small size like `Standard_B1s`.

| VM Name | VNet | Expected Behavior |
| :--- | :--- | :--- |
| **vm-dns1** | `vnet-dns1` | **No auto-registration** (VNet not linked) |
| **vm-dns2** | `vnet-dns2` | **Should auto-register** (VNet is linked) |

#### 7\. Verify Auto-registration

Go to: **Private DNS Zone → `corp.internal` → Records**.

  * You should see an **A** record for `vm-dns2` with its IP address (`10.60.x.x`).
  * The record for `vm-dns1` should **not** appear here.

> **Tip:** If `vm-dns2` is not visible, wait 1-2 minutes for the process to complete.

-----

## 🧪 Testing DNS Resolution

Open **PowerShell** within both Virtual Machines to test name resolution against the Private DNS Zone.

### Test from `vm-dns2` (Same VNet, Linked)

This test confirms resolution within a linked VNet.

```powershell
nslookup vm-dns2.corp.internal
```

**✅ Expected Result:** Success

```
Name:    vm-dns2.corp.internal
Address: 10.60.x.x
```

### Test from `vm-dns1` (Different VNet, Not Linked)

This test confirms that VNet linking is necessary for resolution.

```powershell
nslookup vm-dns2.corp.internal
```

**❌ Expected Result:** Failure

```
*** vm-dns1 can't find vm-dns2.corp.internal: Non-existent domain
```

-----

## 📘 Key Takeaways (AZ-104 Knowledge)

| Topic | What You Must Remember |
| :--- | :--- |
| **Public DNS Zones** | **NS records** must be added to the domain registrar for delegation to work. |
| **Private DNS Zones** | Name resolution only works for **linked VNets**. |
| **Auto-registration** | Only applies to VM NICs in VNets that have **auto-registration enabled** on the VNet link. |
| **Cross-VNet resolution** | Requires linking **both VNets** to the private zone (as a *Resolution VNet*). |
| **SOA record** | Contains the primary nameserver and TTL defaults. |

-----
