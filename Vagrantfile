# -*- mode: ruby -*-
# # vi: set ft=ruby :

require 'fileutils'

# Commands required to setup working docker environment, link
# containers etc.
$setup = <<SCRIPT
  # Stop and remove any existing containers
  docker stop $(docker ps -a -q)
  docker rm $(docker ps -a -q)

  # Run and link the containers
  docker run -d --name fake-sqs -p 4568:4568 bbcnews/fake-sqs
  docker run -d --name fake-s3 -p 4569:4569 bbcnews/fake-s3
  docker run -d --name local-dynamo -p 4570:4570 smaj/local-dynamo
  docker run -d --name memcached_server -p 11211:11211 smaj/memcached
  docker run -d --name fake_elasticache --link memcached_server:memcached01 -p 11212:11212 smaj/fake-elasticache
  docker run -d --name fake_elasticache_local --link memcached_server:memcached01 -p 11213:11212 -e FAKEELASTICACHE_DEFAULT_HOST='127.0.0.1' smaj/fake-elasticache

SCRIPT

  # Commands required to ensure correct docker containers
  # are started when the vm is rebooted.
$start = <<SCRIPT
  docker start fake-s3
  docker start fake-sqs
  docker start local-dynamo
  docker start memcached_server
  docker start fake_elasticache
  docker start fake_elasticache_local
SCRIPT

CLOUD_CONFIG_PATH = "./user-data"
CONFIG= "config.rb"

# Defaults for config options defined in CONFIG
$num_instances = 1
$update_channel = "alpha"
$enable_serial_logging = false
$vb_memory = 1024
$vb_cpus = 1
$expose_docker_tcp = 2375

if File.exist?(CONFIG)
  require_relative CONFIG
end

Vagrant.configure("2") do |config|

  config.vm.box = "coreos-%s" % $update_channel
  config.vm.box_version = ">= 308.0.1"
  config.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant.json" % $update_channel

  config.vm.provider :vmware_fusion do |vb, override|
    override.vm.box_url = "http://%s.release.core-os.net/amd64-usr/current/coreos_production_vagrant_vmware_fusion.json" % $update_channel
  end

  config.vm.provider :virtualbox do |v|
    # On VirtualBox, we don't have guest additions or a functional vboxsf
    # in CoreOS, so tell Vagrant that so it can be smarter.
    v.check_guest_additions = false
    v.functional_vboxsf     = false
  end

  # plugin conflict
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end

  config.vm.hostname = "coreos-%s" % $update_channel

  if $enable_serial_logging
    logdir = File.join(File.dirname(__FILE__), "log")
    FileUtils.mkdir_p(logdir)

    serialFile = File.join(logdir, "%s-serial.txt" % vm_name)
    FileUtils.touch(serialFile)

    config.vm.provider :vmware_fusion do |v, override|
      v.vmx["serial0.present"] = "TRUE"
      v.vmx["serial0.fileType"] = "file"
      v.vmx["serial0.fileName"] = serialFile
      v.vmx["serial0.tryNoRxLoss"] = "FALSE"
    end

    config.vm.provider :virtualbox do |vb, override|
      vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
      vb.customize ["modifyvm", :id, "--uartmode1", serialFile]
    end
  end

  config.vm.network "forwarded_port", guest: 2375, host: 2375, auto_correct: true
  config.vm.network "forwarded_port", guest: 11211, host: 11211, auto_correct: true
  config.vm.network "forwarded_port", guest: 11212, host: 11212, auto_correct: true
  config.vm.network "forwarded_port", guest: 11213, host: 11213, auto_correct: true
  config.vm.network "forwarded_port", guest: 4568, host: 4568, auto_correct: true
  config.vm.network "forwarded_port", guest: 4569, host: 4569, auto_correct: true
  config.vm.network "forwarded_port", guest: 4570, host: 4570, auto_correct: true

  config.vm.provider :virtualbox do |vb|
    vb.memory = $vb_memory
    vb.cpus = $vb_cpus
  end

  config.vm.network :private_network, ip: "172.17.8.100"

  # Uncomment below to enable NFS for sharing the host machine into the coreos-vagrant VM.
  config.vm.synced_folder ".", "/home/core/share", id: "core", :nfs => true, :mount_options => ['nolock,vers=3,udp']

  config.vm.provision "shell", path: "build.sh"
  config.vm.provision "shell", inline: $setup
  config.vm.provision "shell", run: "always", inline: $start

end
