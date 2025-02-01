---
lab:
  title: "06c\_: Vue d’ensemble du déploiement avec le Centre Azure pour les solutions SAP"
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Module AZ 1006 : Concevoir et implémenter une infrastructure pour prendre en charge les charges de travail SAP sur Azure
# Cours en labo sur une journée 6c AZ-1006 : Vue d’ensemble du déploiement avec le Centre Azure pour les solutions SAP

Temps estimé : 60 minutes

Toutes les tâches de ce cours en labo sur une journée AZ-1006 sont effectuées à partir du portail Azure

## Objectifs

À la fin de ce labo, vous serez en mesure de faire ce qui suit :

- Déployer l’infrastructure hébergeant des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

>**Important** : Les conditions préalables implémentées dans cet exercice ne sont *pas* destinées à représenter les meilleures pratiques pour le déploiement de charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP. Leur objectif est de réduire le temps, le coût et les ressources nécessaires pour évaluer la mécanique du déploiement de charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP et d’effectuer des tâches de gestion et de maintenance post-déploiement. L’implémentation des prérequis inclut les activités suivantes :

- Création d’une identité managée affectée par l’utilisateur Microsoft Entra à être utilisé par le Centre Azure pour les solutions SAP pour l’accès de Stockage Azure pendant son déploiement.
- Octroiement de l’identité managée affectée par l’utilisateur Microsoft Entra utilisée pour effectuer l’accès au déploiement à l’abonnement Azure
- Création du réseau virtuel Azure qui héberge toutes les machines virtuelles Azure incluses dans le déploiement.

Ces activités correspondent aux tâches suivantes de cet exercice :

- Tâche 1 : Créer une identité managée affectée par l’utilisateur Microsoft Entra
- Tâche 2 : Configurez les attributions de rôles RBAC (Contrôle d’accès en fonction du rôle) Azure pour l’identité managée affectée par l'utilisateur Microsoft Entra ID
- Tâche 3 : Créez le réseau virtuel Azure

## Instructions

### Exercice 1 : Implémenter les prérequis minimaux au déploiement d’évaluation des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 15 minutes

Dans cet exercice, vous implémentez les prérequis minimaux nécessaires au déploiement d’évaluation des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP. Cette opération inclut les activités suivantes :

>**Important** : Les conditions préalables implémentées dans cet exercice ne sont *pas* destinées à représenter les meilleures pratiques pour le déploiement de charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP. Leur objectif est de réduire le temps, le coût et les ressources nécessaires pour évaluer la mécanique du déploiement de charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP et d’effectuer des tâches de gestion et de maintenance post-déploiement.

L’implémentation des prérequis inclut les activités suivantes :

- Création d’une identité managée affectée par l’utilisateur Microsoft Entra à être utilisé par le Centre Azure pour les solutions SAP pour l’accès à Azure pendant son déploiement.
- Création du réseau virtuel Azure qui héberge toutes les machines virtuelles Azure incluses dans le déploiement.

Ces activités correspondent aux tâches suivantes de cet exercice :

- Tâche 1 : Créer une identité managée affectée par l’utilisateur Microsoft Entra
- Tâche 2 : Créez le réseau virtuel Azure

#### Tâche 1 : Créez une identité managée attribuée par l’utilisateur Microsoft Entra

Durant cette tâche, vous créez une identité managée affectée par l’utilisateur Microsoft Entra devant être utilisée par le Centre Azure pour les solutions SAP pour l’accès de Stockage Azure pendant son déploiement.

1. À partir de l’ordinateur de labo, démarrez un navigateur web, naviguez vers le portail Azure à l’adresse `https://portal.azure.com`, et authentifiez-vous à l’aide d’un compte Microsoft ou Microsoft Entra ID disposant du rôle Propriétaire dans l’abonnement Azure que vous utilisez dans ce labo.
1. Dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Identités managées**.
1. Dans la page **Identités managés**, sélectionnez **+ Créer**.
1. Sous l’onglet **De base** de la page **Créer une identité managée affectée par l’utilisateur**, spécifiez les paramètres suivants, puis sélectionnez **Vérifier + Créer** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|Le nom d’un **nouveau** groupe de ressources **acss-infra-RG**|
   |Région|Le nom de la région Azure que vous utilisez pour le déploiement ACSS|
   |Nom|**acss-infra-MI**|

1. Dans l’onglet **Vérifier**, attendre que le processus de validation se termine puis sélectionner **Créer**.
1. Attendre que le processus d’approvisionnement se termine puis sélectionner **Accéder à la ressource** pour préparer la tâche suivante. L’approvisionnement devrait prendre seulement quelques secondes.

#### Tâche 2 : Configurer le contributeur de l’abonnement contrôle d’accès en fonction du rôle (RBAC) Azure pour l’API Azure pour FHIR

1. Continuer dans le Portail Azure sur la page vue d’ensemble de l’identité managée pour **acss-infra-MI** à partir de l’achèvement de la dernière tâche.
1. Dans le menu de la page identité managée **acss-infra-MI**, choisir **Attributions de rôles Azure**.
1. Sur la page **Attributions de rôle Azure**, sélectionnez **+ Ajouter une attribution de rôle**.

