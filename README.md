# Firewall-Vagrant

This is a project to help me build out a lab to simulate a fully functional linux / unix firewall / GW. Each VM is defined in settings.yaml. Each VM will have an eth0, however that will only be used for management of the VMs by vagrant and in the case of the firewall, it will be used for internet access.

There are three vboxnet networks that are "private" and not routable to each other or the network. Three clients will connect to one of those networks using their eth1 port (default GW set to eth1 so that they can only get out through the firewall). The firewall controller connects to each of those networks using eth1, eth2, and eth3.

```
                  WAN / Public NIC
                 ────────┬────────
                         │
                         │
                         │
          ┌──────────────┴───────────────┐
          │             eth0             │
          │                              │
          │           firewall1          │
          │                              │
          │  eth1       eth2       eth3  │
          └───┬───────────┬─────────┬────┘
  vboxnet1    │           │         │  vboxnet3
 ─────┬───────┴───        │       ──┴───────┬───
      │             ─────┬┴────             │
      │        vboxnet2  │                  │
┌─────┴────┐        ┌────┴────┐         ┌───┴─────┐
│   eth1   │        │  eth1   │         │  eth1   │
│          │        │         │         │         │
│  office1 │        │ server1 │         │ family1 │
│          │        │         │         │         │
│   eth0   │        │  eth0   │         │  eth0   │
└─────┬────┘        └────┬────┘         └───┬─────┘
      │                  │                  │
      │                  │                  │
 ─────┴──────────────────┴──────────────────┴────
             Ports here so that Vagrant
                can manage VM / SSH
```

# Pre-requisites:
* Operating System: Mac OSX or Linux
* Supported Hypervisors
  * [Virtualbox](https://www.virtualbox.org/) >= 6
* [Vagrant](https://www.vagrantup.com/) >= 2.2.8


# Configuration
NOTE: Most configuration needs should be met in the settings.yaml file. You may also use ansible provisioning to deploy configurations by setting `ansible_provisioning: true` in settings.yaml

## Networking
NOTE: you may need to create three private networks in your virtualbox configuration: vboxnet1, vboxnet2, vboxnet3

## Firewall
in the `node_info` settings, you can configure the firewall (role: "firewall") with any number of "networks" and specify which private or public network you wish to attach them to.

## Clients
Using settings.yaml, you can add clients (role: "client") and place them on vboxnetX networks. You may configure them to use the IP address of the firewall as the default GW for the network it is on.

Clients will run a script when the VMs are first built that will remove the default GW from eth0 and assign it to eth1. These nodes will not have access to the internet, however vagrant can still manage them through eth0.

## Ansible
The current ansible configuration will deploy the following components to Rocky linux, Ubuntu, and Debian:
- Secure set of IPTables rules / configurations
- Bind9 server
- ISC DHCP server

# Building out lab for simulation
Run the following command to create the firewall and client VMs:

```
vagrant up
```

NOTE: First time run will take a while to download the vagrant box file. Once its downloaded and cached, the nodes will be up in roughly 5 minutes.

Run the following command to destroy the nodes:
```
vagrant destroy -f
```

Run the following to ssh into any of the nodes:
```
vagrant ssh <node_name>
```

example: `vagrant ssh firewall1`

# After VMs are built:
At this point, it is up to you to configure the firewall as you wish. As a goal, you should ensure the clients can safely get to the internet through the firewall and that the firewall can also manage routing and ACLs between networks.

The sky is the limit...

# Future Improvements
- Configurable vagrant boxes per VM
- Ability to introduce other OS as well as BSD based OS
- Ability to include windows clients

# License
Copyright 2021 Jennifer Reed

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this work except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
