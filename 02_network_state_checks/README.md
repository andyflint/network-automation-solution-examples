# Create a report with information they are collected from network devices

## Target Definition

Get information of the network state for IOS XE, NX OS and ASA. For the network state we will create a file which include hostname, managment IP, modell, firmware, a list of interfaces and IP addresses, routing table and layer2 topologie. Create a playbook which report the gathered information.
We will also collect the running configuration and save this in a file.

## Structure of document and project folder

- 01_ are playbooks which do prechecks
- 02_ are playbooks which get information and save them
- 03_ handle with saved information to generate reports

## 01 Pre Checks

### 01_check_lldp

Check if LLDP is enable, if not it will be enabled.

### 01_check_priv

Check if privilege level is 15 after login to ASA, if not ist configured auto-enable

## 02 Get Facts

[Networktocode](https://github.com/networktocode) publish TextFSM filter [templates](https://github.com/networktocode/ntc-templates/tree/master/templates) for different network device on there github account.
which can be used with [ntc_ansible](https://github.com/networktocode/ntc-ansible) to get structured information direct from the CLI. Follow the instruction details on the github site.
In our lab we will use this tool to get information from the networkd devices and save then for later reports.

### 02_get_facts

We will use the with ansible delivered modules for IOS and NXOS to get the facts (which include hostname, managment IP, modell, firmware, a list of interfaces and IP addresses) from the network devices.
All gathered facts will be saved in folder *saved_state* in file with suffix *_facts*.

#### Issues

For ASA is no official facts module in place for now. But there is one as pull request on [github](https://github.com/ansible/ansible/pull/37298)
Which need to copied to [python_lib_path]/site-packages/ansible/modules/network/asa to be usable.
The ASAv version in the VIRL lab have a bug that a as privilege 15 configured user has privilege 1 after login.
With the playbook 01_check_privilege we can check the current privilege level for ASA. We can use the "become: yes" and "become_mode: enable" to escalte the privilege level to privilege 15. The enable password need to save in the inventory file in the variable ansible_become_pass.
The variables ansible_become and ansible_become_mode did not work with asa_facts.

### 02_get_firmware

Gather information from *show version* with *ntc_show_command*.
All gathered facts will be saved in folder *saved_state* in file with suffix *_firmware*.

### 02_get_lldp_neighbors

Gather information from *show lldp neighbore*"* with *ntc_show_command*.
All gathered facts will be saved in folder *saved_state* in file with suffix *_lldp_neighbors*.

## Create Reports

### 03_report_facts_interfaces

Create a report based on the information from *02_get_facts* and **02_get_firmware** as an HTML table and save it the folder *reports* with suffix *_interfaces.html*.
I use a CSS framework for a nice looking report like: [Milligram](https://milligram.io/)

At first the saved state information will be loaded with the **include_vars** module and will then rentered with the *template* module.

### 03_report_neighbors

Create a network diagram based on information from *03_report_neighbors* as *dot* file and render them to a *SVG*.
I create two different template one that use the *record shape* to group the interfaces to a devices and one which work with subgraphes for grouping.