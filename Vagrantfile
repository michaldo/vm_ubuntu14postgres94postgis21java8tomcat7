# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "forwarded_port", guest: 8080, host: 7070
  config.vm.network "forwarded_port", guest: 5432, host: 4432
  # dedicated folder due problem with permissions on ubuntu host: see http://jeremykendall.net/2013/08/09/vagrant-synced-folders-permissions/ 
  config.vm.synced_folder "logs/", "/tomcat-logs", mount_options:["dmode=777","fmode=666"], create:true

  config.vm.provider "virtualbox" do |v|
	v.memory = 2048
	v.cpus = 1
  end
  config.vm.provision "java8", type:"shell", inline: <<-SHELL
     sudo add-apt-repository ppa:webupd8team/java
     sudo apt-get update
     echo debconf shared/accepted-oracle-license-v1-1 select true | sudo debconf-set-selections
     echo debconf shared/accepted-oracle-license-v1-1 seen true | sudo debconf-set-selections

     sudo apt-get install -y oracle-java8-installer

     sudo apt-get install -y oracle-java8-set-default
  SHELL

  config.vm.provision "tomcat7", type:"shell", inline: <<-SHELL
     # Tomcat will find Java under that link
     ln -s /usr/lib/jvm/java-8-oracle /usr/lib/jvm/default-java

     apt-get install -y tomcat7 tomcat7-admin
     
     /etc/init.d/tomcat7 stop

     #export logs to host
     rm -rf /tomcat-logs/*
     rm /var/lib/tomcat7/logs
     ln -sf /tomcat-logs /var/lib/tomcat7/logs

     #enable admin
     mv /var/lib/tomcat7/conf/tomcat-users.xml /var/lib/tomcat7/conf/tomcat-users.xml.original
     echo '<?xml version="1.0" encoding="utf-8"?>' > /var/lib/tomcat7/conf/tomcat-users.xml
     echo '<tomcat-users><user username="tomcat" password="tomcat" roles="manager-gui,manager-script"/></tomcat-users>' >> /var/lib/tomcat7/conf/tomcat-users.xml
     
     /etc/init.d/tomcat7 start
  SHELL

  config.vm.provision "postgres94postgis21", type:"shell", inline: <<-SHELL
	 sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main" >> /etc/apt/sources.list'
	 wget --quiet -O - http://apt.postgresql.org/pub/repos/apt/ACCC4CF8.asc | sudo apt-key add -
     sudo apt-get update
	 sudo apt-get install -y postgresql-9.4-postgis-2.1
     echo "max_connections = 300 \n max_prepared_transactions = 300\n listen_addresses='*'\n" >>/etc/postgresql/9.4/main/postgresql.conf
	 echo "host    all             all             0.0.0.0/0            md5\n" >> /etc/postgresql/9.4/main/pg_hba.conf
     /etc/init.d/postgresql restart
	 sudo -u postgres psql -c"ALTER USER postgres WITH PASSWORD 'postgres'";
  SHELL


end
