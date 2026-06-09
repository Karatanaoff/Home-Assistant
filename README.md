# Projet IoT : Contrôle d'un Arduino via Matter/Thread sur Home Assistant avec Accès Distant (Ngrok)

## 📝 Description du projet
Ce projet, réalisé dans un cadre scolaire au sein d'un lycée, a pour objectif de piloter un microcontrôleur **Arduino** à l'aide du protocole domotique moderne **Matter (via Thread)**. L'ensemble est centralisé sur un serveur **Home Assistant OS** virtualisé sous **Proxmox VE**.

Une contrainte majeure de ce projet était de rendre l'interface Home Assistant accessible de n'importe où (en 4G/extérieur), sans pouvoir modifier la configuration réseau du lycée (impossible d'ouvrir des ports sur le routeur de l'établissement ou de modifier le pare-feu Fortinet). Cette problématique réseau a été résolue grâce à la mise en place d'un tunnel sécurisé **Ngrok**.

---

## 🛠️ Architecture Technique
* **Matériel :** Arduino (compatible Matter), Clé USB d'extension Sonoff (OpenThread Border Router).
* **Serveur Domotique :** Home Assistant OS (installé sur une machine virtuelle Proxmox).
* **Réseau local :** Protocole Thread (Maillage local sécurisé).
* **Réseau externe :** Tunnel Ngrok (Bannissement des ouvertures de ports NAT/PAT).

---

## 🚀 Étapes de Configuration

### 1. Configuration du réseau Thread & Appairage Matter
Pour que l'Arduino puisse communiquer en local, la clé de bordure Sonoff (Border Router) a été configurée, et les anciens réseaux "fantômes" ont été isolés pour stabiliser le maillage.

1. **Identification du routeur actif :** Dans les configurations Thread de Home Assistant, repérage de la clé Sonoff active (identifiant unique `da398ba2f68b3be9`).
2. **Synchronisation des clés de sécurité :** Assignation de l'icône de sécurité (`Utilisé pour les identifiants Android + iOS`) sur la ligne de la clé active pour autoriser l'appairage avec le smartphone.
3. **Appairage de l'Arduino :** * Synchronisation des identifiants Thread depuis l'application *Home Assistant Compagnon* (`Paramètres > Application Compagnon > Dépannage > Synchroniser les identifiants Thread`).
   * Redémarrage électrique de l'Arduino pour vider le cache des anciens échecs.
   * Ajout de l'appareil via l'intégration Matter en renseignant le code texte fourni par la puce.

### 2. Contournement du Pare-feu du Lycée via Ngrok
Le pare-feu académique bloquant toutes les connexions entrantes, nous avons mis en place un tunnel *sortant* sécurisé qui traverse les protections sans altérer la sécurité du lycée.

1. **Création du compte Ngrok :** Réservation d'un nom de domaine statique et gratuit (ex: `squabble-playroom-spousal.ngrok-free.dev`) et récupération de l'Authtoken secret.
2. **Installation du module Ngrok sur Home Assistant :**
   * Ajout du dépôt de la communauté : `https://github.com/pssc/ha-addon-ngrok`
   * Installation du module complémentaire **ngrok**.
3. **Configuration du module :**
   * Renseignement du champ `auth_token`.
   * Renseignement du champ `hostname` (adresse Ngrok obtenue sans le `https://`) dans les options avancées du tunnel (accessible via l'icône de crayon sur la ligne du tunnel `hass`).
   * **Alignement de la région :** Suppression de la mention `eu` par défaut pour laisser Ngrok basculer sur la région `global` assignée par le tableau de bord web, évitant ainsi l'erreur `ERR_NGROK_3200 (Endpoint offline)`.
   * Activation des options *Lancer au démarrage* et *Chien de garde* (Watchdog).

### 3. Configuration de la sécurité interne de Home Assistant
Par défaut, Home Assistant bloque les connexions issues de serveurs mandataires (Reverse Proxies) comme Ngrok (Erreur `400: Bad Request`). Il a fallu déclarer explicitement les plages réseau internes du tunnel.

1. À l'aide de l'extension **File Editor**, modification du fichier `/config/configuration.yaml`.
2. Ajout du bloc de configuration suivant à la fin du fichier :

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - 172.30.32.0/24
    - 172.30.33.0/24
