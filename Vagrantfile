ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
Vagrant.configure("2") do |config|
  # Базовая конфигурация ВМ.
  config.vm.box = "bento/ubuntu-24.04"
  config.vm.box_version = "1.0.0"
  config.vm.provider :virtualbox do |v|
    v.memory = 1512
    v.cpus = 2
  end

  # Создать две ВМ со статическими IP.
  boxes = [
    { :name => "backup",
      :ip => "192.168.56.160",
    },
    { :name => "client",
      :ip => "192.168.56.150",
    }
  ]
  # Настройки для каждой ВМ.
  boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.hostname = opts[:name]
      config.vm.network "private_network", ip: opts[:ip]
    end
  end
  # Настройки для ВМ backup.
  config.vm.define "backup" do |backup|
    backup.vm.provider "virtualbox" do |vb|
      vb.customize ["createhd",  "--filename", "disk2", "--size", "4096"]
      vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", "2", "--type", "hdd", "--medium", "disk2.vdi"]
      end
  end
  # Настройки ansible для каждой ВМ.
  config.vm.provision "ansible" do |ansible|
    ansible.verbose = "v"
    ansible.playbook = "playbook.yaml"
  end
end
