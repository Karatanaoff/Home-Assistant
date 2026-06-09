# 🏠 Projet IoT : Contrôle d'un Arduino via Matter/Thread sur Home Assistant avec Accès Distant (Ngrok)
## 📝 Description du projet
Ce projet documente la création complète et détaillée d'un serveur domotique performant, installé sur une machine physique (Bare Metal), pour piloter un microcontrôleur **Arduino** à l'aide du protocole **Matter (via Thread)**.
L'objectif principal est de rendre cette installation accessible depuis un smartphone n'importe où dans le monde (en 4G/5G), sans aucune ouverture de port sur le routeur local. Cela permet de contourner les restrictions réseau et les pare-feu stricts (comme le filtrage académique Fortinet d'un lycée).
## 🛠️ Matériel Requis
 * **Serveur Bare Metal :** Un PC dédié, un mini-PC (ex: Intel NUC) ou un ordinateur de récupération. Il doit obligatoirement être doté d'un port **Ethernet** (la connexion filaire est requise pour la stabilité du serveur).
 * **Stockage d'installation :** Une clé USB classique (8 Go minimum) qui sera formatée.
 * **Dongle Réseau IOT :** Une clé USB **Sonoff Zigbee 3.0 USB Dongle Plus** (Impérativement le modèle "E" basé sur la puce EFR32MG21, capable de faire du Thread).
 * **Microcontrôleur :** Un Arduino compatible Matter (ex: Arduino Nano Matter).
 * **Contrôleur Mobile :** Un smartphone (Android ou iOS) avec l'application officielle *Home Assistant* installée.
## 🚀 CHAPITRE 1 : Préparation du Serveur Physique et Installation de Proxmox VE
Proxmox est l'hyperviseur (le système d'exploitation de base) qui est installé directement sur le "métal" de la machine pour gérer nos futures machines virtuelles avec des performances maximales.
**1. Préparation de la clé USB d'installation (via Rufus) :**
 * Téléchargez l'image ISO officielle de **Proxmox VE** sur le site officiel.
 * Branchez votre clé USB sur un PC Windows et lancez le logiciel gratuit **Rufus**.
 * Sélectionnez votre clé USB, puis chargez l'ISO de Proxmox.
 * Cliquez sur **Démarrer**.
 * ⚠️ **ATTENTION CRUCIALE :** Si Rufus affiche une alerte vous demandant de choisir entre le mode "Image ISO" et le "Mode DD", choisissez impérativement le **Mode DD**. Si vous ne le faites pas, le serveur refusera de démarrer sur la clé USB.
**2. Configuration du BIOS du serveur physique :**
 * Insérez la clé USB dans le PC serveur éteint, puis allumez-le.
 * Mitraillez la touche d'accès au BIOS (selon les marques : F2, F12, Suppr ou Échap).
 * **Désactivez le Secure Boot** (Démarrage sécurisé), car il bloque les systèmes d'exploitation tiers comme Proxmox.
 * **Activez la Virtualisation Matérielle** (selon le processeur, cherchez : Intel VT-x / VT-d ou AMD-V / SVM / IOMMU). Sans cela, il sera physiquement impossible de lancer des machines virtuelles.
 * Allez dans l'onglet *Boot* (Démarrage) et placez votre clé USB en première position (Boot Priority #1).
 * Sauvegardez et quittez (généralement la touche F10).
**3. Assistant d'installation Proxmox :**
 * Le serveur démarre sur la clé. Choisissez *Install Proxmox VE (Graphical)*.
 * Suivez l'assistant : sélectionnez le disque dur principal du PC pour l'installation, choisissez votre pays, et définissez un mot de passe administrateur (le nom d'utilisateur par défaut sera root).
 * Renseignez les paramètres réseau ou laissez le DHCP attribuer une adresse automatique.
 * À la fin de la barre de chargement, l'ordinateur redémarre. **Retirez la clé USB**.
 * L'écran noir du serveur affiche alors l'adresse d'administration textuelle. Exemple : https://192.168.1.50:8006. Le serveur est prêt, vous pouvez débrancher son écran, tout se fera désormais à distance.
## 🚀 CHAPITRE 2 : Déploiement Automatisé de Home Assistant OS (HAOS)
Pour éviter les configurations manuelles complexes (disque virtuel, UEFI, RAM), nous utilisons le script officiel mis à jour par la communauté domotique.
**1. Création de la Machine Virtuelle (VM) :**
 * Depuis votre ordinateur principal, ouvrez votre navigateur internet et connectez-vous à l'interface de votre Proxmox (ex: https://192.168.1.50:8006).
 * *Note : Ignorez l'avertissement de sécurité du navigateur en cliquant sur "Paramètres avancés" puis "Continuer vers le site".*
 * Connectez-vous avec l'identifiant root et le mot de passe défini au Chapitre 1.
 * Dans le menu de gauche, cliquez sur le nom de votre serveur (le nœud), puis cliquez sur l'onglet **Shell** (la console de commande).
 * Copiez et collez la commande suivante, puis appuyez sur Entrée :
   bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/vm/haos-vm.sh)"
 * Le script se lance. Lorsqu'il vous pose des questions, appuyez simplement sur Entrée pour accepter les paramètres par défaut (*Default Settings*). Le script va automatiquement télécharger l'image officielle de Home Assistant, allouer la mémoire RAM, créer le disque virtuel et configurer l'UEFI.
