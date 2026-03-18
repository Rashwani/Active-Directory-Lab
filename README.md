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

## Troubleshooting

These are real issues I ran into while building this lab and how I fixed them.

### RDP failed with error code 0x204

**What happened:** I could not connect to the client instance at all. Remote Desktop threw error 0x204 and refused to connect.

**Why it happened:** My public IP address had changed since I originally created the security group. My ISP changed it, so the inbound RDP rule on port 3389 was pointing to an outdated IP and blocking the connection entirely.

**How I fixed it:**
1. Went to EC2 > Security Groups > AD-Lab-SG
2. Clicked Edit inbound rules
3. Changed the source on the RDP rule to My IP
4. Saved and reconnected successfully

I now update the My IP rule at the start of every lab session before trying to connect.

---

### Domain join failed with a DNS timeout

**What happened:** When I tried to join the client to `lab.local`, I got a DNS timeout error referencing `_ldap._tcp.dc._msdcs.lab.local` and the join failed completely.

**Why it happened:** The security group was blocking all traffic between the two EC2 instances. Even though the DNS IP on the client was correctly pointing to the Domain Controller, the machines could not actually talk to each other. Active Directory relies on a range of ports including DNS (53), LDAP (389), Kerberos (88), RPC (135), and SMB (445), and none of that traffic was being allowed through.

**How I fixed it:**
1. Went to EC2 > Security Groups > AD-Lab-SG
2. Added a new inbound rule set to All traffic with the source set to `10.0.0.0/16`
3. Saved the rule and retried the domain join, which succeeded

Setting the source to the VPC range allowed both machines to communicate freely with each other while keeping the environment locked down from the internet.
