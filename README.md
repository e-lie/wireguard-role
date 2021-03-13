Multi Usecases Ansible Role for Wireguard Private Networking
============================================================

Flexible installation for wireguard private networking:
  - single or multiple relay servers
  - mulitple clients configurable using ansible or as external clients config files
  - usable several times to create multiple wireguard interfaces on different overlapping groups of machines

Usecases
--------

TODO

- server to server cluster network
- clients behind NAT with a relay server
- external clients not configurable using ansible
- any mix of the preceding

Be sure to read some wireguard doc to better understand what is happening (network usecases can be tricky). For example: [https://github.com/pirate/wireguard-doc`](https://github.com/pirate/wireguard-docs).

Wireguard Key Generation and Role Design
----------------------------------------

All configs and secret comes from inventory and code variables following Ansible execution best practices. So all the keys have to be generated manually and put in the inventory / ansible vault files. This design also allow easy and partial secret renewing.

#### Key generation

Locally on your ansible execution machine.
- Install wireguard manually.
- Generate a private key: `wg genkey > example.key && cat example.key`
- Generate the associated public key: `wg pubkey < example.key > example.key.pub && cat example.key.pub`
- Then fill up `wireguard_privkey` and `wireguard_pubkey` variables specific to a wireguard host (or external client).
- Ideally use some secret management to store the private key values. For example `ansible-vault edit wireguard-secrets.yml` and create `secret_myclient_wireguard_privkey` variable to store the value (see `ansible-vault` usage and principles).
- Remove the key files.
- Generate keys for the next hosts the same way.

Simple Example
----------------

#### `inventory.ini`

```ini
my_laptop_client ansible_connection=local # need to become root locally => most probably add --ask-become-pass option to ansible-playbook command
wg_server_host ansible_host=222.222.222.222 ansible_user=admin
wg_client_2 ansible_host=111.222.111.222 ansible_user=admin

[wireguard]
wg_server_host wireguard_ip_end="0.1" wireguard_privkey="{{ secret_wg1_privkey_wg_server_host }}" wireguard_pubkey="ArnEBKDmv/86PHLG4D7uUfh/p9WUwAng06p3ke4cgEc="
wg_client_2 wireguard_ip_end="0.2" wireguard_privkey="{{ secret_wg1_privkey_wg_client_2 }}" wireguard_pubkey="sc+27FiMaxukcWbHPoJvlofOC2T3BxcHPD3P6cqBDE8="
my_laptop_client wireguard_ip_end="0.5" wireguard_privkey="{{ secret_wg1_privkey_my_laptop }}" wireguard_pubkey="GfCP6BpaI8d/hgCHNBuNUl2LPuptjjYfDU+a7UhIR8="

[wireguard:vars]
wireguard_interface_state="present"
wireguard_interface_name="wg1"
wireguard_subnet="10.111." # choose a /16 subnet for the private network

[wireguard_servers]
wg_server_host wireguard_public_ip="111.122.133.144" wireguard_listen_port="51820"

[wireguard_clients]
my_laptop_client
wg_client_2
```

#### Example playbook

```yaml
- hosts: wireguard
  become: yes
  roles:
    - name: wireguard
      vars:
        wireguard_group_name: wireguard # Allow to apply role several times on several hosts group and vpn interfaces 
```

More Complete Example
---------------------

TODO

- two wireguard groups/interfaces wg1 et wg2
- multiple wg relay servers (to test / make work)
- group_vars: wg1.yml wg2.yml with external client list

Improvements TODO
-----------------

- define more formally the usecases
- create molecule testing scenarios
- ipv4 and subnet flexibility
- ipv6 ?
- create wireguard install scenarios on multiple linux plateform and ansible versions (with molecule)
- add params doc
- documentation of the multiple options
  - clients config
  - hosts file completion
  - ip-forward activation for relay servers
- link to good wireguard doc : https://github.com/pirate/wireguard-docs

Role Variables
--------------

TODO
See commented `defaults/main.yml`

