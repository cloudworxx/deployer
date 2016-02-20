Vagrant.require_version ">= 1.5.0"

Vagrant.configure("2") do |config|
    # Configure the box
    config.vm.box = "laravel/homestead"
    config.vm.hostname = "deployer"
    config.vm.box_check_update = true

    # Configure SSH
    config.ssh.forward_agent = true

    # Configure a private network IP
    config.vm.network :private_network, ip: "192.168.10.10"

    # Configure VirtualBox settings
    config.vm.provider "virtualbox" do |provider|
        provider.name = "deployer"
        provider.customize ["modifyvm", :id, "--memory", 2048]
        provider.customize ["modifyvm", :id, "--cpus", 1]
        provider.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        provider.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        provider.customize ["modifyvm", :id, "--ostype", "Ubuntu_64"]
    end

    # Configure port forwarding to the box
    config.vm.network "forwarded_port", guest: 80, host: 8000
    config.vm.network "forwarded_port", guest: 443, host: 44300
    config.vm.network "forwarded_port", guest: 3306, host: 33060

    config.vm.synced_folder "./", "/var/www/deployer"

    # Configure The Public Key For SSH Access
    config.vm.provision "shell" do |s|
        s.inline = "echo $1 | grep -xq \"$1\" /home/vagrant/.ssh/authorized_keys || echo $1 | tee -a /home/vagrant/.ssh/authorized_keys"
        s.args = [File.read(File.expand_path("~/.ssh/id_rsa.pub"))]
    end

    # Copy The SSH Private Keys To The Box
    config.vm.provision "shell" do |s|
        s.privileged = false
        s.inline = "echo \"$1\" > /home/vagrant/.ssh/id_rsa && chmod 600 /home/vagrant/.ssh/id_rsa"
        s.args = [File.read(File.expand_path("~/.ssh/id_rsa"))]
    end

    config.vm.provision "file", source: "~/.gitconfig", destination: "~/.gitconfig"
    config.vm.provision "file", source: "~/.composer/auth.json", destination: "~/.composer/auth.json"

    # Install mcrypt
    config.vm.provision "shell", inline: "sudo apt-get update"
    config.vm.provision "shell", inline: "sudo apt-get install php7.0-mcrypt -y"
    config.vm.provision "shell", inline: "sudo phpenmod mcrypt"

    # Remove postgresql
    config.vm.provision "shell", inline: "sudo apt-get remove postgresql-9.4 postgresql-client-9.4 postgresql-common -y"
    config.vm.provision "shell", inline: "sudo apt-get autoremove"
    config.vm.provision "shell", inline: "sudo rm -rf /var/log/postgresql"
    config.vm.provision "shell", inline: "sudo rm -rf /etc/postgresql-common"
    config.vm.provision "shell", inline: "sudo rm -rf /etc/postgresql"
    config.vm.provision "shell", inline: "sudo rm -rf /var/run/postgresql"
    config.vm.provision "shell", inline: "sudo rm -rf /var/lib/postgresql"

    # Install github changelog generator
    config.vm.provision "shell", inline: "sudo apt-get install ruby ruby-dev -y"
    config.vm.provision "shell", inline: "sudo gem install github_changelog_generator"

    # Create DB
    config.vm.provision "shell", inline: "mysql -uhomestead -psecret -e 'DROP DATABASE IF EXISTS deployer'"
    config.vm.provision "shell", inline: "mysql -uhomestead -psecret -e 'CREATE DATABASE deployer DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_unicode_ci'"

    # Install redis commander
    config.vm.provision "shell", inline: "sudo npm install -g redis-commander"

    # Install beanstalk console
    config.vm.provision "shell", inline: "[ ! -d /var/www/beanstalk ] && sudo composer create-project ptrofimov/beanstalk_console -q -n -s dev /var/www/beanstalk"
    config.vm.provision "shell", inline: "[ -d /var/www/html ] && sudo rm -rf /var/www/html"
    config.vm.provision "shell", inline: "sudo chown -R vagrant:vagrant /var/www/beanstalk"

    # Copy dev tools config
    config.vm.provision "shell", inline: "sudo cp /var/www/deployer/examples/dev/nginx.conf /etc/nginx/sites-available/tools.conf"
    config.vm.provision "shell", inline: "sudo ln -fs /etc/nginx/sites-available/tools.conf /etc/nginx/sites-enabled/tools.conf"
    config.vm.provision "shell", inline: "sudo cp /var/www/deployer/examples/dev/redis-commander.conf /etc/supervisor/conf.d/redis-commander.conf"

    # Copy deployer supervisor and cron config
    config.vm.provision "shell", inline: "sudo cp /var/www/deployer/examples/supervisor.conf /etc/supervisor/conf.d/deployer.conf"
    config.vm.provision "shell", inline: "sudo cp /var/www/deployer/examples/crontab /etc/cron.d/deployer"
    config.vm.provision "shell", inline: "sudo cp /var/www/deployer/examples/nginx.conf /etc/nginx/sites-available/deployer.conf"
    config.vm.provision "shell", inline: "sudo ln -fs /etc/nginx/sites-available/deployer.conf /etc/nginx/sites-enabled/deployer.conf"

    # Restart services
    config.vm.provision "shell", inline: "sudo service redis-server restart"
    config.vm.provision "shell", inline: "sudo service beanstalkd restart"
    config.vm.provision "shell", inline: "sudo service supervisor restart"
    config.vm.provision "shell", inline: "sudo service nginx restart"
    config.vm.provision "shell", inline: "sudo service cron restart"
    config.vm.provision "shell", inline: "sudo service php7.0-fpm restart"

    # Update composer and php-cs-fixer
    config.vm.provision "shell", inline: "sudo /usr/local/bin/composer self-update", run: "always"
end