**2. Gestion du Premier Démarrage (L'attente critique) :**
 * Une fois le script terminé, une nouvelle machine virtuelle nommée haos-vm apparaît dans la liste de gauche sous Proxmox. Cliquez dessus et cliquez sur le bouton **Start** (Démarrer) tout en haut.
 * 💡 **Note vitale pour les débutants :** Au tout premier démarrage, le résumé de Proxmox affichera l'erreur *"L'agent invité n'est pas en service"* (Guest Agent not running) et l'adresse IP restera invisible. **C'est 100% normal.** Home Assistant est en train d'initialiser ses fichiers, de créer ses conteneurs Docker internes et de télécharger la dernière version de son interface graphique en arrière-plan.
 * **Ne touchez à rien pendant 10 à 15 minutes.** * Pour suivre l'avancement en temps réel, cliquez sur l'onglet **Console** de la VM dans Proxmox. Dès que l'installation interne est terminée, l'écran affichera la bannière d'accueil Home Assistant avec son adresse IP réseau sous la ligne IPv4 addresses for eth0 (Exemple : 192.168.1.99).
## 🚀 CHAPITRE 3 : Première Connexion et Configuration (Réseau Local)
Avant de configurer l'accès à distance, il faut impérativement réussir à se connecter au serveur en local.
**1. La règle absolue du réseau :**
 * L'appareil qui tente de se connecter (votre PC ou smartphone) et le serveur Proxmox doivent être physiquement sur le **même réseau**. Si le serveur est branché sur un routeur local (ex: 192.168.1.X) et que votre appareil est sur le Wi-Fi général du lycée, ils ne communiqueront jamais.
**2. Connexion depuis le Navigateur ou l'Application :**
 * Ouvrez l'application **Home Assistant** sur votre téléphone (ou votre navigateur web sur PC).
 * Si la détection automatique échoue, cliquez sur **Configurer manuellement**.
 * **Le format de l'adresse est un piège classique.** Vous devez taper **exactement** ceci (sans oublier le port) :
   http://<IP_DE_VOTRE_HOME_ASSISTANT>:8123
   *Exemple concret : http://192.168.1.99:8123*
   *⚠️ Attention : Les navigateurs forcent souvent le mode sécurisé. N'écrivez surtout pas "https://", le protocole initial de HA est uniquement **http://**.*
 * L'écran de bienvenue s'ouvre. Créez votre compte administrateur (Nom, identifiant, mot de passe).
## 🚀 CHAPITRE 4 : Configuration de la Clé USB Sonoff (Réseau OpenThread)
Pour que Home Assistant puisse parler avec l'Arduino, nous devons lier la clé USB physique à la machine virtuelle.
**1. Le Passthrough USB (Transfert vers la VM) :**
 * Branchez la clé USB Sonoff sur le serveur physique Proxmox.
 * Dans l'interface Proxmox, sélectionnez la VM Home Assistant et allez dans l'onglet **Hardware** (Matériel).
 * Cliquez sur **Add** (Ajouter) > **USB Device**.
 * Cochez *Use USB Vendor/Device ID*. Dans la liste déroulante, sélectionnez votre clé Sonoff (souvent nommée *CP210x USB to UART Bridge* ou *Sonoff Dongle Plus*). Cliquez sur **Add**.
 * Redémarrez complètement la VM Home Assistant.
**2. Activation du Routeur de Bordure (Border Router) :**
 * Dans l'interface web de Home Assistant, allez dans **Paramètres** > **Modules complémentaires** > **Boutique des modules**.
 * Cherchez le module officiel **OpenThread Border Router** et installez-le.
 * Cochez *Lancer au démarrage* et *Chien de garde* (Watchdog), puis cliquez sur **Démarrer**.
 * Allez dans **Paramètres** > **Appareils et services**. Home Assistant découvrira automatiquement la clé USB et configurera l'intégration **Thread** et **Matter**.
## 🚀 CHAPITRE 5 : Appairage de l'Arduino (Matter via Thread)
**1. Préparation des clés de sécurité :**
 * Dans Home Assistant, allez dans **Paramètres** > **Appareils et services** > onglet **Intégrations**. Cliquez sur **Configurer** sur la ligne *Thread*.
 * Cliquez sur les 3 petits points à côté de votre routeur actif et sélectionnez : **Utilisé pour les identifiants Android + iOS**. Une icône de smartphone apparaît, confirmant que le réseau est prêt à partager ses clés de chiffrement.
**2. Synchronisation et inclusion :**
 * Sur votre smartphone (connecté au Wi-Fi local), ouvrez l'application Home Assistant.
 * Allez dans **Paramètres** > **Application Compagnon** > **Dépannage** > **Synchroniser les identifiants Thread**. (Le téléphone récupère l'accès au réseau).
 * Branchez l'Arduino Matter sur le secteur.
 * Toujours dans l'application, allez dans **Paramètres** > **Appareils et services** > **Ajouter un appareil** (en bas à droite) > **Ajouter un appareil Matter**.
 * Flashez le QR Code d'appairage ou entrez le code fourni avec l'Arduino. L'appareil est maintenant connecté à votre domotique locale !
## 🚀 CHAPITRE 6 : Configuration du Tunnel Distant Ngrok
Puisque le réseau du lycée bloque les ouvertures de ports (NAT/PAT) et les flux entrants, nous installons un tunnel sortant sécurisé.
**1. Récupération de l'Authtoken :**
 * Créez un compte gratuit sur ngrok.com.
 * Dans le menu, allez dans **Domains**, et réservez un nom de domaine gratuit (Exemple : mon-projet.ngrok-free.dev).
 * Dans **Your Authtoken**, copiez votre longue clé secrète.
**2. Installation de l'add-on Ngrok :**
 * Dans la boutique des modules de Home Assistant, cliquez sur les 3 points en haut à droite > **Dépôts** (Repositories).
 * Ajoutez cette URL : https://github.com/pssc/ha-addon-ngrok. Actualisez la page (F5) et installez **ngrok Client Installer**.
**3. Configuration précise du module :**
 * Allez dans l'onglet **Configuration** du module.
 * Dans le champ **auth_token**, collez votre clé secrète.
 * Dans la section **tunnels**, cliquez sur l'icône **Crayon** sur la ligne grisée hass. Cherchez le champ **hostname** et collez votre adresse réservée (ex: mon-projet.ngrok-free.dev). Ne mettez surtout pas de "https://". Cliquez sur Enregistrer (dans la petite fenêtre).
 * **Crucial pour la stabilité :** Dans le menu principal, cherchez le champ **region**. Supprimez la valeur "eu" par défaut et **laissez la case vide** (ou tapez "global"). Cela évitera que Ngrok ne ferme le tunnel en erreur (ERR_NGROK_3200).
 * Cliquez sur le gros bouton **Enregistrer**, retournez dans l'onglet *Informations* et **Démarrez** le module.
## 🚀 CHAPITRE 7 : Sécurisation du Proxy (Résolution de l'Erreur 400)
Home Assistant bloque nativement les flux provenant de serveurs mandataires (400: Bad Request). Il faut lui indiquer que Ngrok est de confiance.
**1. Modification du fichier YAML :**
 * Installez et démarrez le module **File editor** depuis la boutique.
 * Ouvrez le fichier /config/configuration.yaml.
 * Ajoutez **exactement** ce bloc à la fin du fichier. *Attention, l'indentation (les espaces en début de ligne) est obligatoire en YAML :*
   http:
   use_x_forwarded_for: true
   trusted_proxies:
   - 127.0.0.1
   - 172.30.32.0/24
   - 172.30.33.0/24
 * Sauvegardez en cliquant sur l'icône de disquette rouge.
**2. Application de la règle :**
 * Allez dans **Outils de développement** > **Vérifier la configuration**.
 * Si le voyant est vert, cliquez sur **Redémarrer** (Redémarrer Home Assistant).
## 📱 CHAPITRE 8 : Connexion à Distance (Contournement Pare-Feu Fortinet)
Pour piloter l'Arduino de l'extérieur sans être bloqué par les restrictions académiques, suivez cette procédure :
**1. Configuration du Smartphone :**
 * **Coupez le Wi-Fi.** Utilisez uniquement vos données mobiles personnelles (**4G ou 5G**).
 * *Explication :* Le Wi-Fi du lycée intercepte les certificats SSL (inspection Fortinet), générant une erreur critique SslError qui bloque immédiatement la connexion à l'application.
**2. La connexion finale :**
 * N'ouvrez pas l'application HA tout de suite. Ouvrez votre navigateur internet mobile (Chrome, Safari, Brave) et tapez l'adresse complète :
   https://<VOTRE_DOMAINE_NGROK>
   *(Exemple : https://mon-projet.ngrok-free.dev)*
 * L'avertissement gratuit de Ngrok s'affiche. Cliquez sur le bouton bleu **Visit Site**.
 * L'interface de Home Assistant apparaît ! Identifiez-vous. Vous avez désormais un serveur domotique sécurisé et accessible dans le monde entier.
 * *Vous pouvez maintenant entrer cette même adresse https://... dans les paramètres de votre application mobile pour l'utiliser au quotidien.*
