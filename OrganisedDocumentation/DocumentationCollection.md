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

### Network Map
![NetworkMap](/OrganisedDocumentation/Images/topology.PNG)

## Windows 10 Client Installation:
Creating a Windows client to be latterly connected to the network, domain and SIEM.
## Creating Windows Client in Virtual Box:
* With Virtual Box manager open, click Machine > New...
* Attach Windows 10 iso (Windows 10 Evaluation 22h2 Enterprise)
* Install on to portable SSD
* Skip unattended install
#### Hardware:
* Set Base Memory to 4GB
* Processors set to 2
#### Virtual Hard Disk:
* Create Virtual Hard Disk - Size 50GB
* Press next and then finish.
#### Windows 10 Virtual Machine:
Run the newly installed Windows 10 virtual machine
* Install the standard desktop experience
* When prompted, select no internet connection
* Continue without a Microsoft account
#### Configure IPV4 ethernet adapter:
* IP Address - 192.168.100.20/24
* Gateway - 192.168.100.1
* DNS - 192.168.100.100 / 8.8.8.8
#### Virtual Box Networking:
Set the Windows 10 Virtual Box network adapter
* Internal (intnet) for OPNsense firewall<br>
OR
* Bridged for Palo Alto firewall
#### Check Connectivity:
Open command prompt
* ipconfig /all
* ping 192.168.100.1
* ping 192.168.108.1
* ping 8.8.8.8<br>
* In a web browser connect to 192.168.100.1 to test firewall gui / connectivity.<br>
## Windows Server 22 Installation and DC promotion
## Creating Windows Server in Virtual Box:
* With Virtual Box manager open, click Machine > New...
* Attach Windows Server 22 iso
* Install on to portable SSD
* Skip unattended install
#### Hardware:
* Set Base Memory to 4GB
* Processors set to 2
#### Virtual Hard Disk:
* Create Virtual Hard Disk - Size 50GB
* Press next and then finish.
* Note - Make sure installation is desktop experience and finish installation with no internet/microsoft account.
#### Configure IPV4 ethernet adapter:
* IP Address - 192.168.100.100/24
* Gateway - 192.168.100.1
* DNS - 127.0.0.1, 8.8.8.8
#### Promote server to Domain controller:
* Name the server using your initials - eg. jp
* hostname: initialsserver eg. jpserver
* domain: initials.local - eg. jp.local
* IMPORTANT - Change the hostname before promoting to a DC.<br>
* Added 4 users with usernames and passwords for each member of team
* Created a user group and added the members to that group
* Added Client devices to Domain
#### Set domain group policy 
* Open "Group Policy Manager" 
* Expand Forest:project.production > Domains > project.production > Group Policy Objects
* Right click and edit Default Domain Policy
* Expand Computer Configuration > Policies > Windows Settings > Security Settings > Account Policies > Password Policy
* Enforce password history: 24 passwords remembered
* Max password age: 90 days
* Min password age: 1 day
* Min password length: 10
* Meet complexity requirments Enabled
## OPNSENSE Firewall Installation and Configuration:<br>
## Opnsense Firewall in Virtual Box
* With Virtual Box manager open, click Machine > New...
* ISO Image: OPNsense.iso
* Type: BSD
* Version: FreeBSD (64-bit)
#### Hardware:
* Set Base Memory to 4GB
* Processors set to 2
#### Virtual Hard Disk:
* Create Virtual Hard Disk - Size 20GB
* Press next and then finish.
#### Network:
* Add 2 network adapters cards
* Network > Adapter 1 - Attached to Internal network = LAN
* Network > Adapter 2 - Enable , Attached to Bridged Adapter - Ethernet card on PC (or WIFI) = WAN
* Note: Adapter 1 will be allocated to em0 , Adapter 2 will be allocated to em1 - in OPNsense
#### Finalising Installation:
* Start OPNsense VM
* Login as: installer
* Password: opnsense
* Accept defaults
* Copy files to disk partition select VBOX HARDDISK 10 (20 GB) with down arrow , enter for OK
* Yes to destroy disk contents , left arrow and enter
* Files will be copied from the iso to the VDI file
* Enter root as user, password: opnsense, confirm password, complete installation and reboot.
#### Conifguring OPNsense Firewall:
#### Set up DHCP on LAN interface em0
* Login as root, password opnsense.
* Enter an option: 2 (set interface IP address)
* Enter the number of interface to configure: 1 - LAN (em0)
* Configure IPv4 address LAN interface via DHCP: N
* Enter the New LAN IPv4 Address: 192.168.100.1
* Enter the New LAN IPv4 subnet bit count (1 to 32): 24
* Configure IPv6...: y
* Do you want to enable the DHCP server on LAN: y
* Enter the start address: 192.168.100.40
* Enter the end address: 192.168.100.49
* Do you want to change the web GUI protocol from HTTPS to HTTP? : y
* Restore web GUI access defaults: y
  
