Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
  end
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
    debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'
    PKGS="apache2 libapache2-mod-php mysql-client mysql-server php-mbstring php-xml php-mysql"
    apt-get -y --no-install-recommends install $PKGS

    a2enmod rewrite
    install -m=644 /vagrant/apache2.conf /etc/apache2/
    systemctl reload apache2 # required for php to see php-mbstring and php-xml installed

	rm -fr /var/www/*
	tar -C /var/www -xzf /vagrant/wiki.tar.gz
	chown -R www-data: /var/www/public_html
	(cd /var/www && ln -s public_html html)
	find /var/www -name LocalSettings.php -execdir sed -i \
		-e 's/^\\$wgServer\\s*=.*/$wgServer = "http:\\/\\/127.0.0.1:8080";/' \
		-e 's/^\\$wgDBuser\\s*=.*/$wgDBuser = "root";/' \
		-e 's/^\\$wgDBpassword\\s*=.*/$wgDBpassword = "root";/' \
	{} \\;
	zcat /vagrant/wiki.sql.gz | mysql -uroot -proot
  SHELL
end
