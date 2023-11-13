# - mode: ruby -*-
# vi: set ft=ruby :

MACHINES = {
  :"packer-kernel6" => {
              #Какой vm box будем использовать
              :box_name => "centos8",
            }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|
    # Применяем конфигурацию ВМ
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
#      box.vm.host_name = boxname.to_s
      box.vm.provider "virtualbox" do |v|
        v.customize ["modifyvm", :id, "--memory", "1024"]
      end
    end
  end 
end
