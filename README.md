# Cisco_VLAN_Deployment
VLAN Deployment with Source and Target Vlan

This is a project for deploying and changing access vlans to Cisco devices with Ansible. This playbook will need to be updated with source and target vlan numbers for your case. It will take into account things like trunking the new vlan, changing access ports from a current vlan to a new vlan, and making sure the Layer2 vlan object exists. 

# Notes
* Updated to no longer require the genie parser. All that's needed is the cisco.ios collection!

# Requirements
A couple of requirements for this role need to be installed along with ansible.

- [ ] Install cisco.ios collection `ansible-galaxy collection install cisco.ios`

Once your prerequisites are taken care of, you can move on to prepping the playbook.

# Variables to Change

```
# roles/Deploy_Vlan/defaults/main.yml
target_vlan: <new_vlan_number>
target_vlan_string: <new vlan number in string>
target_vlan_name: <vlan_name>
source_vlan: <current_vlan_number>
trunk_native_vlan: <vlan_to_identify_trunks>
alt_trunk_native_vlan: <second native vlan to identify trunks>

# deploy_vlan.yml
hosts: <hosts,groupnames>
```

# Variables and explanations:

* hosts: Devices you want to run this playbook against. Make sure they are in your ansible inventory file.
* gather_facts: This will need to be set to yes, as the tasks rely on some of this information.
* connection: How to connect to the device.
* target_vlan: This is the variable for the new vlan that you would like to change source_vlan to.
* target_vlan_string: Target vlan in string format for identifying if it's already on a trunk link.
* target_vlan_name: Name for the new L2 vlan object.
* source_vlan: The existing vlan you are looking to update.
* trunk_native_vlan: This is the vlan that will be used to determine which trunk links to add the new vlan to.
Make sure this vlan is the native vlan on all existing interfaces you want the target_vlan added to.
* alt_trunk_native_vlan: secondary option for native vlans.
  - could have additional if added as additional variables and added or statements to when: on set trunk interfaces task.
* ansible_network_os: What operating system the device has (ios, junos, nxos, etc...)

Also update the ```hosts:``` variable at the beginning of the file with the hosts you want to run this against, or groups that you want to run against in your Ansible inventory file.

# Sample Playbook
```
---
- name: Deploy new vlan
  hosts: all
  gather_facts: true
  connection: network_cli

  tasks:
  # create a config backup prior to changes
    - name: Save device configuration
      ios_config:
        backup: true

    # import deploy vlan role
    - name: Include cisco_deploy_vlan role
      import_role:
        name: roles/cisco_deploy_vlan

```

# Running the playbook
Use ```ansible-playbook <filename>``` as the bare-bones command to run the file. You may need additional parameters based on your situation and your ansible.cfg file.

I have included a sample ansible.cfg, and a sample hosts file with some of the options I use. By default ansible will look for these exactly named files in your current directory, so you can just modify these to run this playbook with minimal additional parameters.

Running ```ansible-playbook deploy_vlan.yml``` will run with the user ansible (based on the ansible.cfg file), and will then prompt for a password. Enter the SSH password and you're off to the races.

There are a number of other options you can use with ```ansible-playbook``` like -u to specify a different username (or edit ansible_ssh_user in ansible.cfg). Also, -k to force prompt for a password if you have SSH keys in your path. You can also specify ansible_ssh_key_path in your ansible.cfg file if you want to use key authentication to your devices.

--check is an extremely valuable paramater for ansble-playbook and I often use it as one of my final checks before going to prod. It will run the playbook in "testing" mode where it won't actually change any configurations, it will just tell you "what would happen" if you run it (at least as close as it can).

# Disclaimer
Please please please do your own testing with this in a lab prior to production, and make sure you understand what this playbook is doing. I have done as much testing in my own environment as I can come up with, but it's nearly impossible to account for all scenarios. I can not be held responsible for any damage this might do. I have attempted to write this with as much itempotence as possible, but you should still always test it yourself first. 
