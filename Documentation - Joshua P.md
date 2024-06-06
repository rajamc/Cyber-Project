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
#### Configure IPV4 ethernet adaptor:
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
* In a web browser connect to 192.168.100.1 to test firewall gui / connectivity.

