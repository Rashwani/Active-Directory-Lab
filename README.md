# Building an Active Directory Lab on AWS

In this project I built a fully functional Active Directory environment inside Amazon Web Services. I set up two EC2 instances, a Domain Controller and a domain-joined client, connected through a private VPC to simulate a real corporate network.

---

## What I Built

| Component | Description |
|-----------|-------------|
| Machine 1 | Windows Server 2022 - Domain Controller running Active Directory |
| Machine 2 | Windows Server 2022 - Client machine joined to the domain |
| Network | Private VPC with subnet, Internet Gateway, and Route Table |
| Domain | lab.local |

<img width="1408" height="768" alt="Gemini_Generated_Image_n2bqjpn2bqjpn2bq" src="https://github.com/user-attachments/assets/8cabc0ff-ba22-4964-b783-3d48947a2559" />


---

## What I Did

### Step 1 - Created the Network (VPC)

I started by building the network infrastructure before launching any machines. I created a VPC named `AD-Lab-VPC` with a CIDR block of `10.0.0.0/16`, then created a subnet called `AD-Lab-Subnet` using `10.0.1.0/24` and enabled auto-assign public IPv4 so the instances would be reachable remotely. I then created an Internet Gateway named `AD-Lab-IGW` and attached it to the VPC, and set up a Route Table that directed all outbound traffic through the gateway and associated it with the subnet.

<img width="1776" height="769" alt="Screenshot 2026-03-08 160151" src="https://github.com/user-attachments/assets/4e04f6c1-5e13-4547-9af0-5bc3847419ec" />
<img width="1309" height="373" alt="Screenshot 2026-03-08 160409" src="https://github.com/user-attachments/assets/0480be30-78a1-4a7c-969c-04a021bdb1a1" />
<img width="1310" height="646" alt="Screenshot 2026-03-08 160421" src="https://github.com/user-attachments/assets/abb88206-6402-406c-a257-acdb7aa2b275" />
<img width="1075" height="198" alt="Screenshot 2026-03-08 160638" src="https://github.com/user-attachments/assets/88142dc1-02ab-43e6-834b-628e91fd3edc" />
<img width="1901" height="428" alt="Screenshot 2026-03-08 161040" src="https://github.com/user-attachments/assets/c63dbdf4-de2d-4d78-b3ca-0d4b05a1cc8d" />
<img width="1907" height="490" alt="Screenshot 2026-03-08 161231" src="https://github.com/user-attachments/assets/74dc539b-3501-4662-8545-912274f19438" />


---

### Step 2 - Launched the Domain Controller

I launched a Windows Server 2022 EC2 instance using the `t3.medium` instance type, which provides the 2 vCPUs and 4GB RAM that Active Directory requires. I created a key pair named `AD-Lab-Key`, placed the instance inside `AD-Lab-VPC`, and created a security group named `AD-Lab-SG` with an inbound RDP rule on port 3389 restricted to my IP address only.

---

### Step 3 - Connected via RDP

I retrieved the Administrator password by uploading the `.pem` key file through the EC2 console, then downloaded the RDP file and connected to the server remotely for the first time.
<img width="1842" height="745" alt="Screenshot 2026-03-08 162432" src="https://github.com/user-attachments/assets/204e026e-9190-402b-9f1c-8c9c159ac714" />

---

### Step 4 - Installed Active Directory

Inside the Windows Server RDP session I opened Server Manager, added the Active Directory Domain Services role, and promoted the server to a Domain Controller. I created a new forest with the root domain name `lab.local`, set a DSRM password, and let the wizard complete. The server rebooted automatically and I reconnected as `LAB\Administrator`.
<img width="682" height="347" alt="Screenshot 2026-03-08 164538" src="https://github.com/user-attachments/assets/804f90ed-b539-4b4f-bbf9-2489df77b17d" />
<img width="978" height="697" alt="Screenshot 2026-03-08 164717" src="https://github.com/user-attachments/assets/6a426cae-ef47-42bc-8ebc-0391a9bfc72e" />
<img width="982" height="700" alt="Screenshot 2026-03-08 164742" src="https://github.com/user-attachments/assets/555b533f-817a-4b73-9244-b72eddda9224" />
<img width="980" height="699" alt="Screenshot 2026-03-08 164836" src="https://github.com/user-attachments/assets/b0081971-86b3-454d-8766-e732fe4aec95" />
<img width="516" height="540" alt="Screenshot 2026-03-08 164909" src="https://github.com/user-attachments/assets/e5393fa1-387a-490b-907b-87360a2ba63b" />
<img width="951" height="696" alt="Screenshot 2026-03-08 165522" src="https://github.com/user-attachments/assets/79f53977-285d-4401-8da5-c571a3779a89" />
<img width="944" height="694" alt="Screenshot 2026-03-08 165800" src="https://github.com/user-attachments/assets/5150b19e-1080-416a-9b50-57da968e8347" />

