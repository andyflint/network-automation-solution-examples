# Create a report with information they are collected from network devices

## Target Definition

Get information of the network state for IOS XE, NX OS and ASA. For the network state we will create a file which include hostname, managment IP, modell, firmware, a list of interfaces and IP addresses, the routing table and the layer2 topologie and create a HTML report about the network.

## Structure of document and project folder

- 01_ are playbooks which do prechecks
- 02_ are playbooks which get information and save them
- 03_ handle with saved information to create reports

## 01 Get Facts

### Get Facts

We will use the with ansible delivered modules for IOS and NXOS to get the facts (which include hostname, managment IP, modell, firmware, a list of interfaces and IP addresses) from the network devices.

#### Issues

For ASA is no official facts module in place for now. But there is one as pull request on [github](https://github.com/ansible/ansible/pull/37298)
Which need to copied to [python_lib_path]/site-packages/ansible/modules/network/asa to be usable.
The ASAv version in the VIRL lab have a bug that a as privilege 15 configured user has privilege 1 after login.
With the playbook 01_check_privilege we can check the current privilege level for ASA. We can use the "become: yes" and "become_mode: enable" to escalte the privilege level to privilege 15. The enable password need to save in the inventory file in the variable ansible_become_pass.
The variables ansible_become and ansible_become_mode did not work with asa_facts.

### Get routing information

There is no module in place to get the routing information so we use *_command to the raw output from the CLI.

#### Structure routing information

[Networktocode](https://github.com/networktocode) publish TextFSM filter [templates](https://github.com/networktocode/ntc-templates/tree/master/templates) for different network device on there github account.
In our lab we can use:

- [cisco_asa_show_route.template](https://github.com/networktocode/ntc-templates/blob/master/templates/cisco_asa_show_route.template)
- [cisco_ios_show_ip_route.template](https://github.com/networktocode/ntc-templates/blob/master/templates/cisco_ios_show_ip_route.template)
- [cisco_nxos_show_ip_route.template](https://github.com/networktocode/ntc-templates/blob/master/templates/cisco_nxos_show_ip_route.template)

With [ntc_ansible](https://github.com/networktocode/ntc-ansible) we can get structured information direct from the CLI. Follow the instruction details on the github site.

### Get ping result

To do an ICMP reachabilty test we use the net_ping module.

#### Issue

There is implementation module net_ping for asa. We need to get the raw data with asa_command.

### Report Interface State

#### CSS framwork

We can use a CSS framework for a nice looking report like: [Milligram](https://milligram.io/t)