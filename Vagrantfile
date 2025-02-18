# -*- mode: ruby -*-
# vi: set ft=ruby :

DEFAULT_MACHINE = {
  :domain => 'internal',
  :box => 'rockylinux/9/v4.0.0',
  :cpus => 1,
  :memory => 1024,
  :networks => {},
  :intnets => {},
  :forwarded_ports => [],
}

MACHINES = {
  :'router1' => {
    :intnets => {
      :'net1' => { auto_config: false },
      :'r1-r2' => { auto_config: false },
      :'r1-r3' => { auto_config: false },
    },
  },
  :'router2' => {
    :intnets => {
      :'net2' => { auto_config: false },
      :'r1-r2' => { auto_config: false },
      :'r2-r3' => { auto_config: false },
    },
  },
  :'router3' => {
    :intnets => {
      :'net3' => { auto_config: false },
      :'r1-r3' => { auto_config: false },
      :'r2-r3' => { auto_config: false },
    },
  },
}

Vagrant.configure("2") do |config|
  MACHINES.each do |host_name, host_config|
    host_config = DEFAULT_MACHINE.merge(host_config)
    config.vm.define host_name do |host|
      host.vm.box = host_config[:box]
      host.vm.host_name = host_name.to_s + '.' + host_config[:domain].to_s

      host.vm.provider :virtualbox do |vb|
        vb.cpus = host_config[:cpus]
        vb.memory = host_config[:memory]
      end

      host_config[:networks].each do |network_type, network_args|
        host.vm.network(network_type, **network_args)
      end
      host_config[:intnets].each do |name, intnet|
        intnet[:virtualbox__intnet] = name.to_s
        host.vm.network(:private_network, **intnet)
      end
      host_config[:forwarded_ports].each do |forwarded_port|
        host.vm.network(:forwarded_port, **forwarded_port)
      end

      if MACHINES.keys.last == host_name
        host.vm.provision :ansible do |ansible|
          ansible.playbook = 'playbook.yml'
          ansible.limit = 'all'
          ansible.compatibility_mode = '2.0'
          ansible.raw_arguments = ['--diff']
        end
      end
    end
  end
end
