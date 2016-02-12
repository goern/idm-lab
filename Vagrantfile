# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.define "idm-1" do |idm1|
    idm1.vm.box = "fedora/23-cloud-base"
    idm1.vm.box_check_update = true
    idm1.vm.hostname = "idm-1.goern.example.com"

    idm1.vm.network "forwarded_port", guest: 9090, host: 9091

    # a la https://stackoverflow.com/questions/33117939/vagrant-do-not-map-hostname-to-loopback-address-in-etc-hosts
    config.vm.provision "shell", inline: "hostname --fqdn > /etc/hostname && hostname -F /etc/hostname"
    config.vm.provision "shell", inline: "sed -ri 's/127\.0\.0\.1\s.*/127.0.0.1 localhost localhost.localdomain/' /etc/hosts"

    idm1.vm.provision "shell", inline: "sudo dnf install -y freeipa-server freeipa-server-dns bind bind-dyndb-ldap"
    idm1.vm.provision "shell", inline: "ipa-server-install -U -p foobarfoo -a foobarfoo -n idm-1.goern.example.com -r GOERN.EXAMPLE.COM --setup-dns --no-forwarders"
    idm1.vm.provision "shell", inline: "printf 'foobarfoo\nfoobarfoo\nfoobarfoo\n' | kinit admin@GOERN.EXAMPLE.COM"

  end

  config.vm.define "atomic-host-1" do |atomichost1|
    atomichost1.vm.box = "fedora/23-atomic-host"
    atomichost1.vm.box_check_update = false
    atomichost1.vm.hostname = "atomic-host-1.goern.example.com"

    atomichost1.vm.synced_folder ".", "/home/vagrant/sync", disabled: true

  end

  config.vm.provider :libvirt do |domain|
    domain.memory = 2048
    domain.cpus = 2
  end

end
