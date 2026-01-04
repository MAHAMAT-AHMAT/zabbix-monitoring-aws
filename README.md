# Monitoring Zabbix sur AWS avec Docker
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

![Vue globale de l'architecture](Images/fig0.png)

---

## Deploiement de l'Infrastructure AWS

### 1. Creation du VPC
Mise en place de l'isolation réseau pour l'environnement de monitoring.

![Creation VPC 1](Images/fig1.png)
![Creation VPC 2](Images/fig2.png)
![Creation VPC 3](Images/fig3.png)

### 2. Configuration Reseau et Securite
Configuration du sous-réseau public et des Security Groups pour autoriser le trafic Zabbix et les accès distants (SSH, RDP, HTTP).

![Security Group Regles 1](Images/fig4.png)
![Security Group Regles 2](Images/fig5.png)
![Security Group Regles 3](Images/fig6.png)
![Security Group Regles 4](Images/fig7.png)
![Security Group Regles 5](Images/fig8.png)
![Security Group Regles 6](Images/fig9.png)
![Security Group Regles 7](Images/fig10.png)
![Security Group Regles 8](Images/fig11.png)
![Security Group Regles 9](Images/fig12.png)
![Security Group Regles 10](Images/fig17.png)

### 3. Instances EC2
Déploiement des trois piliers de l'infrastructure :

* Serveur Zabbix : ![Zabbix Server](Images/fig18.png)
* Client Linux : ![Linux Client](Images/fig19.png)
* Client Windows : ![Windows Client](Images/fig20.png)

---

## Installation de Docker et Zabbix

### Installation des outils
![Installation Docker](Images/fig21.png)

### Configuration Docker Compose
Voici le fichier docker-compose.yml utilise :

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

```

---

### Lancement des conteneurs Zabbix
Une fois le fichier docker-compose configuré, les services sont démarrés.
![Démarrage des services](Images/fig28.png)

### Interface Web fonctionnelle
L'accès à l'interface d'administration confirme le bon fonctionnement de la stack.
![Interface Web 1](Images/fig26.png)
![Interface Web 2](Images/fig27.png)

### Installation des Agents (Focus Windows)
Preuve du bon déploiement de l'agent Zabbix sur l'instance Windows Server.
![Agent Windows](Images/fig31.png)

---

##  Monitoring et Résultats

### Configuration des hôtes
Ajout des clients Linux et Windows dans la console Zabbix.
![Ajout Hotes 1](Images/fig29.png)
![Ajout Hotes 2](Images/fig30.png)

### Statut opérationnel (ZBX au vert)
Les indicateurs confirment que la communication entre le serveur et les agents est établie.
![Agents Opérationnels](Images/fig32.png)

### Collecte des données et Graphiques
Visualisation des métriques de performance CPU en temps réel.
![Données CPU](Images/fig35.png)

**Graphiques d'utilisation :**
![Graphique CPU 1](Images/fig33.png)
![Graphique CPU 2](Images/fig34.png)

---
