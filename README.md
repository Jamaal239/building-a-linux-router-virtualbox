# 🌐 Building a Linux Router in VirtualBox

## 📝 Project Overview
As a beginner in networking, I wanted to understand how data moves between different, isolated networks. In this home lab, I built a fully functional virtual network from scratch using **Oracle VirtualBox**. 

Normally, two different network segments (subnets) cannot talk to each other directly. To bridge this gap, I configured an **Ubuntu Linux Server** to act as a custom router, enabling a Windows Server to communicate with a completely separate network segment.

---

## 🏗️ The Network Design (Topology)
* **Subnet A:** 192.168.10.0/24 (The target network)
* **Subnet B:** 192.168.20.0/24 (The host network where my servers live)
* **The Gateway (Router):** An Ubuntu Linux machine sitting with "one foot" in both networks.
  * Adapter 1 (Subnet A Interface): 192.168.10.1
  * Adapter 2 (Subnet B Interface): 192.168.20.1

---

## 🛠️ Step-by-Step Implementation & Documentation

### Phase 1: Creating the Virtual Cables (Hardware Layer)
I started by isolating the virtual machines using VirtualBox's **Internal Network** feature. Think of this like plugging physical computers into an isolated network switch using ethernet cables. 

I attached the second adapter of the Ubuntu Router and the Windows clients to a matching internal network named **`Subnet-B-Client`**.

<img width="781" height="518" alt="screenshot_01" src="https://github.com/user-attachments/assets/331497d2-6005-456a-8ada-742f2da18b6f" />
> Figure 1: VirtualBox Network Adapter Settings
> > This screenshot confirms the hardware layer isolation. The network interface card is explicitly mapped to an Internal Network named `Subnet-B-Client`, ensuring it is securely grouped onto the correct virtual switch segment.

---

### Phase 2: Identifying Router Interfaces
Next, I booted into the Ubuntu server to identify the system names of the network interface cards (NICs) assigned by VirtualBox so I could prepare them for configuration.

<img width="918" height="885" alt="screenshot_02" src="https://github.com/user-attachments/assets/c0c7383e-4602-4330-b7cd-fbe739505bbf" />
> Figure 2: Verifying Available Network Interfaces via Linux CLI >
> Running the `ip link show` command reveals the active network adapters on the Linux kernel. This step was crucial to identify `enp0s3` (connected to Subnet A) and `enp0s8` (connected to Subnet B) before writing the IP configuration files.

---

### Phase 3: Giving the Router a Brain (IP Forwarding & Netplan)
By default, standard operating systems are "selfish"—if they receive a packet meant for a different network, they drop it. To turn the Ubuntu server into an active router:
1. I used **Netplan** to assign static IP addresses to both of the router's network adapters.
2. I modified the Linux configuration file at `/etc/sysctl.conf` and enabled `net.ipv4.ip_forward=1` to allow cross-interface traffic.

<img width="917" height="885" alt="screenshot_03" src="https://github.com/user-attachments/assets/de601701-4afe-4e73-ba91-f1c0235020ca" />
> Figure 3: Verification of Static IP Assignments on the Linux Router
>  > Caption: Using the `ip a` command to verify that our Netplan configurations applied successfully. The output proves that interface `enp0s3` is bound to `192.168.10.1/24` and interface `enp0s8` is bound to `192.168.20.1/24`, allowing the server to sit simultaneously on both subnets.

---

## 🧠 Real-World Troubleshooting & Pivot
During the lab setup, my original Windows 11 Client VM experienced an OS-level administrator account lockout. 

Instead of starting the entire lab over from scratch, I used my engineering and troubleshooting skills to pivot. I reconfigured the VirtualBox network adapters on my active **Windows Domain Controller (DC1)**, moved it onto the `Subnet-B-Client` virtual switch, assigned it the correct static IP properties (`192.168.20.2`), and set its **Default Gateway** to the Linux router (`192.168.20.1`). This saved time and successfully validated the environment.

---

## ⚡ Verification & Final Results

To prove that the Linux router was successfully forwarding packets across subnets, I ran an ICMP connectivity test from the Windows Server command prompt to the **far-side** interface of the router. 

<img width="924" height="1032" alt="screenshot_04" src="https://github.com/user-attachments/assets/93217ef8-fe88-4f43-99bf-cda257b1688a" />
> Figure 4: Successful Cross-Subnet Ping Test from Windows Domain Controller
> The ultimate proof of concept. The first test verifies local connectivity to the gateway (`192.168.20.1`). The second test successfully pings the far-side interface (`192.168.10.1`). Because the packets successfully traversed the Linux kernel from one network segment to another with 0% packet loss, it confirms the Linux router is fully operational.

---

## 🎓 Key Takeaways
* Learned how to multi-home a server using multiple network interface cards (NICs).
* Gained hands-on experience modifying Linux system configuration files via the CLI.
* Understood the vital role a **Default Gateway** plays in letting local devices find external networks.
* Practiced critical thinking and adaptability when virtual lab environments experience technical hurdles.