---

### Step 5 - Set a Static Private IP on the Domain Controller

I noted the private IPv4 address assigned to the DC in the AWS console, then went into the network adapter settings inside Windows and manually set that IP as a static address. I also set the preferred DNS server to `127.0.0.1` so the Domain Controller points to itself, which is required for Active Directory to function correctly.
<img width="461" height="548" alt="Screenshot 2026-03-08 171101" src="https://github.com/user-attachments/assets/14d1a912-5011-46e3-961b-11d4584aae2a" />

---

### Step 6 - Launched the Client Machine

I launched a second Windows Server 2022 instance named `AD-Client` using the same key pair and security group as the Domain Controller so both machines shared the same access rules.

---

### Step 7 - Configured DNS on the Client

I RDP'd into the client machine and updated the TCP/IPv4 settings to point the preferred DNS server at the Domain Controller's private IP address. This is what allows the client to locate the domain and join it.
<img width="458" height="544" alt="Screenshot 2026-03-09 001723" src="https://github.com/user-attachments/assets/1bfc97d1-9aa0-45af-96dd-a53a67911241" />

---

### Step 8 - Joined the Client to the Domain

I opened System Properties on the client, changed the membership from Workgroup to Domain, typed `lab.local`, and entered the Domain Controller's Administrator credentials. I got the "Welcome to the lab.local domain" confirmation, restarted the machine, and logged back in as `LAB\Administrator` using the DC password.
<img width="439" height="515" alt="Screenshot 2026-03-09 002252" src="https://github.com/user-attachments/assets/2a2fd214-7a23-4d2e-97a4-83db2e454237" />
<img width="573" height="373" alt="Screenshot 2026-03-09 005326" src="https://github.com/user-attachments/assets/3a4b0814-5447-491a-b8aa-c2227414baa0" />
<img width="1536" height="1033" alt="Screenshot 2026-03-09 005813" src="https://github.com/user-attachments/assets/09b8c0ea-bd15-4dc7-9ac4-8ec470cd4669" />

---

Step 9 — Creating Organizational Units and User Accounts
I opened Active Directory Users and Computers (dsa.msc) on the Domain Controller and created two Organizational Units under lab.local: Lab Users and Lab Computers. OUs act like folders to keep accounts organized instead of dumping everything into the default Users container.
<img width="939" height="651" alt="Screenshot 2026-03-19 012934" src="https://github.com/user-attachments/assets/6228677d-efdf-41b4-88ae-036b962b9d4b" />

I then created a user account for Jane Smith (jsmith) inside the Lab Users OU through the GUI, setting the password to ******** and disabling the password change requirement for lab purposes.
<img width="540" height="462" alt="Screenshot 2026-03-19 013154" src="https://github.com/user-attachments/assets/8f33c885-68a9-4ac0-9503-bf8a19f3a7be" />

To create additional users faster, I used a PowerShell script that creates accounts directly into the Lab Users OU:

$OU = "OU=Lab Users,DC=lab,DC=local"

$Password = ConvertTo-SecureString "*******" -AsPlainText -Force

New-ADUser -Name "Alice Brown" -GivenName "Alice" -Surname "Brown" `

  -SamAccountName "abrown" -UserPrincipalName "abrown@lab.local" `

  -Path $OU -AccountPassword $Password -Enabled $true

New-ADUser -Name "Tom Green" -GivenName "Tom" -Surname "Green" `

  -SamAccountName "tgreen" -UserPrincipalName "tgreen@lab.local" `

  -Path $OU -AccountPassword $Password -Enabled $true
<img width="1892" height="950" alt="Screenshot 2026-03-19 014637" src="https://github.com/user-attachments/assets/2887649b-c798-41d0-830b-6a5c12804321" />


Step 10 — Creating Security Groups and Assigning Members
I created two Global Security Groups inside the Lab Users OU: IT-Admins and Staff-Users. I added jsmith to IT-Admins and abrown and tgreen to Staff-Users. In Active Directory, permissions are assigned to groups rather than individual users — this is the basis of Role-Based Access Control (RBAC).
<img width="536" height="491" alt="Screenshot 2026-03-19 014942" src="https://github.com/user-attachments/assets/a386059a-27c0-4a0c-a23d-96fe607005c6" />
<img width="529" height="490" alt="Screenshot 2026-03-19 015009" src="https://github.com/user-attachments/assets/29f86f2f-9291-42f4-ac34-a82cd9c67bc0" />


