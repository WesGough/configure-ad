<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Setup Resources in Azure
  1. Create the Domain Controller VM (Windows Server 2022) named "DC-1". Take note of the Resource Group and Virtual Network (Vnet) that get created at this time

![1](https://github.com/WesGough/configure-ad/assets/150361198/0dfd2fb9-036d-441e-8c31-cdb058902abb)

  2. Set Domain Controller’s NIC Private IP address to be static. Networking -> Network Interface -> IP Configurations

![3](https://github.com/WesGough/configure-ad/assets/150361198/dfa1187d-9acc-413b-8c9b-18d72067808d)

  3. Create the Client VM (Windows 10) named “Client-1”. Use the same Resource Group and Vnet that was created in Step 1
  
![4](https://github.com/WesGough/configure-ad/assets/150361198/ddbac78e-a9cd-4825-a07e-1b00eaaa1b0d)

  
  4. Ensure that both VMs are in the same Vnet (you can check the topology with Network Watcher)

![image](https://github.com/WesGough/configure-ad/assets/150361198/7332c517-9d76-4cc8-9d7a-11c0a0d57489)

- Ensure Connectivity between the client and Domain Controller
  1. Login to Client-1 with Remote Desktop and ping DC-1’s private IP address with ping <ip address> 

![image](https://github.com/WesGough/configure-ad/assets/150361198/996f45b9-f10f-4777-91a1-e8f6d6617135)

  2. Login to the Domain Controller and enable ICMPv4 in on the local windows Firewall. Windows Firewall -> Inbound Rules -> Sort by Protocol
 
![image](https://github.com/WesGough/configure-ad/assets/150361198/78a86a4a-7eab-4107-a269-c2352fe74524)
  
  3. Check back at Client-1 to see the ping succeed

![image](https://github.com/WesGough/configure-ad/assets/150361198/e277670f-40f0-4f94-8944-2d0011f712d4)

- Install Active Directory
  1. Login to DC-1 and install Active Directory Domain Services -> Add roles and Features in Server Manager 

![image](https://github.com/WesGough/configure-ad/assets/150361198/d3986d7a-07a6-4c6e-9b3d-f42c2660df90)

  2. Promote as a DC: Setup a new forest as mydomain.com (can be anything, just remember what it is)
 
![image](https://github.com/WesGough/configure-ad/assets/150361198/349b9e11-6262-4c57-b7a6-a0e42008c56b)
  
![image](https://github.com/WesGough/configure-ad/assets/150361198/eb5d5fd9-26d3-4619-859a-58233dc1bd11)

  3. Restart and then log back into DC-1 as user: mydomain.com\labuser

![image](https://github.com/WesGough/configure-ad/assets/150361198/6a537dd7-d6d2-4bec-9d5c-a328d86a72bb)

- Create an Admin and Normal User Account in AD
  1. In Active Directory Users and Computers (ADUC), create an Organizational Unit (OU) called “_EMPLOYEES”

![image](https://github.com/WesGough/configure-ad/assets/150361198/c79b9f03-7c31-4e38-bf9c-1d27a07e1be9)

![image](https://github.com/WesGough/configure-ad/assets/150361198/b686ab2f-0295-4d0f-bcbf-3042998ff13f)

  2. Create a new OU named “_ADMINS”
 
![image](https://github.com/WesGough/configure-ad/assets/150361198/5843f0d5-7c72-43ef-a46e-90d56acbb185)
  
  3. Create a new employee named “Jane Doe” (same password) with the username of “jane_admin”

![image](https://github.com/WesGough/configure-ad/assets/150361198/dd8cf6ae-2252-4697-a517-380dcd0c3c36)
  
  4. Add jane_admin to the “Domain Admins” Security Group. Right-click jane doe -> Properties -> Member Of -> Add

![image](https://github.com/WesGough/configure-ad/assets/150361198/58f7ff18-88d0-49b5-b6c2-8d83909950ab)

![image](https://github.com/WesGough/configure-ad/assets/150361198/50ea1045-51b7-4c8e-8e93-047e7268d268)
  
  5. Log out/close the Remote Desktop connection to DC-1 and log back in as “mydomain.com\jane_admin”
  6. User jane_admin as your admin account from now on
- Join Client-1 to your domain (mydomain.com)
  1. From the Azure Portal, set Client-1’s DNS settings to the DC’s Private IP address

![image](https://github.com/WesGough/configure-ad/assets/150361198/1f4c45af-31d2-40f0-a739-8a58ad872341)

  2. From the Azure Portal, restart Client-1
  
![image](https://github.com/WesGough/configure-ad/assets/150361198/0ab8ff64-274f-4385-815d-930f47276ea4)
  
  3. Login to Client-1 (Remote Desktop) as the original local admin (labuser) and join it to the domain (computer will restart). Right click Start -> System -> Rename this PC -> Change
 
![image](https://github.com/WesGough/configure-ad/assets/150361198/1ddc4257-2d64-479f-b4c4-f68216e8a548)
  
  4. Login to the Domain Controller (Remote Desktop) and verify Client-1 shows up in Active Directory Users and Computers (ADUC) inside the “Computers” container on the root of the domain
 
 ![image](https://github.com/WesGough/configure-ad/assets/150361198/ce386d3d-623a-44d8-a9f9-bccae9201e3d)
 
  5. Create a new OU named “_CLIENTS” and drag Client-1 into there
- Setup Remote Desktop for non-administrative users on Client-1
  1. Log into Client-1 as mydomain.com\jane_admin and open system properties
  2. Click “Remote Desktop”
  3. Allow “domain users” access to remote desktop

![image](https://github.com/WesGough/configure-ad/assets/150361198/84224d6c-a776-40f4-9afa-ef664737ddad)

  4. You can now log into Client-1 as a normal, non-administrative user now
  5. Normally you’d want to do this with Group Policy that allows you to change MANY systems at once (maybe a future lab)
- Create a bunch of additional users and attempt to log into Client-1 with one of the users
  1. Login to DC-1 as jane_admin
  2. Open PowerShell_ise as an administrator
  3. Create a new File and paste the contents of the script into it https://github.com/WesGough/AD_PS/edit/main/README.md

![image](https://github.com/WesGough/configure-ad/assets/150361198/2c4d03bf-2842-4eeb-83a4-9294d8e49c7a)

  4. Run the script and observe the accounts being created
  5. When finished, open ADUC and observe the accounts in the appropriate OU
 
![image](https://github.com/WesGough/configure-ad/assets/150361198/620e1b39-204d-4c0e-adc7-f69c72b021e7)
  
  6. Attempt to log into Client-1 with one of the accounts (take note of the password in the script)

![image](https://github.com/WesGough/configure-ad/assets/150361198/45fdfbcb-4068-4d98-9b48-b0dad0c8bc97)








