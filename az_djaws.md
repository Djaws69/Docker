# I. Premiers pasI. Premiers pas
🌞 Créez une VM depuis le Azure CLI

````
az vm create -g SkinClub -n djaws_vm --image Ubuntu2204 --admin-username djaws --ssh-key-values C:/Users/porti/.ssh/id_rsa.pub
{
  "fqdns": "",
  "id": "/subscriptions/05e7274b-621b-4e0f-b3c7-a068bb954dec/resourceGroups/SkinClub/providers/Microsoft.Compute/virtualMachines/djaws_vm",
  "location": "centralindia",
  "macAddress": "60-45-BD-C6-6A-B7",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "74.225.238.108",
  "resourceGroup": "SkinClub",
  "zones": ""
}`
```

## 🌞 Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique.

````
djaws@djawsvm:~$ dpkg -l | grep walinuxagent
ii  walinuxagent                           2.2.46-0ubuntu5.1                       amd64        Windows Azure Linux Agent
````



````
djaws@djawsvm:~$ cloud-init status
status: done`
```

## II. Un ptit LAN
🌞 Créez deux VMs depuis le Azure CLI

### Creation du LAN 
````

az>> az network vnet create --resource-group SkinClub --name LAN --subnet-name subnet
{
  "newVNet": {
    "addressSpace": {
      "addressPrefixes": [
        "10.0.0.0/16"
      ]
    },
    "enableDdosProtection": false,
    "etag": "W/\"7b3d1efb-ee95-4fc9-aa56-e59e24fe0c0b\"",
    "id": "/subscriptions/05e7274b-621b-4e0f-b3c7-a068bb954dec/resourceGroups/SkinClub/providers/Microsoft.Network/virtualNetworks/LAN",
    "location": "centralindia",
    "name": "LAN",
    "privateEndpointVNetPolicies": "Disabled",
    "provisioningState": "Succeeded",
    "resourceGroup": "SkinClub",
    "resourceGuid": "c9a76add-ece9-4552-808c-b22178c8cea0",
    "subnets": [
      {
        "addressPrefix": "10.0.0.0/24",
        "delegations": [],
        "etag": "W/\"7b3d1efb-ee95-4fc9-aa56-e59e24fe0c0b\"",
        "id": "/subscriptions/05e7274b-621b-4e0f-b3c7-a068bb954dec/resourceGroups/SkinClub/providers/Microsoft.Network/virtualNetworks/LAN/subnets/subnet",
        "name": "subnet",
        "privateEndpointNetworkPolicies": "Disabled",
        "privateLinkServiceNetworkPolicies": "Enabled",
        "provisioningState": "Succeeded",
        "resourceGroup": "SkinClub",
        "type": "Microsoft.Network/virtualNetworks/subnets"
      }
    ],
    "type": "Microsoft.Network/virtualNetworks",
    "virtualNetworkPeerings": []
  }
}

`````
### Creation Première machine dans le LAN
`````

az vm create -g SkinClub -n djaws_vm --image Ubuntu2204 --admin-username djaws --ssh-key-values C:/Users/porti/.ssh/id_rsa.pub --vnet-name LAN --subnet subnet
{
  "fqdns": "",
  "id": "/subscriptions/05e7274b-621b-4e0f-b3c7-a068bb954dec/resourceGroups/SkinClub/providers/Microsoft.Compute/virtualMachines/djaws_vm",
  "location": "centralindia",
  "macAddress": "60-45-BD-C6-6A-B7",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "74.225.238.108",
  "resourceGroup": "SkinClub",
  "zones": ""
}

`````
### Deuxieme VM

````
az vm create -g SkinClub -n djaws_vm2 --image Ubuntu2204 --admin-username djaws --ssh-key-values C:/Users/porti/.ssh/id_rsa.pub --vnet-name LAN --subnet subnet
{
  "fqdns": "",
  "id": "/subscriptions/05e7274b-621b-4e0f-b3c7-a068bb954dec/resourceGroups/SkinClub/providers/Microsoft.Compute/virtualMachines/djaws_vm2",
  "location": "centralindia",
  "macAddress": "60-45-BD-CE-B9-51",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "20.204.99.41",
  "resourceGroup": "SkinClub",
  "zones": ""
}
````
### Ping 

```

djaws@djawsvm:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 60:45:bd:c6:6a:b7 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6245:bdff:fec6:6ab7/64 scope link
       valid_lft forever preferred_lft forever
