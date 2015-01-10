# -*- mode: ruby -*-
# vi: set ft=ruby :

require "yaml"

dir = File.dirname(File.expand_path(__FILE__))

configValues  = YAML.load_file("#{dir}/vagrantConfig.yaml")
config        = configValues['vagrantfile-config']

Vagrant.require_version '>= 1.6.0'

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Begin box section
    config.vm.box     = "#{config['box']}"
    config.vm.box_url = "#{config['box_url']}"
  # End box section

  # Begin network section
    if config['hostname'].to_s.strip.length != 0
      config.vm.hostname = "#{config['hostname']}"
    end

    if config['network']['private_network'].to_s != ''
      config.vm.network 'private_network', ip: "#{config['network']['private_network']}"
    end

    config['network']['forwarded_port'].each do |i, port|
      if port['guest'] != '' && port['host'] != ''
        config.vm.network :forwarded_port, guest: port['guest'].to_i, host: port['host'].to_i
      end
    end
  # Begin network section

  # Begin SSH section
    if !config['ssh']['agent_forwarding'].nil?
      config.ssh.forward_agent = config['ssh']['agent_forwarding']
    end
  # End SSH section

  # Begin shared-folders section
    config['synced_folders'].each do |i, folder|
      if folder['host'] != '' && folder['guest'] != ''
        sync_owner = !folder['sync_owner'].nil? ? folder['sync_owner'] : 'vagrant'
        sync_group = !folder['sync_group'].nil? ? folder['sync_group'] : 'vagrant'

        if folder['sync_type'] == 'nfs'
          config.vm.synced_folder "#{folder['host']}", "#{folder['guest']}", id: "#{i}", type: 'nfs'
        elsif folder['sync_type'] == 'smb'
          config.vm.synced_folder "#{folder['host']}", "#{folder['guest']}", id: "#{i}", type: 'smb'
        elsif folder['sync_type'] == 'rsync'
          rsync_args = !folder['rsync']['args'].nil? ? folder['rsync']['args'] : ['--verbose', '--archive', '-z']
          rsync_auto = !folder['rsync']['auto'].nil? ? folder['rsync']['auto'] : true
          rsync_exclude = !folder['rsync']['exclude'].nil? ? folder['rsync']['exclude'] : ['.vagrant/']

          config.vm.synced_folder "#{folder['host']}", "#{folder['guest']}", id: "#{i}",
            rsync__args: rsync_args, rsync__exclude: rsync_exclude, rsync__auto: rsync_auto, type: 'rsync', group: sync_group, owner: sync_owner
        elsif data['vm']['chosen_provider'] == 'parallels'
          config.vm.synced_folder "#{folder['host']}", "#{folder['guest']}", id: "#{i}",
            group: sync_group, owner: sync_owner, mount_options: ['share']
        else
          config.vm.synced_folder "#{folder['host']}", "#{folder['guest']}", id: "#{i}",
            group: sync_group, owner: sync_owner, mount_options: ['dmode=775', 'fmode=764']
        end
      end
    end
  # End shared-folders section

  # Begin providers section
  if config['vm']['chosen_provider'].empty? || config['vm']['chosen_provider'] == 'virtualbox'
    ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'

    config.vm.provider :virtualbox do |virtualbox|
      config['vm']['provider']['virtualbox']['modifyvm'].each do |key, value|
        if key == 'memory'
          next
        end
        if key == 'cpus'
          next
        end

        if key == 'natdnshostresolver1'
          value = value ? 'on' : 'off'
        end

        virtualbox.customize ['modifyvm', :id, "--#{key}", "#{value}"]
      end

      virtualbox.customize ['modifyvm', :id, '--memory', "#{config['memory']}"]
      virtualbox.customize ['modifyvm', :id, '--cpus', "#{config['cpus']}"]

      if config['vm']['provider']['virtualbox']['modifyvm']['name'].nil? ||
        config['vm']['provider']['virtualbox']['modifyvm']['name'].empty?
        if config['vm']['hostname'].to_s.strip.length != 0
          virtualbox.customize ['modifyvm', :id, '--name', config.vm.hostname]
        end
      end
    end
  end
  # End providers section


  # Begin provisioners section
  if config['chosen_provisioner'].empty? || config['chosen_provisioner'] == 'puppet'

  # vagrantr10k (https://github.com/jantman/vagrant-r10k) config
    if Vagrant.has_plugin?('vagrant-r10k') && !config['provisioner']['puppet']['r10k'].empty?
      config.r10k.puppet_dir = config['provisioner']['puppet']['r10k']['manifests_path']
      config.r10k.puppetfile_path = config['provisioner']['puppet']['r10k']['manifest_file']
      config.r10k.module_path = config['provisioner']['puppet']['r10k']['module_path']
    end

    config.vm.provision "puppet" do |puppet|
      puppet.manifests_path = config['provisioner']['puppet']['manifests_path']
      puppet.manifest_file  = config['provisioner']['puppet']['manifest_file']
      puppet.module_path = config['provisioner']['puppet']['module_path']
    end
 end
 # End provisioners section

end
