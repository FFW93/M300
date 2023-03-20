# M300 Dokumentation

## Theorie

### Virtualbox
Virtualbox ist eine Virtualisierungssoftware.
### Vagrant
Vagrant ist eine Open-Source Software, mit der man das erstellen und konfigurieren von VMs automatisieren kann.
Dafür erstellt man ein Vagrantfile in der man die gewünschten konfigurationen festlegt.
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
</pre></code>

# Umsetzung
1. Tools Installion
    Das Installieren der Tools wie Virtualbox, Vagrant und git habe ich nach der Anleitung gemacht und es ging Problemlos vonstatten.
2. Erste VM mit Vagrant: 
    ![Image](https://github.com/FFW93/M300/blob/main/Bilder/Screenshot%202023-03-06%20151739.png)