1. Sous l’onglet **Ajouter une attribution de rôle** du volet **Ajouter une attribution de rôle**, spécifier les paramètres suivants puis **enregistrer** :

   |Paramètre|Valeur|
   |---|---|
   |Étendue|**Abonnement**|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Rôle|**Contributeur**|

#### Tâche 3 : Configurer les attributions de rôles RBAC (Contrôle d’accès en fonction du rôle) Azure pour le compte d’utilisateur Microsoft Entra ID qui sera utilisé afin d’effectuer le déploiement

##### Ajouter une identité : « Rôle de service du Centre Azure pour les solutions SAP »

1. Dans le Portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **Abonnements**.
1. Dans la page **Abonnements**, sélectionnez l’entrée représentant l’abonnement Azure que vous utiliserez pour ce labo. 
1. Dans la page affichant les propriétés de l’abonnement Azure, sélectionnez **Contrôle d’accès (IAM)**.
1. Sur la page **Contrôle d’accès (IAM)**, sélectionner **+ Ajouter** puis, dans le menu déroulant, sélectionner **Ajouter une attribution de rôle**.
1. Sous l’onglet **Rôle** de la page **Ajouter une attribution** de rôle, dans la liste des **rôles de fonction de travail**, rechercher et sélectionner l’entrée **Rôle de service du Centre Azure pour les solutions SAP**, puis sélectionner **Suivant**.
1. Sous l’onglet **Membres** de la page **Ajouter une attribution de rôle**, pour **Attribuer l’accès à**, sélectionner **Identité managée**, puis cliquer sur **+ Sélectionner des membres**.
1. Dans le volet **Sélectionner des identités managées**, spécifier les paramètres suivants, puis cliquer sur **Sélectionner** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Identité gérée|**Identité managée affectée par l’utilisateur**|
   |Sélectionnez|**acss-infra-RG/subscriptions/...**|

1. De retour sur l’onglet **Membres**, sélectionner **Vérifier + attribuer**.
1. Sous l’onglet **Vérifier + attribuer**, sélectionnez **Vérifier + attribuer**.

##### Ajouter une identité : « Administrateur de Centre Azure pour les solutions SAP

1. De retour sur la page **Contrôle d’accès (IAM)**, sélectionner **+ Ajouter** puis, dans le menu déroulant, sélectionner **Ajouter une attribution de rôle**.
1. Sous l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, dans la liste des **rôles de fonction de travail**, rechercher et sélectionner l’entrée **Administrateur du Centre Azure pour les solutions SAP**, puis sélectionner **Suivant**.
1. Dans l’onglet **Membres**
   - pour **Attribuer l’accès à**, sélectionner **Utilisateur, groupe ou principal du service**
   - Cliquer sur **+ Sélection de Membres**.
1. Dans le volet **Sélectionner des membres**, dans la zone de texte **Sélectionner**, entrer le nom du compte d’utilisateur Microsoft Entra ID que vous avez utilisé pour accéder à l’abonnement Azure que vous utilisez pour ce labo, sélectionnez-le dans la liste des résultats correspondant à votre entrée, puis cliquer sur **Sélectionner**.
1. De retour sous l’onglet **Membres**, sélectionnez **Vérifier + attribuer**.
1. Sous l’onglet **Vérifier + attribuer**, sélectionnez **Vérifier + attribuer**.

##### Ajouter une identité : « Opérateur d’identités gérées »

1. De retour sur la page **Contrôle d’accès (IAM)**, sélectionner **+ Ajouter** puis, dans le menu déroulant, sélectionner **Ajouter une attribution de rôle**.
1. Sous l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, dans la liste des **rôles de fonction de travail**, rechercher et sélectionner l’entrée **Opérateur d’identités gérées**, puis sélectionner **Suivant**.
1. Dans l’onglet **Membres**
   - pour **Attribuer l’accès à**, sélectionner **Utilisateur, groupe ou principal du service**
   - Cliquer sur **+ Sélection de Membres**.
1. Dans le volet **Sélectionner des membres**, dans la zone de texte **Sélectionner**, entrer le nom du compte d’utilisateur Microsoft Entra ID que vous avez utilisé pour accéder à l’abonnement Azure que vous utilisez pour ce labo, sélectionnez-le dans la liste des résultats correspondant à votre entrée, puis cliquer sur **Sélectionner**.
1. De retour sous l’onglet **Membres**, sélectionnez **Vérifier + attribuer**.
1. Sous l’onglet **Vérifier + attribuer**, sélectionnez **Vérifier + attribuer**.




#### Tâche 4 : Créer un réseau virtuel

Dans cette tâche, vous créez le réseau virtuel Azure qui héberge toutes les machines virtuelles Azure incluses dans le déploiement. De plus, au sein du réseau virtuel, vous créez les sous-réseaux suivants :

- bastion
- application : destinée à l’hébergement des serveurs d’instance SAP et d’application SAP Central Services
- db : destiné à l’hébergement de la couche base de données SAP

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Réseaux virtuels**.
1. Dans la page **Réseaux virtuels**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** du panneau **Créer un réseau virtuel**, spécifier les paramètres suivants et sélectionner **Suivant** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|**acss-infra-RG**|
   |Nom du réseau virtuel|**acss-infra-VNET**|
   |Région|le nom de la même région Azure que vous avez utilisée dans la tâche précédente de cet exercice|

