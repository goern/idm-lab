# -*- mode: ruby -*-
# vi: set ft=ruby :

clients = { 'atomic-host-1' => '192.168.1.44',
            'atomic-host-2' => '192.168.1.45',
            'atomic-host-3' => '192.168.1.46'
          }

Vagrant.configure(2) do |config|
  config.ssh.insert_key = false

  # check if the hostmanager plugin is installed
  unless Vagrant.has_plugin?("vagrant-hostmanager")
    raise 'vagrant-hostmanager is not installed! see https://github.com/smdahlen/vagrant-hostmanager'
  end

  # check if the reload plugin is installed
  unless Vagrant.has_plugin?("vagrant-reload")
    raise 'vagrant-reload is not installed! see https://github.com/aidanns/vagrant-reload'
  end

  if Vagrant.has_plugin?('vagrant-registration')
    config.registration.skip = true
  end

  # and configure the hostmanager plugin
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.include_offline = true

  # set some sane defaults for all VMs
  config.vm.provider :libvirt do |domain|
    domain.memory = 2048
    domain.cpus = 2
  end

  # provision and configure IdM service
  config.vm.define "idm-1" do |idm1|
    idm1.vm.box = "fedora/23-cloud-base"
    idm1.vm.box_check_update = true
    idm1.vm.hostname = "idm-1.goern.example.com"

    idm1.vm.synced_folder ".", "/home/vagrant/sync", disabled: true

    idm1.vm.provision "shell", path: "provision-scripts/common.sh"

    idm1.vm.provision "shell", inline: "sudo dnf install -y freeipa-server freeipa-server-dns bind bind-dyndb-ldap"
    idm1.vm.provision "shell", inline: "sudo ipa-server-install --unattended --ds-password=foobarfoo --admin-password=foobarfoo --domain=goern.example.com --realm=GOERN.EXAMPLE.COM --setup-dns --reverse-zone=1.168.192.in-addr.arpa. --no-forwarders"

    idm1.vm.provision "shell", inline: "printf 'foobarfoo\nfoobarfoo\nfoobarfoo\n' | kinit admin@GOERN.EXAMPLE.COM"

    idm1.vm.provision "shell", inline: "atomic run cockpit/ws"

  end

  # provision and enroll Atomic Hosts
  clients.each do |atomic_host, atomic_host_ip_addr|
    config.vm.define atomic_host do |this_atomic_host|
      this_atomic_host.vm.box = "centos/atomic-host"
      this_atomic_host.vm.box_check_update = false
      this_atomic_host.vm.hostname = "#{atomic_host}.goern.example.com"

      this_atomic_host.vm.synced_folder ".", "/home/vagrant/sync", disabled: true

      this_atomic_host.vm.provision "shell", inline: "sudo atomic host upgrade"
      this_atomic_host.vm.provision :reload  # FIXME this should trigger a reboot

      if this_atomic_host.vm.hostname == 'atomic-host-3.goern.example.com'
        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "./kubernetes-ansible/setup.yml"
          ansible.inventory_path = "./inventory"
          ansible.limit = 'all'
        end
      end
    end
  end

  # provision an OpenShift Origin host, basically recycled from https://www.openshift.org/vm/
  # FIXME there is no Atomic Enterprise Platform for CentOS
  config.vm.define "oso-1" do |oso1|
    oso1.vm.box = 'centos/7'
    oso1.vm.box_check_update = true
    oso1.vm.hostname = "oso-1.goern.example.com"

    oso1.vm.synced_folder ".", "/home/vagrant/sync", disabled: true
  end

end