## Switch Configureation

* Switch(config)#hostname SW1
* SW1(config)# enable secret class
* SW1(config)# banner motd $Unauthorised access is strictly prohibited.$
* SW1(config)#service password-encryption
* SW1(config)#username ProdAdmin secret Password1
* SW1(config)#no ip domain-lookup
* SW1(config)#ip domain-name cyber-project.net
* SW1(config)#crypto key generate rsa
* How many bits in the modulus [512]: 1024
* SW1(config)#line vty 0 4
* SW1(config-line)#transport input ssh
* SW1(config-line)#login local
* SW1(config)# ip default-gateway 192.168.100.1
* SW1(config)# interface vlan1
* SW1(config-if)# ip address 192.168.100.254 255.255.255.0
* SW1(config-if)# no shutdown
* SW1(config)#interface g1/0/3
* SW1(config-if)#switchport port-security
* SW1(config-if)#switchport port-security maximum 1
* SW1(config-if)#switchport port-security mac-address sticky
* SW1(config-if)#switchport port-security violation restrict
* SW1(config)#interface range g1/0/1, g1/0/2, g1/0/4 - 36, g1/0/1
* SW1(config-if-range)# shutdown
* SW1(config)#service timestamps log datetime
* SW1(config)#service sequence-numbers
* SW1(config)#logging 192.168.100.200
* SW1(config)#logging trap 4
* SW1(config)#logging facility auth
* SW1(config)#ntp server 139.180.160.82
* SW1(config)#clock timezone AEST +10
* SW1(config)#interface g1/0/3
* SW1(config-if)#switchport mode access
* SW1(config-if)#switchport port-security violation restrict
* SW1(config-if)#interface range g1/0/37 - 48
* SW1(config-if-range)#switchport mode access
* SW1(config-if-range)#switchport port-security violation restrict
* SW1(config-if-range)#exit
* SW1(config)#login on-failure
* SW1(config)#login on-success

### Issues:
To SSH use SSH protocol 1 as the switch is too old to understand the more secure protocal 2

If there are problems with devices connecting to switch, change the port it is connected to as this may help.

## Firewall Configuration

Configure firewall management interface
Settup DNS for internet access in service routes, then settup dynamic updates.
Downloaded updates then installed the updates for antivirus and Application & threats
### Configure DHCP on firewall
* Network>DHCP> add
#### LEASE:
* interface eth1/2 mode auto
* IP pool 192.168.100.50-192.168.100.70
#### OPTIONS:
* Gateway: 192.168.100.1
* Subnet Mask: 255.255.255.0
* primary DNS: windows server (192.168.100.100)
* Secondary DNS: 8.8.8.8
* press > OK > Commit changes

### Configuring Network Interfaces:
Network > Interfaces:
* Ethernet 1/1
 * Interface type: Layer3
 * Assign interface to: VR-1, WAN
 * IPv4: 192.168.108.235/24
* Ethernet 1/2
 * Interface type: Layer3
 * Assign interface to: VR-1, LAN
 * IPv4: 192.168.100.1/24
 * Management Profile: management
* Ethernet 1/3
 * Interface type: Layer3
 * Assign interface to: VR-1, DMZ-Zone
 * IPv4: 192.168.50.1
 * Management Profile: Ping