1. Sous l’onglet **Sécurité**, sous **Azure Bastion**, sélectionnez la **case à cocher** pour **Activer Azure Bastion**.
1. Spécifier les paramètres suivants puis sélectionner **Ajouter**.
   |Paramètre|Valeur|
   |---|---|
   |Nom d’hôte Azure Bastion|**acss-infra-vnet-bastion**|
   |Adresse IP publique Azure Bastion|**(Nouveau) acss-infra-bastion**|

1. Dans l’onglet **Adresses IP**, spécifier les paramètres suivants, puis sélectionner **Ajouter** :

   |Paramètre|Valeur|
   |---|---|
   |Espace d’adressage IP|**10.0.0.0/16 (65 536 adresses)**|

1. Dans la liste des sous-réseaux, sélectionner l’icône corbeille pour **supprimer** le **sous-réseau par défaut**.

1. Sélectionnez **+ Ajouter un sous-réseau**.
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants puis sélectionnez **Ajouter** (conservez les valeurs par défaut des autres paramètres) :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**acss-admin**|
   |Adresse de début|**10.0.0.0**|
   |Taille|**/24 (256 adresses)**|

1. Sélectionnez **+ Ajouter un sous-réseau**.
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants puis sélectionnez **Ajouter** (conservez les valeurs par défaut des autres paramètres) :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**app**|
   |Adresse de début|**10.0.2.0**|
   |Taille|**/24 (256 adresses)**|

1. Sélectionnez **+ Ajouter un sous-réseau**.
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants puis sélectionnez **Ajouter** (conservez les valeurs par défaut des autres paramètres) :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**db**|
   |Adresse de début|**10.0.3.0**|
   |Taille|**/24 (256 adresses)**|

1. Sous l’onglet **Adresses IP**, sélectionnez **Vérifier + créer** :
1. Sous l’onglet **Vérifier + créer**, attendre que le processus de validation se termine puis sélectionner **Créer**.

   >**Remarque** : Attendez environ 3 minutes que le processus d’approvisionnement continue partiellement avant de passer à la tâche suivante. L’approvisionnement entier peut prendre 25 minutes pour Azure Bastion donc nous **n’attendrons pas**.

### Exercice 2 : Déployer l’infrastructure qui héberge des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 20 minutes

Dans cet exercice, vous effectuez le déploiement du Centre Azure pour les solutions SAP. Ceci inclut les activités suivantes :

- Utiliser le Centre Azure pour les solutions SAP pour déployer l’infrastructure capable d’héberger les charges de travail SAP dans un abonnement Azure.

Cette activité correspond à la tâche suivante de cet exercice :

- Tâche 1 : Créer une instance virtuelle pour les solutions SAP (VIS)

>**Remarque** : Une fois le déploiement réussi, vous pouvez procéder à l’installation de logiciels SAP à l’aide du Centre Azure pour les solutions SAP. Toutefois, l’installation du logiciel SAP n’est pas incluse dans ce labo.

>**Remarque** : Pour plus d’informations sur l’installation de logiciels SAP à l’aide du Centre Azure pour les solutions SAP, se reporter à la documentation Microsoft Learn qui explique comment [Obtenir un support d’installation SAP](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/get-sap-installation-media) et [Installer un logiciel SAP](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/install-software).

#### Tâche 1 : Créer une instance virtuelle pour les solutions SAP (VIS)

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **Solutions Azure Center pour SAP**.
1. Dans la page **Centre Azure pour les solutions SAP\| Vue d’ensemble**, sélectionnez **Créer un système SAP**.
1. Sous l’onglet **Informations de base** de la page **Créer une instance virtuelle pour les solutions SAP**, spécifiez les paramètres suivants et sélectionnez **Suivant : Machines virtuelles**

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|**acss-infra-RG**|
   |Nom (SID)|**VI1**|
   |Région|le nom de la région Azure hébergeant le déploiement SAP inscrit par ACSS ou une autre région dans la même zone géographique|
   |Type d’environnement|**Hors production**|
   |Produit SAP|**S/4HANA**|
   |Base de données|**HANA**|
   |Méthode de mise à l’échelle HANA|**Effectuer un scale-up (recommandé)**|
   |Type de déploiement|**Distributed**|
   |Réseau virtuel|**acss-infra-VNET**|
   |Sous-réseau d’application|**app**|
   |Sous-réseau de base de données|**db**|
   |Options d’image du système d’exploitation d’application|**Utiliser une image de marketplace**|
   |Image du système d’exploitation d’application|**Red Hat Enterprise Linux 8.6 pour applications SAP - x64 Gen2 version la plus récente**|
   |Options d’image du système d’exploitation de base de données|**Utiliser une image de marketplace**|
   |Image du système d’exploitation de base de données|**Red Hat Enterprise Linux 8.6 pour applications SAP - x64 Gen2 version la plus récente**|
   |Options de transport SAP|**Créer un répertoire de transport SAP**|
   |Groupe de ressources de transport|**acss-infra-RG**|
   |Nom du compte de stockage|*blank*|
   |Type d'authentification|**SSH public**|
   |Nom d’utilisateur|**contososapadmin**|
   |Source de la clé publique SSH|**Générer une nouvelle paire de clés**|
   |Nom de la paire de clés|**contosovi1key**|
   |Nom de domaine complet SQP|**sap.contoso.com**|
   |Source d’identité managée|**Utiliser une identité managée affectée par l’utilisateur existante**|
   |Nom de l’identité managée|**acss-infra-MI**|

