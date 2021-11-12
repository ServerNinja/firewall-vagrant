# Loading Settings from settings.yml file
require 'yaml'
settings = YAML.load_file("settings.yml")

dictNodeInfo = settings['node_info']
defaultVagrantBox = settings['vagrant_box']

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
    end
  end
end