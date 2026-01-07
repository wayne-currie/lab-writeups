# Active Directory Homelab in Azure 

### Project Overview

In this project, I built an Active Directory environment in Azure by deploying a Windows Server 2022 domain controller in Azure and connected my local Windows 11 virtual machine running in VMware to the domain through a Point‑to‑Site VPN.

---
## Project Steps

### 1. Create an Azure Virtual Network (VNet)
Created a virtual network in Azure to provide isolated network infrastructure for the Active Directory environment.

**Configuration Details:**
- **Address Space:** 10.0.0.0/16
- **Subnets:**
  - GatewaySubnet: 10.0.1.0/24 (for VPN Gateway)
  - ServerSubnet: 10.0.2.0/24 (for Domain Controller)
- **Region:** [Your Azure Region]
- **Resource Group:** EntraID-Homelab-RG

---

### 2. Create a Virtual Network Gateway (VPN)
Deployed a VPN Gateway to establish secure Point-to-Site connectivity between local machines and Azure resources.

**Gateway Configuration:**
- **Gateway Type:** VPN
- **VPN Type:** Route-based
- **SKU:** VpnGw1 (or appropriate tier)
- **Address Pool:** 10.1.0.0/24 (for VPN clients)
- **Tunnel Type:** IKEv2
- **Deployment Time:** ~30-45 minutes

---

### 3. Generate Root and Client Certificates
Generated self-signed certificates for VPN authentication using PowerShell.

Run PowerShell as Administrator:

```powershell
# Step 1: Create a Self-Signed Root Certificate
$params = @{
    Type = 'Custom'
    Subject = 'CN=AzureRoot'
    KeySpec = 'Signature'
    KeyExportPolicy = 'Exportable'
    KeyUsage = 'CertSign'
    KeyUsageProperty = 'Sign'
    KeyLength = 2048
    HashAlgorithm = 'sha256'
    NotAfter = (Get-Date).AddMonths(24)
    CertStoreLocation = 'Cert:\CurrentUser\My' 
}
$cert = New-SelfSignedCertificate @params

# Export the root certificate public key (for Azure)
Export-Certificate -Cert $cert -FilePath "C:\AzureRoot.cer"
```

```powershell
# Step 2: Create Client Certificate
$clientParams = @{
    Type = 'Custom'
    Subject = 'CN=AzureClient'
    DnsName = 'AzureClient'
    KeySpec = 'Signature'
    KeyExportPolicy = 'Exportable'
    KeyLength = 2048
    HashAlgorithm = 'sha256'
    NotAfter = (Get-Date).AddMonths(18)
    CertStoreLocation = 'Cert:\CurrentUser\My'
    Signer = $cert
    TextExtension = @('2.5.29.37={text}1.3.6.1.5.5.7.3.2')
}
New-SelfSignedCertificate @clientParams
```

**Certificate Export Notes:**
- Root certificate exported as `.cer` file (public key only)
- Client certificate remains in personal certificate store
- Root certificate uploaded to Azure VPN Gateway configuration

---

### 4. Configure Point-to-Site in the Azure VPN Gateway
Configured P2S VPN settings in the Virtual Network Gateway to allow remote client connections.

**Steps Completed:**
1. Navigated to VPN Gateway → Point-to-site configuration
2. Uploaded root certificate public key (AzureRoot.cer)
3. Configured address pool: 10.1.0.0/24
4. Selected tunnel types: IKEv2 and OpenVPN (SSL)
5. Enabled Azure Active Directory authentication (optional)
6. Saved configuration and generated VPN client package

---

### 5. Download and Install the VPN Client
Downloaded the VPN client configuration package from Azure and installed it on the local Windows 11 machine.

**Installation Steps:**
1. Downloaded VPN client package from Azure Portal
2. Extracted ZIP file and ran `WindowsAmd64/VpnClientSetupAmd64.exe`
3. Verified VPN connection profile in Windows Network Settings
4. Ensured client certificate was installed in personal certificate store

---

### 6. Connect to the Azure P2S VPN
Established VPN connection from local Windows 11 machine to Azure VNet.

**Connection Verification:**
- VPN Status: Connected
- Assigned IP: 10.1.0.x (from VPN address pool)
- Gateway IP: 10.0.0.4
- Connection Protocol: IKEv2

