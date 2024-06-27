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


## **Kali Purple installation for SIEM**

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

**UP TO NUMBER 5**
