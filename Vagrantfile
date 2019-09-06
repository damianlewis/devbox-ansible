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
            if settings["servers"].size > 1
                machine_name = "#{server["name"] ||= "devbox"}_vm#{server_count}"
                machine_hostname = "#{server["hostname"] ||= "devbox"}-vm#{server_count}"
            else
                machine_name = "devbox_vm"
                machine_hostname = "devbox"
            end
 
            groups[server["group"] ||= "all"] << machine_name

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

            config.vm.define machine_name do |machine|
                # Configure the vagrant box.
                machine.vm.hostname = machine_hostname
                machine.vm.box = server["box"] ||= "damianlewis/#{os ||= default_os}-#{os_version ||= default_os_version}"
                machine.vm.box_version = server["box_version"] ||= ">= 0"

                # Configure VirtualBox settings.
                machine.vm.provider "virtualbox" do |vb|
                    vb.name = machine_name
                    vb.memory = server["memory"] ||= "1024"
                    vb.cpus = server["cpus"] ||= "1"
                    # vb.linked_clone = true
                    if server.has_key?("gui") && server["gui"] == true
                        vb.gui = true
                    end
                end

                # Configure private network.
                if server.has_key?("ip")
                    machine.vm.network "private_network", ip: server["ip"]
                else
                    machine.vm.network "private_network", type: "dhcp"
                end

                # Configure public/bridged network.
                if server.has_key?("public_ip")
                    machine.vm.network "public_network", ip: server["public_ip"], bridge: server["bridge"] ||= "en1: Wi-Fi (AirPort)"
                end

                # Configure ports to forward.
                if server.has_key?("forward_ports")
                    server["forward_ports"].each do |port|
                        machine.vm.network "forwarded_port", guest: port["guest"], host: port["host"] + port_count, auto_correct: true
                    end

                    port_count += 1
                end

                # Disable default shared folder.
                machine.vm.synced_folder ".", "/vagrant", disabled: true

                # Configure shared folders.
                if server.has_key?("folders")
                    server["folders"].each do |folder|
                        if folder["type"] == "nfs"
                            machine.vm.synced_folder folder["map"], folder["to"],
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
                            machine.vm.synced_folder folder["map"], folder["to"],
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

                # Provision the servers.
                machine.vm.provision "ansible" do |ansible|
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
