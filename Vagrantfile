# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

PFB_SHARED_FOLDER_TYPE = ENV.fetch("PFB_SHARED_FOLDER_TYPE", "nfs")

if PFB_SHARED_FOLDER_TYPE == "nfs"
  if Vagrant::Util::Platform.linux? then
    PFB_MOUNT_OPTIONS = ['rw', 'vers=3', 'tcp', 'nolock', 'actimeo=1']
  else
    PFB_MOUNT_OPTIONS = ['vers=3', 'udp', 'actimeo=1']
  end
else
  if ENV.has_key?("PFB_MOUNT_OPTIONS")
    PFB_MOUNT_OPTIONS = ENV.fetch("PFB_MOUNT_OPTIONS").split
  else
    PFB_MOUNT_OPTIONS = ["rw"]
  end
end
VAGRANT_NETWORK_OPTIONS = { auto_correct: false }

ROOT_VM_DIR = "/vagrant"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network :private_network, ip: ENV.fetch("PFB_PRIVATE_IP", "192.168.111.111")
  config.vm.synced_folder '.', ROOT_VM_DIR, type: PFB_SHARED_FOLDER_TYPE, mount_options: PFB_MOUNT_OPTIONS

  config.vm.provision "shell", path: 'install_pip.sh'
  config.vm.provision "shell", inline: 'pip install urllib3 pyopenssl ndg-httpsclient pyasn1'

  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "site.yml"
    ansible.verbose = true
    ansible.sudo = true
    ansible.raw_arguments = ["--timeout=60",
                             "--extra-vars",
                             "dev_user=#{ENV.fetch("USER", "vagrant")}"]
    ansible.install = true
    ansible.install_mode = "pip"
    ansible.version = "2.2.1.0"
  end
end
