# 🔐 Cybersecurity & Pentesting Lab Setup — Week 1

Setting up my own isolated virtual lab for cybersecurity practice, from scratch.

---

## 📌 About This Project

This is my Week 1 project for the Cybersecurity program at **Networkwalks (Batch B083)**. The goal was simple on paper but genuinely useful to build hands-on: set up a virtual lab where I can safely run security tools, scan networks, and test things out without touching my real machine or anyone else's network.

I used **Oracle VirtualBox** as the hypervisor and installed **Kali Linux** as the main security-testing VM, connected through a private, isolated virtual network so nothing inside the lab can reach or be reached from the outside internet unintentionally.

This is meant to be the foundation — future weeks will likely add more VMs (targets, other distros) into the same lab network.

## 🎯 What I Set Out to Do

- Install and configure VirtualBox
- Import and get Kali Linux running as a VM
- Build a private internal network (`LAB-NET`) so lab machines can talk to each other in isolation
- Configure and verify the Kali VM's network adapter settings
- Confirm network connectivity and DNS resolution actually work
- Take a clean snapshot as a recovery baseline before I start breaking things on purpose later
- Document everything properly — screenshots, steps, and the problems I ran into

## *?*  Why Build This

An isolated lab is basically a sandbox — it means I can practice reconnaissance, scanning, and other security techniques without any risk of it affecting a real network. Everything here stays contained.

>  **Note:** This lab is strictly for learning and for testing systems I own or have explicit permission to test. Not to be used against anything else.

## ⚙️ Lab Configuration

| Component | Details |
|---|---|
| Hypervisor | Oracle VirtualBox 7.2.16 |
| Security OS | Kali Linux 2026.2 |
| Ram Allocated | 2056 MB |
| CPU Cores | 3 Cores |
| Network Type | Internal Network |
| Network Name | `LAB-NET` |
| Adapter | Adapter 2 (Intel PRO/1000 MT Desktop) |
| Snapshot Baseline | `KALI-CLEAN-LAB` |
| Kali IP Address | 10.0.2.15/24 |
| Internal Network | 10.0.0.3/24 |
| Network Address | 10.0.2.0/24 |


##  How I Set It Up
 
### 1. Installing VirtualBox
 
Started with VirtualBox as the hypervisor since it's free, well-documented, and widely used across most cybersecurity courses, so anything I learn here carries over. Installed it on my host machine and left resource allocation settings at default for now — no VMs running yet at this point.

------------------------------------------------------------------------------------------------------------------------------------------------------------
### 2. Importing Kali Linux
 
Downloaded the official Kali Linux VM image from kali.org rather than installing from ISO, since the pre-built image saves setup time and comes with the standard security toolset already installed. Imported it into VirtualBox and adjusted its settings:
 
- Allocated **2048 MB RAM**, balancing performance against my host machine's limits
- Left the virtual disk at default size since I'm not doing anything storage-heavy yet
- Kept CPU cores conservative so my host stays responsive while the VM runs
-------------------------------------------------------------------------------------------------------------------------------------------------------------
### 3. Creating the Internal Network
 
This was the part I spent the most time getting right. Instead of using a regular NAT adapter, I created a dedicated **Internal Network** named `LAB-NET`:
 
- Went into the VM's **Network → Adapter 2** settings
- Set **Attached to: Internal Network**, named it `LAB-NET`
- Adapter type set to **Intel PRO/1000 MT Desktop**
- **Promiscuous Mode: Deny** — no need for packet sniffing across other VMs at this stage
- Enabled **Virtual Cable Connected** so the adapter is actually live
I picked Internal Network specifically because it keeps the lab fully isolated from my host and the outside internet — only VMs attached to `LAB-NET` can see each other. That isolation matters more here than outbound internet access would.
-------------------------------------------------------------------------------------------------------------------------------------------------------------- 
### 4. Verifying Network Settings on Kali
 
