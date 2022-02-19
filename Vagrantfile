# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :"backup" => {
        :box_name => "centos/7",
        :box_version => "1804.02",
        :ip_addr => '192.168.11.160',
        :net => [],
  :"disks" => {
        :sata1 => {
            :dfile => home + '/VirtualBoxHDD/VHDD/sata1.vdi',
            :size => 2048,
            :port => 1
#        },
#        :sata2 => {
#            :dfile => home + '/VirtualBoxHDD/VHDD/sata2.vdi',
#            :size => 1024, # Megabytes
#            :port => 2
      }
    }
  },
}

Vagrant.configure("2") do |config|

    config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|  
      config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      box.vm.network "private_network", ip: boxconfig[:ip_addr]

      #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset 

       # Additional network config if present
#     if boxconfig.key?(:net)
#        boxconfig[:net].each do |ipconf|
#        box.vm.network "private_network", ipconf
#     end
# end
        # Создаем память и диски 
        box.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "1024"]
        needsController = false
        boxconfig[:disks].each do |dname, dconf|
        unless File.exist?(dconf[:dfile])
        vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
        needsController =  true
    end  
end
         # Добавление контроллера и дисков в контроллер
      if needsController == true
         vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
         boxconfig[:disks].each do |dname, dconf|
         vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
      end
   end
end     
         # Добавление шаре папки и provision скриптов
         config.vm.synced_folder ".", "/vagrant",  
         type: "rsync",
         rsync_auto: "true",
         rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
         #config.vm.provision "shell", path: "provision/add-hdd.sh"

         # ssh-pub add in server
         config.vm.provision "shell", inline: <<-SHELL
         cat /vagrant/provision/ssh/vagrant-key.pub >> /home/vagrant/.ssh/authorized_keys
         SHELL
      end
   end
end

Vagrant.configure(2) do |client|
  client.vm.box = "centos/7"
  # ВТОРАЯ ВИРТУАЛЬНАЯ МАШИНА
    client.vm.define "backup-client" do |client|
    # имя виртуальной машины
    client.vm.box = 'centos/7'
    client.vm.provider "virtualbox" do |vb|
    vb.name = "backup-client"
end
    # hostname виртуальной машины
    client.vm.hostname = "backup-client"
    # настройки сети
    client.vm.network "private_network", ip: "192.168.11.150"
    client.vm.synced_folder ".", "/vagrant",  
          type: "rsync",
          rsync_auto: "true",
          rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
          #client.vm.provision "shell", path: "provision-client/nfs-client.sh"
    # ssh-pub add in server
    client.vm.provision "shell", inline: <<-SHELL
    cat /vagrant/provision/ssh/vagrant-key.pub >> /home/vagrant/.ssh/authorized_keys
    SHELL
  end
end
