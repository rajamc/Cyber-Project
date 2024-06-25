# Documentation
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
