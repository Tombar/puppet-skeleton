# -*- mode: ruby -*-
# vi: set ft=ruby :

nodes = {
  'node0' => {:ip => '172.16.10.10', :memory => 512},
#  'node1' => {:ip => '172.16.10.11'},
#  'node2' => {:ip => '172.16.10.12'},
}
node_defaults = {
  :domain => 'dev',
  :memory => 384,
}

Vagrant.configure("2") do |config|
  config.vm.box     = "Debian-7.6.0-amd64-netboot"
  config.vm.box_url = "http://repo.crvt.net/vagrant-boxes/linux/Debian-7.6.0-amd64-netboot.box"

  config.vm.synced_folder 'puppet', '/opt/puppet'

  config.vm.provision :shell,
    :inline => 'exec /usr/local/bin/run-librarian'

  nodes.each do |node_name, node_opts|
    config.vm.define node_name do |node|
      node_opts = node_defaults.merge(node_opts)
      fqdn = "#{node_name}.#{node_opts[:domain]}"

      node.vm.hostname = fqdn

      if node_opts[:ip]
        node.vm.network(:private_network, :ip => node_opts[:ip])
      end

      node.vm.provider :virtualbox do |vb|
        modifyvm_args = ['modifyvm', :id]
        modifyvm_args << "--name" << fqdn
        if node_opts[:memory]
          modifyvm_args << "--memory" << node_opts[:memory]
        end
        # Isolate guests from host networking.
        modifyvm_args << "--natdnsproxy1" << "on"
        modifyvm_args << "--natdnshostresolver1" << "on"
        vb.customize(modifyvm_args)
      end

      config.vm.provision :shell,
        :inline => "exec /usr/local/bin/run-puppet #{node_name}.pp $@",
        :args   => '--verbose --summarize --environment development'

    end
  end
end
