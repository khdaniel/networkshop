## раскоментируйте если вам не доступен ванльный репозиторий vagrant
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/jammy64" # Базовый образ
    config.vm.boot_timeout = 600
    config.vm.box_download_insecure=true
    config.vm.synced_folder ".", "/vagrant", disabled: true

    # Мапа с настройками CPU и памяти для разных машин
    machine_resources = {
        "router" => { "cpus" => 1, "memory" => 512 },
        "client" => { "cpus" => 1, "memory" => 512 },
        "office" => { "cpus" => 1, "memory" => 512 },
    }

    # Задайте правильные адреса на внешних и внутренних интерфейсах роутеров
    # формат:  "имя ВМ" => { "int_ip" => "Адрес в сети Офиса", "ext_ip" => "Адрес в сети для WG", "vagrant_ip" => "Адрес для работы с ансиблом" },
    routers = {
        "router-1" => { "int_ip" => "192.168.101.1/24", "ext_ip" => "192.168.100.1/24", "vagrant_ip" => "192.168.110.1/24" },
        "router-2" => { "int_ip" => "192.168.102.1/24", "ext_ip" => "192.168.100.2/24", "vagrant_ip" => "192.168.110.2/24" },
        "router-3" => { "int_ip" => "192.168.103.1/24", "ext_ip" => "192.168.100.3/24", "vagrant_ip" => "192.168.110.3/24" }, 
        "router-4" => { "int_ip" => "192.168.104.1/24", "ext_ip" => "192.168.100.4/24", "vagrant_ip" => "192.168.110.4/24" }
    }

    routers.each do |name, ip_addr|
        config.vm.define name do |router|
            router.vm.hostname = name
            router.vm.network "private_network", ip: ip_addr["int_ip"], virtualbox__intnet: name  # Сеть для хоста 
            router.vm.network "private_network", ip: ip_addr["ext_ip"], virtualbox__intnet: "net-router"  # Промежуточная сеть
            router.vm.network "private_network", ip: ip_addr["vagrant_ip"] # Сеть для работы ансибла как провиженера
            # router.vm.provision "ansible" do |ansible|
            #     ansible.playbook = "ansible/routers.playbook.yml"
            #     ansible.inventory_path = "ansible/inventory.yml"
            #     ansible.extra_vars = { "hostname" => name }
            # end
            router.vm.provider "virtualbox" do |vb|
                vb.name = name
                vb.memory = machine_resources["router"]["memory"]
                vb.cpus = machine_resources["router"]["cpus"]
            end
        end
    end

    # Задайте правильные адреса для представителей офисов
    # формат:  "имя ВМ" => { "ip" => "адрес в сети офиса", "name" => "имя сети к которой подключается (к какому роутеру)", "getaway" => "адрес офисного роутера который открывает выход в впн сети" },
    offices = {
        "office-1" => { "ip" => "192.168.101.2/24", "name" => "router-1", "getaway" => "192.168.101.1" },
        "office-2" => { "ip" => "192.168.102.2/24", "name" => "router-2", "getaway" => "192.168.102.1" },
        "office-3" => { "ip" => "192.168.103.2/24", "name" => "router-3", "getaway" => "192.168.103.1" }
    }

    offices.each do |name, network|
        config.vm.define name do |office|
            office.vm.hostname = name
            office.vm.network "private_network", ip: network["ip"], virtualbox__intnet: network["name"]  # Сеть для хоста 
            office.vm.provider "virtualbox" do |vb|
                vb.name = name
                vb.memory = machine_resources["office"]["memory"]
                vb.cpus = machine_resources["office"]["cpus"]
            end
            # # Настройка маршрута для доступа через роутер 
            #     office.vm.provision "shell", inline: <<-SHELL
            #     sudo ip route add <Сеть Wireguard которой связаны роутеры> via #{network["getaway"]}
            #     sudo ip route add <сеть user2sie vpn / openvpn> via #{network["getaway"]}
            # SHELL
        end
    end

    #  OpenVPN клиент
    config.vm.define "client" do |client|
        client.vm.hostname = "client"
        client.vm.network "private_network", ip: "192.168.104.2", virtualbox__intnet: "router-4" # 
        client.vm.network "private_network", ip: "192.168.110.5" # Сеть для работы ансибла как провиженера
        # client.vm.provision "ansible" do |ansible|
        #     ansible.playbook = "ansible/client.playbook.yml"
        # end
        client.vm.provider "virtualbox" do |vb|
            vb.name = "client"
            vb.memory = machine_resources["client"]["memory"]
            vb.cpus = machine_resources["client"]["cpus"]
        end
        # Ставим дополнтельные утилиты для проверки маршрутизации
        client.vm.provision "shell", inline: <<-SHELL
            sudo apt install -y traceroute mtr
        SHELL
    end
  
end    