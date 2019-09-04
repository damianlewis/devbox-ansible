# DevBox Ansible

### Introduction
DevBox Ansible provides a multi server development environment using Vagrant and VirtualBox and is provisioned using Ansible.


### Requirements
- Vagrant
- VirtualBox
- Ansible


### Configuration
The `config.yaml` file allows you to configure the servers by adding individual server settings to the `servers` list property. The example `config.yaml` file and accompanying playbooks, demonstrate setting up 4 servers in a load balanced infrastructure.


#### VM name
Use the `name` property to update the VirtualBox VM name. By default this is `devbox`.
```yaml
name: devbox_www
```


#### Hostname
Use the `hostname` property to update the server hostname. By default this is `devbox`.
```yaml
hostname: devbox-www
```


#### Memory
Use the `memory` property to update the amount of memory allocated to the VM . By default this is `1024`.
```yaml
memory: 2048
```


#### CPU
Use the `cpus` property to update the number of CPU's allocated to the VM . By default this is `1`.
```yaml
cpus: 2
```


#### Networks
By default, DevBox will create a private network with automatically assigned IP addresses for all the servers. Use `ifconfig` on the server to determine it's IP address. A static IP address can be assigned to a server by adding the `ip` property. The [reserved private address space](https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces) must be used when assigning IP addresses.
```yaml
ip: "192.168.2.40"
```

A public IP address can also be given to a server by adding the `public_ip` property.
```yaml
public_ip: "192.168.178.41"
```

By default, DevBox forwards port 22 (for SSH) on the guest machine to port 2222 on the host. To forward additional ports, add the `forward_ports` list property with `guest` and `host` properties set to the ports you want to forward.
```yaml
forward_ports:
- guest: 80
  host: 8000
- guest: 443
  host: 44300
```


#### OS
Currently, DevBox only supports Ubuntu and defaults to version 16.04. To use a different version, change the `os_version` property. Supported versions are `14.04`, `16.04'`and `18.04`.
```yaml
os: "ubuntu"
os_version: "16.04"
```

You can also specify a particular Vagrant box to use. The will override the OS and server type properties.
```yaml
box: "nginx-php"
```

If you require a particular version of a Vagrant box, use the `box_version` property.
```yaml
box_version: "1"
```


#### Folders
By default, DevBox doesn't share/sync any folders between the guest and host machines. To share folders with the guest machine add the `folders` list property and `map` the host folder `to` the guest folder.
```yaml
folders:
- map: ./public
  to: /var/www/public
```

To use NFS for a shared folder, add the `type` property.
```yaml
folders:
- map: ./public
  to: /var/www/public
  type: nfs
```

To change the owner or group of the shared folder on the guest machine, add the `owner` and `group` properties.
```yaml
folders:
- map: ./public
  to: /var/www/public
  owner: root
  group: www-data
```

To change the permissions of the shared folder on the guest machine, add the `dmode` and `fmode` properties. The default directory permissions (dmode) are 755 and the default file permissions (fmode) are 644.
```yaml
folders:
- map: ./public
  to: /var/www/public
  dmode: 775
  fmode: 664
```

If the host path doesn't exsist, it can be created by setting the `create` property to `true`. By default, this is `false`.
```yaml
folders:
- map: ./public
  to: /var/www/public
  create: true
```


#### Ansible inventory groups
DevBox creates an inventory for use with Ansible. By default all servers will be added to a 'devbox' inventory group. You can override this and add servers to different groups by using the `group` property.
```yaml
group: devbox_www
```


### Ansible provisioning
The `provision.yaml` file should be used to create an Ansible playbook used to provision the servers.


#### Ansible Galaxy Roles
To import Galaxy roles, create a [requirements YAML file](https://docs.ansible.com/ansible/latest/reference_appendices/galaxy.html#installing-multiple-roles-from-a-file) and set the `galaxy_role_file` property to it's location.
```yaml
galaxy_role_file: requirements.yml
```

By default, DevBox will create a `roles` folder to store the imported roles. Use the `galaxy_roles_path` property to set the path of the roles folder to an alternative location.
```yaml
galaxy_roles_path: ~/.ansible/roles
```

#### Debugging
Set the `debug` property to `true` to obtain detailed logging with connection debugging.
```yaml
debug: true
```


### Bash aliases
A number of default bash aliases are created for the VM. These can be found in the `aliases` file. Add any further aliases you require to this file before creating the VM.
