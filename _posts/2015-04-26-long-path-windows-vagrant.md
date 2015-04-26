---
layout: post
title: Long file path in VirtualBox on Windows using Vagrant
---

Windows by default limits the path length to 260 characters.
This became a problem for me when I shared a folder from VirtualBox containing a node_modules folder.
Nested dependencies can create some long paths ([a well known problem](https://github.com/joyent/node/issues/6960)) and we weren''t willing to use any workarounds in the code since using Vagrant should already abstract the developers OS away.
I found a solution in [issue#1953](https://github.com/mitchellh/vagrant/issues/1953) in the Vagrant repository.

So here is a vagrant file using the fix on Windows:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

VM_HOSTNAME = "long-path-vm"
SHARED_FOLDER_NAME = "app"
SHARED_FOLDER_LINUX_PATH = "/home/vagrant/app"
SHARED_FOLDER_WINDOWS_RELATIVE_PATH = ""
MAX_MEMORY = 1024

Vagrant.configure("2") do |config|
    config.vm.hostname = VM_HOSTNAME
    config.vm.box = "ubuntu/trusty64"

    unless Vagrant::Util::Platform.windows?
        config.vm.synced_folder ".", SHARED_FOLDER_PATH
    end

    config.vm.provider "virtualbox" do |v, override|
        v.name = VM_HOSTNAME
        v.memory = MAX_MEMORY

        if Vagrant::Util::Platform.windows?
            v.customize ["sharedfolder", "add", :id, "--name",
                SHARED_FOLDER_NAME,
                "--hostpath",(("//?/" + File.dirname(__FILE__)
                + SHARED_FOLDER_WINDOWS_RELATIVE_PATH )
                .gsub("/","\\"))]
            v.customize ["setextradata", :id,
                "VBoxInternal2/SharedFoldersEnableSymlinksCreate/"
                +SHARED_FOLDER_NAME, "1"]
            
            override.vm.provision :shell, inline: "mkdir -p "
                + SHARED_FOLDER_LINUX_PATH
            override.vm.provision :shell,
                inline: "mount -t vboxsf -o uid=`id -u vagrant`"
                + ",gid=`getent group vagrant | cut -d: -f3` "
                + SHARED_FOLDER_NAME + " "
                + SHARED_FOLDER_LINUX_PATH, run: "always"
        end
    end
end
```