1. Dans l’onglet **Machines Virtuelles**, spécifier les paramètres suivants :

   |Paramètre|Valeur|
   |---|---|
   |Générer une recommandation basée sur|**SAP Application Performance Standard (SAPS) : sélectionnez cette option pour fournir une valeur SAPS pour la couche Application et la taille de la mémoire de base de données, puis cliquez sur Générer des recommandations**|
   |SAPS pour la couche Application|**1 000**|
   |Taille de la mémoire pour la base de données (Gio)|**128**|

1. Sélectionnez **Générer une recommandation**.
1. Passez en revue la taille et le nombre de machines virtuelles pour les machines virtuelles ASCS, d’application et de base de données.

   >**Remarque** : Si nécessaire, ajuster les tailles recommandées en sélectionnant le lien **Afficher toutes les tailles** pour chaque ensemble de machines virtuelles et en choisissant une autre taille. Par défaut, le type de déploiement distribué avec haute disponibilité, ainsi que la taille de mémoire SAPS et de la mémoire de base de données de la couche Application spécifiée ci-dessus entraînent les recommandations de référence SKU de machine virtuelle minimale suivantes :
   - 1 x Standard_D4ds_v4 pour les machines virtuelles ASCS (4 processeurs virtuels et 16 Gio de mémoire chacun)
   - 1 x Standard_D4ds_v4 pour les machines virtuelles d’application (4 processeurs virtuels et 16 Gio de mémoire chacun)
   - 1 x Standard_E16ds_v5 pour les machines virtuelles de base de données (16 processeurs virtuels et 128 Gio de mémoire chacun)

   >**Remarque** : Si nécessaire, vous pouvez demander une augmentation du quota en sélectionnant le lien **Demande de Quota** pour une référence SKU spécifique des machines virtuelles et en envoyant une demande d’augmentation de quota. Le traitement d’une demande prend généralement quelques minutes.

   >**Remarque** : Le Centre Azure pour les solutions SAP applique l’utilisation des références SKU de machine virtuelle prises en charge par SAP pendant le déploiement.

1. Sous l’onglet **Machines virtuelles**, dans la section **Disques de données**, sélectionnez le lien **Afficher et personnaliser la configuration**.
1. Dans la page **Configuration du disque de base de données**, passez en revue la configuration recommandée sans apporter de modifications, puis sélectionnez **Fermer**.
1. De retour sous l’onglet **Machines virtuelles**, sélectionnez **Suivant : Visualiser l’architecture**.
1. Sous l’onglet **Visualiser l’architecture**, passez en revue le diagramme illustrant l’architecture recommandée et sélectionnez **Vérifier + créer**.
1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine, cochez la case confirmant que vous disposez d’un quota suffisant dans la région de déploiement pour éviter de rencontrer une erreur « Quota insuffisant », puis sélectionnez **Créer**.
1. À l’invite, dans la fenêtre **Générer une nouvelle paire de clés**, sélectionnez **Télécharger la clé privée et créer une ressource**.

   >**Remarque** : La clé privée requise pour se connecter aux machines virtuelles Azure incluses dans le déploiement sera téléchargée sur l’ordinateur à partir duquel vous exécutez ce labo.

   >**Remarque** : Attendez seulement 3 minutes pour que le processus d’approvisionnement continue partiellement avant de passer à la tâche suivante. L’ensemble du provisionnement peut prendre 25 minutes, mais nous **n’attendrons pas**.

   >**Remarque** : Après le déploiement, vous pourrez procéder à l’installation de logiciels SAP à l’aide du Centre Azure pour les solutions SAP. Dans ce labo, vous allez explorer les capacités du Centre Azure pour les solutions SAP sans installer de logiciels SAP.

### Exercice 3 : Explorer les charges de travail VIS pour SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 25 minutes

Dans cet exercice, vous visionnez les propriétés et les fonctions au sein d’un centre Azure pour les solutions SAP VIS. Vous allez également vous connecter à une machine virtuelle créée par ACSS et explorer les répertoires créés.

#### Tâche 1 : Passer en revue la page VIS ACSS

1. Une fois le déploiement de VIS terminé, visionner la page **VI1 Instance virtuelle pour les solutions SAP** et explorez les informations disponibles dans le menu de la page Instance virtuelle pour les solutions SAP, notamment :

    1. **Vue d’ensemble**
        - L’onglet **Démarrer** affiche les options pour « Installer le logiciel SAP » et pour « Confirmer le logiciel SAP déjà installé ».
        - L’onglet **Propriétés** affiche les machines virtuelles déployées.
        - L’onglet **Surveillance** affiche « L’utilisation processeur de machine virtuelle du serveur App + Central », « La consommation d’IOPS de disque de base de données » et « L’Utilisation du processeur de machine virtuelle de base de données ».
    1. **Surveillance ** > ** des Informations de qualité** sur la page Informations de qualité :
        - **Machines Virtuelles** : explorez la liste de calcul, les extensions de calcul et le disque de calcul+système d’exploitation.
        - **Vérifications de configuration** : explorez les choix de sous-éléments : Mise en réseau accélérée, adresse IP publique, sauvegarde et équilibreur de charge.
    1. **Gestion des Coûts** > **Analyse des Coûts**
        - Développez les éléments sous la colonne **Ressource**.

