# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

settings = YAML::load(File.read("devbox_config.yaml"))
aliases = "aliases"

supported_os = ["ubuntu"]
supported_ubuntu_versions = ["14.04", "16.04", "18.04"]

default_os = "ubuntu"
default_os_version = "16.04"

Vagrant.configure("2") do |config|
    if settings.has_key?("servers")
        groups = Hash.new {|h,k| h[k]=[]}
        port_count = 0

        settings["servers"].each_with_index do |server, server_count|
            name = server["name"] ||= "devbox"
            hostname = server["hostname"] ||= "devbox"
            group = server["group"] ||= "devbox"
            machine = "vm#{server_count}_#{name}"
            groups[server["group"]] << machine

            if server.has_key?("os")
                os = server["os"]

                unless supported_os.include?("#{os}")
                    abort("OS #{os} not recognised. Only #{supported_os} are currenly supported with DevBox.")
                end
            end

            if server.has_key?("os_version")
                supported_os_version = "supported_#{os}_versions"
                os_version = server["os_version"]

                unless supported_ubuntu_versions.include?("#{os_version}")
                    abort("#{os} version #{os_version} not recognised. Only #{supported_os_version} are currenly supported with DevBox.")
                end
            end

            config.vm.define machine do |node|
                # Configure the vagrant box.
                node.vm.hostname = "vm#{server_count}-#{hostname}"
                node.vm.box = server["box"] ||= "damianlewis/#{os ||= default_os}-#{os_version ||= default_os_version}"
                node.vm.box_version = server["box_version"] ||= ">= 0"

                # Configure VirtualBox settings.
                node.vm.provider "virtualbox" do |vb|
                    vb.name = machine
                    vb.memory = server["memory"] ||= "1024"
                    vb.cpus = server["cpus"] ||= "1"
                    # vb.linked_clone = true
                    if server.has_key?("gui") && server["gui"] == true
                        vb.gui = true
                    end
                end

                # Configure private network.
                if server.has_key?("ip")
                    node.vm.network "private_network", ip: server["ip"]
                else
                    node.vm.network "private_network", type: "dhcp"
                end

                # Configure public/bridged network.
                if server.has_key?("public_ip")
                    node.vm.network "public_network", ip: server["public_ip"], bridge: server["bridge"] ||= "en1: Wi-Fi (AirPort)"
                end

                # Configure ports to forward.
                if server.has_key?("forward_ports")
                    server["forward_ports"].each do |port|
                        node.vm.network "forwarded_port", guest: port["guest"], host: port["host"] + port_count, auto_correct: true
                    end

                    port_count += 1
                end

                # Disable default shared folder.
                node.vm.synced_folder ".", "/vagrant", disabled: true

                # Configure shared folders.
                if server.has_key?("folders")
                    server["folders"].each do |folder|
                        if folder["type"] == "nfs"
                            node.vm.synced_folder folder["map"], folder["to"],
                            create: folder["create"] ||= false,
                            owner: folder["owner"] ||= "",
                            group: folder["group"] ||= "",
                            type: "nfs",
                            mount_options: [
                              "dmode=#{folder['dmode'] ||= 755}",
                              "fmode=#{folder['fmode'] ||= 644}",
                              "actimeo=1",
                              "nolock"
                            ]
                        else
                            node.vm.synced_folder folder["map"], folder["to"],
                            create: folder["create"] ||= false,
                            owner: folder["owner"] ||= "",
                            group: folder["group"] ||= "",
                            mount_options: [
                              "dmode=#{folder['dmode'] ||= 755}",
                              "fmode=#{folder['fmode'] ||= 644}"
                            ]
                        end
                    end
                end

                # Create bash aliases.
                if File.exists? aliases
                    node.vm.provision "file", source: aliases, destination: "~/.bash_aliases"
                end

                # Provision the servers.
                node.vm.provision "ansible" do |ansible|
                    if settings.has_key?("galaxy_role_file")
                        ansible.galaxy_role_file = settings["galaxy_role_file"]
                        ansible.galaxy_roles_path = settings["galaxy_roles_path"] ||= "roles"
                    end
                    ansible.playbook = "configure.yml"
                    ansible.groups = groups
                    if settings.has_key?("debug") and settings["debug"]
                        ansible.verbose = '-vvvv'
                    end
                end
            end
        end
    end
end