Booted into Kali and checked that the adapter was recognized correctly at the OS level, not just in the VirtualBox settings panel. Made sure the interface came up and was actually attached to `LAB-NET` rather than silently falling back to NAT. 
┌──(adox㉿root)-[~]
└─$ ip -br a
lo               UNKNOWN        127.0.0.1/8 ::1/128 
eth0             UP             10.0.2.15/24 fe80::ae00:9584:306c:b5c5/64 
eth1             UP             10.0.0.3/24 

┌──(adox㉿root)-[~]
└─$ ip route
default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100 
10.0.0.0/24 dev eth1 proto kernel scope link src 10.0.0.3 metric 101 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100

-------------------------------------------------------------------------------------------------------------------------------------------------------------- 
### 5. Testing Connectivity and DNS
Ran basic connectivity checks across both active interfaces (eth0 and eth1) to confirm the network layer was fully operational:

Internal Network (eth1 - 10.0.0.3/24): Verified local subnet reachability across 10.0.0.0/24 to ensure the host could properly route traffic within the isolated lab environment.

External NAT Network (eth0 - 10.0.2.15/24): Confirmed outbound routing via default gateway 10.0.2.2 and verified public DNS resolution to ensure seamless internet access for updates and external tooling.

 --------------------------------------------------------------------------------------------------------------------------------------------------------------
### 6. Taking a Clean Snapshot
 
Once the base setup was confirmed working, I created a `KALI-CLEAN-LAB` snapshot immediately. This gives me a known-good rollback point before I start running anything experimental or destructive in future weeks — standard practice before doing real security testing work.

----------
----------
# Lab Verification
|  Test |  Command |  Expected Result |
| --- | --- | --- |
| **🌐 Check IP address** | `ip -br a` | `eth0: 10.0.2.15/24`<br><br>`eth1: 10.0.0.3/24` |
| **📡 Test gateway** | `ping -c 2 10.0.2.2` | Successful replies (0% packet loss) |
| **🌍 Test Internet connectivity** | `ping -c 2 1.1.1.1` | Successful replies (0% packet loss) |
| **🔎 Test DNS resolution** | `nslookup kali.org` | Domain resolves (`104.18.4.159`, `104.18.5.159`) |
| **🧰 Verify Nmap** | `nmap --version` | Nmap version 7.99 displayed |

---

### Example Results

**IP Addresses:**

* `eth0`: `10.0.2.15/24`
* `eth1`: `10.0.0.3/24`

**Gateway:**

* `10.0.2.2` (via `eth0`)

**DNS Server:**

* `10.157.197.170` (Resolved `kali.org` to `104.18.4.159` / `104.18.5.159`)

**Nmap Version:**

* `7.99`
##  Problems I Ran Into

Documenting this because I think the troubleshooting is honestly more useful than the steps that just worked first try.

- **Ran into a few hiccups getting the network adapter to behave correctly** on the first pass — had to double check the adapter type and internal network naming matched on both ends before it worked cleanly.
- **Had to make sure the VM's resource allocation was reasonable for my host machine** — didn't want to over-allocate and slow down my system while the VM was running.

*(Adding more here as I run into new issues in future weeks.)*

##  What I Actually Learned

- The difference between how VirtualBox handles **NAT**, **NAT Network**, and **Internal Network** modes — and why isolation matters for a security lab specifically
- How virtual network adapters connect VMs to each other (or don't)
- Why taking a clean snapshot *before* you start experimenting is non-negotiable
- That documenting your setup properly — including what went wrong — is part of doing the work professionally, not an afterthought

## 🔗 Tools Used

- [VirtualBox](https://www.virtualbox.org/)
- [Kali Linux](https://www.kali.org/get-kali/)

## 👤 About Me

Cybersecurity student, Networkwalks Batch B083.
Week 1 of the Cybersecurity & Pentesting program, under the guidance of Waqas Karim (CCIE) and the Networkwalks team.

---

📌 **Project:** Cybersecurity & Pentesting Lab Setup — Week 1 · **Program:** Networkwalks Cybersecurity Batch B083
