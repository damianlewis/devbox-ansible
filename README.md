# DevBox Ansible

### Introduction
DevBox Ansible provides a multi server development environment using Vagrant and VirtualBox. It can be provisioned to support numerous configurations via Ansible.


### Requirements
- Vagrant
- VirtualBox
- Ansible


### Configuration
The `config.yaml` file allows you to configure the servers by adding individual server settings to the `servers` list property. The example `config.yaml` file demonstrates setting up four servers in a load balanced infrastructure.


#### VM name
Use the `name` property to update the VirtualBox VM name. By default this is 'devbox'.
```yaml
name: devbox_app
```


#### Hostname
Use the `hostname` property to update the server hostname. By default this is 'devbox'.
```yaml
hostname: devbox-app
```


#### Networks
By default, DevBox will create a private network with automatically assigned IP addresses for all the servers. Use `ifconfig` on the server to determine it's IP address. A static IP address can be assigned to a server by adding the `ip` property. The [reserved private address space](https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces) must be used when assigning IP addresses.
```yaml
ip: "192.168.2.40"
```

A public IP address can also be given to a server by adding the `public-ip` property.
```yaml
public-ip: "192.168.178.41"
```

By default, DevBox forwards port 22 (for SSH) on the guest machine to port 2222 on the host. To forward additional ports, add the `forward-ports` list property with `guest` and `host` properties set to the ports you want to forward.
```yaml
forward-ports:
    - guest: 80
      host: 8000
    - guest: 443
      host: 44300
```


#### OS
Currently, DevBox only supports Ubuntu which defaults to version 16.04. To use a different version, change the `os-version` property. Supported versions are `14.04`, `16.04'`and `18.04`. 
```yaml
os: "ubuntu"
os-version: "16.04"
```

#### Vagrant box
You can use a number of predefined Vagrant box types for your server. The current list is `lamp`, `lemp`, `apache-php`, `nginx-php` and `mysql`. To use a particular type, add the `type` property and specify which type you require.
```yaml
type: "nginx-php"
```

You can also specify a particular Vagrant box to use. The will override the OS and server type properties.
```yaml
box: "nginx-php"
```

If you require a particular version of a Vagrant box, use the `box-version` property.
```yaml
box-version: "1"
```


#### Folders
By default, DevBox doesn't share/sync any folders between the guest and host machines. To share folders with the guest machine add the `folders` list property and `map` the host folder `to` the guest folder.
```yaml
folders:
    - map: ~/code
      to: /vagrant/code
```

To use NFS for a shared folder, add the `type` property.
```yaml
folders:
    - map: ~/code
      to: /vagrant/code
      type: nfs
```


#### Ansible inventory groups
DevBox creates an inventory for use with Ansible. By default all servers will be added to a 'devbox' inventory group. You can override this and add servers to different groups by using the `group` property.
```yaml
group: app
```


### Ansible provisioning
The `provision.yaml` file should be used to create the Ansible playbook used to provision the servers.


### Bash aliases
A number of default bash aliases are created for the VM. These can be found in the `aliases` file. Add any further aliases you require to this file before creating the VM.