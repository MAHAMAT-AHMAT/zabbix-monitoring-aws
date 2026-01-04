# Monitoring Zabbix sur AWS avec Docker

Ce projet présente le déploiement d'une infrastructure de supervision centralisée sur AWS. L'objectif est de surveiller un environnement hybride (Linux et Windows) en utilisant Zabbix déployé via Docker.

---

## Presentation
L’objectif principal est de collecter, centraliser et visualiser des métriques système (CPU, mémoire, disponibilité, etc.) à partir de plusieurs instances clientes via une interface web Zabbix unique.

## Technologies utilisees
* Cloud : AWS (EC2, VPC, Security Groups)
* Conteneurisation : Docker & Docker Compose
* Supervision : Zabbix Server & Zabbix Agents
* OS Clients : Ubuntu Server & Windows Server

---

## Architecture du Projet
L’infrastructure repose sur un réseau segmenté et sécurisé :
* Serveur Zabbix : Ubuntu + Docker.
* Clients : 1 Instance Ubuntu + 1 Instance Windows Server.
* Réseau : VPC avec sous-réseau public et règles de filtrage (Ports 80, 10050, 10051, 22, 3389).

![Vue globale de l'architecture](images/fig0.png)

---

## Deploiement de l'Infrastructure AWS

### 1. Creation du VPC
Mise en place de l'isolation réseau pour l'environnement de monitoring.

![Creation VPC 1](images/fig1.png)
![Creation VPC 2](images/fig2.png)
![Creation VPC 3](images/fig3.png)

### 2. Configuration Reseau et Securite
Configuration du sous-réseau public et des Security Groups pour autoriser le trafic Zabbix et les accès distants (SSH/RDP).

![Security Groups](images/fig4.png)
![Security Group Regles 2](images/fig5.png)
![Security Group Regles 3](images/fig6.png)
![Security Group Regles 4](images/fig7.png)
![Security Group Regles 5](images/fig8.png)
![Security Group Regles 6](images/fig9.png)
![Security Group Regles 7](images/fig10.png)
![Security Group Regles 8](images/fig11.png)
![Security Group Regles 9](images/fig12.png)
![Security Group Regles 10](images/fig17.png)

* Ces captures illustrent le filtrage des ports 80, 10050, 10051, 22 et 3389 pour assurer la communication entre le serveur et les agents. 



### 3. Instances EC2
Déploiement des trois piliers de l'infrastructure :

* Serveur Zabbix : ![Zabbix Server](images/fig18.png)
* Client Linux : ![Linux Client](images/fig19.png)
* Client Windows : ![Windows Client](images/fig20.png)

---

## Installation de Docker et Zabbix

### Installation des outils
![Installation Docker](images/fig21.png)

### Configuration Docker Compose
Voici le fichier docker-compose.yml utilise pour orchestrer la stack Zabbix (Serveur, Base de donnees Postgres et Interface Web) :

```yaml
services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: zabbix
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbixpass
    volumes:
      - db_data:/var/lib/postgresql/data
    restart: unless-stopped

  zabbix-server:
    image: zabbix/zabbix-server-pgsql:alpine-latest
    environment:
      DB_SERVER_HOST: db
      POSTGRES_DB: zabbix
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbixpass
    depends_on:
      - db
    ports:
      - "10051:10051"
    restart: unless-stopped

  zabbix-web:
    image: zabbix/zabbix-web-nginx-pgsql:alpine-latest
    environment:
      DB_SERVER_HOST: db
      POSTGRES_DB: zabbix
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbixpass
      ZBX_SERVER_HOST: zabbix-server
      PHP_TZ: Africa/Casablanca
    depends_on:
      - db
      - zabbix-server
    ports:
      - "80:8080"
    restart: unless-stopped

volumes:
  db_data: