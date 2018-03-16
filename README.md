= Vagrant config to play with your mediawiki locally =

This is a Vagrantfile + necessary helper files to make sure you can esaily set up local Vagrant copy of mediawiki.

How to use:

0. Create wiki.tar.gz and wiki.sql.gz from your existing (remote) mediawki installation:
    tar czf wiki.tar.gz public_html
    mysql -u<mysql_user_name> -p --databases <mysql_db_name> | gzip > wiki.sql.gz

1. `git clone https://github.com/ashutosh108/vagrant-mediawiki.git`
2. cd vagrant-mediawiki`
3. Copy wiki.tar.gz and wiki.sql.gz to your cloned repository (wiki.tar.gz and wiki.sql.gz are already in the .gitignore file)
4. `vagrant up`
5. Your new mediawiki should be available at http://127.0.0.1:8080/