#### Tâche 2 : Se connecter à la machine virtuelle de base de données et passer en revue la configuration ACSS

1. Dans le portail Azure, sélectionner les Machines Virtuelles et sélectionnez la machine virtuelle de base de données créée dans ACSS, **vi1dbvm**.
1. Sélectionner Connecter > Azure Bastion, puis choisir les paramètres suivants, puis sélectionner **Se connecter** :
    - Type d’authentification **Clé privée SSH à partir d’un fichier local**
    - Nom d’utilisateur **contososapadmin** 
    - Fichier local ***Clé privée que vous avez téléchargé***
    
1. À l’invite Bash, entrer : `mount` et cherchez la sortie comme suit pour le mappage :

```output
/dev/sdb1 on /mnt type ext4 (rw,relatime,x-systemd.requires=cloud-init.service,_netdev)
/dev/mapper/vg_sap-lv_usrsap on /usr/sap type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/vg_hana_shared-lv_hana_shared on /hana/shared type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/vg_hana_backup-lv_hana_backup on /hana/backup type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/vg_hana_data-lv_hana_data on /hana/data type xfs (rw,relatime,attr2,inode64,sunit=512,swidth=2048,noquota)
/dev/mapper/vg_hana_log-lv_hana_log on /hana/log type xfs (rw,relatime,attr2,inode64,sunit=128,swidth=384,noquota)
```

1. À l’invite, entrer : `more fstab` et chercher une sortie similaire à ce qui suit, que le mappage pour le Media SAP (`sapmedia`) partage :

```output
10.100.1.8:/vi2nfs9fbec656c6a60a7/sapmedia on /usr/sap/install type nfs4 (rw,relatime,vers=4.1,rsize=65536,wsize=65536,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=10.100.1.6,local_
lock=none,addr=10.100.1.8)
```

1. À l’invite, entrer : `cd /usr/sap/install` pour naviguer vers le répertoire mappé vers `sapmedia`.

1. Revenir à la page VIS ACSS et passer en revue les éléments suivants :
1. Azure UI : Lien privé > Points de terminaison privés de l’IU Azure : Afficher l’interface utilisateur du logiciel d’installation d’ACSS

### Exercice 4 (facultatif) : Maintenir les charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 20 minutes

Dans cet exercice, vous passez en revue la gestion post-déploiement et la surveillance des charges de travail SAP à l’aide du Centre Azure pour les solutions SAP. Cette opération inclut les activités suivantes :

- Passez en revue les conditions préalables à la sauvegarde des charges de travail SAP gérées par le Centre Azure pour les solutions SAP
- Passez en revue les conditions préalables à la récupération d'urgence des charges de travail SAP gérées par le Centre Azure pour les solutions SAP
- Examen des options de supervision disponibles pour les charges de travail SAP gérées par le Centre Azure pour les solutions SAP
- Suppression de toutes les ressources Azure approvisionnées dans ce labo.

Ces activités correspondent aux tâches suivantes de cet exercice :

- Tâche 1 : Passez en revue les conditions préalables à la sauvegarde des charges de travail SAP gérées par le Centre Azure pour les solutions SAP
- Tâche 2 : Passez en revue les conditions préalables à la récupération d'urgence des charges de travail SAP gérées par le Centre Azure pour les solutions SAP
- Tâche 3 : Examen des options de supervision pour les charges de travail SAP gérées par le Centre Azure pour les solutions SAP
- Tâche 4 : Supprimer les ressources Azure approvisionnées dans ce labo

#### Tâche 1 : Passez en revue les conditions préalables à la sauvegarde des charges de travail SAP gérées par le Centre Azure pour les solutions SAP

>**Remarque** : Lorsque vous configurez Sauvegarde Azure au niveau des ressources VIS dans le Centre Azure pour les solutions SAP, vous pouvez, en une seule étape, activer la sauvegarde pour les machines virtuelles Azure hébergeant la base de données, les serveurs d’applications et l’instance des services centraux SAP, et pour la base de données HANA. Pour la sauvegarde de base de données HANA, le Centre Azure pour les solutions SAP exécute automatiquement le script de préinscription de sauvegarde.

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **Solutions Azure Center pour SAP**.
1. Dans la page **Solutions Azure Center pour SAP \| Vue d’ensemble**, dans le menu de navigation vertical situé à gauche, sélectionner **Solutions Instances Virtuelles pour SAP** et dans la liste des instances virtuelles, sélectionner l’instance que vous avez déployée dans l’exercice précédent.
1. Dans la page de l’instance virtuelle, dans le menu de navigation vertical sur le côté gauche, dans la section **Opérations** , sélectionner **Sauvegarde (préversion)**.
1. Noter le message indiquant que la sauvegarde ne peut pas être configurée car l’installation/l’inscription de logiciels SAP pour ce système SAP n’est pas terminée.

   >**Remarque** : Ceci est normal. Vous ne pourrez pas configurer la sauvegarde de cette façon tant que l’installation du logiciel SAP n’est pas terminée. Toutefois, la fin de l’installation implique également des prérequis supplémentaires, notamment la création de coffres et de stratégies de sauvegarde, que nous allons examiner ici. 

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **Centre de sauvegarde**. 
1. Sur la page **Centre de sauvegarde**, dans le menu de navigation vertical situé à gauche, dans la section **Gérer**, sélectionner **Coffres**.
1. Sur la page **Centre de sauvegarde \| Coffres**, sélectionner **+ Coffre**.
1. Sur le **Démarrage : Page créer un coffre**, passer en revue les types de coffres disponibles, vérifier que **le coffre Recovery Services** (qui prend en charge **les machines virtuelles Azure** et les types de sources de données **SAP HANA dans les machines virtuelles Azure**) est sélectionné, puis sélectionner **Continuer**.
1. Sous l’onglet **Bases** de la page **Créer un coffre Recovery Services**, spécifier les paramètres suivants et sélectionner **Suivant : Redondance**

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|Le nom d’un **nouveau** groupe de ressources **acss-mgmt-RG**|
   |Nom du coffre|**acss-backup-RSV**|
   |Région|le nom de la région Azure hébergeant le déploiement SAP inscrit par ACSS|

