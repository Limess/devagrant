# -*- mode: ruby -*-
# vi: set ft=ruby :

# Load default configuration values
require "yaml"

conf = YAML.load_file('vagrant-default.yaml')

# Load per-project overrides if they exist
if File.exist?('vagrant.yaml')
    user_conf = YAML.load_file('vagrant.yaml')
    conf.merge!(user_conf)
end

Vagrant.require_version ">= 1.6.0"

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Begin box section
    config.vm.box     = conf['box']
    config.vm.box_url = conf['box_url']
  # End box section

  # Begin network section
    if conf['hostname'].to_s.strip.length != 0
      config.vm.hostname = "#{conf['hostname']}"
    end

    if conf['network']['private_network'].to_s != ''
      config.vm.network 'private_network', ip: "#{conf['network']['private_network']}"
    end

    if !conf['network']['forwarded_port'].nil?
      conf['network']['forwarded_port'].each do |i, port|
        if port['guest'] != '' && port['host'] != ''
          config.vm.network :forwarded_port, guest: port['guest'].to_i, host: port['host'].to_i
        end
      end
    end
  # End network section


  # Begin SSH section
    if !conf['ssh']['agent_forwarding'].nil?
      config.ssh.forward_agent = conf['ssh']['agent_forwarding']
    end
  # End SSH section

  # Begin shared-folders section
    conf['synced_folders'].each do |i, folder|
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
        elsif conf['chosen_provider'] == 'parallels'
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
  if conf['chosen_provider'].empty? || conf['chosen_provider'] == 'virtualbox'
    ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'

    config.vm.provider :virtualbox do |virtualbox|
      conf['provider']['virtualbox']['modifyvm'].each do |key, value|
        if key == 'memory'
          next
        end
        if key == 'cpus'
          next
        end

        if key == 'natdnshostresolver1'
          value = value ? 'on' : 'off'
        end

        if key == 'natdnsproxy1'
          value = value ? 'on' : 'off'
        end

        virtualbox.customize ['modifyvm', :id, "--#{key}", "#{value}"]
      end

      virtualbox.customize ['modifyvm', :id, '--memory', "#{conf['resources']['memory']}"]
      virtualbox.customize ['modifyvm', :id, '--cpus', "#{conf['resources']['cpus']}"]

      if conf['provider']['virtualbox']['modifyvm']['name'].nil? ||
        conf['provider']['virtualbox']['modifyvm']['name'].empty?
        if conf['hostname'].to_s.strip.length != 0
          virtualbox.customize ['modifyvm', :id, '--name', config.vm.hostname]
        end
      end
    end
  end
  # End providers section

  # Begin provisioners section

  # Begin shell section
  if conf['chosen_provisioners'].include?('shell')
    config['provisioner']['shell'].each do |i, script|
      conf.vm.provision "shell", path: ['script']
    end
  end
  # End shell section

  # Begin Puppet section (default)
  if conf['chosen_provisioners'].empty? || conf['chosen_provisioners'].include?('puppet')

    # config.vm.provision "puppet" do |puppet|
    #   puppet.manifests_path = conf['provisioner']['puppet']['manifests_path']
    #   puppet.manifest_file  = conf['provisioner']['puppet']['manifest_file']
    #   puppet.module_path = conf['provisioner']['puppet']['module_path']

    #   if !conf['provisioner']['puppet']['facter'].nil?
    #     conf['provisioner']['puppet']['facter'].each do |key, value|
    #       puppet.facter = {
    #         "#{key}" => "#{value}"
    #       }
    #     end
    #   end
    # end

    # Begin vagrantr10k (https://github.com/jantman/vagrant-r10k) section
    if Vagrant.has_plugin?('vagrant-r10k') && !conf['provisioner']['puppet']['r10k'].empty?
      # config.r10k.puppet_dir = conf['provisioner']['puppet']['r10k']['manifests_path']
      # config.r10k.puppetfile_path = conf['provisioner']['puppet']['r10k']['manifest_file']
      # config.r10k.module_path = conf['provisioner']['puppet']['r10k']['module_path']
    end
    # End vagrantr10k section

 end
 # End Puppet section

 # End provisioners section

end
