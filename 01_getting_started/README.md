# Labsetup

## Ansible host setup
### virtual python environment
| Package | Version |
| --- | --- |
| ansible  | 2.7.5 |
| asn1crypto | 0.24.0 |
| bcrypt | 3.1.5 |
| cffi | 1.11.5 |
| cryptography | 2.4.2 |  
| idna |2.8 |
| Jinja2 | 2.10 |
| MarkupSafe | 1.1.0 |
| paramiko | 2.4.2 |
| pip | 18.1 |
| pyasn1 | 0.4.5 | 
| pycparser | 2.19 |   
| PyNaCl | 1.3.0 |
| PyYAML | 3.13 |
| setuptools | 40.6.3 |
| six | 1.12.0 |
| wheel | 0.32.3| 

## VIRL Setup
### VIRL host
- 8×Intel(R) Core(TM) i7-6770HQ CPU @ 2.60GHz
- 32GB RAM
- 450GB SSD HD

### VIRL lab
| Node | Subtype | Management IPs |
| --- | --- | --- |
| lab-acc-asa-01-pri | ASAv | 172.16.1.13 |
| lab-acc-asa-01-sec | ASAv | 172.16.1.14 |
| lab-acc-sw-01 | CSR1000v | 172.16.1.12 |
| lab-core-sw-01 | NX-OSv | 172.16.1.10 |
| lab-core-sw-02 | NX-OSv | 172.16.1.11 |
| lab-mgmt-asa-01 | ASAv | 172.16.1.15 |

My NUC has only one NIC, so I use OpenVPN to connect to Managment Network.
I configured FLAT-1 as Managment Network, to connect direct from my ansible host to the network devices.

## Ansible lab setup

### Connect to devices with ssh
At first we need to connect to all devices with ssh. To ensure that ssh is correct configure. 
Cisco VIRL AutoNetkit ssh default configurations are not combatible to actual and hardend ssh client configurations. So we need to log in via console port and need to change the ssh server configuation.

#### Issues IOS XE
```
ssh cisco@172.16.1.12
ssh_dispatch_run_fatal: Connection to 172.16.1.12 port 22: Invalid key length
```
```
lab-acc-sw-01(config)#crypto key generate rsa modulus 2048                      
% You already have RSA keys defined named lab-acc-sw-01.virl.info.              
% They will be replaced.                                                        
                                                                                
% The key modulus size is 2048 bits                                             
% Generating 2048 bit RSA keys, keys will be non-exportable...                  
[OK] (elapsed time was 1 seconds)                                               
                                                                                
lab-acc-sw-01(config)#  
```
#### Issues ASA
```
ssh cisco@172.16.1.13
Unable to negotiate with 172.16.1.13 port 22: no matching key exchange method found. Their offer: diffie-hellman-group1-sha1
```
```
lab-acc-asa-01-pri(config)# ssh key-exchange group dh-group14-sha1 
```
```
ssh cisco@172.16.1.13
ssh_dispatch_run_fatal: Connection to 172.16.1.13 port 22: Invalid key length
```
```
lab-acc-asa-01-pri(config)# crypto key generate rsa modulus 2048                                                                                                                                                                                                                                                                                                        
WARNING: You have a RSA keypair already defined named <Default-RSA-Key>.                                                                                                                                                                                                                                                                                                
                                                                                                                                                                                                                                                                                                                                                                        
Do you really want to replace them? [yes/no]: yes                                                                                                                                                                                                                                                                                                                       
Keypair generation process begin. Please wait...                                                                                                                                                                                                                                                                                                                        
lab-acc-asa-01-pri(config)# 
```   

### Inventory
Before we can connect the first time with ansible to the device we need to create the inventory file. To connect we will use network_cli which support the most network os, but need *ansible_network_os* as an additional parameter. For this we create the file *inventory*
```
# Group of all Cisco ASA
[asa]
lab-acc-asa-01-pri ansible_host=172.16.1.13
lab-acc-asa-01-sec ansible_host=172.16.1.14
lab-mgmt-asa-01 ansible_host=172.16.1.15

# Group of all Cisco IOS XE
[iosxe]
lab-acc-sw-01 ansible_host=172.16.1.12

# Group of all Cisco NX-OS
[nxos]
lab-core-sw-01 ansible_host=172.16.1.10
lab-core-sw-02 ansible_host=172.16.1.11   

# Define variables for all devices
[all:vars]
ansible_user=cisco
ansible_ssh_pass=cisco

#Define group specific variables
[asa:vars]
ansible_network_os=asa

[iosxe:vars]
ansible_network_os=ios

[nxos:vars]
ansible_network_os=nxos
```



## Connect to devices with ansible
We can connect to the CLI of the device direct from the ansible host terminal. For this we need to specify *where* and *how*, we like to connect and *what* we like to do.
```
ansible all -i inventory -c network_cli -m cli_command -a "command='show version | i up'"
```
*ansible* start ansible from terminal
*all* reference *all* devices in the inventory, we can use host name that are specify in the inventory file, too (like: **lab-acc-asa-01-pri**) or group names (like: **asa**)
*-i inventory* specify which inventory file ansible will use
*-c network_cli* specify that we like to connect via the connection type [network_cli](https://docs.ansible.com/ansible/latest/plugins/connection/network_cli.html?highlight=network_cli) to device
*-m cli_command -a "command='show version | i up'"* specify that ansible will use the module [cli_command](https://docs.ansible.com/ansible/latest/modules/cli_command_module.html) to perform the command **'show version | i up'** at the device.


```
ansible all -i lab-01.inventory -c network_cli -m cli_command -a "command='show version | i up'"                            
lab-acc-asa-01-pri | SUCCESS => {
    "changed": false,                                                                                                                                                                     
    "stdout": "Config file at boot was \"startup-config\"\nlab-acc-asa-01-pri up 14 hours 28 mins",                                                                                       
    "stdout_lines": [
        "Config file at boot was \"startup-config\"",
        "lab-acc-asa-01-pri up 14 hours 28 mins"                                                                                                                                          
    ]
}
lab-acc-asa-01-sec | SUCCESS => {
    "changed": false,
    "stdout": "Config file at boot was \"startup-config\"\nlab-acc-asa-01-sec up 23 hours 41 mins",                                                                                       
    "stdout_lines": [
        "Config file at boot was \"startup-config\"",                                                                                                                                     
        "lab-acc-asa-01-sec up 23 hours 41 mins"
    ]
}
lab-acc-sw-01 | SUCCESS => {
    "changed": false,
    "stdout": "Technical Support: http://www.cisco.com/techsupport\nlab-acc-sw-01 uptime is 23 hours, 41 minutes",                                                                        
    "stdout_lines": [
        "Technical Support: http://www.cisco.com/techsupport",
        "lab-acc-sw-01 uptime is 23 hours, 41 minutes"
    ]
}

```