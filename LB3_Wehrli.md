# Dokumentation LB3

## Theorie

### Container Virtualisierung

Container-Virtualisierung ist eine Methode zur Virtualisierung von Anwendungen, bei der eine isolierte Umgebung für eine Anwendung bereitgestellt wird, um ihre Ausführung und Verwaltung zu erleichtern. Container verwenden den Betriebssystem-Kernel des Host-Systems, um Ressourcen gemeinsam zu nutzen und die Effizienz zu maximieren. Im Gegensatz zu virtuellen Maschinen benötigen Container keine vollständige Kopie des Betriebssystems, was zu einer höheren Dichte und Skalierbarkeit führt. Container werden oft in Cloud-Umgebungen eingesetzt, um Anwendungen schnell bereitzustellen, zu skalieren und zu verwalten.

### Docker

Docker ist eine Open-Source-Software, mit der Anwendungen in Containern ausgeführt werden können. Mit Docker können Entwickler Anwendungen zusammen mit ihren Abhängigkeiten in einer standardisierten Umgebung verpacken und auf verschiedenen Systemen bereitstellen, ohne sich um Abhängigkeiten und Konfigurationsunterschiede sorgen zu müssen. Docker ist leichtgewichtig und erfordert weniger Ressourcen als andere Virtualisierungstechnologien. Es bietet auch eine einfache Möglichkeit, Anwendungen zu skalieren und zu verwalten.

### Prometheus

Prometheus ist ein Open-Source-System zur Überwachung von Anwendungen und Infrastrukturen. Es sammelt Metriken von verschiedenen Quellen, wie z.B. Anwendungen und Servern, und speichert sie in einer Datenbank. Die Daten können dann verwendet werden, um Leistungsprobleme zu identifizieren und Engpässe zu beseitigen. Prometheus unterstützt auch Alarmierungen, die automatisch ausgelöst werden, wenn bestimmte Schwellenwerte überschritten werden. Es ist ein leistungsstarkes Tool, das bei der Überwachung von Systemen sehr hilfreich sein kann, aber es erfordert ein gewisses Mass an technischem Wissen und Erfahrung, um effektiv eingesetzt zu werden.

## Praxis

#### Anmerkung

Die Idee für diese Projekt habe ich von Valerio Visisni und er hat mir auch geholfen.
Wie man unten in den Screenshots sieht habe ich dieses Projekt aber bei mir selber umgesetzt und die Umgebung erfolgreich eingerichtet.

### mysql-exporter
Zuerst habe ich das mysql-exporter yaml file erstellt. Der exporter ist selbsterklärend für den Export der Daten aus dem MYSQL Container zuständig.
<pre><code>
---
config:
  username: FCZ
  password: Passwort
  datasource: "tcp(db:3306)/mydb"
  collect.info_schema.INNODB_METRICS: "ON"
  collect.global_status: "ON"
  collect.global_variables: "ON"
</code></pre>

### docker Compose

Als zweiten Schritt habe ich dann das docker-compose yaml file erstellt.
<pre><code>
#Version von Docker Compose
version: '3' 

# Definitionen von den Services
services:
  # MySQL Container
  db:
    # image für MySQL
    image: mysql:latest
    
    # MySQL Port
    ports:
      - "3306:3306"
      
    # MYSQL user und database Informationen  
    environment:
      MYSQL_ROOT_PASSWORD: FCZ
      MYSQL_DATABASE: mydb
      MYSQL_USER: FCZ
      MYSQL_PASSWORD: Passwort
      
      
  # Prometheus Container
  prometheus:
    # image für Prometheus
    image: prom/prometheus:latest
    
    #Prometheus Port
    ports:
      - "9090:9090"
      
    volumes:
      # Konfigurationsdatei für Prometheus
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
  
  
  # MySQL Exporter Container
  mysqlexporter:
    image: prom/mysqld-exporter:latest
    
    # Exporter Port
    ports:
      - "9104:9104"
      
    environment:
      # Datenquelle für den MySQL Exporter
      DATA_SOURCE_NAME: root:secret@(db:3306)/mydb
      
    volumes:
      # Konfigurationsdatei des Exporters 
      - ./mysql-exporter.yml:/etc/mysql-exporter/config.yml
      
    command:
      #Konfiguration des Exporters
      - "--config.my-cnf=/etc/mysql-exporter/config.cnf"
      # Port freigabe
      - "--web.listen-address=0.0.0.0:9104"
</code></pre>

## prometheus 

Als letztes habe ich dann noch das prometheus.yml erstellt.
<code><pre>

global:
  # Refresh Interval
  scrape_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      # Prometheus Container
      - targets: ['localhost:9090']

  - job_name: mysql
    static_configs:
      # MySQL Exporter Container 
      - targets: ['mysqlexporter:9104']

</code></pre>






### Reflexion