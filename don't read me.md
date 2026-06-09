# 🏠 Projet Domotique : Arduino Matter + Home Assistant + Ngrok
L'objectif de ce projet est très simple : Contrôler un Arduino Matter depuis son téléphone en 4G, sans être bloqué par le pare-feu du lycée.
## 🛠️ ÉTAPE 1 : Installer le Serveur (Proxmox)
 1. Prenez une clé USB.
 2. Sur votre PC Windows, téléchargez **Rufus** et l'ISO de **Proxmox VE**.
 3. Flashez la clé avec Rufus. ⚠️ **Très important :** Si Rufus vous demande, choisissez le **Mode DD**.
 4. Branchez la clé sur le PC qui servira de serveur et allumez-le.
 5. Suivez l'installation (choisissez le mot de passe, le pays, etc.).
 6. À la fin, l'écran vous donne une adresse IP (ex: https://192.168.1.50:8006). Le serveur est prêt.
## 🚀 ÉTAPE 2 : Installer Home Assistant
 1. Sur votre PC principal, ouvrez internet et tapez l'IP de Proxmox.
 2. Connectez-vous (Utilisateur : root / Mot de passe : celui de l'étape 1).
 3. Cliquez sur votre serveur à gauche, puis allez dans l'onglet **Shell**.
 4. Tapez EXACTEMENT cette commande (la bonne ! 😉) et faites Entrée :
   bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/vm/haos-vm.sh)"
 5. Faites juste Entrée pour dire "Yes" à tout. La machine virtuelle se crée toute seule.
 6. **ATTENTION :** Démarrez la machine virtuelle et **attendez 15 minutes** sans rien toucher. Le système s'installe en fond.
## ⚙️ ÉTAPE 3 : Premier lancement de Home Assistant
 1. Ouvrez un nouvel onglet internet.
 2. Tapez l'IP de votre Home Assistant avec le port 8123. Exemple :
   http://192.168.1.99:8123
   *(N'écrivez pas "https", juste "http" !)*
 3. Créez votre compte administrateur (votre prénom, mot de passe).
## 📻 ÉTAPE 4 : Configurer la Clé Sonoff (Thread & Matter)
 1. Branchez la clé USB Sonoff sur le PC serveur.
 2. Dans Proxmox, allez sur votre machine virtuelle > **Hardware** > **Add** > **USB Device**.
 3. Choisissez votre clé Sonoff et redémarrez Home Assistant.
 4. Dans Home Assistant, allez dans **Paramètres** > **Modules complémentaires** > **Boutique**.
 5. Installez **OpenThread Border Router** et démarrez-le.
 6. Allez dans **Paramètres** > **Appareils et services**. Home Assistant va détecter la clé et configurer **Thread** et **Matter**.
## 📱 ÉTAPE 5 : Lier l'Arduino
 1. Dans les paramètres **Thread**, cliquez sur les 3 points à côté du réseau et choisissez : *Utilisé pour les identifiants Android + iOS*.
 2. Sur votre téléphone, ouvrez l'application Home Assistant (en Wi-Fi).
 3. Allez dans **Paramètres** > **Application Compagnon** > **Dépannage** > **Synchroniser les identifiants Thread**.
 4. Branchez l'Arduino.
 5. Dans l'application, faites **Ajouter un appareil** > **Matter**.
 6. Scannez ou tapez le code de l'Arduino. Il est connecté !
## 🌍 ÉTAPE 6 : Accès depuis l'extérieur (Ngrok)
 1. Créez un compte gratuit sur le site de **Ngrok**.
 2. Allez dans **Domains** et réservez un nom gratuit (ex: mon-projet.ngrok-free.dev).
 3. Copiez votre **Authtoken**.
 4. Dans la boutique Home Assistant, ajoutez ce dépôt : https://github.com/pssc/ha-addon-ngrok et installez **ngrok**.
 5. Dans sa configuration :
   * Mettez l'Authtoken.
   * Mettez le domaine que vous avez réservé dans hostname (sans https).
   * **Laissez la case region VIDE !**
   * Démarrez Ngrok.
## 🔓 ÉTAPE 7 : Débloquer la sécurité de Home Assistant
Par défaut, Home Assistant bloque Ngrok. Il faut le débloquer :
 1. Installez le module **File editor**.
 2. Ouvrez le fichier /config/configuration.yaml.
 3. Ajoutez ceci TOUT EN BAS (respectez bien les espaces) :
   http:
   use_x_forwarded_for: true
   trusted_proxies:
   - 127.0.0.1
   - 172.30.32.0/24
   - 172.30.33.0/24
 4. Redémarrez Home Assistant.
## 🎉 C'EST GAGNÉ !
Pour tester en direct du lycée :
 1. **Coupez votre Wi-Fi**, mettez-vous en **4G** (pour contourner le pare-feu du lycée).
 2. Ouvrez votre navigateur internet sur le téléphone.
 3. Tapez votre adresse Ngrok complète : https://mon-projet.ngrok-free.dev
 4. Connectez-vous, et pilotez votre Arduino !
