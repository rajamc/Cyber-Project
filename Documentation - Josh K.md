**Kali Purple installation for SIEM**

  Attatched Kali Purple iso to VirtualBox (kali-linux-2023.4-installer-purple-amd64.iso)
  Installed onto SSD as with Debian 11 bullseye 64-bit with 20064MB RAM and 4 cores with 50GB of storage.
  Set Adapter 1 it NAT Networking to download updates for installation.

  Followed Graphical setup/install
  
  Hostname: PurplePrototype (cloned the prototype VM to production)
    Domain: (blank)
    Username: zoey
    Password: Password1
  
  Disk partition:
    used entire disk
    all files installed on one partition
    wrote changes to disk
    Partition disks: yes

  Software selection:
    accepted defaults
    install grub -y
    on /dev/sda

  rebooted and logged in

  Checked connectivity:

```

  ip a  (DHCP assigned IP because currently connected to e.208 network for updates)
  ping 8.8.8.8

```

  Downloaded and installed updates:

  ```
  sudo apt update && sudo apt upgrade -y
  ```

  Assigning static IP via GUI:

    Edit connections
    eth0
    IPv4 settings
    IP: 192.168.100.200/24


    Gateway: 192.168.100.1
    DNS Servers: 8.8.8.8


    restart machine to refresh eth0

  Testing connectivity on Kali Purple

    ip a   
    ping 192.168.100.1  
    ping 192.168.108.1  
    ping 8.8.8.8

  Testing PaloAlto Firewall is accessable

    Navigate to https://192.168.100.1 on Firefox
    log in with username: admin    (Requires Management profile on Eth1/2 (The interface that goes to LAN))
                password: Password1
    
**Configuring Kali Purple for remote management**

    sudo systemctl enable ssh --now  

    sudo apt update && sudo apt install xrdp 

    sudo systemctl enable xrdp --now 

    sudo wget -P /etc/polkit-1/localauthority/50-local.d https://gitlab.com/kalilinux/documentation/kali-purple/-/raw/main/301_kali-purple/overlays/etc/polkit-1/localauthority/50-local.d/45-allow-colord.pkla