![image](https://github.com/rajamc/Cyber-Project/assets/163961365/e8b73a63-da34-49e4-b4ee-a6cb3a6d7539)

### Configuring Virtual Router:
Network > Virtual Routers > Add
* Router Settings:
  * Name: VR-1
  * Interfaces: <br>
    &nbsp;&nbsp;ethernet1/1 <br>
    &nbsp;&nbsp;ethernet1/2 <br>
    &nbsp;&nbsp;ethernet1/3 <br>

![image](https://github.com/rajamc/Cyber-Project/assets/163961365/62492952-e3f0-4bca-9a7a-804f089e0c14)

* Static Routes > IPv4 > Add:
 * Name: Default-route
 * Destination: 0.0.0.0/0
 * Type: ip-address
 * Value: 192.168.108.1

![image](https://github.com/rajamc/Cyber-Project/assets/163961365/b4f56537-ddbf-4aa2-96e1-42939f65a408)

### Configure Interface management profile
Network > Network Profiles > Interface Mgmt > Add
* Interface management profile:
 * Name: Ping
 * Network services: Ping

![image](https://github.com/rajamc/Cyber-Project/assets/163961365/0082539b-a61a-4f8a-940a-ffbaf19d2c5d)

### Configure Security Zones:
Network > Zones > Add:
* Name: WAN
 * Type: Layer3
	* Interfaces: ethernet1/1
	* \> OK > Add:
* Name: LAN
	* Type: Layer3
	* User Identification ACL: Tick box next to Enable User Identification
	* Interfaces: ethernet1/2
	* \> OK > Add:
* Name: DMZ
	* Type: Layer3
	* Interfaces: ethernet1/3

![image](https://github.com/rajamc/Cyber-Project/assets/163961365/7ff3b775-fd87-436c-9ac4-d7c80fe54e22)

### Create Tags for each Zone
```
Objects > Tags > Add: x3
	Name: DMZ 		Colour: Orange
	Name: WAN 		Colour: Black
	Name: LAN		Colour: Green
```
![image](https://github.com/rajamc/Cyber-Project/assets/163961365/22a37583-a54e-4601-9a19-0eaeedd53c7a)

### Create Security profiles then group:
Objects > Security Profiles <br>
Clone all predefined security profiles and call them production for: 
* Antivirus
* Anti-Spyware
* Vulnerability Protections
* URL Filtering
* File Blocking
* WildFire Alalysis
Create a securty profile group that uses all the above profiles by going to Objects > Security Profile Groups.

![image](https://github.com/rajamc/Cyber-Project/assets/163961365/c9c5ab57-8d2e-464f-a654-9186c1390f5f)


### Create NAT Policies
Policies > Nat > Add: <br>
&nbsp; \> General<br>
&nbsp;&nbsp;&nbsp; Name: Internet_access_to_LAN<br>
&nbsp;&nbsp;&nbsp; Description: Translates traffic from LAN to WAN<br>
&nbsp;&nbsp;&nbsp; Tags: LAN<br>
&nbsp; \> Original Packet<br>
&nbsp;&nbsp;&nbsp; Source Zone: LAN<br>
&nbsp;&nbsp;&nbsp; Destination Zone: WAN<br>
&nbsp;&nbsp;&nbsp; Interface: ethernet1/1<br>
&nbsp;&nbsp;&nbsp; Any source and destination address<br>
&nbsp; \> Translated Packet > Source Address Translation:<br>
&nbsp;&nbsp;&nbsp; Translation Type: Dynamic IP And Port<br>
&nbsp;&nbsp;&nbsp; Address Type: Interface Address<br>
&nbsp;&nbsp;&nbsp; Interface: ethernet1/1<br>
&nbsp;&nbsp;&nbsp; IP address 192.168.108.235/24<br>
&nbsp; \> OK<br>
Add:<br>
&nbsp; \> General<br>
&nbsp;&nbsp;&nbsp; Name: Internet_access_to_DMZ<br>
&nbsp;&nbsp;&nbsp; Description: Translates traffic from DMZ to WAN<br>
&nbsp;&nbsp;&nbsp; Tags: DMZ<br>
&nbsp; \> Original Packet<br>
&nbsp;&nbsp;&nbsp; Source Zone: DMZ<br>
&nbsp;&nbsp;&nbsp;	Destination Zone: WAN<br>
&nbsp;&nbsp;&nbsp;	Interface: ethernet1/1<br>
&nbsp;&nbsp;&nbsp;	Any source and destination address<br>
&nbsp;	\> Translated Packet > Source Address Translation:<br>
&nbsp;&nbsp;&nbsp;	Translation Type: Dynamic IP And Port<br>
&nbsp;&nbsp;&nbsp;	Address Type: Interface Address<br>
&nbsp;&nbsp;&nbsp;	Interface: ethernet1/1<br>
&nbsp;&nbsp;&nbsp;	IP address 192.168.108.235/24<br>
&nbsp;	\> OK<br>
Add:<br>
&nbsp; \> General<br>
&nbsp;&nbsp;&nbsp;	Name: dstNat-WAN-to-DMZ<br>
&nbsp;&nbsp;&nbsp;	Description: Allow internet traffic to the DMZ using the IP: 192.168.108.135<br>
&nbsp;	\> Original Packet<br>
&nbsp;&nbsp;&nbsp;	Source Zone: WAN<br>
&nbsp;&nbsp;&nbsp;	Destination Zone: WAN<br>
&nbsp;&nbsp;&nbsp;	Interface: any<br>
&nbsp;&nbsp;&nbsp;	Any source address <br>
&nbsp;&nbsp;&nbsp;	Destination address: 192.168.108.135<br>
&nbsp;	\> Translated Packet > Destination Address Translation:<br>
&nbsp;&nbsp;&nbsp;	Translation Type: Static IP<br>
&nbsp;&nbsp;&nbsp;	Translated Address: 192.168.50.100<br>
&nbsp;	\> OK<br>

![image](https://github.com/rajamc/Cyber-Project/assets/163961365/d0f78662-e244-4167-a357-c6768447d5ed)

### Configure Security Policy
Policies > Security
* Interzone-default logging:
  * Interzone-default > Override:
    * Log Setting > Log at Session End
    * Log Forwarding: KaliPurpleSIEMProfile

### Create Security Policies:
Policies > Security > Add<br>
DMZ-to-Lan
* General:
  * Name: DMZ-to-LAN
  * Description: DMZ connectivity to LAN
  * Tags: Allow
* Source:
  * Source Zone: DMZ Zone
  * Source Address: 192.168.50.100
* Destination:
  * Destination Zone: LAN
  * Destination Address: 192.168.100.200
* Actions
  * Action: Allow
  * Log Setting > Log at Session End
  * Log Forwarding: KaliPurpleSIEMProfile
  * Profile Group: Production Security Profiles

LAN-to-WAN
* General:
  * Name: LAN-to-WAN
  * Description: User PCs to Internet
  * Tags: Allow
* Source:
  * Source Zone: LAN
* Destination:
  * Destination Zone: WAN
* Service:
  * Service: application-default
* Actions:
  * Action: Allow
  * Log Setting > Log at Session End
  * Log Forwarding: KaliPurpleSIEMProfile
  * Profile Group: Production Security Profiles

LAN-to-DMZ
* General:
  * Name: LAN-to-DMZ
  * Description: LAN to access DMZ
  * Tags: Allow
* Source:
  * Source Zone: LAN
* Destination:
  * Destination Zone: DMZ Zone
* Actions:
  * Action: Allow
  * Log Setting > Log at Session End
  * Log Forwarding: KaliPurpleSIEMProfile
  * Profile Group: Production Security Profiles

DMZ-to-WAN
* General: 
  * Name: DMZ-to-WAN
  * Description: DMZ zone to internet
  * Tags: Allow
* Source:
  * Source Zone: DMZ-Zone
* Destination:
  * Destination Zone: WAN
* Service:
  * Service: application-default
* Actions:
  * Action: Allow
  * Log Setting > Log at Session End
  * Log Forwarding: KaliPurpleSIEMProfile
  * Profile Group: Production Security Profiles
	
WAN-to-DMZ:
* General: 
  * Name: WAN-to-DMZ
  * Description: Internet access to DMZ
  * Tags: Allow
* Source:
  * Source Zone: WAN
* Destination:
  * Destination Zone: DMZ-Zone
  * Destination Address: 192.168.108.135
* Application:
  * Applications: web-browsing
* Service:
  * Service: service-http
* Actions:
  * Action: Allow
  * Log Setting > Log at Session End
  * Log Forwarding: KaliPurpleSIEMProfile
  * Profile Group: Production Security Profiles

WAN-to-DMZ-REMOTE-MGT
* General: 
  * Name: WAN-to-DMZ-REMOTE-MGT
  * Description: Remote management for DMZ
  * Tags: Allow
* Source:
  * Source Zone: WAN
  * Source Address: 192.168.108.16
* Destination:
  * Destination Zone: DMZ-Zone
  * Destination Address: 192.168.108.135
* Application:
  * Applications: ms-rdp, mysql, ssh, ssh-tunnel
* Service:
  * Service: application-default
* Actions:
  * Action: Allow
  * Log Setting > Log at Session End
  * Log Forwarding: KaliPurpleSIEMProfile
  * Profile Group: Production Security Profiles

![image](https://github.com/rajamc/Cyber-Project/assets/163961365/21361a1b-1ceb-43b6-a099-152c6e1f1605)

## SIEM Configuration:
### Kali Purple installation

&nbsp;&nbsp;Attatched Kali Purple iso to VirtualBox (kali-linux-2023.4-installer-purple-amd64.iso)
&nbsp;&nbsp;Installed onto SSD as with Debian 11 bullseye 64-bit with 20064MB RAM and 4 cores with 50GB of storage.
&nbsp;&nbsp;Set Adapter 1 it NAT Networking to download updates for installation.

&nbsp;&nbsp;Followed Graphical setup/install
 
&nbsp;&nbsp;Hostname: 

&nbsp;&nbsp;&nbsp;&nbsp;PurplePrototype (cloned the prototype VM to production)

&nbsp;&nbsp;&nbsp;&nbsp;Domain: (blank)

&nbsp;&nbsp;&nbsp;&nbsp;Username: zoey

&nbsp;&nbsp;&nbsp;&nbsp;Password: Password1
  
&nbsp;&nbsp;Disk partition:

&nbsp;&nbsp;&nbsp;&nbsp;used entire disk

&nbsp;&nbsp;&nbsp;&nbsp;all files installed on one partition

&nbsp;&nbsp;&nbsp;&nbsp;wrote changes to disk

&nbsp;&nbsp;&nbsp;&nbsp;Partition disks: yes

&nbsp;&nbsp;Software selection:
    
&nbsp;&nbsp;&nbsp;&nbsp;accepted defaults
    
&nbsp;&nbsp;&nbsp;&nbsp;install grub -y

&nbsp;&nbsp;&nbsp;&nbsp;on /dev/sda

&nbsp;&nbsp;Next reboot and log in

&nbsp;&nbsp;Checked connectivity:

```
    ip a  (DHCP assigned IP because currently connected to e.208 network for updates)
    ping 8.8.8.8
```

&nbsp;&nbsp;Downloaded and installed updates:

```
    sudo apt update && sudo apt upgrade -y
```

&nbsp;&nbsp;Assigning static IP via GUI:
```
    Edit connections
    eth0
    IPv4 settings
    IP: 192.168.100.200/24
    
    Gateway: 192.168.100.1
    DNS Servers: 8.8.8.8
    
    
    restart machine to refresh eth0
```
Testing connectivity on Kali Purple
```
    ip a   
    ping 192.168.100.1  
    ping 192.168.108.1  
    ping 8.8.8.8
```
&nbsp;&nbsp;Testing PaloAlto Firewall is accessable
```
    Navigate to https://192.168.100.1 on Firefox
    log in with username: admin    (Requires Management profile on Eth1/2 (The interface that goes to LAN))
                password: Password1
``` 
**Configuring Kali Purple for remote management**
```
    sudo systemctl enable ssh --now  
    sudo apt update && sudo apt install xrdp 
    sudo systemctl enable xrdp --now 
    sudo wget -P /etc/polkit-1/localauthority/50-local.d https://gitlab.com/kalilinux/documentation/kali-purple/-/raw/main/301_kali-purple/overlays/etc/polkit-1/localauthority/50-local.d/45-allow-colord.pkla
```
**Installing Elastic + Kibana on Kali Purple**

&nbsp;&nbsp;&nbsp;Installing Elasticsearch:
```
  sudo apt update && sudo apt upgrade 
  sudo bash -c "export HOSTNAME=kali-purple.kali.purple; apt-get install elasticsearch -y"

  Generated Superuser Password: zvTy6gUPonVCKwbNduFY 
```
&nbsp;&nbsp;&nbsp;Single-node setup
```
  sudo sed -e '/cluster.initial_master_nodes/ s/^#*/#/' -i /etc/elasticsearch/elasticsearch.yml 
  echo "discovery.type: single-node" | sudo tee -a /etc/elasticsearch/elasticsearch.yml
```
&nbsp;&nbsp;Installing Kibana
   
&nbsp;&nbsp;&nbsp;&nbsp;adding Kali maching to /etc/hosts/
  ```
  sudo cat /etc/hosts  
  sudo nano /etc/hosts 
  ```
&nbsp;&nbsp;&nbsp;&nbsp;Should look like this:
```
  127.0.0.1       localhost
  127.0.1.1       PurplePrototype   (called that cause Prod-SIEM cloned from Prototype)
  192.168.100.200 kali-purple.kali.purple
  
  # The following lines are desirable for IPv6 capable hosts
  ::1     localhost ip6-localhost ip6-loopback
  ff02::1 ip6-allnodes
  ff02::2 ip6-allrouters
```
&nbsp;&nbsp;&nbsp;&nbsp;Installing Kibana + generating encryption keys
```
  sudo apt install kibana  
  sudo /usr/share/kibana/bin/kibana-encryption-keys generate –q
  
  Encryption keys: xpack.encryptedSavedObjects.encryptionKey: 39ad9b931ff617316623e780f300ae00
                   xpack.reporting.encryptionKey: 0e90a402d329e27cab63545a03bad871
                   xpack.security.encryptionKey: c1b5e7f0732a0b4783e2abe85fd67b44

  echo "server.host: \"kali-purple.kali.purple\"" | sudo tee -a /etc/kibana/kibana.yml 
  sudo systemctl enable elasticsearch kibana --now
```

**Enrolling Kibana**

&nbsp;&nbsp;Verify Kibana is running
```
  systemctl status kibana 
  systemctl status elasticsearch
```
&nbsp;&nbsp;Generate enrolement token
```
  sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
  Enrollment-token: eyJ2ZXIiOiI4LjEzLjMiLCJhZHIiOlsiMTkyLjE2OC4xMDAuMjAwOjkyMDAiXSwiZmdyIjoiYjkzYzY3MDllMjY0ZDFlNWY2ZGU5MmZjNzg3ZjQxMWY1YWVkNzFhZmYxZDY0N2M4MjMzZjI0MDI4NGFlMTcyMCIsImtleSI6IkVzSDVPNDhCOUx2cXBNMXlWalFrOnNhTG1LMENmUW4yaGJDZUVmYnpHaFEifQ==
```
&nbsp;&nbsp;&nbsp;&nbsp;Navigate to http://192.168.100.200:5601 and paste the enrollment token > press "Configure Elastic"

&nbsp;&nbsp;Generate a verification code
```
 sudo /usr/share/kibana/bin/kibana-verification-code
```
&nbsp;&nbsp;&nbsp;&nbsp;enter verification code into elastic GUI in browser

&nbsp;&nbsp;&nbsp;&nbsp;press verify

&nbsp;&nbsp;&nbsp;&nbsp;Login as:

&nbsp;&nbsp;&nbsp;&nbsp;username: elastic

&nbsp;&nbsp;&nbsp;&nbsp;Password: zvTy6gUPonVCKwbNduFY

&nbsp;&nbsp;&nbsp;&nbsp;Enrollment and login to Kibana is successful

**Enable HTTPS for Kibana**

&nbsp;&nbsp;Generate required certificates and keys

&nbsp;&nbsp;commands needed:
```
  sudo /usr/share/elasticsearch/bin/elasticsearch-certutil ca 
  sudo /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12 --dns kali-purple.kali.purple,kali-purple.kali.purple,siem --out kibana-server.p12 
  sudo openssl pkcs12 -in /usr/share/elasticsearch/elastic-stack-ca.p12 -clcerts -nokeys -out /etc/kibana/kibana-server_ca.crt 
  sudo openssl pkcs12 -in /usr/share/elasticsearch/kibana-server.p12 -out /etc/kibana/kibana-server.crt -clcerts -nokeys 
  sudo openssl pkcs12 -in /usr/share/elasticsearch/kibana-server.p12 -out /etc/kibana/kibana-server.key -nocerts -nodes 
  sudo chown root:kibana /etc/kibana/kibana-server_ca.crt 
  sudo chown root:kibana /etc/kibana/kibana-server.key 
  sudo chown root:kibana /etc/kibana/kibana-server.crt 
  sudo chmod 660 /etc/kibana/kibana-server_ca.crt 
  sudo chmod 660 /etc/kibana/kibana-server.key 
  sudo chmod 660 /etc/kibana/kibana-server.crt 
   
  echo "server.ssl.enabled: true" | sudo tee -a /etc/kibana/kibana.yml 
  echo "server.ssl.certificate: /etc/kibana/kibana-server.crt" | sudo tee -a /etc/kibana/kibana.yml 
  echo "server.ssl.key: /etc/kibana/kibana-server.key" | sudo tee -a /etc/kibana/kibana.yml 
  echo "server.publicBaseUrl: \"https://kali-purple.kali.purple:5601\"" | sudo tee -a /etc/kibana/kibana.yml 

```

&nbsp;&nbsp;Copy encryption keys into /etc/kibana/kibana.yml that were generated earlier

```
  sudo nano /etc/kibana/kibana.yml
  copy the prior generated keys to the bottom of file
  sudo systemctl restart kibana
```

&nbsp;&nbsp;&nbsp;&nbsp; To log into Kibana open browser and navigate to https://192.168.100.200:5601

&nbsp;&nbsp;&nbsp;&nbsp;Log in information:

&nbsp;&nbsp;&nbsp;&nbsp;Username: elastic

&nbsp;&nbsp;&nbsp;&nbsp;Password: zvTy6gUPonVCKwbNduFY

&nbsp;&nbsp;Creating an elastic user account

&nbsp;&nbsp;&nbsp;&nbsp; Management > Stack Management > Users > Create user

&nbsp;&nbsp;&nbsp;&nbsp;Username: zoey

&nbsp;&nbsp;&nbsp;&nbsp;Password: Password1

&nbsp;&nbsp;&nbsp;&nbsp;Priveleges: superuser

&nbsp;&nbsp;&nbsp;&nbsp;Create user

**Fleet Server Installation**

&nbsp;&nbsp;&nbsp;&nbsp;Before doing anything run
```
  sudo apt update && sudo apt upgrade -y
```

&nbsp;&nbsp;&nbsp;&nbsp;After that's done:

&nbsp;&nbsp;&nbsp;&nbsp;Open Firefox > Navigate to Kibana > Fleet > Add Fleet server

&nbsp;&nbsp;&nbsp;&nbsp;Name: Fleet server policy

&nbsp;&nbsp;&nbsp;&nbsp;URL: https://192.168.100.200:5601

&nbsp;&nbsp;&nbsp;&nbsp;Press: - Generate Fleet server policy button

&nbsp;&nbsp;Install fleet server to central host

&nbsp;&nbsp;&nbsp;&nbsp;copy the command generated into a terminal window 
```
(example)
curl -L -O https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.X.X-linux-x86_64.tar.gz
tar xzvf elastic-agent-8.X.X-linux-x86_64.tar.gz
cd elastic-agent-8.7.0-linux-x86_64
sudo ./elastic-agent install \ --fleet-server-es=https://192.168.100.200:9200 \ --fleet-server-service-token= eyJ2ZXIiOiI4LjEzLjMiLCJhZHIiOlsiMTkyLjE2OC4xMDAuMjAwOjkyMDAiXSwiZmdyIjoiYjkzYzY3MDllMjY0ZDFlNWY2ZGU5MmZjNzg3ZjQxMWY1YWVkNzFhZmYxZDY0N2M4MjMzZjI0MDI4NGFlMTcyMCIsImtleSI6IkVzSDVPNDhCOUx2cXBNMXlWalFrOnNhTG1LMENmUW4yaGJDZUVmYnpHaFEifQ==
```
&nbsp;&nbsp;&nbsp;&nbsp;Wait until files are copied, extracted, and elastic agent is installed and connection has been confirmed, then type "y" to run it as a service, it should then display :Elastic Agent has been successfully installed.

&nbsp;&nbsp;&nbsp;&nbsp;Fleet will then display: Fleet server connected

&nbsp;&nbsp;&nbsp;&nbsp;Press: Continue enrolling Elastic Agent button

&nbsp;&nbsp;&nbsp;&nbsp;Browse to Elastic: Fleet > Agents 

&nbsp;&nbsp;&nbsp;&nbsp;Make sure it say's Agent is Healthy

&nbsp;&nbsp;Adding Elastic Defend and Windows agents

&nbsp;&nbsp;&nbsp;&nbsp;Go to: Management > Integration > add security integrations

&nbsp;&nbsp;&nbsp;&nbsp;select and add Elastic Defend

&nbsp;&nbsp;&nbsp;&nbsp;Integration name: Windows defend

&nbsp;&nbsp;&nbsp;&nbsp;select: traditional endproints, Next gen antivirus

&nbsp;&nbsp;Where to add this integration

&nbsp;&nbsp;&nbsp;&nbsp;New Hosts - create agent policy

&nbsp;&nbsp;&nbsp;&nbsp;New agent policy name: Windows defend policy

&nbsp;&nbsp;&nbsp;&nbsp;Make sure "Collect system logs and metrics" is selected

&nbsp;&nbsp;&nbsp;&nbsp;Save and continue

&nbsp;&nbsp;&nbsp;&nbsp;Add elastic agent to your Host

&nbsp;&nbsp;&nbsp;&nbsp;Select Windows enrollment token for powershell script (For the DMZ server enrollment, select Linux Tar script instead)

&nbsp;&nbsp;Installing Elastic Agent on Windows host

&nbsp;&nbsp;&nbsp;&nbsp;First verify connectivity with the following pings:
```
  ping 192.168.100.200
  ping 8.8.8.8
```
&nbsp;&nbsp;&nbsp;&nbsp;Copy the script into an elevated powershell window after adding --insecure to the end of the code

&nbsp;&nbsp;&nbsp;&nbsp;Example script:
```
$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest -Uri https://artifacts.elastic.co/downloads/beats/elastic-agent/elastic-agent-8.13.4-windows-x86_64.zip -OutFile elastic-agent-8.13.4-windows-x86_64.zip
Expand-Archive .\elastic-agent-8.13.4-windows-x86_64.zip -DestinationPath .
cd elastic-agent-8.13.4-windows-x86_64
.\elastic-agent.exe install --url=https://192.168.100.200:8220 --enrollment-token=REtiRXZZOEJNQ2Z6T09GZzNscDk6UnNSR2JMNG9TVk83dzlGSXdvZkdUdw== --insecure
```

&nbsp;&nbsp;&nbsp;&nbsp;Select 'y' to all prompts (should only be asked to run elastic as a service)

&nbsp;&nbsp;&nbsp;&nbsp;On windows client, wait until you see "Successfully enrolled elastic agent, Elastic agent has been successfully installed

&nbsp;&nbsp;&nbsp;&nbsp;In the Kibana GUI wait for: Agent enrolment confirmed, Incoming data confirmed 

&nbsp;&nbsp;&nbsp;&nbsp;(traffic may have to be generated try pinging the Kali machine again)

&nbsp;&nbsp;Verify Endpoint protection is properly configured 

&nbsp;&nbsp;&nbsp;&nbsp;Browse to: Elastic > Integrations > Elastic Defend

&nbsp;&nbsp;&nbsp;&nbsp;Check that the windows client is running

&nbsp;&nbsp;&nbsp;&nbsp;Browse to: Elastic > Fleet > Agents

&nbsp;&nbsp;&nbsp;&nbsp;Check that the Windows client and Windows defend Policy is healthy

&nbsp;&nbsp;&nbsp;&nbsp;Browse to: Elastic > Fleet > Agent Policies > Windows defend policy > edit integration

&nbsp;&nbsp;&nbsp;&nbsp;Make sure Protection level = prevent, and windows events are collected

&nbsp;&nbsp;Check that data is being forwarded to elastic via fleet

&nbsp;&nbsp;&nbsp;&nbsp;Browse to: Elastic > Discover

&nbsp;&nbsp;&nbsp;&nbsp;Filter with "agent.name : ''" to see collected logs


&nbsp;&nbsp;If fleet server is unhealthy:


&nbsp;&nbsp;&nbsp;&nbsp;If the fleet server is unhealthy restarting it may help:
```
  Kali:
  sudo elastic-agent restart

  Windows:
  PowerShell: C:\Windows\system32>Stop-Service Elastic Agent
  PowerShell: C:\Windows\system32>Start-Service Elastic Agent
```

&nbsp;&nbsp;Incompatible Version

&nbsp;&nbsp;&nbsp;&nbsp;Agents can be enrolled onto a outdated fleet server but logs wont be collected and forwarded to elastic,

&nbsp;&nbsp;&nbsp;&nbsp;To update: Fleet > Agents >Fleet Server Policy - Actions: Upgrade agent

**Configure integration policy for Elastic Defend**

&nbsp;&nbsp;&nbsp;&nbsp;Elastic Security > Manage > Policies - Policy settings

&nbsp;&nbsp;&nbsp;&nbsp;Make sure preventions against:Malware, Ransomware, Memory threats, and malicious behaviour is enabled

&nbsp;&nbsp;&nbsp;&nbsp;Policy settings: Save

**Selecting security rules**

&nbsp;&nbsp;&nbsp;&nbsp;Elastic Security > Alerts - manage rules\

&nbsp;&nbsp;&nbsp;&nbsp;Select all rules

&nbsp;&nbsp;&nbsp;&nbsp;Bulk actions - enable