![VPN Connection Status](https://github.com/user-attachments/assets/768f5c6a-b776-4187-9064-74847a66a125)

---

### 7. Deploy Windows Server 2022
Deployed a Windows Server 2022 virtual machine in Azure to serve as the foundation for the Active Directory environment.

**VM Specifications:**
- **VM Size:** Standard_B2s (2 vCPUs, 4 GB RAM)
- **OS:** Windows Server 2022 Datacenter
- **Disk:** 127 GB Premium SSD
- **Private IP:** 10.0.2.10 (static)
- **Network Security Group:** Allowed RDP (3389), DNS (53), AD traffic

**Initial Configuration:**
- Set static IP address in Azure NIC configuration
- Configured Windows Firewall rules for AD and DNS
- Disabled IE Enhanced Security Configuration
- Installed Windows Updates

![Windows Server VM Deployment](https://github.com/user-attachments/assets/630c10e4-15bf-4b2c-9db7-40d91ee84d89)

---

### 8. Configure Windows Server 2022 as a Domain Controller
Promoted the Windows Server 2022 instance to a Domain Controller and configured Active Directory services.

**Services Configured:**
- **Active Directory Domain Services (AD DS)**
- **DNS Server**
- **Domain Controller Promotion**

**Configuration Steps:**

1. **Install AD DS Role:**
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

2. **Promote to Domain Controller:**
```powershell
Install-ADDSForest `
    -DomainName "homelab.local" `
    -DomainNetbiosName "HOMELAB" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns:$true `
    -SafeModeAdministratorPassword (ConvertTo-SecureString "YourPassword" -AsPlainText -Force) `
    -Force:$true
```

3. **Post-Promotion Configuration:**
   - Verified DNS zones (Forward and Reverse Lookup)
   - Created Organizational Units (OUs) for Users, Computers, and Groups
   - Configured DNS forwarders (8.8.8.8, 1.1.1.1)
   - Set Azure VNet DNS to point to Domain Controller (10.0.2.10)

**Domain Information:**
- **Domain Name:** homelab.local
- **NetBIOS Name:** HOMELAB
- **Forest/Domain Functional Level:** Windows Server 2016
- **DNS Server:** 10.0.0.4

![Domain Controller Configuration](https://github.com/user-attachments/assets/4920f984-aeb5-4398-ac98-9e548ccc2e19)

---

### 9. Connect Windows 11 Host to Domain using Point-to-Site VPN
Successfully joined the local Windows 11 machine to the Azure-hosted domain through the P2S VPN connection.

**Connectivity Verification:**

**Pinging Windows Server Deployed in Azure from Windows 11:**

![Ping to Azure Server](https://github.com/user-attachments/assets/b0c38e16-b5b3-42ff-927e-dd4e4f4bf529)

**Pinging Domain Name from Windows 11 (homelab.local):**
Although I was able to ping the IP address of the Domain Controller, I was not able to ping the domain. Powershell was used to configure DNS via PowerShell
```bash
# See VPN interface details
Get-NetAdapter | Where-Object {$_.Name -like "*p2s*"}

# Set DNS to DC
Set-DnsClientServerAddress -InterfaceAlias "p2sVPN-VNet1" -ServerAddresses "10.0.0.4"

# Verify it worked
Get-DnsClientServerAddress -InterfaceAlias "p2sVPN-VNet1"
```

<img width="973" height="497" alt="image" src="https://github.com/user-attachments/assets/b102a06d-3174-430c-b4b7-9a0ddfbbe567" />

**Domain Join Steps:**
1. Joined domain via System Properties → Computer Name → Change
2. Entered domain credentials (HOMELAB\Administrator)
3. Restarted Windows 11 machine
4. Logged in with domain account

<img width="885" height="355" alt="image" src="https://github.com/user-attachments/assets/fd03877a-4f96-433b-8827-3453a70f1cd0" />

<img width="1045" height="293" alt="image" src="https://github.com/user-attachments/assets/66390d4a-6fdf-4918-88c6-11a775b79f52" />


## Resources
- [Microsoft Entra Documentation](https://learn.microsoft.com/en-us/entra/)
- [Active Directory Domain Services Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [Azure VPN Gateway Documentation](https://learn.microsoft.com/en-us/azure/vpn-gateway/)
- [PowerShell AD Module Reference](https://learn.microsoft.com/en-us/powershell/module/activedirectory/)
- [P2S VPN](https://www.cloudspress.com/how-to-configure-point-to-site-vpn-to-an-azure-vnet/)
---

