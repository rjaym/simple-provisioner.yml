
Using base.yml
==============

## Motivation

I want to use ansible-playbook on a host that is specified in a hosts file, but 
doesn't yet physically exist.


## Goals

- Import into VirtualBox a minimally configured 'base' machine which has: 
  	- An extra network interface with static IP address (e.g. 10.254.254.254)
	- A known username and password (e.g. 'base'/'base')
	- An OpenSSH-server installation using the default port 22
- 'Morph' the base machine using details from an extra-vars-specified target
	- Create an ansible user matching hostvars[target].ansible_user
	- Change the IP address/host to match hostvars[target].ansible_host
	- Add a known SSH pubkey provided via the extra-vars
	- Change the openssh-server's port to match hostvars[target].ansible_port
- Reset the network adapter, and continue the parent playbook without rebooting

With the above, the process of provisioning a new machine turns into:

1. Import and boot a base machine using the VirtualBox Manager
2. ansible-playbook blah.yml --extra-vars="target=example pubkey=??/id_rsa.pub"


## Benefits when using this approach

- Simplify configuration by using details from the hosts file entry itself, 
  rather than manually 'hard-coding' in that information into a playbook
- No need to repeatedly install an operating system for each new machine
- Adding new machines is reduced to simply adding a host to your hosts file
- Using the secondary host-only adapter provides development-friendly local 
  access to the target machine while still keeping it behind NAT for WAN access


## Setting up VirtualBox and creating your own 'base' machine
- Add a host-only network via VirtualBox > File > Preferences > Network
	- IPv4 Address: 10.0.0.1
	- Netmask: 255.0.0.0
	- Leave everything else blank, also no DHCP server needed
	- IPv6 Network Mask Length: 0
- Create a new Ubuntu VM, and add a secondary host-only network adapter that 
  uses the host-only network you created above (this will be eth1)
- Download and install Ubuntu server into a VirtualBox VM
	- Use username "base", password "base"
	- Install OpenSSH-server
- sudo vi /etc/network/interfaces

```
auto eth1
iface eth1 inet static
address 10.254.254.254
netmask 255.0.0.0
# no gateway!
```

- Finally, power off the machine, and export it; this is the base machine

You should now be able to run the example.yml playbook from this directory: 
ansible-playbook -i hosts main.yml --extra-vars="target=example pubkey=/??/id_rsa.pub"

* Replace ?? with the path to your pubkey
* The above has been tested with Ansible 2.1 and VirtualBox 5.1.2


## Gotchas

- VirtualBox may disconnect both network cables when importing; just reconnect
- Ansible will complain about known_hosts on your first run, so just ssh in 
  manually with the following command: "ssh base@10.254.254.254"