1. Dans l’onglet **Redondance**, spécifier les paramètres suivants et sélectionner **Suivant : Propriétés du coffre**

   |Paramètre|Valeur|
   |---|---|
   |Redondance du stockage de sauvegarde|**Géo-redondant**|
   |Restauration entre régions|**Activer**|

1. Sous l’onglet **Propriétés du coffre**, passer en revue le paramètre **Activer l’immuabilité** sans l’activer, puis sélectionner **Suivant : Mise en réseau**
1. Sous l’onglet **Mise en réseau**, accepter l’option par défaut pour **Autoriser l’accès public à partir de tous les réseaux**, puis sélectionner **Vérifier + créer**
1. Dans cet exercice **ne pas** sélectionnez **Créer**, car nous examinons uniquement.
1. Sous l’onglet **Révision + création**, attendre que le processus de validation se termine, puis revenir à la page Centre Azure pour les solutions SAP (les paramètres de sauvegarde seront perdus).  

   >**Remarque** : La fonctionnalité [Sauvegarde (préversion)](https://learn.microsoft.com/azure/sap/center-sap-solutions/acss-backup-integration) de l’interface utilisateur de Centre Azure pour les solutions SAP deviendra une méthode de choix pour terminer la configuration de la sauvegarde après sa publication sur la *disponibilité générale à* partir de la *préversion*.

   >**Remarque** : Lors de la configuration de la sauvegarde au niveau du VIS de l’interface du Centre Azure pour les solutions SAP, vous pourrez tirer parti du coffre existant et de ses stratégies.

   >**Remarque** : Une fois la sauvegarde configurée au niveau du VIS, vous pouvez surveiller l’état des travaux de sauvegarde des machines virtuelles Azure et de la base de données HANA à partir de l’interface VIS dans le Portail Azure.

#### Tâche 2 : Passez en revue les conditions préalables à la récupération d'urgence des charges de travail SAP gérées par le Centre Azure pour les solutions SAP

>**Remarque** : Alors que le service Centre Azure pour les solutions SAP est un service inter-zone redondant, il n’existe aucun basculement lancé par Microsoft en cas de panne de région. Pour corriger un tel scénario, vous devrez configurer la récupération d’urgence pour les systèmes SAP déployés à l’aide du Centre Azure pour les solutions SAP en suivant les instructions décrites dans [Vue d’ensemble de la récupération d’urgence et les instructions d’infrastructure pour la charge de travail SAP](https://learn.microsoft.com/en-us/azure/sap/workloads/disaster-recovery-overview-guide), ce qui implique l’utilisation de Azure Site Recovery (ASR). Dans cette tâche, vous allez passer en revue le processus d’implémentation d’une solution de récupération d’urgence basée sur ASR qui s’appuie sur ces conseils.

>**Remarque** : ASR est la solution recommandée pour les serveurs d’applications et les instances des services centraux SAP. Pour les serveurs de base de données, vous devriez envisager d’utiliser leur fonction de réplication native.

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **coffres Recovery Services**.
1. Dans la page **Coffres Recovery Services**, sélectionner **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un coffre Recovery Services**, spécifier les paramètres suivants (conserver les valeurs par défaut pour les autres), puis sélectionner **Suivant : Redondance**.

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|Le nom d’un **nouveau** groupe de ressources **acss-dr-RG**|
   |Nom du coffre|**acss-dr-RSV**|
   |Région|le nom de la région Azure associée au déploiement SAP inscrit par ACSS|

   >**Remarque** : Pour identifier la région associée à celle qui héberge vos charges de travail de production, reportez-vous à la documentation MS Learn décrivant les [Régions Azure associées](https://learn.microsoft.com/en-us/azure/reliability/cross-region-replication-azure#azure-paired-regions).

1. Dans l’onglet **Redondance**, spécifier les paramètres suivants et sélectionner **Suivant : Propriétés du coffre**

   |Paramètre|Valeur|
   |---|---|
   |Redondance du stockage de sauvegarde|**Localement redondant**|

1. Sous l’onglet **Propriétés du coffre**, passer en revue le paramètre **Activer l’immuabilité** sans l’activer, puis sélectionner **Suivant : Mise en réseau**
1. Sous l’onglet **Mise en réseau**, accepter l’option par défaut pour **Autoriser l’accès public à partir de tous les réseaux**, puis sélectionner **Vérifier + créer**
1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.

   >**Remarque** : Ne pas attendre pas que le processus d’approvisionnement se termine, mais passer à la tâche suivante. L’approvisionnement peut durer environ 2 minutes.

   >**Remarque** : Vous allez maintenant configurer l’environnement de récupération d’urgence dans la région jumelée dans laquelle vous avez créé le coffre Recovery Services. Cet environnement va inclure un réseau virtuel qui hébergera des réplicas des machines virtuelles Azure actuellement hébergées dans la région primaire où vous avez provisionné Virtual Instance pour SAP. 

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **coffres Recovery Services**.
1. Sur la page **Coffres Recovery Services**, sélectionner **acss-dr-RSV**.
1. Sur la page **acss-dr-RSV**, dans le menu de navigation vertical situé à gauche, dans la section **Prise en main**, sélectionner **Site Recovery**.
1. Sur la page **acss-dr-RSV \| Site Recovery**, dans la section **machines virtuelles Azure**, sélectionner **1. Activer la réplication**. 
1. Sous l’onglet **Source** de la page **Activer la réplication**, spécifier les paramètres suivants, puis sélectionner **Suivant** :

   |Paramètre|Valeur|
   |---|---|
   |Région|le nom de la région Azure hébergeant votre instance virtuelle pour SAP (VIS)|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|**acss-vi-RG**|
   |Modèle de déploiement de machine virtuelle|**Resource Manager**|
   |Récupération d'urgence entre zones de disponibilité|**Aucun**|

   >**Remarque** : La **Récupération d’urgence entre les zones de disponibilité** peut ne pas être paramétrable en fonction de si la région source prend en charge les zones de disponibilité ou non.

1. Sous l’onglet **Machines virtuelles**, sélectionner les quatre premières machines virtuelles de la liste (**vi1appvm0**, **vi1appvm1**, **vi1ascsvm0** et **vi1ascsvm0**) puis sélectionner **Suivant**.

   >**Remarque** : Comme mentionné précédemment, la réplication basée sur ASR sera appliquée aux serveurs d’applications et aux instances des services centraux SAP. Les serveurs de base de données resteront en synchronisation en s’appuyant sur la fonctionnalité de réplication de base de données native.

1. Sous l’onglet **Paramètres de réplication**, effectuez les actions suivantes :

   1. Si nécessaire, dans la liste déroulante **Emplacement cible**, sélectionner la région Azure dans laquelle vous avez créé le coffre Recovery Services **acss-dr-RSV**.
   1. Vérifiez que le nom de l’abonnement Azure utilisé dans ce labo apparaît dans la liste déroulante **Abonnement cible**.
   1. Dans la liste déroulante **Groupe de ressources cible**, sélectionner **acss-dr-RG**.
   1. Sous la liste déroulante **Réseau virtuel de basculement**, sélectionner **Créer nouveau**.
   1. Dans le volet **Créer un réseau virtuel**, dans la zone de texte **Nom** , entrer **CONTOSO-VNET-asr**
   1. Dans la section **Espace d’adressage**, dans la zone de texte **Plage d’adresses**, remplacer l’entrée par défaut par **10.10.0.0/16**.
   1. Dans la section **Sous-réseaux**, dans la zone de texte **Nom du sous-réseau**, entrer **application** et dans la zone de texte **Plage d’adresses**, entrer **10.10.0.0/24**.
   1. Sous l’entrée de sous-réseau récemment ajoutée, dans la section **Sous-réseaux**, dans la zone de texte **Nom du sous-réseau**, entrer l**base de données** puis dans la zone de texte **plage d’adresses**, entrer **10.10.2.0/24**.
   1. Dans le volet **Créer un réseau virtuel**, sélectionner **OK**.
   1. De retour sur la page **Activer la réplication**, vérifier que l’entrée **(nouvelle) application (10.10.0.0/24)** s’affiche dans la liste déroulante **sous-réseau de basculement**.
   1. Dans la section **Stockage**, sélectionner le lien **configuration d’affichage/modification du stockage**.
   1. Dans la page **Personnaliser les paramètres cibles**, passer en revue la configuration résultante sans apporter de modifications, puis sélectionner **Annuler**.
   1. Dans la section **Options de disponibilité**, sélectionner le lien **Afficher/modifier les options de disponibilité**.
   1. Dans la page **Options de disponibilité**, vous avez la possibilité d’implémenter des groupes de placement de proximité pour les ressources cibles, mais n’apportez aucune modification et sélectionnez **Annuler**.

   >**Remarque** : Vous avez également la possibilité de configurer la réservation de capacité.

   >**Remarque** : Vous avez également la possibilité de configurer les paramètres de stratégie de réplication.

   >**Important** : Notez que les espaces d’adressage IP diffèrent entre le réseau virtuel dans les régions primaires et secondaires. Cela est intentionnel, car il permettra de connecter les deux réseaux virtuels ensemble, ce qui est nécessaire pour configurer la réplication entre les serveurs de base de données hébergés dans les deux régions. Cette connexion peut être établie à l’aide de l’appairage de réseaux virtuels.

1. Sous l’onglet **Paramètres de réplication** de la page **Activer la réplication**, sélectionner **Suivant**.
1. Sous l’onglet **Gérer** de la page **Activer la réplication**, sélectionner **Suivant**.
1. Sous l’onglet **Révision** de la page **Activer la réplication**, passer en revue les paramètres. Pour cet exercice, nous **n’activerons pas** la réplication.

   >**Remarque** : La réplication initiale prend généralement beaucoup de temps à terminer. Compte tenu du temps limité alloué à ce laboratoire, se reporter à l’instructeur en ce qui concerne les étapes supplémentaires à effectuer dans le cadre de cette tâche. En l’absence de conseils spécifiques, passer directement à la tâche suivante.

   >**Remarque** : Vous pouvez approvisionner Azure Bastion et Pare-feu Azure à ce stade. Toutefois, vous devez automatiser leur approvisionnement dans le cadre de votre procédure de basculement de récupération d’urgence. Cela réduit les frais associés à la maintenance de l’environnement de récupération d’urgence. La même chose doit s’appliquer à d’autres composants de cet environnement qui reflète la configuration de l’instance virtuelle principale pour SAP, comme les partages de fichiers Azure Premium et le routage personnalisé.

#### Tâche 3 : Examen des options de supervision pour les charges de travail SAP gérées par le Centre Azure pour les solutions SAP

>**Remarque** : Comme avec la sauvegarde, vous ne pourrez pas bénéficier pleinement des fonctionnalités de supervision du Centre Azure pour les solutions SAP. Cela nécessite l’installation de logiciels SAP ou l’inscription d’une instance existante de Azure Monitor pour les solutions SAP. Au lieu de cela, dans cette tâche, vous allez parcourir l’interface disponible dans l’instance virtuelle pour les solutions SAP pour identifier et examiner ces fonctionnalités.

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **Instance virtuelle Azure pour les solutions SAP**. 
1. Dans la page **Instances virtuelles pour les solutions SAP**, passer en revue les informations d’état résumées pour l’instance **VI1**, y compris les indicateurs visuels d’intégrité et d’état globaux.
1. Dans la page **Instances virtuelles pour les solutions SAP**, sélectionner **VI1**.
1. Dans la page **VI1** , dans le menu de navigation vertical situé à gauche, sélectionner **Vue d’ensemble** puis dans le volet situé à droite, sélectionner **Surveillance**.
1. Passez en revue les données de télémétrie de surveillance affichées dans le volet surveillance.

   >**Remarque** : Le volet de surveillance comprend des graphiques et des métriques d’utilisation de processeurs virtuels pour les serveurs d’applications, les serveurs de base de données et les instances des services centraux SAP. Il inclut également des serveurs de base données avec des statistiques IOPS de disque de base de données. 

1. Dans la page **VI1**, dans le menu de navigation vertical situé à gauche, dans la section **Surveillance**, sélectionner **Aperçus de qualité**.
1. Dans la page **VI1 \| Aperçus de qualité \| classeur 1**, passer en revue l’onglet **Recommandation Advisor**, qui est destiné à fournir des recommandations pour optimiser l’instance virtuelle pour les solutions SAP (VIS), l’instance de serveur central, les instances App Service et les bases de données.

   >**Remarque** : Ces recommandations nécessitent l’installation complète du logiciel SAP.

1. Dans la page **VI1 \| Aperçus qualité \| Classeur 1**, sélectionner l’onglet **Machine virtuelle** et passer en revue le contenu des onglets **Azure Compute**, **Liste de Calcul**, **Extensions de Calcul**, **Calcul + Disque de système d’exploitation** et **Calcul + Disques de données**.

   >**Remarque** : Chacun de ces onglets doit inclure des données réelles collectées à partir des machines virtuelles Azure qui font partie de l’instance virtuelle pour les solutions SAP.

1. Dans la page **VI1 \| Aperçus de Qualité \| Classeur 1** , sélectionner l’onglet **Vérifications de configuration** et examiner le contenu des onglets **Réseau accéléré**, **IP publique**, **Sauvegarde** et **Équilibreur de charge**. Ce contenu fournit une vue d’ensemble rapide des paramètres de performances et de sécurité des composants de calcul et de réseau de l’instance virtuelle pour les solutions SAP. L’onglet **Équilibreur de charge** inclut les informations du **moniteur Load Balancer** qui affiche les métriques d’équilibreur de charge clés.
1. Dans la page **VI1**, dans le menu de navigation vertical situé à gauche, dans la section **Surveillance**, sélectionner **Azure Monitor pour les solutions SAP**.
1. Dans la page **VI1 \| Azure Monitor pour les solutions SAP**, noter le message indiquant que AMS ne peut pas être configuré, car l’installation de logiciels SAP\inscription pour VIS n’est pas terminée.

   >**Remarque** : Une fois que vous avez installé le logiciel SAP, vous pourrez l’intégrer à une ressource de solutions Azure Monitor pour SAP nouvelle ou existante. Les solutions Azure Monitor pour SAP s’appuient sur les fonctionnalités de Azure Monitor de Log Analytics et de classeurs pour fournir une surveillance complète des charges de travail SAP hébergées sur des machines virtuelles Azure, notamment la prise en charge des visualisations personnalisées, des requêtes et des alertes.
