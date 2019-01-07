# Labsetup

## Ansible Host
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

## VIRL host:
- 8×Intel(R) Core(TM) i7-6770HQ CPU @ 2.60GHz
- 32GB RAM
- 450GB SSD HD

## VIRL lab:
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
