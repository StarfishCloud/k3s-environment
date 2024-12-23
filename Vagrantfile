Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"
  config.vm.network "public_network"

  public_key_path = File.expand_path("~/.ssh/id_rsa.pub")
  public_key_content = File.read(public_key_path)

  config.vm.define "k3s-master01" do |master01|
    master01.vm.hostname = "k3s-master01"
    master01.vm.network "private_network", ip: "192.168.56.101" # IP privado
    master01.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
    master01.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
      sudo apt-get upgrade -y
      sudo apt-get install -y curl

      mkdir -p ~/.ssh
      sudo echo "#{public_key_content}" >> ~/.ssh/authorized_keys
    SHELL
  end

  config.vm.define "k3s-master02" do |master02|
    master02.vm.hostname = "k3s-master02"
    master02.vm.network "private_network", ip: "192.168.56.102" # IP privado
    master02.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
    master02.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
      sudo apt-get upgrade -y
      sudo apt-get install -y curl

      mkdir -p ~/.ssh
      sudo echo "#{public_key_content}" >> ~/.ssh/authorized_keys
    SHELL
  end

  config.vm.define "k3s-master03" do |master03|
    master03.vm.hostname = "k3s-master03"
    master03.vm.network "private_network", ip: "192.168.56.103" # IP privado
    master03.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
    master03.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
      sudo apt-get upgrade -y
      sudo apt-get install -y curl

      mkdir -p ~/.ssh
      echo "#{public_key_content}" >> ~/.ssh/authorized_keys
      chmod 600 ~/.ssh/authorized_keys
      chmod 700 ~/.ssh
    SHELL
  end

  config.vm.define "k3s-worker01" do |worker01|
    worker01.vm.hostname = "k3s-worker01"
    worker01.vm.network "private_network", ip: "192.168.56.201" # IP privado
    worker01.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
    worker01.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
      sudo apt-get upgrade -y
      sudo apt-get install -y curl
      mkdir -p ~/.ssh
      sudo echo "#{public_key_content}" >> ~/.ssh/authorized_keys
    SHELL
  end

  config.vm.define "k3s-worker02" do |worker02|
    worker02.vm.hostname = "k3s-worker02"
    worker02.vm.network "private_network", ip: "192.168.56.202" # IP privado
    worker02.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
      vb.cpus = 1
    end
    worker02.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
      sudo apt-get upgrade -y
      sudo apt-get install -y curl

      mkdir -p ~/.ssh
      sudo echo "#{public_key_content}" >> ~/.ssh/authorized_keys
    SHELL
  end

    # Configuração da VM do PostgreSQL
    config.vm.define "postgres" do |postgres|
      postgres.vm.hostname = "postgres"
      postgres.vm.network "private_network", ip: "192.168.56.50"
  
      postgres.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 1
      end
  
      postgres.vm.provision "shell", inline: <<-SHELL
        # Instalar PostgreSQL
        sudo apt-get update -y
        sudo apt-get install -y postgresql postgresql-contrib
  
        # Configurar o PostgreSQL
        sudo -u postgres psql -c "CREATE DATABASE kubernetes;"
        sudo -u postgres psql -c "CREATE USER k3s_user WITH PASSWORD 'k3s_password';"
        sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE kubernetes TO k3s_user;"
  
        # Configurar o PostgreSQL para aceitar conexões externas
        echo "listen_addresses = '*'" | sudo tee -a /etc/postgresql/13/main/postgresql.conf
        echo "host all all 0.0.0.0/0 md5" | sudo tee -a /etc/postgresql/13/main/pg_hba.conf
  
        # Reiniciar o PostgreSQL
        sudo systemctl restart postgresql
        
        mkdir -p ~/.ssh
        sudo echo "#{public_key_content}" >> ~/.ssh/authorized_keys
      SHELL
    end
end
