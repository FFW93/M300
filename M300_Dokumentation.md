# M300 Dokumentation

## Theorie

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
![Image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-06%20151739.png)
## LB2
Bei der LB2 habe ich mich für das Projekt https://gitlab.com/ch-tbz-it/Stud/m300/M300/-/tree/master/vagrant/mmdb entschieden.
Hierfür habe ich zuerst das Vagrantfile und die sb.sh Datei heruntergeladen und mit Vagrant up eine neue Maschine erstellt:
![image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-20%20143203.png?raw=true)
Beim ersten Versuch kam wie im Screenshot gezeigt eine Fehlermeldung und es wurden nich alle VMs richtig erstellt.
Ich habe nicht herausgefunden an was es gelegen hat, denn als ich den Befehl ein zweites mal eingegeben habe funktionierte es einwandfrei.
![image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-20%201432031.png?raw=true)
![image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-20%20151513.png?raw=true)
