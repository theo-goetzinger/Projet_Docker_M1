## Théo Goetzinger

## Projet Master 1 Docker 

## présentation du projet

```

Etant en cybersécurité, j'ai décidé de m'intéresser au monitoring avec la technologie docker. 
Pour effectuer mon projet j'ai décider de déployer une stack de monitoring comprenant un container Telegraf qui permet de collecter les données systèmes, un container InfluxDB qui sera la base de données, et un container Grafana afin d'avoir des dashboards des différentes métriques. 

Pour cela j'ai donc réaliser un fichier docker-compose.yml afin de déployer la stack 

```

## docker-compose.yml : 

```
version: "3"
services:
  influxdb:
    image: influxdb:latest
    container_name: influxdb
    restart: always
    hostname: influxdb
    networks:
      - lan
    ports:
      - 8086:8086
    environment:
      INFLUX_DB: "telegraf"  # nom de la base de données créée à l'initialisation d'InfluxDB
      INFLUXDB_USER: "telegraf_user" # nom de l'utilisateur pour gérer cette base de données
      INFLUXDB_USER_PASSWORD: "telegraf_password"  #mot de passe de l'utilisateur pour gérer cette base de données
    volumes:
      - influxdb-data:/var/lib/influxdb # volume pour stocker la base de données InfluxDB

  telegraf:
    image: telegraf:latest
    depends_on:
      - influxdb # indique que le service influxdb est nécessaire
    container_name: telegraf
    restart: always
    networks:
      - lan
    links:
      - influxdb:influxdb
    tty: true
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./telegraf/telegraf.conf:/etc/telegraf/telegraf.conf # fichier de configuration de Telegraf

    

  grafana:
    image: grafana/grafana:latest
    depends_on:
      - influxdb # indique que le service influxdb est nécessaire
    container_name: grafana
    restart: always
    networks:
      - lan
    ports:
      - 3000:3000 # port pour accéder à l'interface web de Grafana
    links:
      - influxdb:influxdb
    environment:
      GF_INSTALL_PLUGINS: "grafana-clock-panel,\
                          grafana-influxdb-08-datasource,\
                          grafana-kairosdb-datasource,\
                          grafana-piechart-panel,\
                          grafana-simple-json-datasource,\
                          grafana-worldmap-panel"
      GF_SECURITY_ADMIN_USER: "grafana_user" # nom de l'utilisateur créé par défaut pour accéder à Grafana

      GF_SECURITY_ADMIN_PASSWORD: "grafana_password" # mot de passe de l'utilisateur créé par défaut pour accéder à Grafana
    volumes:
    volumes:
      - grafana-data:/var/lib/grafana
volumes:
  influxdb-data:
  grafana-data:

networks:
  lan:

  ```

## Variables d'environnements
```
ayant intégrer des varibales d'environnement dans le docker-compose, il a fallut les déclarer dans un fichier .env

```
## .env 

```
INFLUX_DB=telegraf
INFLUXDB_USER=telegraf_user
INFLUXDB_USER_PASSWORD=telegraf_password
GF_SECURITY_ADMIN_USER=grafana_user
GF_SECURITY_ADMIN_PASSWORD=grafana_password

```


##  Configuration du fichier telegraf.conf

```
Avant de lancer la stack, il est nécessaire de réaliser quelques modification dans le fichier de configuration de telegraf. Par exemple : 

hostname = "telegraf"
urls = ["http://influxdb:8086"]
database = "telegraf"
username = "telegraf_user"
password = "telegraf_password"
endpoint = "unix:///var/run/docker.sock"

```

## lancement de la stack 

```
docker-compose up -d 
Creating influxdb ... done
Creating grafana  ... done
Creating telegraf ... done

```
## Accès à Grafana 

```

On accède ensuite à grafana depuis un navigateur avec l'adresse ip de notre container docker et le port 3000

http://172.25.0.4:3000

on s'identifie avec les identifiants précédement défini dans le fichier .env

```

## Problème rencontrés 

Suite à la mise en place de la stack, aucun problème n'a été rencontré dans la partie mise en place et déploiement de celle ci. 

En revanche, arriver sur l'interface web au moment d'ajouter un datasource, qui est la base de données InfluxDb, impossible de réaliser l'action. 
celle ci me retournais une error 401. 
J'ai donc commencer par vérifier mon docker-compose mais aucun soucis à ce niveau la. Puis j'ai regarder le fichier telegraf.conf et pareil la configuration était bonne.
Je me suis donc interessé aux droits sur de telegraf sur la bdd et aux credentials sans réussir à résoudre mon problème.

