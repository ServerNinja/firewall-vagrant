# Loading Settings from settings.yml file
require 'yaml'
settings = YAML.load_file("settings.yml")

ansibleProvisioning = settings['ansible_provisioning']

#if ansibleProvisioning
#  system("
#      if [ '#{ARGV[0]}' = 'up' ] || [ '#{ARGV[0]}' = 'provision' ]; then
#        ansible-galaxy collection install community.kubernetes -p provisioning/ansible_collections/
#      fi
#  ")
#end

dictNodeInfo = settings['node_info']
defaultVagrantBox = settings['vagrant_box']
defaultGuestOSType = settings['vagrant_guest_os'] || "linux"

ansibleFirewallGroup = []

Vagrant.configure(2) do |config|
  #Define the number of nodes to spin up

  dictNodeInfo.each_with_index do |(hostname), index|
    nodeInfo = dictNodeInfo[hostname]

    config.vm.define hostname do |node|
      node.vm.box = nodeInfo['vagrant_box'] || defaultVagrantBox
      node.vm.provider "virtualbox" do |vb|
        vb.memory = nodeInfo['memory'] || 1096
        vb.cpus = nodeInfo['cpus'] || 1
      end

      node.ssh.forward_agent = true

      node.vm.guest = nodeInfo['vagrant_guest_os'] || defaultGuestOSType
      node.vm.hostname = hostname

      # Loop through networks and add NICs
      nodeInfo['networks'].each do |net|
        if net['mode'] == "public"
          node.vm.network :public_network,
            ip: net['ip'],
            mac: net['mac'] || nil,
            netmask: net['netmask'] || "255.255.255.0"
        else
          node.vm.network :private_network,
            name: net['name'],
            ip: net['ip'],
            mac: net['mac'] || nil,
            netmask: net['netmask'] || "255.255.255.0"
        end
      end

      $script = ''
      # VM Provision Shell Script for client only (override default router to eth1)
      # - basically removes default GW to default NAT network set up by VirtualBox
      if nodeInfo['role'] == 'client'
        # use only the default router defined on the first private net
        defaultRouter = nodeInfo['networks'][0]['default_router']
        nameServers = nodeInfo['networks'][0]['nameservers']

        $script = <<-SCRIPT
ip route delete default
ip route add default via #{defaultRouter}
cat <<EOF>/etc/netplan/99-route-overrides.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4-overrides:
        use-routes: false
        use-dns: false
      nameservers:
        addresses: []
    eth1:
      gateway4: #{defaultRouter}
      nameservers:
        addresses: #{nameServers}
EOF
SCRIPT

        node.vm.provision :shell, :inline => $script
      end

      # Ansible stuff
      if nodeInfo['role'] == "firewall"
        ansibleFirewallGroup.append(hostname)
      end 

      if index == dictNodeInfo.keys.length() - 1 and ansibleProvisioning
        node.vm.provision "ansible" do |ansible|
          ansible.limit    = "all"
          ansible.verbose  = "vv"
          ansible.playbook = "provisioning/playbook.yml"
          ansible.galaxy_role_file = "provisioning/requirements.yaml"
          ansible.galaxy_command = 'ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path}'
          ansible.config_file = "provisioning/ansible.cfg"

          ansible.groups = {
            "firewall" => ansibleFirewallGroup,
          }
          ansible.extra_vars = {
            "fw_role_internet_gateway": true,
            "fw_wan_if": "eth0",
            "fw_office_if": "eth1",
            "fw_server_if": "eth2",
            "fw_family_if": "eth3",
            "fw_dmz_if": "eth4",
            "fw_office_net_enabled": true,
            "fw_server_net_enabled": true,
            "fw_family_net_enabled": true,
            "fw_ssh_from_wan_net_enabled": true,
            "fw_dhcp_server": true,
            "fw_dns_server": true,
            "fw_dynamic_dns": true,
            "fw_iptables_enabled": true,
            "fw_ipset_enabled": true,
            "fw_nat_enabled": true,
            "fw_network_internal_domain": "reedfamilyninjas.local",
            "fw_xbox_hosts": [{
              "name": "XBox Wired",
              "hostname": "xbox-wired",
              "mac": "2C:54:91:CD:5C:19",
              "ip": "192.168.58.49"
            }, {
              "name": "XBox Wireless",
              "hostname": "xbox-wireless",
              "mac": "2C:54:91:CD:5C:1B",
              "ip": "192.168.58.48"
            }],
            "fw_chromecast_hosts": [{
              "name": "Upstairs Chromecast",
              "hostname": "chromecast-upstairs",
              "mac": "3C:8D:20:02:30:DB",
              "ip": "192.168.58.47"
            }],
            "fw_firetv_hosts": [{
              "name": "Master Bedroom FireTV",
              "hostname": "firetv-master-bed",
              "mac": "A0:D0:DC:60:9C:22",
              "ip": "192.168.58.46"
            }],
            "fw_printer_hosts": [{
              "name": "Office Brother Printer",
              "hostname": "brother-mfc",
              "mac": "AC:D1:B8:79:6D:34",
              "ip": "192.168.57.49"
            }],
            "fw_server_net_services": [{
              "name": "Minecraft Bedrock",
              "ip": "192.168.1.45",
              "port": "19132",
              "protocol": "tcp"
            }],
          }
        end
      end 
    end
  end
end
