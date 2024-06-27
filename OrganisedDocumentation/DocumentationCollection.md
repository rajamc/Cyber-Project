# 2023CyberProject - CSOC Cybersecurity Operations Centre for SME's
### Project overview
This project entails the set-up, configuration, and testing of a prototype CSOC infrsutructure model. There will be an initial prototype done using virtual machines, before moving on to physical hardware for the network. This will give room for understanding and pre implimentation testing. Once the netwok has been configured, there will be Red and Blue team activities to test the security.

### Project Requirements 
The network will be set up and configured to use the following devices:
* Windows Client Devices (Windows 10)
* Windows Domain Controller Server (Windows Server 22)
* Cisco Switch
* KaliPurple SIEM
* OPENsense Firewall (Virtual Prototype)
* Palo Alto Firewall (Physical Production Prototype)
* Ubuntu DMZ Web Server
 
Testing will then take place to ensure that the network is secure.

The Initial prototype will use OPNsense for the firewall, while the production prototype will use Palo Alto.

### 1.0 IP adresses
```
- KaliPurple ---------------------- 192.168.100.200
- Windows 22 DC ------------------- 192.168.100.100
- DHCP Clients -------------------- 192.168.100.50-70
- Internal Management (Windows) --- 192.169.100.11-17
- Kali Inside --------------------- (DHCP)
- Switch VLAN1 -------------------- 192.168.100.254
- Palo Fw E1/2 -------------------- 192.168.100.1
- Palo Fw E1/1 -------------------- 192.168.108.235
- Palo Fw E1/3 -------------------- 192.168.50.1
- DMZ Web Server ------------------ 192.168.50.100
- Kali Outside -------------------- 192.168.108.116 
```
