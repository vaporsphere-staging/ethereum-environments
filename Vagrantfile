# -*_ mode: ruby _*_
# vi: set ft=ruby :
VAGRANTFILE_API_VERSION = "2"
node = ENV['ETH_NODE']
name = ENV['ETH_NAME'] || node
gui  = ENV['ETH_GUI'] || 'false'
branch = ENV['ETH_BRANCH'] || 'master'
fqdn = ENV['ETH_FQDN']

Vagrant.configure(VAGRANTFILE_API_VERSION) do |vagrant|

  vagrant.vm.define "virtualbox-#{name}", primary: true do |config|
    config.vm.box       = 'ubuntu14.04'
    config.vm.host_name = node
    config.vm.box_url = 'http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box'
    config.vm.provider "virtualbox" do |v|
      v.customize ["modifyvm", :id, '--memory', 2048]
    end
    config.vm.provision :shell do |shell|
      shell.inline = "sudo apt-get -y install puppet; sudo rm -f /tmp/hiera; sudo ln -sf /vagrant/puppet/hiera /tmp/hiera"
    end
    config.vm.provision :puppet do |puppet|
      puppet.options        = "--verbose"
      puppet.facter = {
        "fqdn" => "local.leaf",
        "hostname" => name,
        "gui" => gui,
        "branch" => branch,
        "environment" => 'virtualbox'
      }
      puppet.hiera_config_path = "puppet/hiera.yaml"
      # puppet.working_directory = "/vagrant"
      puppet.module_path    = 'puppet/modules'
      puppet.manifests_path = 'puppet/manifests'
      puppet.manifest_file  = "#{node}.pp"
    end
  end

  # vagrant config for a remote ec2 AMI built by packer
  # before you can use vagrant for remote provisioning, you need:
  # - ec2 access and setup with a security group AWS_SECURITY_GROUP allowing ssh
  # - install vagrant-aws plugin
  # - set environment variables
  # - run packer on the template
  # - add box to vagrant
  # - vagrant up/ssh/provision/destroy available: ETH_NODE=go-ethereum vagrant up aws-go-ethereum --provider=aws

  vagrant.vm.define "aws-#{name}" do |config|
    config.vm.provider :aws do |aws, override|
      aws.access_key_id = ENV['AWS_ACCESS_KEY_ID']
      aws.secret_access_key = ENV['AWS_SECRET_KEY']
      aws.security_groups = [ ENV['AWS_SECURITY_GROUP'] ]
      aws.keypair_name = ENV['AWS_KEYPAIR_NAME']
      aws.region = ENV['AWS_REGION']
      override.ssh.username = "ubuntu"
      override.ssh.private_key_path = ENV['AWS_PRIVATE_KEY_FILE']
    end
    config.vm.box = "aws-#{name}"
    config.vm.provision :shell do |shell|
      shell.inline = "sudo apt-get -y install puppet; sudo rm -f /tmp/hiera; sudo ln -sf /vagrant/puppet/hiera /tmp/hiera"
    end
    config.vm.provision :puppet do |puppet|
      puppet.facter = {
        "fqdn" => fqdn,
        "hostname" => name,
        "gui" => gui,
        "branch" => branch,
        "environment" => 'aws'
      }
      puppet.options        = "--verbose"
      puppet.hiera_config_path = "puppet/hiera.yaml"
      puppet.working_directory = "/vagrant"
      puppet.module_path    = 'puppet/modules'
      puppet.manifests_path = 'puppet/manifests'
      puppet.manifest_file  = "#{node}.pp"
    end
  end

end
