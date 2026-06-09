# 🏠 Projet IoT : Contrôle d'un Arduino via Matter/Thread sur Home Assistant avec Accès Distant (Ngrok)

## 📝 Description du projet
Ce projet, réalisé dans un cadre scolaire, explique comment créer un serveur domotique complet en partant de zéro, pour piloter un microcontrôleur **Arduino** avec le protocole **Matter (via Thread)**. 

L'objectif final est de rendre cette installation accessible depuis l'extérieur (en 4G), sans ouvrir de ports sur la box internet et en contournant les pare-feux stricts (comme le filtrage Fortinet d'un réseau de lycée).

---

## 🛠️ Matériel Requis
* Un PC ou mini-PC dédié pour servir de serveur (ex: Intel NUC, vieux PC portable).
* Une clé USB classique (8 Go minimum) pour l'installation.
* Une clé USB domotique **Sonoff Zigbee 3.0 USB Dongle Plus** (modèle "E" basé sur puce EFR32MG21).
* Un microcontrôleur compatible Matter (ex: Arduino Nano Matter).
* Un smartphone avec l'application Home Assistant installée.

---

## 🚀 ÉTAPE 1 : Installation du serveur Proxmox VE

Proxmox est un système d'exploitation (hyperviseur) qui permet de faire tourner des machines virtuelles. C'est lui qui va héberger notre serveur domotique.

**1. Préparer la clé USB d'installation :**
* Téléchargez l'image ISO de **Proxmox VE** sur le site officiel.
* Téléchargez le logiciel gratuit **Rufus**.
* Insérez votre clé USB vierge dans votre PC Windows.
* Ouvrez Rufus, sélectionnez votre clé USB, choisissez l'ISO de Proxmox que vous venez de télécharger, et cliquez sur **Démarrer** (choisissez le mode "DD" si Rufus le demande).

**2. Installer Proxmox sur la machine serveur :**
* Branchez la clé USB sur le PC qui servira de serveur et allumez-le.
* Accédez au BIOS (touches F2, F12, ou Suppr) et forcez le démarrage (Boot) sur la clé USB.
* Suivez l'assistant d'installation Proxmox à l'écran (choisissez votre disque dur, votre pays, et définissez un mot de passe).
* À la fin, l'écran affichera une adresse IP (ex: `https://192.168.1.50:8006`). Le serveur est prêt ! Vous pouvez y accéder depuis le navigateur web de votre PC principal.

---

## 🚀 ÉTAPE 2 : Installation de Home Assistant OS

Maintenant que Proxmox tourne, nous allons y installer Home Assistant OS (HAOS) de la manière la plus simple possible, via un script de la communauté.

**1. Lancer le script d'installation :**
* Depuis votre PC principal, ouvrez l'interface web de Proxmox (`https://IP_PROXMOX:8006`).
* Dans le menu de gauche, cliquez sur votre "Nœud" (le nom de votre serveur), puis sur **Shell**.
* Copiez-collez cette commande exacte (créée par tteck) pour générer automatiquement la machine virtuelle :
`bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/vm/haos.sh)"`
* Répondez "Yes" aux questions en laissant les paramètres par défaut.

**2. Premier démarrage :**
* Une fois terminé, une nouvelle machine virtuelle (VM) apparaît dans la liste à gauche. Démarrez-la.
* Ouvrez un nouvel onglet dans votre navigateur web et tapez l'adresse IP de votre Home Assistant sur le port 8123 (ex: `http://192.168.1.51:8123`).
* Créez votre compte administrateur.

---

## 🚀 ÉTAPE 3 : Configuration de la Clé Sonoff (OpenThread)

Pour que Home Assistant puisse parler en "Thread" avec l'Arduino, il faut lui connecter la clé USB Sonoff et la configurer comme "Routeur de Bordure" (Border Router).

[cliquer sur moi pour accéder au github qui vous permetra de parametrer votre clé SONOFF](https://github.com/Karatanaoff/Zibee)

**1. Relier la clé USB à la machine virtuelle :**
* Branchez la clé Sonoff sur un port USB du serveur Proxmox.
* Dans l'interface Proxmox, cliquez sur votre machine virtuelle Home Assistant, puis sur **Hardware** (Matériel).
* Cliquez sur **Add** (Ajouter) > **USB Device**.
* Choisissez "Use USB Vendor/Device ID" et sélectionnez votre clé Sonoff dans la liste. Redémarrez la machine virtuelle Home Assistant.

**2. Installer le module OpenThread :**
* Dans Home Assistant, allez dans **Paramètres** > **Modules complémentaires** > **Boutique**.
* Installez et démarrez le module **OpenThread Border Router**.
* Allez dans **Paramètres** > **Appareils et services**. Home Assistant devrait détecter automatiquement la clé (intégration Thread/Matter) et configurer le réseau.

---

## 🚀 ÉTAPE 4 : Appairage de l'Arduino (Matter/Thread)

**1. Préparer le réseau Thread :**
* Dans Home Assistant, allez dans **Paramètres** > **Appareils et services** > **Thread** (Configurer).
* Identifiez votre routeur de bordure actif dans la liste.
* Cliquez sur les 3 points à côté du bon réseau et choisissez **Utilisé pour les identifiants Android + iOS** (une icône de smartphone apparaîtra).

**2. L'appairage :**
* Sur votre smartphone, ouvrez l'application Home Assistant.
* Allez dans **Paramètres** > **Application Compagnon** > **Dépannage** > **Synchroniser les identifiants Thread**.
* Branchez votre Arduino Matter.
* Dans l'application, allez dans **Appareils et services** > **Ajouter un appareil** > **Matter**.
* Entrez le code texte d'appairage fourni avec l'Arduino. La connexion s'établit en local !

---

## 🚀 ÉTAPE 5 : Accès Distant Sécurisé (Contournement Pare-feu via Ngrok)

Le lycée bloquant les ouvertures de port (NAT/PAT), nous installons un tunnel Ngrok qui établit une connexion "sortante" autorisée par le pare-feu.

**1. Préparer Ngrok :**
* Créez un compte gratuit sur le site de Ngrok.
* Allez dans **Domains** et réclamez un domaine gratuit (ex: `mon-projet.ngrok-free.dev`).
* Allez dans **Your Authtoken** et copiez votre clé secrète.

**2. Installer le module dans Home Assistant :**
* Allez dans la Boutique des modules de Home Assistant.
* Ajoutez ce dépôt personnalisé (via les 3 petits points en haut à droite) : `https://github.com/pssc/ha-addon-ngrok`
* Installez le module **ngrok**.

**3. Configurer le tunnel :**
* Dans l'onglet **Configuration** du module Ngrok, collez votre `auth_token`.
* Sur la ligne du tunnel (en bas), cliquez sur l'icône **Crayon** et collez votre adresse dans le champ `hostname` (sans "https://").
* **Très important :** Supprimez "eu" dans le champ `region` pour le laisser vide (ou tapez "global" selon votre tableau de bord Ngrok) afin d'éviter l'erreur de déconnexion.
* Sauvegardez et démarrez le module.

---

## 🚀 ÉTAPE 6 : Autoriser Ngrok dans Home Assistant

Par défaut, Home Assistant bloque les Reverse Proxies externes par sécurité (Erreur 400 Bad Request). 

**1. Modifier le fichier de configuration :**
* Installez le module **File editor** depuis la boutique Home Assistant.
* Ouvrez le fichier `/config/configuration.yaml`.
* Ajoutez ce bloc de code à la fin du fichier :

```yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - 172.30.32.0/24
    - 172.30.33.0/24
