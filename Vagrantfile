
# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "debian/jessie64"

    config.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbook.yml"
        ansible.groups = {
            "clients" => ["client"],
            "ecmp-routers" => ["r1"],
            "servers" => ["s1", "s2", "s3"]
        }
    end

    config.vm.define "r1" do |r1|
        r1.vm.hostname = "r1"
        r1.vm.network "private_network", ip: "10.1.1.254"
        r1.vm.network "private_network", ip: "10.1.2.254"
        r1.vm.network "private_network", ip: "10.1.3.254"
        r1.vm.network "private_network", ip: "192.168.100.254"
    end

    config.vm.define "client" do |client|
        client.vm.hostname = "client"
        client.vm.network "private_network", ip: "192.168.100.100"
    end

    config.vm.define "s1" do |s1|
        s1.vm.hostname = "s1"
        s1.vm.network "private_network", ip: "10.1.1.2"
    end

    config.vm.define "s2" do |s2|
        s2.vm.hostname = "s2"
        s2.vm.network "private_network", ip: "10.1.2.2"
    end

    config.vm.define "s3" do |s3|
        s3.vm.hostname = "s3"
        s3.vm.network "private_network", ip: "10.1.3.2"
    end

end
