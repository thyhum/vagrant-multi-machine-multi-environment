# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'
require 'socket' # Do we still require socket?

if ENV['THYVAGRANT'].nil?
  abort 'Use "./thyvagrant" instead of "vagrant" command'
end

root_path = File.dirname __FILE__

vagrant_conf = YAML.load_file "#{root_path}/vagrant_conf.yml"

vagrant_env_file = "#{root_path}/envs/#{ENV['VAGRANT_ENV']}.yml"
if File.exist? vagrant_env_file
  vagrant_env = YAML.load_file vagrant_env_file
  vagrant_conf['machine_default'].merge! vagrant_env['machine_default']
  vagrant_conf['machines'] = vagrant_env['machines']
else
  abort "VAGRANT_ENV #{ENV['VAGRANT_ENV']} (#{vagrant_env_file}) doesn't exist"
end

# puts vagrant_conf # For DEBUG

Vagrant.configure("2") do |config|

  vagrant_conf['machines'].each_with_index do |machine, idx|

    machine = vagrant_conf['machine_default'].merge(machine)
    # print "#{machine}\n\n"

    virtualbox_setting = []
    unless machine['virtualbox_setting'].nil?
      machine['virtualbox_setting'].each do |virtualbox_setting_yaml|
        virtualbox_setting += virtualbox_setting_yaml
      end
    end

    config.vm.define machine['name'],
        primary:     machine['primary'],
        autostart:   machine['autostart'] do |vm_machine|

      vm_machine.vm.box              = machine['box']
      vm_machine.vm.hostname         = machine['hostname'] || machine['name'] + machine['domain']
      vm_machine.vm.box_check_update = machine['vm.box_check_update']

      unless machine['network'].nil?

        machine['network'].each do |machine_network|
          if machine_network['addr'] == 'dhcp'
            vm_network_params = {type: 'dhcp'}
          else
            vm_network_params = {
                ip: machine_network['addr'],
                netmask: machine_network['netmask'] ||
                    vagrant_conf['default']["#{machine_network['type']}_ips"]['netmask']
            }
          end

          vm_network_params.merge! ({bridge: machine_network['bridges'] ||
              vagrant_conf['default']['public_ips']['bridges']}) if machine_network['type'] == 'public'

          vm_machine.vm.network "#{machine_network['type']}_network", vm_network_params
        end


      end

      unless machine['forwarded_port'].nil?
        machine['forwarded_port'].each do |forwarded_port|
          vm_machine.vm.network 'forwarded_port', {
              guest: forwarded_port['guest'],
              host: forwarded_port['host']
          }
        end
      end

      vm_machine.vm.provider 'virtualbox' do |vm_virtualbox_config|
        vm_virtualbox_config.linked_clone = machine['linked_clone']
        vm_virtualbox_config.memory       = machine['memory']
        vm_virtualbox_config.gui          = machine['gui']

        vm_virtualbox_config.customize [
                         'modifyvm', :id,
                         '--groups', machine['group'],
                         '--cpus',   machine['cpus'],
                     ] + virtualbox_setting

      end

      vm_machine.ssh.insert_key       = machine['ssh.insert_key']
      vm_machine.ssh.private_key_path = machine['private_key_path'] unless machine['private_key_path'].nil?

      vm_machine.vbguest.auto_update  = machine['vbguest.auto_update']

      unless machine['ansible_playbook'].nil?
        vm_machine.vm.provision 'ansible' do |ansible|
          ansible.playbook = machine['ansible_playbook']
          ansible.compatibility_mode = '2.0'
        end
      end

      # if idx == vagrant_conf['machines'].size - 1
      #   vm_machine.vm.provision 'ansible' do |ansible|
      #     ansible.limit = 'all'
      #     ansible.playbook = 'ansible/main.yml'
      #     ansible.compatibility_mode = '2.0'
      #   end
      # end
    end

  end


end