djaws@djawsvm:~$ ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=5.70 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=3.14 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=1.22 ms
64 bytes from 10.0.0.5: icmp_seq=4 ttl=64 time=1.59 ms
64 bytes from 10.0.0.5: icmp_seq=5 ttl=64 time=2.09 ms
^C
--- 10.0.0.5 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 1.221/2.749/5.697/1.609 ms
djaws@djawsvm:~$

```

# Partie 2 

### 2. Gooooo
➜ Sur votre PC, créez un fichier cloud-init.txt avec le contenu suivant :


````
az vm create -g SkinClub -n djaws_cloud --image Ubuntu2204 --admin-username djaws --ssh-key-values C:/Users/porti/.ssh/


{
  "fqdns": "",
  "id": "/subscriptions/05e7274b-621b-4e0f-b3c7-a068bb954dec/resourceGroups/SkinClub/providers/Microsoft.Compute/virtualMachines/djaws_cloud",
  "location": "centralindia",
  "macAddress": "60-45-BD-E8-2B-C5",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.6",
  "publicIpAddress": "52.172.148.127",
  "resourceGroup": "SkinClub",
  "zones": ""
}
az>>
````
## 🌞 Vérifier que cloud-init a bien fonctionné

```
PS C:\Users\porti> ssh djaws@52.172.148.127
The authenticity of host '52.172.148.127 (52.172.148.127)' can't be established.
ED25519 key fingerprint is SHA256:Y8aq+NPYmBePi7xMy1++SIQsaq26Rat//IubQVTHnos.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '52.172.148.127' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Mar 18 13:32:58 UTC 2025

  System load:  0.02              Processes:             106
  Usage of /:   5.2% of 28.89GB   Users logged in:       0
  Memory usage: 8%                IPv4 address for eth0: 10.0.0.6
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

djaws@djawscloud:~$
````


# 3. Write your own

## 🌞 Utilisez cloud-init pour préconfigurer la VM :

````
az vm create -g SkinClub -n djaws_cloud2 --image Ubuntu2204 --admin-username djaws --ssh-key-values C:/Users/porti/.ssh
{
  "fqdns": "",
  "id": "/subscriptions/05e7274b-621b-4e0f-b3c7-a068bb954dec/resourceGroups/SkinClub/providers/Microsoft.Compute/virtualMachines/djaws_cloud2",
  "location": "centralindia",
  "macAddress": "60-45-BD-CF-06-09",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "20.204.151.215",
  "resourceGroup": "SkinClub",
  "zones": ""
}
az>>
````

### Config Cloud-init.txt

````
#cloud-config
packages:
  - docker.io

users:
  - name: djaws
    passwd: "Kali33"  
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDBGvK6IPkq8Bp7/XQKNdbFgvN6gGE7B1LJXt839918NeYZ21ArGTtAINzULEMU2UbdDae1zQ8tAq+Rz8of95Vbmjg6EMZI7AYva3bGrpPW4iL7gAWxy5KzIQL4lu4I00GhFtnCwxz69U335euEUktLFw3ld495FzevvB/fVyp5QX5eKFH0SOZeTqS2l2IJP+G83sf2fVBN5JmwOYMPJelnS1R8T5d9GDaDDclj9YjOS5TK7Y8yeHmSRpYYpWPoSOS5VzKBnBsrPfoMM0htrcp92KN/Xb6qVtZMFFi1l08Cf1gquFsodVfVjR6dfDC08L/vh3Nr9G+U6kvl54jni5/8fcqY+3HT7by2etRAasBZGSH1Ne43uDvaDmuiteXMUGRtcU8h9sHrJe/vCktrsRrjE7HJ1M945ueYz8T8BV2S8VFsWj4BIZ+h2jNpX5hk6dQj1crf/r+7liopPk5gaRxQpVIRq5DAYcbfuRkB5FPwct3+HwNVoyI4GtH3X/j70l1YmOU2467AK7RLPwz3LDqrYGhT2lcVSENf/uLWvA6K+2/PZypm4f4ffHdUFf7DLMnGFTzaAAW72rv4EYIO+nAhZIWEO6cskPLOVA2mplwflJbnYX0nbPoQMO/K1m94LKkxj6JdU3xJVIRQR+7ByQYkUCWXojRhIgUSJZ2qO034Dw==

    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    shell: /bin/bash
    groups: [docker]

runcmd:
  - docker pull alpine:latest  


systemctl:
  enabled:
    - docker

````
### Connection SSH 

