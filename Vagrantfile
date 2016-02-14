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
#    idm1.vm.network "private_network", ip: "192.168.1.33"

    # a la https://stackoverflow.com/questions/33117939/vagrant-do-not-map-hostname-to-loopback-address-in-etc-hosts
    config.vm.provision "shell", inline: "hostname --fqdn > /etc/hostname && hostname -F /etc/hostname"
    config.vm.provision "shell", inline: "sed -ri 's/127\.0\.0\.1\s.*/127.0.0.1 localhost localhost.localdomain/' /etc/hosts"

    idm1.vm.provision "shell", inline: "sudo dnf install -y atomic freeipa-server freeipa-server-dns bind bind-dyndb-ldap"
    idm1.vm.provision "shell", inline: "sudo ipa-server-install --unattended --ds-password=foobarfoo --admin-password=foobarfoo --domain=goern.example.com --realm=GOERN.EXAMPLE.COM --setup-dns --reverse-zone=1.168.192.in-addr.arpa. --no-forwarders"

    idm1.vm.provision "shell", inline: "printf 'foobarfoo\nfoobarfoo\nfoobarfoo\n' | kinit admin@GOERN.EXAMPLE.COM"

#    clients.each do |atomic_host, atomic_host_ip_addr|
#      idm1.vm.provision "shell", inline: "sudo ipa host-add --force --ip-address=#{atomic_host_ip_addr} #{atomic_host}.goern.example.com --password=happyjaja"
#    end

    # lets enable/start docker and get cockpit webservice running
#    idm1.vm.provision "shell", inline: "sudo systemctl enable docker && sudo systemctl start docker && sudo atomic run cockpit/ws"
  end

  # provision and enroll Atomic Hosts
  clients.each do |atomic_host, atomic_host_ip_addr|
    config.vm.define atomic_host do |this_atomic_host|
      this_atomic_host.vm.box = "fedora/23-atomic-host"
      this_atomic_host.vm.box_check_update = false
      this_atomic_host.vm.hostname = "#{atomic_host}.goern.example.com"
#      this_atomic_host.vm.network "private_network", ip: atomic_host_ip_addr

      this_atomic_host.vm.synced_folder ".", "/home/vagrant/sync", disabled: true

#      this_atomic_host.vm.provision "shell", inline: "atomic install sssd --password barbazbar"
#      this_atomic_host.vm.provision "shell", inline: "sudo atomic host upgrade"
    end
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
