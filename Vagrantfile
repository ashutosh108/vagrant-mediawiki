require 'zlib'
require 'rubygems/package'

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
  end

  sed_code = ""
  append_code = ""

  # mount all directories in extensions under the wiki
  tar_extract = Gem::Package::TarReader.new(Zlib::GzipReader.open('wiki.tar.gz'))
  tar_extract.each do |entry|
    if match = entry.full_name.match(/(^.*)\/LocalSettings.php$/)
      wiki_path_from_www_root = match.captures[0] # e.g. public_html/wiki
      Dir["extensions/*/"].each{|d|
        ext = d.chomp('/') # e.g. "extentions/SampleExtension"
        ext_basename = ext.sub(/^extensions\//, '')
        config.vm.synced_folder ext, "/var/www/#{wiki_path_from_www_root}/#{ext}"
        Dir[ext+"/extension.json"].each{|f|
            if File.file?(f)
                #enable extension in the LocalSettings.php
                sed_code += "-e '/^\\s*#*\\s*wfLoadExtension(\\s*\'#{ext_basename}\'\\s*);\\s*$/d' "
                append_code += "wfLoadExtension( '#{ext_basename}' );\n"
            end
        }
      }
      break
    end
  end
  tar_extract.close

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
    debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'
    PKGS="apache2 libapache2-mod-php mysql-client mysql-server php-mbstring php-xml php-mysql"
    apt-get -y --no-install-recommends install $PKGS

    a2enmod rewrite
    install -m=644 /vagrant/apache2.conf /etc/apache2/
    systemctl reload apache2 # required for php to see php-mbstring and php-xml installed

    find /var/www -mindepth 1 -xdev -delete 2>/dev/null
    tar -C /var/www -xzf /vagrant/wiki.tar.gz
    chown -R www-data: /var/www/public_html
    (cd /var/www && ln -s public_html html)
    patch_files() {
        for file; do
            sed -i $file \
                -e 's/^\\$wgServer\\s*=.*/$wgServer = "http:\\/\\/127.0.0.1:8080";/' \
                -e 's/^\\$wgDBuser\\s*=.*/$wgDBuser = "root";/' \
                -e 's/^\\$wgDBpassword\\s*=.*/$wgDBpassword = "root";/' \
                #{sed_code}
            echo -n "#{append_code}" >> $file
        done
    }
    export -f patch_files
    find /var/www -name LocalSettings.php -execdir bash -c 'patch_files {}' \\;
    echo "[client]
user=root
password=root" > ~/.my.cnf
    zcat /vagrant/wiki.sql.gz | mysql
  SHELL
end
