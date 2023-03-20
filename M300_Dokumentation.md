# M300 Dokumentation

## Theorie

### Github

GitHub ist eine webbasierte Plattform zur Versionsverwaltung von Softwareprojekten. Sie ermöglicht es Entwicklern, gemeinsam an Code zu arbeiten, Änderungen zu verfolgen, Pull-Requests zu erstellen und Reviews durchzuführen. GitHub bietet auch eine Vielzahl von Tools zur Zusammenarbeit und zur Verwaltung von Projekten, wie z.B. Issue-Tracking und Projekt-Boards. Es ist eine wichtige Ressource für die Open-Source-Community und wird von vielen Unternehmen und Entwicklern genutzt, um ihre Arbeit zu teilen und zu verbessern.

### Visual Studio Code
Visual Studio Code ist ein Open-Source-Code-Editor den man für fast alle Programmiersprachen benutzen kann.
Auch gibt es viele weitere funktionen wie die zum Beispiel die integrierung von von git, die ich im folgenden Screenshot aufgezeige:
![image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-20%20154225.png)
### Virtualbox
Virtualbox ist eine Virtualisierungssoftware.
### Vagrant
Vagrant ist eine Open-Source Software, mit der man das erstellen und konfigurieren von VMs automatisieren kann.
Dafür erstellt man ein Vagrantfile.
Ein Vagrantfile ist eine Konfigurationsdatei in einer eigenen Domain-spezifischen Sprache (DSL), die die Spezifikationen einer Vagrant-Umgebung definiert. Es enthält Informationen wie die gewünschte Box, Netzwerkkonfigurationen, Speichereinstellungen, Skripte zur automatischen Einrichtung der VM und andere Details, die Vagrant benötigt, um eine VM zu erstellen und zu konfigurieren.
Hier ein Beispiel für den Inhalt eines solchen files:
<pre><code> 
Vagrant.configure("2") do |config|
        config.vm.define :apache do |web|
            web.vm.box = "bento/ubuntu-16.04"
            web.vm.provision :shell, path: "config_web.sh"
            web.vm.hostname = "srv-web"
            web.vm.network :forwarded_port, guest: 80, host: 4567
            web.vm.network "public_network", bridge: "en0: WLAN (AirPort)"
        end
</code></pre>
Der Zweite wichtige Bestandteil von Vagrant sind Boxen.
Vagrant Boxen sind vordefinierte virtuelle Maschinen (VMs), die als Basis für die Erstellung neuer VMs dienen. Eine Box ist eine einzige Datei, die ein vorinstalliertes Betriebssystem und eine Reihe von vordefinierten Anwendungen enthält. Mit einer Box können Entwickler schnell und einfach VMs erstellen, ohne das Betriebssystem und die Anwendungen manuell installieren zu müssen.

# Umsetzung
## 10 Toolumgebung
Das Installieren der Tools wie Virtualbox, Vagrant und git habe ich nach der Anleitung gemacht und es ging Problemlos vonstatten.
## 20 Infrastruktur-Automatisierung 
Hier ein Screenshot von der erstellung der esten Vagrant VM:
![Image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-06%20151739.png)
## LB2
Bei der LB2 habe ich mich für das Projekt https://gitlab.com/ch-tbz-it/Stud/m300/M300/-/tree/master/vagrant/mmdb entschieden.
Hier die Vagrantfiles die ich dafür benutzt habe:

<pre><code>
# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.define "database" do |db|
    db.vm.box = "ubuntu/bionic64"
	db.vm.provider "virtualbox" do |vb|
	  vb.memory = "512"  
	end
    db.vm.hostname = "db01"
    db.vm.network "private_network", ip: "192.168.55.100"
    # MySQL Port nur im Private Network sichtbar
	# db.vm.network "forwarded_port", guest:3306, host:3306, auto_correct: false
  	db.vm.provision "shell", path: "db.sh"
  end
  
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/bionic64"
    web.vm.hostname = "web01"
    web.vm.network "private_network", ip:"192.168.55.101" 
	web.vm.network "forwarded_port", guest:80, host:8080, auto_correct: true
	web.vm.provider "virtualbox" do |vb|
	  vb.memory = "512"  
	end     
  	web.vm.synced_folder ".", "/var/www/html"  
	web.vm.provision "shell", inline: <<-SHELL
		sudo apt-get update --fix-missing
		sudo apt-get -y install debconf-utils apache2 nmap
		sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password admin'
		sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password admin'
		sudo apt-get -y install php libapache2-mod-php php-curl php-cli php-mysql php-gd mysql-client  
		# Admininer SQL UI 
		sudo mkdir /usr/share/adminer
		sudo wget "http://www.adminer.org/latest.php" -O /usr/share/adminer/latest.php
		sudo ln -s /usr/share/adminer/latest.php /usr/share/adminer/adminer.php
		echo "Alias /adminer.php /usr/share/adminer/adminer.php" | sudo tee /etc/apache2/conf-available/adminer.conf
		sudo a2enconf adminer.conf 
		sudo service apache2 restart 
	  echo '127.0.0.1 localhost web01\n192.168.55.100 db01' > /etc/hosts
SHELL
	end  
 end

</code></pre>

<pre><code>

#!/bin/bash
#
#	Datenbank installieren und Konfigurieren
#

# root Password setzen, damit kein Dialog erscheint und die Installation haengt!
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password S3cr3tp4ssw0rd'
sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password S3cr3tp4ssw0rd'

# Installation
sudo apt-get update --fix-missing
sudo apt-get install -y mysql-server

# MySQL Port oeffnen
sudo sed -i -e"s/^bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/mysql.conf.d/mysqld.cnf

# User fuer Remote Zugriff einrichten - aber nur fuer Host web 192.168.55.101
mysql -uroot -pS3cr3tp4ssw0rd <<%EOF%
	CREATE USER 'root'@'192.168.55.101' IDENTIFIED BY 'admin';
	GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.55.101';
	FLUSH PRIVILEGES;
%EOF%

# Restart fuer Konfigurationsaenderung
sudo service mysql restart
</code></pre>

Mit dem Befehl "Vagrant up" habe ich dann eine neue Maschine erstellt:
![image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-20%20143203.png?raw=true)

Beim ersten Versuch kam wie im Screenshot gezeigt eine Fehlermeldung und es wurden nicht alle VMs richtig erstellt.

Ich habe nicht herausgefunden an was es gelegen hat, denn als ich den Befehl ein zweites mal eingegeben habe funktionierte es einwandfrei.

### Tests

1. Als erstes habe ich überprüft ob die VMs auch wikrlich erstellt wurden:
![image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-20%201432031.png?raw=true)

Dies ist wie man im Screenshot sehen kann gelungen.

2. Der zweite Test ging dann um die erreichbarkeit des Webservers bzw. der DB
![image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-20%20151513.png?raw=true)

Dies ist wie man im Screenshot sehen kann, ebenfalls gelungen.

Als weiteren Test habe ich dann noch versucht mich einzuloggen.
Dies hat nicht funktioniert. 
Ich konnte aus Zeitgründen noch nicht herausfinden warum dies nicht funktioniert hat.
![image]()