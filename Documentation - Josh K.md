**Kali Purple installation for SIEM**

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
