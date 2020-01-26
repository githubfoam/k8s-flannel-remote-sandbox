# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "512"
    vb.cpus = 2 # not less than the required 2
  end

  config.vm.define "k8s-master01" do |k8scluster|
      k8scluster.vm.box = "bento/ubuntu-16.04"
      k8scluster.vm.hostname = "k8s-master01"
      k8scluster.vm.network "private_network", ip: "192.168.50.10"
      k8scluster.vm.network "forwarded_port", guest: 8001, host: 8001
      k8scluster.vm.provider "virtualbox" do |vb|
          vb.name = "k8s-master01"
          vb.memory = "4096"
      end
      k8scluster.vm.provision "ansible_local" do |ansible|
        ansible.playbook = "provisioning/deploy.yml"
        ansible.become = true
        ansible.compatibility_mode = "2.0"
        ansible.version = "2.9.4" # ubuntu-19.04s
        ansible.inventory_path = 'inventory'
      end
      k8scluster.vm.provision "shell", inline: <<-SHELL
      echo "===================================================================================="
                    hostnamectl status
      echo "===================================================================================="
      echo "         \   ^__^                                                                  "
      echo "          \  (oo)\_______                                                          "
      echo "             (__)\       )\/\                                                      "
      echo "                 ||----w |                                                         "
      echo "                 ||     ||                                                         "
      SHELL
    end

    config.vm.define "worker01" do |k8scluster|
        k8scluster.vm.box = "bento/ubuntu-16.04"
        k8scluster.vm.hostname = "worker01"
        k8scluster.vm.network "private_network", ip: "192.168.50.11"
        k8scluster.vm.provider "virtualbox" do |vb|
            vb.name = "worker01"
            vb.memory = "1024"
        end
        k8scluster.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "provisioning/deploy.yml"
          ansible.become = true
          ansible.compatibility_mode = "2.0"
          ansible.version = "2.9.4" # ubuntu-16.04
          ansible.inventory_path = 'inventory'
        end
        k8scluster.vm.provision "shell", inline: <<-SHELL
        echo "===================================================================================="
                      hostnamectl status
        echo "===================================================================================="
        echo "         \   ^__^                                                                  "
        echo "          \  (oo)\_______                                                          "
        echo "             (__)\       )\/\                                                      "
        echo "                 ||----w |                                                         "
        echo "                 ||     ||                                                         "
        SHELL
      end


      config.vm.define "worker02" do |k8scluster|
          k8scluster.vm.box = "bento/ubuntu-16.04"
          k8scluster.vm.hostname = "worker02"
          k8scluster.vm.network "private_network", ip: "192.168.50.12"
          k8scluster.vm.provider "virtualbox" do |vb|
              vb.name = "worker02"
              vb.memory = "1024"
          end
          k8scluster.vm.provision "ansible_local" do |ansible|
            ansible.playbook = "provisioning/deploy.yml"
            ansible.become = true
            ansible.compatibility_mode = "2.0"
            ansible.version = "2.9.4"
            ansible.inventory_path = 'inventory'
          end
          k8scluster.vm.provision "shell", inline: <<-SHELL
          echo "===================================================================================="
                        hostnamectl status
          echo "===================================================================================="
          echo "         \   ^__^                                                                  "
          echo "          \  (oo)\_______                                                          "
          echo "             (__)\       )\/\                                                      "
          echo "                 ||----w |                                                         "
          echo "                 ||     ||                                                         "
          SHELL
        end

        config.vm.define "remotecontrol01" do |k8scluster|
            k8scluster.vm.box = "bento/ubuntu-16.04" # OK
            k8scluster.vm.hostname = "remotecontrol01"
            k8scluster.vm.network "private_network", ip: "192.168.50.13"
            k8scluster.vm.network "forwarded_port", guest: 8001, host: 8002
            k8scluster.vm.provider "virtualbox" do |vb|
                vb.name = "remotecontrol01"
            end
            k8scluster.vm.provision "ansible_local" do |ansible|
              ansible.playbook = "provisioning/deploy.yml"
              ansible.become = true
              ansible.compatibility_mode = "2.0"
              ansible.version = "2.9.4" # ubuntu-16.04
              ansible.inventory_path = 'inventory'
            end
            k8scluster.vm.provision "shell", inline: <<-SHELL
            echo "===================================================================================="
                          hostnamectl status
            echo "===================================================================================="
            echo "         \   ^__^                                                                  "
            echo "          \  (oo)\_______                                                          "
            echo "             (__)\       )\/\                                                      "
            echo "                 ||----w |                                                         "
            echo "                 ||     ||                                                         "
            SHELL
          end

          config.vm.define "remotecontrol02" do |k8scluster|
              k8scluster.vm.box = "bento/centos-7.7" # OK
              k8scluster.vm.hostname = "remotecontrol02"
              k8scluster.vm.network "private_network", ip: "192.168.50.14"
              k8scluster.vm.network "forwarded_port", guest: 8001, host: 8003
              k8scluster.vm.provider "virtualbox" do |vb|
                  vb.name = "remotecontrol02"
              end
              k8scluster.vm.provision "ansible_local" do |ansible|
                ansible.playbook = "provisioning/deploy.yml"
                ansible.become = true
                ansible.compatibility_mode = "2.0"
                ansible.version = "2.9.2"
                ansible.inventory_path = 'inventory'
              end
              k8scluster.vm.provision "shell", inline: <<-SHELL
              # yum update -y && yum install -y python3 #centos7 python3
              echo "===================================================================================="
                            hostnamectl status
              echo "===================================================================================="
              echo "         \   ^__^                                                                  "
              echo "          \  (oo)\_______                                                          "
              echo "             (__)\       )\/\                                                      "
              echo "                 ||----w |                                                         "
              echo "                 ||     ||                                                         "
              SHELL
            end

end
