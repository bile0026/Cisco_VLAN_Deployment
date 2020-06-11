# Cisco_VLAN_Deployment
VLAN Deployment with Source and Target Vlan

This is a project for deploying vlans to Cisco devices with Ansible. This playbook will need to be updated with source and target vlan numbers for your case. It will take into account things like trunking the new vlan, changing access ports from a current vlan to a new vlan, and making sure the Layer2 vlan object exists. You can enable or disable these functions by commenting out the specific playbooks in the ```roles/Deploy_Vlan/tasks/main.yml``` file.

The current production version is ```Prod/deploy_vlan.yml```. There are a few things that will need to be updated for your intended results. These are outlined below. Deploy_Vlans has been converted to a role, so there's some additional considerations to take into place.

# Requirements
A couple of requirements for this role need to be installed along with ansible.

- [ ] Install Python3 if it is not already installed ```sudo apt/yum install python3``` (Most distributions include this by default now).
- [ ] The pyats/genie applications this playbook uses are required to be run in a python virtual environment.
    - [ ] Create a working directory with ```mkdir <directory_name>```
    - [ ] Cd into the new directory ```cd <directory_name>```
    - [ ] Run ```Python3 -m venv .venv``` (This creates the virtual environment in your current directory)
    - [ ] Run ```source .venv/bin/activate``` to place your session into the Python virtual environment. (You can use this same command in the future to place you back into the virtual environment for future work)
- [ ] pip3 needs to be installed inside the virtual environment. ```sudo apt/yum install python3-pip```
- [ ] Use the requirements file included in this repository to install the necessary prerequisites. ```sudo pip3 -r requirements.txt```

Once your prerequisites are taken care of, you can move on to prepping the playbook.

# Files to Change

```
# roles/Deploy_Vlan/defaults/main.yml
target_vlan: <new_vlan_number>
target_vlan_name: <vlan_name>
source_vlan: <current_vlan_number>
trunking_vlan: <vlan_to_identify_trunks>

# deploy_vlan.yml
hosts: <hosts,groupnames>
```

# Variables and explanations:

* hosts: Devices you want to run this playbook against. Make sure they are in your ansible inventory file.
* gather_facts: This will need to be set to yes, as the tasks rely on some of this information.
* connection: How to connect to the device.
* target_vlan: This is the variable for the new vlan that you would like to change source_vlan to.
* target_vlan_name: Name for the new L2 vlan object.
* source_vlan: The existing vlan you are looking to update.
* trunking_vlan: This is the vlan that will be used to determine which trunk links to add the new vlan to.
Make sure this vlan is trunked on all existing interfaces you want the target_vlan added to.
* spanning_tree_command: Used by the parser to know which vlan to check spanning-tree information on.
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

    # bring in the genie parser plugin for processing show command results
    - name: Read in parse_genie role
      include_role:
        name: clay584.parse_genie

    # import deploy vlan role
    - name: Include deploy_vlan role
      import_role:
        name: Deploy_Vlan

```

# Running the playbook
Use ```ansible-playbook <filename>``` as the bare-bones command to run the file. You may need additional parameters based on your situation and your ansible.cfg file.

I have included a sample ansible.cfg, and a sample hosts file with some of the options I use. By default ansible will look for these exactly named files in your current directory, so you can just modify these to run this playbook with minimal additional parameters.

Running ```ansible-playbook deploy_vlan.yml``` will run with the user ansible (based on the ansible.cfg file), and will then prompt for a password. Enter the SSH password and you're off to the races.

There are a number of other options you can use with ```ansible-playbook``` like -u to specify a different username (or edit ansible_ssh_user in ansible.cfg). Also, -k to force prompt for a password if you have SSH keys in your path. You can also specify ansible_ssh_key_path in your ansible.cfg file if you want to use key authentication to your devices.

--check is an extremely valuable paramater for ansble-playbook and I often use it as one of my final checks before going to prod. It will run the playbook in "testing" mode where it won't actually change any configurations, it will just tell you "what would happen" if you run it (at least as close as it can).

# Disclaimer
Please please please do your own testing with this in a lab prior to production, and make sure you understand what this playbook is doing. I have done as much testing in my own environment as I can come up with, but it's nearly impossible to account for all scenarios. I can not be held responsible for any damage this might do. I have attempted to write this with as much itempotence as possible, but you should still always test it yourself first. 
