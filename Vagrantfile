# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :lvm => {
        :box_name => "centos/7",
#	:box_version => "1804.02",
        :ip_addr => '192.168.56.3',
    :disks => {
        :sata1 => {
            :dfile => home + '/VirtualBox VMs/sata1.vdi',
            :size => 256,
            :port => 1
        },
        :sata2 => {
            :dfile => home + '/VirtualBox VMs/sata2.vdi',
            :size => 256, # Megabytes
            :port => 2
        },
        :sata3 => {
            :dfile => home + '/VirtualBox VMs/sata3.vdi',
            :size => 256, # Megabytes
            :port => 3
        },
        :sata4 => {
            :dfile => home + '/VirtualBox VMs/sata4.vdi',
            :size => 256,
            :port => 4
        },
        :sata5 => {
            :dfile => home + '/VirtualBox VMs/sata5.vdi',
            :size => 256,
            :port => 5
        },
        :sata6 => {
            :dfile => home + '/VirtualBox VMs/sata6.vdi',
            :size => 256,
            :port => 6
        },
        :sata7 => {
            :dfile => home + '/VirtualBox VMs/sata7.vdi',
            :size => 256,
            :port => 7
        },
        :sata8 => {
            :dfile => home + '/VirtualBox VMs/sata8.vdi',
            :size => 256,
            :port => 8
        }
    }
  },
}

Vagrant.configure("2") do |config|

 #   config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|
  
        config.vm.define boxname do |box|
  
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s
  
            #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
  
            box.vm.network "private_network", ip: boxconfig[:ip_addr]
  
            box.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "256"]
                    needsController = false
            boxconfig[:disks].each do |dname, dconf|
                unless File.exist?(dconf[:dfile])
                  vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                  needsController =  true
                            end
  
            end
                    if needsController == true
                       vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                       boxconfig[:disks].each do |dname, dconf|
                           vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                       end
                    end
            end
  
        box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
            #install zfs repo
            yum install -y http://download.zfsonlinux.org/epel/zfs-release.el7_8.noarch.rpm
            #import gpg key
            rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-zfsonlinux
            #install DKMS style packages for correct work ZFS
            yum install -y epel-release kernel-devel zfs
            #change ZFS repo
            yum-config-manager --disable zfs
            yum-config-manager --enable zfs-kmod
            yum install -y zfs
            #Add kernel module zfs
            modprobe zfs
            #install wget
            yum install -y wget
          SHELL
  
        end
    end
  end
  
