# Target Definition

Get Ansible Facts for IOS XE, NX OS and ASA.

## Issues

For ASA is no official facts module in place for now. But there is one as pull request on [github](https://github.com/ansible/ansible/pull/37298)
Which need to copied to [python_lib_path]/site-packages/ansible/modules/network/asa to be usable.
The ASAv version in the VIRL lab have a bug that a as privilege 15 configured user has privilege 1 after login.
With the playbook 01_check_privilege we can check the current privilege level for ASA. We can use the "become: yes" and "become_mode: enable" to escalte the privilege level to privilege 15. The enable password need to save in the inventory file in the variable ansible_become_pass.
The variables ansible_become and ansible_become_mode did not work with asa_facts.