````
PS C:\Users\porti> ssh djaws@4.213.63.126
The authenticity of host '4.213.63.126 (4.213.63.126)' can't be established.
ED25519 key fingerprint is SHA256:WLGpbEdlxHuRD6+K0YV2RONKnKYfxou7teib5hXUfi0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '4.213.63.126' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Mar 18 14:16:42 UTC 2025

  System load:  0.01              Processes:             111
  Usage of /:   7.2% of 28.89GB   Users logged in:       0
  Memory usage: 11%               IPv4 address for eth0: 10.0.0.5
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

14 updates can be applied immediately.
8 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

djaws@djawscloud2:~$
````

````

djaws@djawscloud2:~$ docker --version
Docker version 26.1.3, build 26.1.3-0ubuntu1~22.04.1
djaws@djawscloud2:~$
````

# PART 3 

## Terraform

### 🌞 Constater le déploiement

`````
az>> az vm show --name deuxtp-vm --resource-group deuxtp-resources -o table
Name       ResourceGroup     Location    Zones
---------  ----------------  ----------  -------
deuxtp-vm  deuxtp-resources  westeurope

````
# 3. Do it yourself

### 🌞 Créer un plan Terraform avec les contraintes suivantes

````
djaws@node-node1:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 7c:ed:8d:0a:b6:d5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::7eed:8dff:fe0a:b6d5/64 scope link
       valid_lft forever preferred_lft forever
djaws@node-node1:~$ ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=2.64 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=0.882 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=0.852 ms
^C
--- 10.0.0.5 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2018ms
rtt min/avg/max/mdev = 0.852/1.456/2.636/0.834 ms
djaws@node-node1:~$
````

# 4. cloud-iniiiiiiiiiiiiit

A. Un premier tf + cloud-init
🌞 Intégrer la gestion de cloud-init

### Fichier main.tf

``````
provider "azurerm" {
  features {}
  subscription_id = "my id azure"
}

#
resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}


resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}


resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.0.0/24"]
}


resource "azurerm_public_ip" "pip_node" {
  name                = "${var.prefix}-pip-node"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}


resource "azurerm_network_interface" "nic_node" {
  name                = "${var.prefix}-nic-node"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip_node.id
  }
}

resource "azurerm_linux_virtual_machine" "node" {
  name                            = "${var.prefix}-node"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "azureuser"  # Utilisateur créé par Azure
  network_interface_ids = [
    azurerm_network_interface.nic_node.id,
  ]
  
  # Cloud-init 
  custom_data = base64encode(templatefile("cloud-init.txt", {
    ssh_public_key = var.ssh_public_key
    user_password  = var.user_password
  }))
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  
  admin_ssh_key {
    username   = "azureuser"
    public_key = var.ssh_public_key
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

````

### Ficher cloud-init

````
#cloud-config


users:
  - name: djaws
    gecos: "DJaws User"
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    groups:
      - docker
    ssh_authorized_keys:
      - "${ssh_public_key}"


chpasswd:
  list: |
    djaws:${user_password}
  expire: False


packages:
  - docker.io


runcmd:
  - systemctl start docker
  - systemctl enable docker


runcmd:
  - docker pull alpine:latest

````

### Fichier variable 

`````
variable "prefix" {
  description = "da name"
  default     = "tp2magueule-v2"
}

variable "location" {
  description = "da loc"
  default     = "West Europe"
}

variable "ssh_public_key" {
  description = "The SSH public key to be authorized for the new user"
  type        = string
}

variable "user_password" {
  description = "Password for the new user"
  type        = string
  sensitive   = true
}

````

### Fichier terraform.tfvars

````
ssh_public_key = "hihi"
user_password  = "djaws"
````

### Connection SSH

````
PS C:\Users\porti> ssh djaws@20.73.97.58
The authenticity of host '20.73.97.58 (20.73.97.58)' can't be established.
ED25519 key fingerprint is SHA256:PjPFjI6xYJdKWGCjzkyl5mU6Xpy4oTU1yxfDtRwJMmY.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '20.73.97.58' (ED25519) to the list of known hosts.
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Mar 20 11:15:42 UTC 2025

  System load:  0.0               Processes:             124
  Usage of /:   7.2% of 28.89GB   Users logged in:       0
  Memory usage: 9%                IPv4 address for eth0: 10.0.0.4
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

15 updates can be applied immediately.
9 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

djaws@tp2magueule-v2-node:~$ docker --version
Docker version 26.1.3, build 26.1.3-0ubuntu1~22.04.1
djaws@tp2magueule-v2-node:~
````

# B. Go further
🌞 Moar cloud-init and Terraform configuration

## Cloud init

````
#cloud-config


