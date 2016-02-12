# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  # check if the hostmanager plugin is installed
  unless Vagrant.has_plugin?("vagrant-hostmanager")
    raise 'vagrant-hostmanager is not installed! see https://github.com/smdahlen/vagrant-hostmanager'
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

    # a la https://stackoverflow.com/questions/33117939/vagrant-do-not-map-hostname-to-loopback-address-in-etc-hosts
    config.vm.provision "shell", inline: "hostname --fqdn > /etc/hostname && hostname -F /etc/hostname"
    config.vm.provision "shell", inline: "sed -ri 's/127\.0\.0\.1\s.*/127.0.0.1 localhost localhost.localdomain/' /etc/hosts"

    idm1.vm.provision "shell", inline: "sudo dnf install -y freeipa-server freeipa-server-dns bind bind-dyndb-ldap"
    idm1.vm.provision "shell", inline: "ipa-server-install -U -p foobarfoo -a foobarfoo -n idm-1.goern.example.com -r GOERN.EXAMPLE.COM --setup-dns --no-forwarders"
    idm1.vm.provision "shell", inline: "printf 'foobarfoo\nfoobarfoo\nfoobarfoo\n' | kinit admin@GOERN.EXAMPLE.COM"
    idm1.vm.provision "shell", inline: "sudo atomic run cockpit/ws"

  end

  # provision and enroll an Atomic Host
  config.vm.define "atomic-host-1" do |atomichost1|
    atomichost1.vm.box = "fedora/23-atomic-host"
    atomichost1.vm.box_check_update = false
    atomichost1.vm.hostname = "atomic-host-1.goern.example.com"

    atomichost1.vm.synced_folder ".", "/home/vagrant/sync", disabled: true

    atomichost1.vm.provision "shell", inline: "atomic install sssd --password barbazbar"
  end

  # provision an OpenShift Origin host, basically recycled from https://www.openshift.org/vm/
  config.vm.define "openshift-origin-1" do |oso1|
    oso1.vm.box = 'fedora/23-cloud-base'
    oso1.vm.box_check_update = true
    oso1.vm.hostname = "oso-1.goern.example.com"

    oso1.vm.synced_folder ".", "/vagrant", disabled: true

    oso1.vm.provision "shell", inline: "sudo dnf install -y docker wget; sudo systemctl enable docker; sudo systemctl start docker"
    oso1.vm.provision "shell", inline: "wget -q https://github.com/openshift/origin/releases/download/v1.1.1/openshift-origin-server-v1.1.1-e1d9873-linux-64bit.tar.gz && tar -xaf openshift-origin-server-v1.1.1-e1d9873-linux-64bit.tar.gz && cd openshift-origin-server-v1.1.1-e1d9873-linux-64bit"
    oso1.vm.provision "shell", inline: 'sudo echo "[Unit]\nDescription=OpenShift Master\nDocumentation=https://github.com/openshift/origin\nAfter=network.target\n\n[Service]\nType=notify\nExecStart=/home/vagrant/openshift-origin-server-v1.1.1-e1d9873-linux-64bit/openshift start master " >/usr/lib/systemd/system/openshift-origin.service && sudo systemctl daemon-reload && sudo systemctl enable openshift-origin && sudo systemctl start openshift-origin'
  end

end
