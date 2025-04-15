# 🌐 On-Premises Active Directory in the Cloud (Azure)

<p align="center">
  <img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo" width="400"/>
</p>

This walkthrough outlines how to implement on-premises-style Active Directory within Microsoft Azure using cloud-hosted virtual machines.

---

## 🧰 Environments and Technologies Used

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

## 🖥️ Operating Systems Used

- Windows Server 2022
- Windows 10 (21H2)

---

## ⚙️ Deployment & Configuration Steps

### 🔹 Overview
<img src="https://github.com/user-attachments/assets/408f1246-4054-4ae8-b12f-eef80d4beb42" width="600"/>

- Create a Server 2022 VM to serve as the domain controller (DC-1) and configure it with the domain `mydomain.com`.
- Set up a second VM, Client-1, running Windows 11 to join the domain and authenticate using AD-created users.

---

## 🏗️ Create the Domain Controller (DC-1)

### VM Setup
- Create a resource group and a virtual network (VNet).
- Launch a VM with:
  - **OS**: Windows Server 2022 Datacenter (Azure Edition)
  - **Network**: Active-Directory VNet
  - **Region**: Same as VNet
  - Set an admin account and credentials.

<img src="https://github.com/user-attachments/assets/d93456ea-bf7e-427e-94b1-b7d5ab97f700" width="600"/>

### Set Static IP for DC-1
- To ensure reliability, set the NIC’s private IP to **static**.

<img src="https://github.com/user-attachments/assets/b8e96c08-5ffe-4a4a-95e1-a4531071ef91" width="600"/>

### Disable Windows Firewall on DC-1
- RDP into DC-1 and run `wf.msc`.
- Disable Domain, Private, and Public firewalls.

<img src="https://github.com/user-attachments/assets/2f926f97-3e9b-4cf7-a32c-22028b30b028" width="600"/>

---

## 🖥️ Create the Client VM (Client-1)

- Use **Windows 10 Pro** with 2 vCPUs.
- Assign to same resource group and virtual network.
- After creation, set **DNS to DC-1’s private IP**.

<img src="https://github.com/user-attachments/assets/9bb926d3-40f2-44c4-b28f-c96c4dd62b67" width="600"/>

### Test Connectivity
- Restart the VM, RDP into Client-1.
- Ping DC-1’s IP and verify DNS with `ipconfig /all`.

<img src="https://github.com/user-attachments/assets/0d87934c-ecbf-4062-8e22-dd29b5c00add" width="600"/>
<img src="https://github.com/user-attachments/assets/b04a7381-7056-461d-96b4-1ed1543810f6" width="600"/>

---

## 📡 Deploying Active Directory on DC-1

### Install AD DS
- Open Server Manager → Manage → Add Roles and Features → Select **Active Directory Domain Services** → Install

<img src="https://github.com/user-attachments/assets/d7c9cfe3-0875-48ef-875f-8b116e31f1bd" width="600"/>

### Promote Server to Domain Controller
- After install, click "Promote this server to a domain controller".
- Select “Add new forest” → Use domain name: `mydomain.com` → Install

---

## 👤 Create Admin User & OUs

- In **ADUC**, create the following OUs:
  - `_EMPLOYEES`
  - `_ADMINS`

- Add user **Jane Doe** (`jane_admin`) to **Domain Admins** group.
- Log out and test login with: `mydomain\jane_admin`

<img src="https://github.com/user-attachments/assets/84a4e05e-d937-473e-a8fe-2ec84b4bf7a4" width="600"/>
<img src="https://github.com/user-attachments/assets/f60f5921-823c-42d3-9d8b-6dc8923fcfe4" width="600"/>
<img src="https://github.com/user-attachments/assets/17f30fe2-26d6-4b06-bb2f-57b8dc6b6854" width="600"/>

---

## 🖇️ Join Client-1 to Domain

- Go to **Settings → About → Advanced Settings** on Client-1.
- Under Domain, enter: `mydomain.com` and credentials.
- In ADUC, create `_CLIENTS` OU and move Client-1 into it.

<img src="https://github.com/user-attachments/assets/1d913fa5-ef26-48a9-bbfd-2370d4e6b134" width="600"/>
<img src="https://github.com/user-attachments/assets/6572d67c-f41d-4e61-a082-290d778832f5" width="600"/>

---

## 🖥️ Enable Remote Desktop Access for Domain Users

- Log into Client-1 as `jane_admin`
- Go to System Properties → Remote Desktop → Allow "Domain Users"

<img src="https://github.com/user-attachments/assets/9abbcb67-29c5-43d2-ad48-52c23216c072" width="600"/>

---

## 🧪 Create AD Users at Scale with PowerShell

- Log into DC-1 as `jane_admin`
- Open **PowerShell ISE as Administrator**
- Paste this [script](https://github.com/PouyaDini10/configure-ad/blob/main/CreateADusernames) to auto-generate users

<img src="https://github.com/user-attachments/assets/e101c903-73ee-4286-9bef-03e0f76d7463" width="600"/>

- Run the script → Creates 1000+ AD users with passwords in `_EMPLOYEES` OU
- Validate in ADUC

<img src="https://github.com/user-attachments/assets/192ca7cb-b456-449a-a8ed-b6337dd43486" width="600"/>
<img src="https://github.com/user-attachments/assets/7ce2eef6-3672-43b8-848e-eebcc5d21fe7" width="600"/>
<img src="https://github.com/user-attachments/assets/7c2bc838-d907-4c31-88e8-e9ed947fa65d" width="600"/>

---

## ✅ Summary
- We built an entire Active Directory environment from scratch — in the cloud.
- Implemented domain controllers, DNS, user/group provisioning, and domain-joined a client.
- Demonstrated remote login and automation via PowerShell scripting.
