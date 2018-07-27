# nxos-vpc-facts

This Ansible role retrieves facts about the VPC configuration of a Cisco Nexus switch.  These facts can then be used to check the status before and after a maintance operation.

# Requirements

The inventory must be properly setup for the platforms being used from the [Ansible Network Modules](https://docs.ansible.com/ansible/latest/network/index.html).

# Example Playbook

To run the role to just collect the facts:
    - hosts: vpc_switches
      gather_facts: no
      tasks:
        - include_role:
            name: nxos-vpc-facts

These facts are make available from `show vpc brief`:
* nxos_vpc_domain_id
* nxos_vpc_role
* nxos_vpc_peer_status
* nxos_vpc_keepalive_status
* nxos_vpc_config_consistency_status
* nxos_vpc_vlan_consistency_status
* nxos_vpc_type2_consistency_status

These facts can then be checked in the playbook:

    - hosts: vpc_switches
      gather_facts: no
      tasks:
        - include_role:
            name: nxos-vpc-facts

        - debug:
            msg: 'VPC Domain ID: {{ nxos_vpc_domain_id }}'

        - debug:
            msg: 'VPC Role: {{ nxos_vpc_role }}'

        - debug:
            msg: '*WARNING* The following VPCs are down: {{ nxos_vpc_down_list | join(",") }}'
          when: nxos_vpc_down_list

        - name: Check VPC Status
          assert:
            that:
              - nxos_vpc_peer_status is search('formed ok')
              - nxos_vpc_keepalive_status is search('alive')
              - nxos_vpc_config_consistency_status == 'success'
              - nxos_vpc_vlan_consistency_status == 'success'
              - nxos_vpc_type2_consistency_status == 'success'
            msg: "VPC must be consisteany and have a reachable peer"

# License

GPL-3

# Author Information

Steven Carter
