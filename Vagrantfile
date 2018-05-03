# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

vagrantConfig = YAML.load_file 'config.yaml'

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "magento-server"
  config.vm.network "private_network", ip: vagrantConfig['vm']['ip']

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = vagrantConfig['vm']['memory']
    vb.cpus = vagrantConfig['vm']['cpus']
  end
  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get install -y whois git
    sudo apt-get install python-software-properties
    sudo add-apt-repository ppa:ondrej/php
    sudo apt-get update
    sudo apt-get install nginx -y
    sudo update-rc.d nginx defaults
    curl https://raw.githubusercontent.com/robinhuy/static-files/master/nginx/magento-vagrant/magento2.conf > /etc/nginx/sites-enabled/magento
    sudo rm /etc/nginx/sites-enabled/default
    sudo apt-get install php5.6-fpm php5.6-mcrypt php5.6-mbstring php5.6-zip php5.6-curl php5.6-cli php5.6-mysql php5.6-imagick php5.6-gd php5.6-xsl php5.6-json php5.6-intl php5.6-dev php5.6-common libcurl3 curl -y
    sudo rm /etc/php/5.6/fpm/php.ini
    sudo rm /etc/php/5.6/cli/php.ini
    curl https://raw.githubusercontent.com/robinhuy/static-files/master/php/magento/php.ini > /etc/php/5.6/fpm/php.ini
    sudo cp /etc/php/5.6/fpm/php.ini /etc/php/5.6/cli/php.ini
    sudo service php5.6-fpm restart
    export DEBIAN_FRONTEND="noninteractive"
    sudo debconf-set-selections <<< "mysql-server mysql-server/root_password password #{vagrantConfig['db']['root_pass']}"
    sudo debconf-set-selections <<< "mysql-server mysql-server/root_password_again password #{vagrantConfig['db']['root_pass']}"
    sudo apt-get install -y mysql-server-5.6
    mysql_secure_installation
    sudo mysql --user=root --password=#{vagrantConfig['db']['root_pass']} -e "CREATE DATABASE #{vagrantConfig['db']['db_name']};"
    cd ~/ && curl -sS https://getcomposer.org/installer | php && sudo mv composer.phar /usr/bin/composer
    cd /home/vagrant
    sudo wget https://github.com/magento/magento2/archive/2.1.0.tar.gz
    sudo tar -xzvf 2.1.0.tar.gz
    sudo rm -rf 2.1.0.tar.gz
    sudo mv magento2-2.1.0 magento2
    sudo mkdir /root/.composer
    echo '{\n\t"http-basic": {\n\t\t"repo.magento.com": {\n\t\t\t"username": "#{vagrantConfig['repo_magento_com']['username']}",\n\t\t\t\"password": "#{vagrantConfig['repo_magento_com']['password']}"\n\t\t}\n\t}\n}' > auth.json
    sudo mv auth.json /root/.composer/
    cd magento2 && sudo composer install -v
    sudo chmod u+x bin/magento
    php bin/magento setup:install --backend-frontname="#{vagrantConfig['magento']['backend_frontname']}" \
    --db-host="localhost" \
    --db-name="#{vagrantConfig['db']['db_name']}" \
    --db-user="root" \
    --db-password="#{vagrantConfig['db']['root_pass']}" \
    --language="#{vagrantConfig['magento']['language']}" \
    --currency="#{vagrantConfig['magento']['currency']}" \
    --timezone="#{vagrantConfig['magento']['timezone']}" \
    --use-rewrites=1 \
    --use-secure=0 \
    --base-url="#{vagrantConfig['magento']['base_url']}" \
    --admin-user="#{vagrantConfig['magento']['admin_user']}" \
    --admin-password="#{vagrantConfig['magento']['admin_password']}" \
    --admin-email="#{vagrantConfig['magento']['admin_email']}" \
    --admin-firstname="#{vagrantConfig['magento']['admin_firstname']}" \
    --admin-lastname="#{vagrantConfig['magento']['admin_lastname']}" \
    --cleanup-database
    sudo chown -R vagrant:www-data /home/vagrant/magento2
    sudo chmod g+s /home/vagrant/magento2/var
    sudo service nginx restart
  SHELL
end
