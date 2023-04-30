# -*- mode: ruby -*-
# vi: set ft=ruby :

def stdopt(
  config, hostname:, cpus: 1, memory_mb: 1024, box: "ubuntu/jammy64"
)
  config.vm.box = box
  config.vm.hostname = hostname

  config.ssh.forward_agent = true

  config.vm.synced_folder ".", "/host_share"

  config.vm.provider "virtualbox" do |vbox|
    vbox.linked_clone = true
    vbox.cpus = cpus
    vbox.memory = memory_mb
    # Force allocation of all specified VM memory at start. Change the 1 below
    # to 0 to allow for "as needed allocation".
    vbox.customize ["setextradata", :id, "VBoxInternal/RamPreAlloc", 1]
  end
end

def stdopt_ansible(ansible, verbose: 'v')
  ansible.compatibility_mode = "2.0"
  ansible.verbose = verbose
  ansible.extra_vars = {
    # It's 2023 so we always use Python 3.
    'ansible_python_interpreter': '/usr/bin/python3',
  }
end

def find_playbook(fname)
  return File.join(ENV.fetch('PWD'), 'ansible', fname)
end

def opt_network(
  config, network_name:, ip:, forwarded_ports: {}
)
  config.vm.network :private_network, name: network_name, ip: ip

  # Set up the port forwarding.
  forwarded_ports.each do |guest_port, host_port|
    config.vm.network "forwarded_port", guest: guest_port, host: host_port
  end
end

def provision_timezone(config, tz:, extra_vars: {})
  config.vm.provision "tz", type: "ansible" do |ansible|
    stdopt_ansible(ansible)
    ansible.playbook = find_playbook('tz.yml')
    ansible.extra_vars = ansible.extra_vars.merge({
      'tz': tz,
    }).merge(extra_vars)
  end
end

def provision_app_client(config, extra_vars: {})
  config.vm.provision "app-client", type: "ansible", run: "always" do |ansible|
    stdopt_ansible(ansible)
    ansible.playbook = find_playbook('ssh-tunnel.yml')
    ansible.extra_vars = ansible.extra_vars.merge(extra_vars)
  end
end

def provision_host(config, name:, playbook:, extra_vars: {})
  config.vm.provision name, type: "ansible", run: "always" do |ansible|
    stdopt_ansible(ansible)
    ansible.playbook = find_playbook(playbook)
    ansible.extra_vars = ansible.extra_vars.merge(extra_vars)
  end
end

Vagrant.configure("2") do |config|
  config.vm.define "app-client" do |app_client|
    stdopt(app_client, hostname: "ssh-demo-app-client", memory_mb: 1024)
    opt_network(app_client, network_name: "vboxnet0", ip: "192.168.56.10")

    provision_timezone(app_client, tz: 'US/Eastern')
    provision_host(app_client, name: "app-client", playbook: "app-client.yml")
  end

  config.vm.define "client-ssh-sidecar" do |client_ssh_sidecar|
    stdopt(
      client_ssh_sidecar,
      hostname: "ssh-demo-client-ssh-sidecar",
      memory_mb: 1024,
    )
    opt_network(
      client_ssh_sidecar,
      network_name: "vboxnet0",
      ip: "192.168.56.20",
    )

    provision_timezone(client_ssh_sidecar, tz: 'US/Eastern')
    provision_host(
      client_ssh_sidecar,
      name: "client-ssh-sidecar",
      playbook: "ssh-sidecar.yml",
    )
  end

  config.vm.define "server-ssh-sidecar" do |server_ssh_sidecar|
    stdopt(
      server_ssh_sidecar,
      hostname: "ssh-demo-server-ssh-sidecar",
      memory_mb: 1024,
    )
    opt_network(
      server_ssh_sidecar,
      network_name: "vboxnet0",
      ip: "192.168.56.30",
    )

    provision_timezone(server_ssh_sidecar, tz: 'US/Eastern')
    provision_host(
      server_ssh_sidecar,
      name: "server-ssh-sidecar",
      playbook: "ssh-sidecar.yml",
    )
  end

  config.vm.define "app-server" do |app_server|
    stdopt(app_server, hostname: "ssh-demo-app-server", memory_mb: 1024)
    opt_network(app_server, network_name: "vboxnet0", ip: "192.168.56.40")

    provision_timezone(app_server, tz: 'US/Eastern')
    provision_host(app_server, name: "app-server", playbook: "app-server.yml")
  end
end
