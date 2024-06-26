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
