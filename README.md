# Deploying Active Directory

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

(Step 1 overview) 

- setting up Active Directory infrastructure. I will configure and interconnect two virtual machines, each assuming distinct roles. The first virtual machine will be designated as the Domain Controller. The second virtual machine will be configured as the Client.

![image](https://github.com/user-attachments/assets/e9d78826-d6ae-4a80-98f2-19b1f67bfe15)

<h2></h2>

1. Create two VMs (Azure)in the same VNET. One will be a Domain Controller, the other will be a Client machine.

- Create a virtual machine (Domain Controller)on Azure.
- Name it DC-01
- Select Windows Server 2022: Azure Edition - x64 Gen2 as the image
- size    (Standard_D2s_v3 - 2 vcpus, 8 GiB memory)
- Create a username and password for your VM DC-1
![image](https://github.com/user-attachments/assets/2341bf39-8667-420b-953f-caeb221df308)


2. Create a virtual machine this will be our client machine

- Once again create a new VM and
- name it Client-01.
- select Windows 10 pro as the image
- select at least 2 vcpus and 16 GiB memory.
- Create a username and password for your VM client-1
- Make sure to select the same resource group and Virtual networkfrom the DC-01 VM

![image](https://github.com/user-attachments/assets/d592b6eb-f3e6-4410-88da-555c82419bbb)

3. Set the Domain Controller's Private IP to static

*We will change the DC to a static IP because its offering Active Directory services to the client machine (we do not want the private IP address to change so we will put it as static). Client machine will be joined to the domain. we're gonna tell client one to use DC one as the DNS server. And we need to manually configure the DNS settings for client one to use this private IP address. And if this private IP address changes this DNS server, will  no longer be valid.

- Once the VM has been deployed, proceed to the VM overview page and select "Networking" -> network setting  on the left side.

![image](https://github.com/user-attachments/assets/678e0d6a-b2bd-46a8-94ad-bb972353e47f)



- Select Network Interface Card -> IP configurations -> ipconfig1 and set Private IP address allocation to static.


![image](https://github.com/user-attachments/assets/3b93fcde-31d0-444f-81d1-54103b6b5a5e)


4.Login to DC-01 using remote desktop (for mac user use windows app)

- Get the public IP address from the VM DC-1 
- Enter the password and username you created for your VM 
- Login

![image](https://github.com/user-attachments/assets/8240c4c0-2281-4d35-86ea-f631e2ce9180)


5. Once logged in to DC_1 i will enable inbound ICMP traffic to allow for Client-1's ping

- open windows defender firewall and select advanced settings.
- Sort by protocol and find both ICMPv4 echo requests and enable both these rules by right clicking and selecting enable rule.


   ![image](https://github.com/user-attachments/assets/723d74a4-63af-480b-8fe6-49df3656c820)




6. Client one will connect to DC-1 to ensure connectivity

*To ensure connectivity between the two VM's, we will ping the domain controller from the client. At first the ping will not work correctly. We have to enable ICMPv4 on the firewall on DC-1. Now we can ping DC-1 successfully from Client-1

- Login to VM client-1 using remote desktop (for mac user use windows app)

- Find DC-1's private IP address in the Azure Portal and copy it. Proceed to Client-1 and open the terminal and type "ping (DC-01 private ip address)"

- Now we can ping DC-1 successfully from Client-1  

![image](https://github.com/user-attachments/assets/8cd1563d-0faa-40ce-955f-6f953739a0a5)


<h2></h2>


(Step 2 overview) 

Configure and install Active Directory services on the designated Domain Controller virtual machine.


1. Install Active Directory in DC-01


- In the Server Manager dashboard
- click Add roles and features
- continue the setup
- Select Active Directory Domain Services and finish the installation
![image](https://github.com/user-attachments/assets/42d65a5e-05d3-441f-91c9-78177aa4dbe2)

2. Promote DC-01 to Domain Controller

- click on the flag
- promote DC-1 to Domain Controller
- Setup a new forest as mydomain.com
- Restart and then log back into DC-1 as user: mydomain.com\labuser
![image](https://github.com/user-attachments/assets/5758ca96-5749-4c68-b22e-f06613ff3c68)

-  Run Active Directory Users & Computers as shown below.

![image](https://github.com/user-attachments/assets/dbe8e3bd-8036-4c30-ba29-3ae9954ebef0)


3. create an Organizational Unit(OU) named _EMPLOYEES and _ADMINS

- Right click on mydomain.com and select new and click on Organizational Unit

![image](https://github.com/user-attachments/assets/e99b4669-08dc-4396-a2ac-67eadfab1757)

- create OU _EMPLOYEES AND _ADMINS

![image](https://github.com/user-attachments/assets/1550f54f-b186-48d0-8b95-222997eb2a9f)

4. Create a Domain Admin user within the domain

-Right click on the OU _ADMINS and create a new -> user named Jane Doe with the username jane_admin

![image](https://github.com/user-attachments/assets/abf82555-2d3b-43df-bb94-2bb0d375942f)

5. Add jane_admin to the “Domain Admins” Security Group

- turn Jane Doe into an admin by right clicking her name-> properties-> member of and adding her to the “Domain Admins” Security Group

![image](https://github.com/user-attachments/assets/16c77d30-25ac-4905-8c36-a6b04b9d41e2)

6. Log out / close the connection to DC-1 and log back in as "mydomain.com jane_admin"

- Use jane _admin as your admin account from now on

![image](https://github.com/user-attachments/assets/0c062ab7-29b0-4a14-a4d3-35ddaea00639)




7. Set Client-1’s DNS settings to DC-1’s Private IP address


*In order for the client-1 to join the domain “mydomain.com”. We need to tell the client-1 to use our domain controller as the DNS server

- In the Azure Portal, select Client-01 -> Networking setting -> Network interface and click on DNS servers

- Select a custom DNS server and type in the private ip address of DC-01 and save then restart Client-01

![image](https://github.com/user-attachments/assets/8258c825-8030-4c55-85db-9228cceb7ed3)

- From Client-1, open PowerShell and run ipconfig /all
- The output for the DNS settings should show DC-1's private IP Address I

![image](https://github.com/user-attachments/assets/6a262bb6-d5be-4328-9e50-4d5540ecaa88)

8. Login to Client-1 as the original local admin and join it to the domain (computer will restart)

- right click the start menu -> click system -> click rename this pc advanced

- under the computer name tab click on change

- join it to the domain "mydomain.com

![image](https://github.com/user-attachments/assets/eb9dced2-f7ed-4e7e-a018-bd3ca8f7c184)


- Enter the name and password of an account with permission to join the domain.

- use the account mydomain.com\jane_admin

- Once Client-01 has been added, the VM will restart.

![image](https://github.com/user-attachments/assets/23a90fa4-f86d-4725-93ce-e86a869530ea)


9. Login to the Domain Controller and verify Client-1 shows up in ADUC

- Expand mydomain.com then go to computers and verify 

![image](https://github.com/user-attachments/assets/5e8526f3-2184-4cb2-abb8-d58384430be7)


10. Create a new OU named “_CLIENTS” and drag Client-1 into there


![image](https://github.com/user-attachments/assets/a2b66ee6-7ad6-4ffb-b911-2e9b7f5f76bd)


<h2></h2>


(Step 3 overview) 

Setting up Remote Desktop for non-administrative users on Client-1 and creating user accounts using a powershell script to populate our domain 

1. Setup Remote Desktop for non-administrative users on client - 1

- We have to log into Client-1 as an admin (using mydomain.com\jane_admin) right click the start menu and open system.

- Click on "Remote Desktop" on User Accounts and click “Select users that can remotely access this PC”

- allow "domain users" access to remote desktop.

-  After completing those steps you should be able to log into Client-1 as a normal user.

![image](https://github.com/user-attachments/assets/6fecd11d-7d7f-4724-a05e-075883efbe60)

2.  Run PowerShell script


-  use a Powershell script to generate a number of users for our Active Directory Domain.

- Login to DC-01 as mydomain.com\jane_admin

- Open PowerShell_ise as an administrator and create a new file then save 


- [https://github.com/rafaelgarridoIT1/Create-New-Users/blob/main/README.md]



- paste the script to generate thousands of users into the domain.

- run script

![image](https://github.com/user-attachments/assets/2d58ce4a-9b87-4f79-b26d-e3a7de058c30)


- When finished, open ADUC and observe the accounts in the appropriate OU　(_EMPLOYEES)

![image](https://github.com/user-attachments/assets/37ac0585-6020-4915-9931-3252b20ede7a)





3.  Login as a user

- Now you can try logging in as one of the users to further verify that the script has worked.

 - Take note of the default password in the script (Password1) for all users 

![image](https://github.com/user-attachments/assets/f7ed89f4-794c-437e-9733-ab5d1a46ac1c)


- As you can see the Powershell script created a user with the username "bid.tuk" We were able to login to Client-1 with his credentials as a normal user.


![image](https://github.com/user-attachments/assets/e1f609d6-a6dc-42ba-ad54-22d6d2d87006)