Step 11 — Configuring Shared Folder Permissions
On the Domain Controller I created a folder at C:\SharedDrive with two subfolders: IT-Only and AllStaff. I shared the parent folder and configured both Share Permissions and NTFS Permissions:
<img width="1265" height="409" alt="Screenshot 2026-03-19 151205" src="https://github.com/user-attachments/assets/8ea61ba5-f609-4239-a82f-e23f812f1e00" />

IT-Admins — Full Control (both Share and NTFS)
Staff-Users — Read only (both Share and NTFS)
<img width="451" height="553" alt="Screenshot 2026-03-19 151357" src="https://github.com/user-attachments/assets/f8c1336c-d16c-4429-bda4-e12aaad90923" />
<img width="477" height="549" alt="Screenshot 2026-03-19 151653" src="https://github.com/user-attachments/assets/ab0bcb6c-f417-473a-9624-03b03ecb031a" />
<img width="476" height="542" alt="Screenshot 2026-03-19 151743" src="https://github.com/user-attachments/assets/8b4c31b6-4b77-4031-9b21-41901c9d5d7a" />

I removed the default Everyone entry from the share permissions so access is controlled entirely through group membership.


Step 12 — Testing Permissions from the Client Machine
I granted RDP access to both IT-Admins and Staff-Users on the client machine through Remote Desktop settings, then tested:

Logged in as LAB\jsmith (IT-Admins member) and navigated to \\<DC-PRIVATE-IP>\SharedDrive. I was able to browse and create files — Full Control worked as expected.
<img width="1126" height="664" alt="Screenshot 2026-03-19 160026" src="https://github.com/user-attachments/assets/59fbd40e-e61d-4d38-9b1e-495a6699edf9" />

Logged in as LAB\tgreen (Staff-Users member) and tried the same. I could browse the folder but creating a file was denied — Read only access confirmed.
<img width="1160" height="694" alt="Screenshot 2026-03-19 161030" src="https://github.com/user-attachments/assets/412b501f-9d66-4fee-952b-c842dee719f5" />


Step 13 — Configuring Windows Firewall Rules
I opened Windows Defender Firewall with Advanced Security (wf.msc) on the Domain Controller and verified that all the critical AD inbound rules were enabled (LDAP on TCP 389, Kerberos on TCP 88, DNS on TCP/UDP 53, RDP on TCP 3389).

I then created custom rules:

Block Telnet (Port 23) — Inbound block rule on TCP port 23 across all profiles
Allow HTTPS (Port 443) — Inbound allow rule on TCP port 443 across all profiles
Allow svchost (Domain) — Program-based inbound rule allowing svchost.exe on the Domain profile only


Step 14 — Configuring Firewall via Group Policy
I opened the Group Policy Management Console (gpmc.msc), created a new GPO called Firewall Policy linked to lab.local, and configured it to enforce the firewall state as On with inbound connections blocked and outbound connections allowed across all profiles. This pushes the same firewall baseline to every domain-joined machine automatically.
<img width="489" height="555" alt="Screenshot 2026-03-19 163616" src="https://github.com/user-attachments/assets/3a1d2368-7111-48d1-9151-2da690346c27" />
<img width="488" height="553" alt="Screenshot 2026-03-19 163632" src="https://github.com/user-attachments/assets/7c1b2f93-6c7e-4f40-b380-ec4989abcc84" />
<img width="483" height="556" alt="Screenshot 2026-03-19 163646" src="https://github.com/user-attachments/assets/a1608e38-918b-4dcb-a6e8-e929ca5c8722" />


Step 15 — Testing Firewall Rules from the Client
I ran Test-NetConnection from the client machine against the DC's private IP on three ports:

Port 23 (Telnet) — TcpTestSucceeded: False — Blocked by the custom firewall rule, working as intended.
Port 389 (LDAP) — TcpTestSucceeded: True — AD's LDAP service is running and reachable.
Port 443 (HTTPS) — TcpTestSucceeded: False — The firewall allows it, but no service is actually listening on that port. The rule is correct; there's just nothing behind it.
<img width="812" height="631" alt="Screenshot 2026-03-19 170043" src="https://github.com/user-attachments/assets/cd485032-8c87-451d-9093-34fa5bbacdc7" />
<img width="767" height="249" alt="Screenshot 2026-03-19 170137" src="https://github.com/user-attachments/assets/5cad5c01-e8ec-4582-8cde-a4afa7be3490" />


