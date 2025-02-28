# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.8.1"

Vagrant.configure("2") do |config|
  config.vm.provider :virtualbox do |v, override|
    v.memory = 2048
    override.vm.box = "ubuntu/focal64"
    override.vm.box_version = "= 20220426.0.0 "
    override.vm.provision "shell", path: "deployment/provision_vagrant_vm.sh"

    # disable logfile
    if Vagrant::Util::Platform.windows?
      v.customize [ "modifyvm", :id, "--uartmode1", "file", "nul" ]
    else
      v.customize [ "modifyvm", :id, "--uartmode1", "file", "/dev/null" ]
    end
  end

  config.vm.provider :docker do |d, override|
    d.image = "ubuntu:focal"
    # Docker container really are supposed to be used differently. Hacky way to make it into a "VM".
    d.cmd = ["tail", "-f", "/dev/null"]

    # Workaround for no SSH server as long as https://github.com/hashicorp/vagrant/issues/8145 is still open
    override.trigger.before :provision do |trigger|
      trigger.ruby do |env, machine| system("vagrant docker-exec -it -- /evap/deployment/provision_vagrant_vm.sh") end
    end
    override.trigger.before :ssh do |trigger|
      trigger.ruby do |env, machine| system("vagrant docker-exec -it -- sudo -H -u evap bash") end
    end
  end

  # port forwarding
  config.vm.network :forwarded_port, guest: 8000, host: 8000 # django server
  config.vm.network :forwarded_port, guest: 80, host: 8001 # apache
  config.vm.network :forwarded_port, guest: 6379, host: 6379 # redis. helpful when developing on windows, for which redis is not available

  # override username to be evap instead of vagrant, just as it is on production.
  # This is necessary so management script can assume evap is the correct user to
  # execute stuff as.
  # Mounting with uid and gid is necessary as the provision script will create
  # this user, so mounting before does not work with an owner specified by name.
  # Also, provision needs to use vagrant as ssh user (since evap does not exist yet)
  config.vm.synced_folder ".", "/evap", mount_options: ["uid=1042", "gid=1042"]
  if ARGV[0] == "ssh" or ARGV[0] == "ssh-config"
    config.ssh.username = 'evap'
  end
end
