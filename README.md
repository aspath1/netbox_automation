[![Build Status](https://travis-ci.org/ksator/netbox_automation.svg?branch=master)](https://travis-ci.org/ksator/netbox_automation)

## Documentation structure
[**About Netbox**](README.md#about-netbox)  
[**About this repo**](README.md#about-this-repo)  
[**How to use this repo**](README.md#how-to-use-this-repo)    
[**Netbox Installation**](README.md#netbox-installation)  
[**install the requirements to use the automation content hosted in this repository**](README.md#install-the-requirements-to-use-the-automation-content-hosted-in-this-repository)  
[**clone this repository**](README.md#clone-this-repository)  
[**Define your variables**](README.md#define-your-variables)  
[**Configure Netbox with automation**](README.md#configure-netbox-with-automation)  
[**Generate the ansible inventory from Netbox API**](README.md#generate-the-ansible-inventory-from-netbox-api)  
[**Generate yaml files for Jinja templates or ansible playbooks from Netbox API**](README.md#generate-yaml-files-for-jinja-templates-or-ansible-playbooks-from-netbox-api)  
[**Use the yaml files generated from the Netbox API**](README.md#use-the-yaml-files-generated-from-the-netbox-api)  
[**Delete Netbox configuration with automation**](README.md#delete-netbox-configuration-with-automation)  
[**Continuous integration with Travis CI**](README.md#continuous-integration-with-travis-ci)  
[**Looking for more automation solutions**](README.md#looking-for-more-automation-solutions)  


## About Netbox

Netbox is a popular open source IP address management (IPAM) and data center infrastructure management (DCIM) tool maintained by Jeremy Stretch.  

Netbox has RESTful API. The NetBox API represents all objects in JSON.  
There is a NAPALM integration to pull some data from devices for validation purpose. 

Here's the [doc](https://netbox.readthedocs.io/en/latest/)   
Here's the [code](https://github.com/digitalocean/netbox)  

## About this repo

Netbox is supposed to be the source of truth for the the assets it manages:

So this new repo is about: 
-	Netbox configuration with automation (with a focus on Juniper devices) 
-	ansible inventory file generated with automation from Netbox API   
-	yaml files (variables) for Jinja templates or ansible playbooks generated with automation from Netbox API 

This repository doesn’t install Netbox. You still need to install Netbox yourself.

This repository has been tested using: 
- Ubuntu 16.04
- Python 2.7
- Netbox v2.2.9
    
## How to use this repo

The steps are:  
- Install Netbox. 
   - This repository doesn’t install Netbox. You still need to install Netbox yourself.  
- Install the requirements to use the python scripts hosted in this repository  
- Clone this repository
- Edit the file [**variables.yml**](variables.yml) to indicate what you want to configure on Netbox
- To configure Netbox, execute the script [**configure_netbox.py**](configure_netbox.py). 
   - It uses the variables defined in the file [**variables.yml**](variables.yml) and configures Netbox 
- To generate an Ansible inventory file from Netbox API, execute the script [**ansible_dynamic_inventory.py**](ansible_dynamic_inventory.py)
- To generate yaml files for jinja templates and ansible playbooks from Netbox API, execute the script [**generate_vars_for_jinja_from_netbox.py**](generate_vars_for_jinja_from_netbox.py)   
- To delete Netbox configuration, execute the script [**delete_netbox_configuration.py**](delete_netbox_configuration.py)

## Netbox Installation
See the [**installation guide**](http://netbox.readthedocs.io/en/stable/)  
This repository has been tested with Netbox version v2.2.9  

Once netbox is installed, you can use it: 
- netbox GUI http://your_netbox_server
- netbox API http://your_netbox_server/api/
- netbox API doc http://your_netbox_server/api/docs/

This repository uses Netbox REST API. So you need to create a Netbox API Token. 


## install the requirements to use the automation content hosted in this repository  
The python scripts hosted in this repository use the library **requests** to make REST calls to Netbox.   
Run these commands on your laptop:
```
sudo -s
pip install requests
```

## clone this repository
Run these commands on your laptop:
```
sudo -s
git clone https://github.com/ksator/netbox_automation.git 
cd netbox_automation
```

## Define your variables

The file [**variables.yml**](variables.yml) defines variables.
On your laptop, edit it to indicate details such as:
- The IP address of your Netbox server
- The Netbox user token you want to use
- The Netbox details you want to configure with automation: 
   - a tenant
   - sites
   - device-roles
   - devices
   - prefixes-roles
   - prefixes
   - device management ip addresses
   - ip addresses to assign to devices interfaces
   - interface connections (physical topology) 
   - In addition to this, there are others Netbox details that will be configured automatically: 
     - device-types (Juniper qfx5100-48s-6q and qfx10002-36q) 
     - a platform (Junos platform with a junos napalm_driver)

Run these commands on your laptop:
```
vi variable.yml
```
```
$ more variables.yml 
---
# Edit this file to define the details to configure on Netbox

# netbox server ip @
ip: 192.168.233.152

# netbox user token you want to use
token: 'b1b0f72bed6946d352b78781030e8d626f5e8c28'

# netbox tenants you want to create. please create one single tenant.
tenants: 
    - evpn-vxlan-demo

# netbox sites you want to create. the sites are assigned to the tenant.
sites:
    - dc1
    - dc2

# netbox device-roles you want to create
device-roles: 
    - "spine_switch"
    - "leaf_switch"   

# device-types qfx5100-48s-6q and qfx10002-36q are automatically created. 
# interface_templates are automatically created for device-types qfx5100-48s-6q and qfx10002-36q
# power_port_templates are automatically created for device-types qfx5100-48s-6q and qfx10002-36q
# Juniper Junos platform is automatically created with a junos napalm_driver

# prefix roles you want to create
prefix_roles: 
    - out_of_band_management
    - devices_interconnection

# prefixes you want to create. 
# These prefixes are assigned to the tenant.
prefixes:
    - prefix: 10.0.2.0/24
      role: devices_interconnection
    - prefix: 172.25.90.0/24
      role: out_of_band_management
    
# devices you want to create. the devices are assigned to the tenant.
devices: 
    - name: QFX5100-183
      device_type: qfx5100-48s-6q
      device_role: leaf_switch
      site: dc1
    - name: QFX5100-186
      device_type: qfx5100-48s-6q
      device_role: leaf_switch
      site: dc1
    - name: QFX10K2-178
      device_type: qfx10002-36q
      device_role: spine_switch
      site: dc1
    - name: QFX10K2-180
      device_type: qfx10002-36q
      device_role: spine_switch
      site: dc1
    - name: QFX10K2-181
      device_type: qfx10002-36q
      device_role: spine_switch
      site: dc1
    - name: QFX10K2-174
      device_type: qfx10002-36q
      device_role: spine_switch
      site: dc2
    - name: QFX10K2-175
      device_type: qfx10002-36q
      device_role: spine_switch
      site: dc2
    
# management ip addresses you want to create. The IP addresses are assigned to the tenant.
management_addresses: 
    - ip: 172.25.90.183
      device: QFX5100-183
      interface: vme0
      mgmt_only: True
    - ip: 172.25.90.186
      device: QFX5100-186
      interface: vme0
      mgmt_only: True
    - ip: 172.25.90.178
      device: QFX10K2-178
      interface: em0
      mgmt_only: True
    - ip: 172.25.90.174
      device: QFX10K2-174
      interface: em0
      mgmt_only: True
    - ip: 172.25.90.175
      device: QFX10K2-175
      interface: em0
      mgmt_only: True
    - ip: 172.25.90.180
      device: QFX10K2-180
      interface: em0
      mgmt_only: True
    - ip: 172.25.90.181
      device: QFX10K2-181
      interface: em0
      mgmt_only: True

# Other ip addresses you want to create. The IP addresses are assigned to the tenant.
ip_addresses:
    - ip: 10.0.2.13/31
      device: QFX10K2-180
      interface: et-0/0/0
    - ip: 10.0.2.23/31
      device: QFX10K2-180
      interface: et-0/0/1
    - ip: 10.0.2.15/31
      device: QFX10K2-181
      interface: et-0/0/0
    - ip: 10.0.2.25/31
      device: QFX10K2-181
      interface: et-0/0/1
    - ip: 10.0.2.9/31
      device: QFX10K2-178
      interface: et-0/0/0
    - ip: 10.0.2.19/31
      device: QFX10K2-178
      interface: et-0/0/1

# define how the interfaces are connected (physical topology). 
interface_connections:
    - device_a: QFX5100-183
      interface_a: et-0/0/48
      device_b: QFX10K2-178
      interface_b: et-0/0/2
      connection_status: Connected
    - device_a: QFX10K2-181
      interface_a: et-0/0/3
      device_b: QFX5100-186
      interface_b: et-0/0/49
      connection_status: Connected
```
## Configure Netbox with automation

The script [**configure_netbox.py**](configure_netbox.py) configures Netbox using the file [**variables.yml**](variables.yml) 

```
$ python configure_netbox.py 
device role spine_switch successfully created
device role leaf_switch successfully created
tenant evpn-vxlan-demo successfully created
site dc1 successfully created
site dc2 successfully created
device type qfx5100-48s-6q successfully created
device type qfx10002-36q successfully created
prefix role out_of_band_management successfully created
prefix role devices_interconnection successfully created
prefix 10.0.2.0/24 successfully created
prefix 172.25.90.0/24 successfully created
platform junos with a junos napalm_driver successfully created
device QFX5100-183 successfully created
device QFX5100-186 successfully created
device QFX10K2-178 successfully created
device QFX10K2-180 successfully created
device QFX10K2-181 successfully created
device QFX10K2-174 successfully created
device QFX10K2-175 successfully created
address ip 172.25.90.183 successfully created
address ip 172.25.90.186 successfully created
address ip 172.25.90.178 successfully created
address ip 172.25.90.174 successfully created
address ip 172.25.90.175 successfully created
address ip 172.25.90.180 successfully created
address ip 172.25.90.181 successfully created
address ip 10.0.2.13/31 successfully created
address ip 10.0.2.23/31 successfully created
address ip 10.0.2.15/31 successfully created
address ip 10.0.2.25/31 successfully created
address ip 10.0.2.9/31 successfully created
address ip 10.0.2.19/31 successfully created
interface connection between QFX5100-183 et-0/0/48 and QFX10K2-178 et-0/0/2 successfully created
interface connection between QFX10K2-181 et-0/0/3 and QFX5100-186 et-0/0/49 successfully created

```

## Generate the ansible inventory from Netbox API

The script  [**ansible_dynamic_inventory.py**](ansible_dynamic_inventory.py) generates the ansible inventory [**hosts**](hosts) from Netbox API  

```
$ python ansible_dynamic_inventory.py
```
```
$ more hosts 
#Ansible dynamic inventory file generated from Netbox API

[juniper:children]
qfx10002-36q
qfx5100-48s-6q

[qfx10002-36q]
QFX10K2-174 junos_host=172.25.90.174
QFX10K2-175 junos_host=172.25.90.175
QFX10K2-178 junos_host=172.25.90.178
QFX10K2-180 junos_host=172.25.90.180
QFX10K2-181 junos_host=172.25.90.181

[qfx5100-48s-6q]
QFX5100-183 junos_host=172.25.90.183
QFX5100-186 junos_host=172.25.90.186

[dc1]
[QFX10K2-178]
[QFX10K2-180]
[QFX10K2-181]
[QFX5100-183]
[QFX5100-186]

[dc2]
[QFX10K2-174]
[QFX10K2-175]

[leaf_switch]
[QFX5100-183]
[QFX5100-186]

[spine_switch]
[QFX10K2-174]
[QFX10K2-175]
[QFX10K2-178]
[QFX10K2-180]
[QFX10K2-181]
```

The playbook [**pb_print_junos_facts.yml**](pb_print_junos_facts.yml) has been tested with ansible 2.4.2.0.  
It uses the ansible inventory file generated from Netbox API. 

Run these commands on Ubuntu 16.04 to install Ansible and its requirements for Junos:
```
sudo -s
apt-get update
apt-get install -y python-dev libxml2-dev python-pip libxslt1-dev build-essential libssl-dev libffi-dev git
pip install junos-eznc jxmlease wget ansible==2.4.2.0 requests ipaddress cryptography 
```

```
# ansible-playbook pb_print_junos_facts.yml

PLAY [Get Facts] ********************************************************************************************************************************************************

TASK [Retrieve information from devices running Junos] ******************************************************************************************************************
ok: [QFX10K2-178]
ok: [QFX10K2-180]
ok: [QFX10K2-174]
ok: [QFX10K2-175]
ok: [QFX10K2-181]
ok: [QFX5100-183]
ok: [QFX5100-186]

TASK [Print some facts] *************************************************************************************************************************************************
ok: [QFX10K2-174] => {
    "msg": "device QFX10K2-174 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX10K2-175] => {
    "msg": "device QFX10K2-175 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX10K2-178] => {
    "msg": "device QFX10K2-178 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX10K2-180] => {
    "msg": "device QFX10K2-180 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX10K2-181] => {
    "msg": "device QFX10K2-181 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX5100-183] => {
    "msg": "device QFX5100-183 is a qfx5100-48s-6q running junos version 17.4R1-S1.9"
}
ok: [QFX5100-186] => {
    "msg": "device QFX5100-186 is a qfx5100-48s-6q running junos version 17.4R1-S1.9"
}

PLAY RECAP **************************************************************************************************************************************************************
QFX10K2-174                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-175                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-178                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-180                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-181                : ok=2    changed=0    unreachable=0    failed=0   
QFX5100-183                : ok=2    changed=0    unreachable=0    failed=0   
QFX5100-186                : ok=2    changed=0    unreachable=0    failed=0   
```
```
# ansible-playbook pb_print_junos_facts.yml --limit QFX10K2-180

PLAY [Get Facts] ********************************************************************************************************************************************************

TASK [Retrieve information from devices running Junos] ******************************************************************************************************************
ok: [QFX10K2-180]

TASK [Print some facts] *************************************************************************************************************************************************
ok: [QFX10K2-180] => {
    "msg": "device QFX10K2-180 is a qfx10002-36q running junos version 17.4R1-S1.9"
}

PLAY RECAP **************************************************************************************************************************************************************
QFX10K2-180                : ok=2    changed=0    unreachable=0    failed=0   
```
```
# ansible-playbook pb_print_junos_facts.yml --limit dc2

PLAY [Get Facts] ********************************************************************************************************************************************************

TASK [Retrieve information from devices running Junos] ******************************************************************************************************************
ok: [QFX10K2-174]
ok: [QFX10K2-175]

TASK [Print some facts] *************************************************************************************************************************************************
ok: [QFX10K2-174] => {
    "msg": "device QFX10K2-174 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX10K2-175] => {
    "msg": "device QFX10K2-175 is a qfx10002-36q running junos version 17.4R1-S1.9"
}

PLAY RECAP **************************************************************************************************************************************************************
QFX10K2-174                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-175                : ok=2    changed=0    unreachable=0    failed=0   
```
```
# ansible-playbook pb_print_junos_facts.yml --limit spine_switch

PLAY [Get Facts] ********************************************************************************************************************************************************

TASK [Retrieve information from devices running Junos] ******************************************************************************************************************
ok: [QFX10K2-180]
ok: [QFX10K2-175]
ok: [QFX10K2-174]
ok: [QFX10K2-178]
ok: [QFX10K2-181]

TASK [Print some facts] *************************************************************************************************************************************************
ok: [QFX10K2-174] => {
    "msg": "device QFX10K2-174 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX10K2-175] => {
    "msg": "device QFX10K2-175 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX10K2-178] => {
    "msg": "device QFX10K2-178 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX10K2-180] => {
    "msg": "device QFX10K2-180 is a qfx10002-36q running junos version 17.4R1-S1.9"
}
ok: [QFX10K2-181] => {
    "msg": "device QFX10K2-181 is a qfx10002-36q running junos version 17.4R1-S1.9"
}

PLAY RECAP **************************************************************************************************************************************************************
QFX10K2-174                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-175                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-178                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-180                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-181                : ok=2    changed=0    unreachable=0    failed=0   

```
```
# ansible-playbook pb_print_junos_facts.yml --limit qfx5100-48s-6q

PLAY [Get Facts] ********************************************************************************************************************************************************

TASK [Retrieve information from devices running Junos] ******************************************************************************************************************
ok: [QFX5100-183]
ok: [QFX5100-186]

TASK [Print some facts] *************************************************************************************************************************************************
ok: [QFX5100-183] => {
    "msg": "device QFX5100-183 is a qfx5100-48s-6q running junos version 17.4R1-S1.9"
}
ok: [QFX5100-186] => {
    "msg": "device QFX5100-186 is a qfx5100-48s-6q running junos version 17.4R1-S1.9"
}

PLAY RECAP **************************************************************************************************************************************************************
QFX5100-183                : ok=2    changed=0    unreachable=0    failed=0   
QFX5100-186                : ok=2    changed=0    unreachable=0    failed=0   
```
## Generate yaml files for Jinja templates or ansible playbooks from Netbox API

The script [**generate_vars_for_jinja_from_netbox.py**](generate_vars_for_jinja_from_netbox.py) generates yaml files for jinja templates and ansible playbooks from Netbox API. 


There is no variable available: 
```
# ls host_vars
ls: cannot access 'host_vars': No such file or directory
```

Generate the variables: 
```
# python generate_vars_for_jinja_from_netbox.py
```
```
# ls host_vars
QFX10K2-174  QFX10K2-175  QFX10K2-178  QFX10K2-180  QFX10K2-181  QFX5100-183  QFX5100-186
```
```
# more host_vars/QFX10K2-181/vars_from_netbox_api.yml 
vars_from_netbox_api:
- address: 10.0.2.15/31
  interface: et-0/0/0
- address: 10.0.2.25/31
  interface: et-0/0/1
- address: 172.25.90.181/32
  interface: em0
```
```
# more host_vars/QFX10K2-178/vars_from_netbox_api.yml 
vars_from_netbox_api:
- address: 10.0.2.9/31
  interface: et-0/0/0
- address: 10.0.2.19/31
  interface: et-0/0/1
- address: 172.25.90.178/32
  interface: em0
```

## Use the yaml files generated from the Netbox API 

The playbook [**pb_check_interfaces.yml**](pb_check_interfaces.yml) uses: 
- the ansible inventory we generated from Netbox API
- the variables we generated from Netbox API 

```
# ansible-playbook pb_check_interfaces.yml

PLAY [check interface states] *******************************************************************************************************************************************

TASK [check if interfaces admin status is up] ***************************************************************************************************************************
ok: [QFX10K2-174] => (item={u'interface': u'em0', u'address': u'172.25.90.174/32'})
ok: [QFX10K2-175] => (item={u'interface': u'em0', u'address': u'172.25.90.175/32'})
ok: [QFX10K2-180] => (item={u'interface': u'et-0/0/0', u'address': u'10.0.2.13/31'})
ok: [QFX10K2-178] => (item={u'interface': u'et-0/0/0', u'address': u'10.0.2.9/31'})
ok: [QFX10K2-181] => (item={u'interface': u'et-0/0/0', u'address': u'10.0.2.15/31'})
ok: [QFX10K2-180] => (item={u'interface': u'et-0/0/1', u'address': u'10.0.2.23/31'})
ok: [QFX10K2-181] => (item={u'interface': u'et-0/0/1', u'address': u'10.0.2.25/31'})
ok: [QFX10K2-178] => (item={u'interface': u'et-0/0/1', u'address': u'10.0.2.19/31'})
ok: [QFX10K2-180] => (item={u'interface': u'em0', u'address': u'172.25.90.180/32'})
ok: [QFX10K2-181] => (item={u'interface': u'em0', u'address': u'172.25.90.181/32'})
ok: [QFX10K2-178] => (item={u'interface': u'em0', u'address': u'172.25.90.178/32'})

TASK [check if interfaces op status is up] ******************************************************************************************************************************
ok: [QFX10K2-174] => (item={u'interface': u'em0', u'address': u'172.25.90.174/32'})
ok: [QFX10K2-175] => (item={u'interface': u'em0', u'address': u'172.25.90.175/32'})
ok: [QFX10K2-178] => (item={u'interface': u'et-0/0/0', u'address': u'10.0.2.9/31'})
ok: [QFX10K2-181] => (item={u'interface': u'et-0/0/0', u'address': u'10.0.2.15/31'})
ok: [QFX10K2-180] => (item={u'interface': u'et-0/0/0', u'address': u'10.0.2.13/31'})
ok: [QFX10K2-178] => (item={u'interface': u'et-0/0/1', u'address': u'10.0.2.19/31'})
ok: [QFX10K2-180] => (item={u'interface': u'et-0/0/1', u'address': u'10.0.2.23/31'})
ok: [QFX10K2-181] => (item={u'interface': u'et-0/0/1', u'address': u'10.0.2.25/31'})
ok: [QFX10K2-178] => (item={u'interface': u'em0', u'address': u'172.25.90.178/32'})
ok: [QFX10K2-180] => (item={u'interface': u'em0', u'address': u'172.25.90.180/32'})
ok: [QFX10K2-181] => (item={u'interface': u'em0', u'address': u'172.25.90.181/32'})

PLAY RECAP **************************************************************************************************************************************************************
QFX10K2-174                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-175                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-178                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-180                : ok=2    changed=0    unreachable=0    failed=0   
QFX10K2-181                : ok=2    changed=0    unreachable=0    failed=0   
```

The playbook [**pb_render_template.yml**](pb_render_template.yml) renders the template [**template.j2**](template.j2) using the yaml files built from Netbox API. It saves the generated files in the directory [**render**](render).   

```
# more pb_render_template.yml 
---
 - name: create a directory 
   hosts: localhost
   gather_facts: no
   
   tasks:
    
   - name: create the directory render
     file: 
        path: "{{playbook_dir}}/render" 
        state: directory

 - name: render junos configuration template using vars from Netbox API
   hosts: juniper
   connection: local
   gather_facts: no

   tasks:

    - name: render junos configuration template using vars from Netbox API
      template: 
        src: "{{ playbook_dir }}/template.j2" 
        dest: "{{ playbook_dir }}/render/{{ inventory_hostname }}.conf"

```
```
# more template.j2 
interfaces {
{% for item in vars_from_netbox_api %}
    {{ item.interface }} {
        unit 0 {
            family inet {
                address {{ item.address }};
            }
        }
    }
{% endfor %}
}
protocols {
    lldp {
{% for item in vars_from_netbox_api %}
        interface "{{ item.interface }}";
{% endfor %}
    }
}

```
```
# ls render
ls: cannot access 'render': No such file or directory
```
```
# ansible-playbook pb_render_template.yml 

PLAY [create a directory] ***********************************************************************************************************************************************

TASK [create the directory render] **************************************************************************************************************************************
changed: [localhost]

PLAY [render junos configuration template using vars from Netbox API] ***************************************************************************************************

TASK [render junos configuration template using vars from Netbox API] ***************************************************************************************************
changed: [QFX10K2-181]
changed: [QFX10K2-174]
changed: [QFX10K2-180]
changed: [QFX10K2-175]
changed: [QFX10K2-178]
changed: [QFX5100-183]
changed: [QFX5100-186]

PLAY RECAP **************************************************************************************************************************************************************
QFX10K2-174                : ok=1    changed=1    unreachable=0    failed=0   
QFX10K2-175                : ok=1    changed=1    unreachable=0    failed=0   
QFX10K2-178                : ok=1    changed=1    unreachable=0    failed=0   
QFX10K2-180                : ok=1    changed=1    unreachable=0    failed=0   
QFX10K2-181                : ok=1    changed=1    unreachable=0    failed=0   
QFX5100-183                : ok=1    changed=1    unreachable=0    failed=0   
QFX5100-186                : ok=1    changed=1    unreachable=0    failed=0   
localhost                  : ok=1    changed=1    unreachable=0    failed=0   
```
```
# ls render
QFX10K2-174.conf  QFX10K2-175.conf  QFX10K2-178.conf  QFX10K2-180.conf  QFX10K2-181.conf  QFX5100-183.conf  QFX5100-186.conf
```
```
# more render/QFX10K2-181.conf 
interfaces {
    et-0/0/0 {
        unit 0 {
            family inet {
                address 10.0.2.15/31;
            }
        }
    }
    et-0/0/1 {
        unit 0 {
            family inet {
                address 10.0.2.25/31;
            }
        }
    }
    em0 {
        unit 0 {
            family inet {
                address 172.25.90.181/32;
            }
        }
    }
}
protocols {
    lldp {
        interface "et-0/0/0";
        interface "et-0/0/1";
        interface "em0";
    }
}
```
## Delete Netbox configuration with automation
The script [**delete_netbox_configuration.py**](delete_netbox_configuration.py) delete the Netbox configuration: 
   - all tenants
   - all sites
   - all device-roles
   - all device-types
   - all platforms
   - all interface connections
   - all devices
   - all prefixes-roles
   - all prefixes
   - all ip addresses
```
$ python delete_netbox_configuration.py 
```

## Continuous integration with Travis CI

There is a github webhook with Travis CI
The syntax of the python scripts and ansible playbooks in this repository are tested automatically by Travis CI. 
The files [**.travis.yml**](.travis.yml) and [**requirements.txt**](requirements.txt) at the root of this repository are used for this.  

Here's the last build status
[![Build Status](https://travis-ci.org/ksator/netbox_automation.svg?branch=master)](https://travis-ci.org/ksator/netbox_automation)


## Looking for more automation solutions

https://github.com/ksator?tab=repositories  
https://gitlab.com/users/ksator/projects  
https://gist.github.com/ksator/  