users:
  - default
  - name: djaws
    gecos: "DJaws User"
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    shell: /bin/bash
    groups:
      - docker
    ssh_authorized_keys:
      - "${ssh_public_key}"


chpasswd:
  list: |
    djaws:${user_password}
  expire: False


apt:
  sources:
    docker.list:
      source: deb [arch=amd64] https://download.docker.com/linux/ubuntu $RELEASE stable
      keyid: 9DC858229FC7DD38854AE2D88D81803C0EBFCD88

packages:
  - apt-transport-https
  - ca-certificates
  - curl
  - gnupg
  - lsb-release
  - sudo
  - docker-ce
  - docker-ce-cli


runcmd:
  - sudo mkdir -p /opt/wikijs


write_files:
  - path: /opt/wikijs/docker-compose.yml
    content: |
      version: "3"
      services:
        db:
          image: postgres:15-alpine
          environment:
            POSTGRES_DB: wiki
            POSTGRES_PASSWORD: wikijsrocks
            POSTGRES_USER: wikijs
          logging:
            driver: none
          restart: unless-stopped
          volumes:
            - db-data:/var/lib/postgresql/data

        wiki:
          image: ghcr.io/requarks/wiki:2
          depends_on:
            - db
          environment:
            DB_TYPE: postgres
            DB_HOST: db
            DB_PORT: 5432
            DB_USER: wikijs
            DB_PASS: wikijsrocks
            DB_NAME: wiki
          restart: unless-stopped
          ports:
            - "10101:3000"  

      volumes:
        db-data: {} 
runcmd:
  - cd /opt/wikijs && sudo docker compose up -d
````
## main.tf

````
provider "azurerm" {
  features {}
  subscription_id = "05e7274b-621b-4e0f-b3c7-a068bb954dec"
}


resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}


resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}


resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.0.0/24"]
}


resource "azurerm_public_ip" "pip_node" {
  name                = "${var.prefix}-pip-node"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}


resource "azurerm_network_interface" "nic_node" {
  name                = "${var.prefix}-nic-node"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip_node.id
  }
}


resource "azurerm_network_security_group" "ssh_and_wiki" {
  name                = "ssh_and_wiki"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  
  # Règle pour SSH (port 22)
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }

  
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "wiki-port"
    priority                   = 101
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "10101"
    destination_address_prefix = "*"
  }
}


resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.nic_node.id
  network_security_group_id = azurerm_network_security_group.ssh_and_wiki.id
}


resource "azurerm_linux_virtual_machine" "node" {
  name                            = "${var.prefix}-node"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "azureuser"  
  network_interface_ids = [
    azurerm_network_interface.nic_node.id,
  ]
  
  # Cloud-init (encodé en base64)
  custom_data = base64encode(templatefile("cloud-init.txt", {
    ssh_public_key = var.ssh_public_key
    user_password  = var.user_password
  }))

  admin_ssh_key {
    username   = "azureuser"
    public_key = var.ssh_public_key
  }
  
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}


````

### Curl 

````
djaws@tp2magueule-v2-node:~$ curl 127.0.0.1:10101
<!DOCTYPE html><html><head><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta charset="UTF-8"><meta name="viewport" content="user-scalable=yes, width=device-width, initial-scale=1, maximum-scale=5"><meta name="theme-color" content="#1976d2"><meta name="msapplication-TileColor" content="#1976d2"><meta name="msapplication-TileImage" content="/_assets/favicons/mstile-150x150.png"><title>Wiki.js Setup</title><link rel="apple-touch-icon" sizes="180x180" href="/_assets/favicons/apple-touch-icon.png"><link rel="icon" type="image/png" sizes="192x192" href="/_assets/favicons/android-chrome-192x192.png"><link rel="icon" type="image/png" sizes="32x32" href="/_assets/favicons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16" href="/_assets/favicons/favicon-16x16.png"><link rel="mask-icon" href="/_assets/favicons/safari-pinned-tab.svg" color="#1976d2"><link rel="manifest" href="/_assets/manifest.json"><script>var siteConfig = {"title":"Wiki.js"}
</script><link type="text/css" rel="stylesheet" href="/_assets/css/setup.22871ffac1b643eed4d9.css"><script type="text/javascript" src="/_assets/js/runtime.js?1738531300"></script><script type="text/javascript" src="/_assets/js/setup.js?1738531300"></script></head><body><div id="root"><setup wiki-version="2.5.306"></setup></div></body></html>djaws@tp2magueule-v2-node:~$
````

# Bingo
