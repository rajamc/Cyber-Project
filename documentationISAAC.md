# Documentation
## Cabled Network
## Firewall Configuration:
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
* 
### Set up security policy and NAT policy:
A Security policy was created for LAN to access the WAN zone in an Allow security policy. 
A NAT policy was created to give internet translation between LAN and WAN to allow for internet access.
A backup config was created using InitialNetworkConfig15-05-2024.
##Switch Configureation
"Note Issues with port 17 perhaps"
* Switch(config)#hostname SW1
* SW1(config)# enable secret Password1
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
### to SSH use SSH protocol 1

## Domain Controller Configuration
* Configured server as Domain Controller
* Added 4 users with usernames and passwords for each member of team
* Created a user group and added the members to that group
* Added Clients to Domain

#### Set domain group policy 
* Open "Group Policy Manager" 
* 
*
*