Troubleshooting
These are real issues I ran into while building this lab and how I fixed them.
RDP failed with error code 0x204
What happened: I could not connect to the client instance at all. Remote Desktop threw error 0x204 and refused to connect.

Why it happened: My public IP address had changed since I originally created the security group. My ISP changed it, so the inbound RDP rule on port 3389 was pointing to an outdated IP and blocking the connection entirely.

How I fixed it:

Went to EC2 → Security Groups → AD-Lab-SG
Clicked Edit inbound rules
Changed the source on the RDP rule to My IP
Saved and reconnected successfully

I now update the My IP rule at the start of every lab session before trying to connect.


Domain join failed with a DNS timeout
What happened: When I tried to join the client to lab.local, I got a DNS timeout error referencing _ldap._tcp.dc._msdcs.lab.local and the join failed completely.

Why it happened: The security group was blocking all traffic between the two EC2 instances. Even though the DNS IP on the client was correctly pointing to the Domain Controller, the machines could not actually talk to each other. Active Directory relies on a range of ports including DNS (53), LDAP (389), Kerberos (88), RPC (135), and SMB (445), and none of that traffic was being allowed through.

How I fixed it:

Went to EC2 → Security Groups → AD-Lab-SG
Added a new inbound rule set to All traffic with the source set to 10.0.0.0/16
Saved the rule and retried the domain join, which succeeded

Setting the source to the VPC range allowed both machines to communicate freely with each other while keeping the environment locked down from the internet.


PowerShell user creation ran without errors but users didn't appear
What happened: I ran the New-ADUser PowerShell script and got no errors, but the users didn't show up in the Lab Users OU in ADUC.

Why it happened: The script can fail silently if the AD module hasn't fully loaded or if domain services are still settling after a recent reboot. The OU existed and the syntax was correct, but the first execution just didn't take.

How I fixed it: I re-ran the same script a second time and the users were created successfully. To verify, I ran:

Get-ADUser -Filter * -SearchBase "OU=Lab Users,DC=lab,DC=local" | Select Name, SamAccountName


Could not log in as a domain user via RDP on the client machine
What happened: After creating domain users, I tried to RDP into the client machine as LAB\jsmith but it wouldn't let me connect.

Why it happened: By default, only the Administrator account has Remote Desktop access on domain machines. Regular domain users need to be explicitly granted permission.

How I fixed it:

RDP'd into the client as LAB\Administrator
Went to System → Remote Desktop → Select users that can remotely access this PC
Added the IT-Admins and Staff-Users groups
Signed out and reconnected as LAB\jsmith successfully


"Check Names" prompted for credentials when adding groups
What happened: When I tried to add IT-Admins to the Remote Desktop users list and clicked Check Names, Windows asked me for credentials instead of resolving the name.

Why it happened: Windows needed to verify the group name against Active Directory and the current session wasn't passing the domain credentials automatically for that lookup.

How I fixed it: I entered LAB\Administrator with the Domain Controller's Administrator password. This is the domain-wide admin account so it works on any domain-joined machine. After entering the credentials, the group name resolved and underlined correctly.


Firewall port tests failed for LDAP (port 389) from the client
What happened: I ran Test-NetConnection from the client to test port 389 on the DC and it returned TcpTestSucceeded: False, even though the LDAP firewall rule was enabled on the Domain Controller.

Why it happened: The AWS Security Group AD-Lab-SG only had an inbound rule for port 3389 (RDP). All other traffic between the two EC2 instances was being blocked at the AWS level before it ever reached Windows Firewall.

How I fixed it:

Went to EC2 → Security Groups → AD-Lab-SG
Edited inbound rules and added a new rule: Type set to All traffic, Source set to 10.0.0.0/16
Saved the rule and re-ran the test — port 389 returned True

This allows all internal VPC communication between the DC and client while keeping the environment locked down from the internet.


Port 443 test showed False even after creating an allow rule
What happened: I created a Windows Firewall rule to allow inbound HTTPS on port 443, but Test-NetConnection still showed TcpTestSucceeded: False.

Why it happened: A firewall allow rule only opens the port — it doesn't start a service. Since there was no web server or any application listening on port 443, there was nothing to accept the connection.

How I confirmed it: I ran the following on the DC and it returned nothing, confirming no service was bound to port 443:

Get-NetTCPConnection -LocalPort 443 -ErrorAction SilentlyContinue

The firewall rule itself was working correctly. The test failed because the port was open but empty.

