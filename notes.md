# Projet réseau - Association MAVIE

## Réseau interne

### Router1

```bash

enable
configure terminal
interface GigabitEthernet0/0
no shutdown
ip address 192.168.10.254 255.255.255.0
exit
ip dhcp pool dhcp-info
network 192.168.50.0 255.255.255.0
default-router 192.168.50.254
exit
interface GigabitEthernet0/1
no shutdown
ip address 10.0.0.1 255.0.0.0
exit
interface GigabitEthernet0/2
no shutdown
ip address 11.0.0.1 255.0.0.0
exit
ip route 192.168.40.0 255.255.255.0 10.0.0.2
```



## Réseau internet

### Router2

```bash
enable
configure terminal
interface GigabitEthernet0/0
no shutdown
ip address 11.0.0.2 255.0.0.0
exit
interface GigabitEthernet0/1
no shutdown
ip address 192.168.60.1 255.255.255.0
exit
```



## Salle Servers

### Router0

```bash
enable
configure terminal
interface GigabitEthernet0/0
no shutdown
ip address 192.168.40.254 255.255.255.0
exit
interface GigabitEthernet0/1
no shutdown
ip address 10.0.0.2 255.0.0.0
exit
ip dhcp pool dhcp-info2
network 192.168.40.0 255.255.255.0
default-router 192.168.40.254
exit
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

### Server0 - DNS

Définir l'adresse IP en DHCP (on obtient : 192.168.40.1)

### Server1 - Web

Définir l'adresse IP en DHCP (on obtient : 192.168.40.2)



Le domaine intranet.mavie pointe vers le serveur 192.168.40.2 via le DNS du server0

## Configuration pour les Réseaux virtuels (VLANs)

### Switch2

```bash
enable
configure terminal
vlan 10
name pedagogie
exit
interface range fastEthernet 0/1 - 10
switchport mode access
switchport access vlan 10
exit
vlan 40
name administration
exit
interface fastEthernet 0/2
switchport mode access
switchport access vlan 40
exit
interface fastEthernet 0/1
switchport mode trunk
switchport trunk allowed vlan 10,40
exit
```

### Switch1

```bash
enable
configure terminal
vlan 20
name comptabilite
exit
vlan 40
name administration
exit
vlan 10
name pedagogie
exit
interface range fastEthernet 0/1 - 10
switchport mode access
switchport access vlan 20
exit
interface fastEthernet 0/1
switchport mode trunk
switchport trunk allowed vlan 10,40
exit
interface fastEthernet 0/2
switchport mode access
switchport access vlan 40
exit
interface fastEthernet 0/3
switchport mode trunk
switchport trunk allowed vlan 10,20,40
exit
interface fastEthernet 0/6
switchport mode access
switchport access vlan 40
exit
```

### Switch3

```bash
enable
configure terminal
vlan 40
name administration
exit
interface range fastEthernet 0/1 - 24
switchport mode access
switchport access vlan 40
exit
```

### Router1

```bash
enable
configure terminal
interface GigabitEthernet0/0
no shutdown
exit
vlan 10
name pedagogie
exit
vlan 20
name comptabilite
exit
vlan 40
name administration
exit

# VLAN 10
interface gigabitEthernet 0/0.1
encapsulation dot1Q 10
ip address 192.168.20.254 255.255.255.0
no shutdown
exit
ip dhcp pool dhcp-vlan10
network 192.168.20.0 255.255.255.0
default-router 192.168.20.254
dns-server 192.168.40.1
exit
ip dhcp excluded-address 192.168.20.254

# VLAN 20
interface gigabitEthernet 0/0.2
encapsulation dot1Q 20
ip address 192.168.30.254 255.255.255.0
no shutdown
exit
ip dhcp pool dhcp-vlan20
network 192.168.30.0 255.255.255.0
default-router 192.168.30.254
dns-server 192.168.40.1
exit
ip dhcp excluded-address 192.168.30.254

# VLAN 40
interface gigabitEthernet 0/0.3
encapsulation dot1Q 40
ip address 192.168.50.254 255.255.255.0
no shutdown
exit
ip dhcp pool dhcp-vlan40
network 192.168.50.0 255.255.255.0
default-router 192.168.50.254
dns-server 192.168.40.1
exit
ip dhcp excluded-address 192.168.50.254
```

### Router0

```bash
ip route 192.168.20.0 255.255.255.0 10.0.0.1
ip route 192.168.30.0 255.255.255.0 10.0.0.1
ip route 192.168.50.0 255.255.255.0 10.0.0.1
```

### PC0 - PC1 - PC2 -PC3 - PC4 - PC5 - PC6 - PC7 - PC8

Définition des ip via le serveur DHCP

```bash
ipconfig /renew
```

## NAT

### Router2

```bash
enable
configure terminal
interface gigabitEthernet 0/1
ip nat outside
exit
interface gigabitEthernet 0/0
ip nat inside
exit

# Define local IPs
ip route 192.168.10.0 255.255.255.0 11.0.0.1
ip route 192.168.20.0 255.255.255.0 11.0.0.1
ip route 192.168.30.0 255.255.255.0 11.0.0.1
ip route 192.168.40.0 255.255.255.0 11.0.0.1
ip route 192.168.50.0 255.255.255.0 11.0.0.1

access-list 1 permit 192.168.10.0 0.0.0.255
access-list 1 permit 192.168.20.0 0.0.0.255
access-list 1 permit 192.168.30.0 0.0.0.255
access-list 1 permit 192.168.50.0 0.0.0.255
access-list 1 permit 192.168.60.0 0.0.0.255

ip nat inside source list 1 interface gigabitEthernet 0/1 overload

```

### Router1

```bash
enable
configure terminal
ip route 0.0.0.0 0.0.0.0 11.0.0.2
```





