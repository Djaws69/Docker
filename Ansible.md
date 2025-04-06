# 1. Mise en place

## A. Setup Azure

### 🌞 Vous me livrerez vos deux fichiers en compte-rendu


### main.tf

````
provider "azurerm" {
  features {}
  subscription_id = "my id "  
}


variable "prefix" {
  description = "da names"
  default     = "ansible-vms"
}

variable "location" {
  description = "da loc"
  default     = "West Europe"
}

variable "ssh_public_key" {
  description = "SSH public key for user access"
  type        = string
}

variable "user_password" {
  description = "Password for the user"
  type        = string
  sensitive   = true
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


resource "azurerm_public_ip" "pip_vm1" {
  name                = "${var.prefix}-pip-vm1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_public_ip" "pip_vm2" {
  name                = "${var.prefix}-pip-vm2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}


resource "azurerm_network_interface" "nic_vm1" {
  name                = "${var.prefix}-nic-vm1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip_vm1.id
  }
}

resource "azurerm_network_interface" "nic_vm2" {
  name                = "${var.prefix}-nic-vm2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip_vm2.id
  }
}


resource "azurerm_linux_virtual_machine" "vm1" {
  name                            = "${var.prefix}-vm1"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "azureuser"
  network_interface_ids = [
    azurerm_network_interface.nic_vm1.id,
  ]
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


resource "azurerm_linux_virtual_machine" "vm2" {
  name                            = "${var.prefix}-vm2"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "azureuser"
  network_interface_ids = [
    azurerm_network_interface.nic_vm2.id,
  ]
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

### terraform.tfvars

````
ssh_public_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDBGvK6IPkq8Bp7/XQKNdbFgvN6gGE7B1LJXt839918NeYZ21ArGTtAINzULEMU2UbdDae1zQ8tAq+Rz8of95Vbmjg6EMZI7AYva3bGrpPW4iL7gAWxy5KzIQL4lu4I00GhFtnCwxz69U335euEUktLFw3ld495FzevvB/fVyp5QX5eKFH0SOZeTqS2l2IJP+G83sf2fVBN5JmwOYMPJelnS1R8T5d9GDaDDclj9YjOS5TK7Y8yeHmSRpYYpWPoSOS5VzKBnBsrPfoMM0htrcp92KN/Xb6qVtZMFFi1l08Cf1gquFsodVfVjR6dfDC08L/vh3Nr9G+U6kvl54jni5/8fcqY+3HT7by2etRAasBZGSH1Ne43uDvaDmuiteXMUGRtcU8h9sHrJe/vCktrsRrjE7HJ1M945ueYz8T8BV2S8VFsWj4BIZ+h2jNpX5hk6dQj1crf/r+7liopPk5gaRxQpVIRq5DAYcbfuRkB5FPwct3+HwNVoyI4GtH3X/j70l1YmOU2467AK7RLPwz3LDqrYGhT2lcVSENf/uLWvA6K+2/PZypm4f4ffHdUFf7DLMnGFTzaAAW72rv4EYIO+nAhZIWEO6cskPLOVA2mplwflJbnYX0nbPoQMO/K1m94LKkxj6JdU3xJVIRQR+7ByQYkUCWXojRhIgUSJZ2qO034Dw=="
user_password  = "djaws"
`````

### cloud-init.txt

````
#cloud-config


users:
  - default
  - name: djaws
    gecos: "Djaws User"
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    shell: /bin/bash
    groups:
      - sudo
    ssh_authorized_keys:
      - "${ssh_public_key}"

chpasswd:
  list: |
    djaws:${user_password}
  expire: False


packages:
  - python3
  - python3-pip
  - python3-dev
  - gcc
  - libffi-dev
  - sudo

runcmd:
  - sudo pip3 install ansible


````

# B. Setup sur votre poste


## 🌞 Pour le compte-rendu

### ngnix.yml here 

````
- name: Install nginx
  hosts: web
  become: true

  tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present

  - name: Insert Certificat
    template:
      src: localhost.crt
      dest: /etc/pki/tls/certs/tp3.crt

  - name: Insert Key
    template:
      src: /etc/pki/tls/private/tp3.key
      dest: /

  - name: Create dir tp3 site
    ansible.builtin.file:
      path: /var/www/tp3_site
      state: directory
      mode: '0755'
      owner: www-data
      group: www-data

  - name: Insert index
    template:
      src: index.html
      dest: /var/www/tp3_site/index.html

  - name: Insert conf nginx
    template:
      src: static-site-config
      dest: /etc/nginx/sites-enabled/default

  - name: Start NGiNX
    service:
      name: nginx
      state: started
````

# hosts.ini

````

[web]
172.201.26.128
````

# Le curl

````
 curl -k https://172.201.26.128/

TP3 - Site HTTPS en Ansible !
````

# B. MariaDB ou MySQL


## 🌞 Pour le compte-rendu



### mysql.yml
````

- name: Setup mysql
  hosts: db
  become: true

  tasks:
    - name: Installing Mysql  and dependencies
      package:
       name: "{{item}}"
       state: present
       update_cache: yes
      loop:
       - mysql-server
       - mysql-client 
       - python3-mysqldb
       - libmysqlclient-dev
      become: yes
  
    - name: start and enable mysql service
      service:
        name: mysql
        state: started
        enabled: yes    

    - name: creating mysql user
      mysql_user:
        name: "djaws"
        password: "djaws69"
        priv: '*.*:ALL'
        host: '%'
        state: present    

    - name: creating db
      mysql_db:
        name: "BDD"
        state: present    

    - name: Enable remote login to mysql
      lineinfile:
        path: /etc/mysql/mysql.conf.d/mysqld.cnf
        regexp: '^bind-address'
        line: 'bind-address = 0.0.0.0'
        backup: yes
      notify:
        - Restart mysql  

  handlers:
    - name: Restart mysql
      service:
        name: mysql
        state: restarted
````

### hosts.ini 

````
[db]
172.201.27.20
````

# RANGE  TA CHAMBRE

### Done

# III Repeaaaaaaatttttt


## Ngninx

### 🌞 En compte-rendu...

vhost.yml

````
- name: NGINX Virtual Host
  become: true
  template:
    src: vhost.conf.j2
    dest: /etc/nginx/conf.d/{{ item.nginx_servername }}.conf
  loop: "{{ vhosts }}"
  notify: Restart Nginx

- name: "dir webroot for vhost"
  become: true
  file:
    path: "{{ item.nginx_webroot }}"
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'
  loop: "{{ vhosts }}"

- name: "html file for vhost"
  become: true
  copy:
    dest: "{{ item.nginx_webroot }}/index.html"
    content: "{{ item.nginx_index_content }}"
    owner: www-data
    group: www-data
    mode: '0644'
  loop: "{{ vhosts }}"
````

vhost.conf.j2


````

server {
    listen {{ item.nginx_port }};
    server_name {{ item.nginx_servername }};
    root {{ item.nginx_webroot }};

    location / {
        index index.html;
    }
}
````


handlers/main.yml

`````


- name: Restart Nginx
  become: true
  service:
    name: nginx
    state: reload


````

# 2. Common

## 🌞 En compte-rendu...

users.yml

````
- name: Cree user avec ssh et sudo
  become: true
  block:
    - name: Cree user avec password et ssh acces
      user:
        name: "{{ item.username }}"
        state: present
        password: "{{ item.password }}"
        home: "/home/{{ item.username }}"
        shell: /bin/bash
        groups: admin
        append: true
      loop: "{{ users }}"

    - name:  .ssh directory exists
      file:
        path: "/home/{{ item.username }}/.ssh"
        state: directory
        owner: "{{ item.username }}"
        group: "{{ item.username }}"
        mode: '0700'
      loop: "{{ users }}"

    - name: Add SSH public key to authorized_keys
      copy:
        dest: "/home/{{ item.username }}/.ssh/authorized_keys"
        content: "{{ item.ssh_key }}"
        owner: "{{ item.username }}"
        group: "{{ item.username }}"
        mode: '0600'
      loop: "{{ users }}"

````

all.yml


````
users:
  - username: Victor
    password: "$6$G6fYlU9/"  
  ssh_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2I9zqAk6gFkw5nwn9XV4oL6yR0aIG+W3cfZrFdfV3B5YJK2m9zZMyB1hw+zEvEMxNjGvEZXytJ+qR+H+pF0feg6lZZhQ0AdE1CQNFgTVr3vGePShYVRZkgpyd3u6BJ1EJzZsR3hdR+e9du1hQWrEa7SgU7sU+N9K9Pu7NduMQvhTduXo2Xdp42rJAsF3E8Eg9bKTN3TDhnyb5+ul7VO2pOufG2gJL1PjTxz4PZ31xN9gxv==
"
  - username: Enzo
    password: "$6$TR3NqVp/"
    ssh_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDkSNeQeZzOvX34OJRo3nPSMeHE+xnAfGkUL9+VFVtn+JzddqHwRMqIGuq0mAmMxxFTkHrOSY6x+2Zm9KMwzImzRAXCcvOYopLfTAYZuqPSstYxv68rZCCbDQzAxzrC5cWE1cTI5GZXLWqkQPYi3Wa3ytLwMSpbkmBgaowVfQQMv9v==
"
````

# 3. Dynamic loadbalancer

### 🌞 En compte-rendu...

3eme vm dans le main.tf


````
resource "azurerm_linux_virtual_machine" "vm3" {
  name                            = "${var.prefix}-vm3"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_F2"
  admin_username                  = "azureuser"
  network_interface_ids = [
    azurerm_network_interface.nic_vm3.id,
  ]
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

hosts.ini

````
[webapp]
172.201.27.250
172.201.26.12

[rproxy]
172.201.27.127
````

rproxy.yml

````
- name: Install nginx and setup reverse proxy
  hosts: rproxy
  become: true

  tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: present

  - name: NGINX conf
    become: true
    template:
      src: rproxy.conf.j2
      dest: /etc/nginx/conf.d/rproxy.conf

  - name: supprimez le site de base ngnix
    file:
      path: /etc/nginx/sites-enabled/default
      state: absent
  
  - name: Start NGiNX
    service:
      name: nginx
      state: started

````

rproxy.conf.j2

````
upstream coffeeTime {
    172.201.27.250;
    172.201.26.12;
}

server {
    server_name anyway.com;

    location / {
        proxy_pass http://coffeeTime;
        proxy_set_header    Host $host;
    }
}

````