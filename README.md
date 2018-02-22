Vagrant - Multi-Machine-Multi-Environment
------------------------------

Vagrant is a very good tool in defining and controling multiple guest per Vagrantfile. When you need to have another set of multiple guest, you need to create another Vagrantfile.

Why don't we use a single Vagrantfile for Multi-Machine and Multi-Environment?

Here is why you can use this repo. We YAMLize Vagrant's Multi-Machine setting into multiple .yml files in `envs/` folder. Each .yml represents an environment.

# Requirements
* Vagrant-vbguest
```bash
$ vagrant plugin install vagrant-vbguest
```
 
# Usage
There is `thyvagrant` shell script, it's just a wrapper to vagrant binary file. 

You use `thyvagrant` the same way with how you run `vagrant` command but with additional `VAGRANT_ENV` variable to configure which environment vagrant will use.

You can either put `VAGRANT_ENV` in front of `thyvagrant`; or export it first and run `thyvagrant` separately 

```bash
$ VAGRANT_ENV=docker ./thyvagrant status
Vagrant environment: docker
Current machine states:

docker                    poweroff (virtualbox)
docker2                   not created (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.

$ export VAGRANT_ENV=docker 
$ ./thyvagrant status
Vagrant environment: docker
Current machine states:

docker                    poweroff (virtualbox)
docker2                   not created (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.

```

## Environment files

* It's stored in `envs/<VAGRANT_ENV>.yml`.
* Default environment is `main.yml`, when you run `thyvagrant` without `VAGRANT_ENV` set.

Sample for an environment with two docker hosts (`env/docker.yml`)

```yaml
machines:
  - name      : docker
    memory    : 1024
    primary   : true
    autostart : true
    network:
    - type: private
      addr: 10.255.255.36
    - type: public
      addr: 192.168.250.36

  - name      : docker2
    memory    : 4096
    network:
    - type: private
      addr: 10.255.255.37
    - type: public
      addr: 192.168.250.37

machine_default:
  vbguest.auto_update : true
  box:  centos/7
```

# It's still under development

It's still under development and not well tested yet :)

* Updates and enhancements keep coming.
* It's tested in Ubuntu. I don't foresee any issue with using it on another Linux distribution.
* Further testing need to be done on MacOS/Windows

