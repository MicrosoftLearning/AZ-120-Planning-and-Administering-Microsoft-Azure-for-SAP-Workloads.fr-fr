---
lab:
  title: "06a\_: Vue d’ensemble des prérequis pour le déploiement du Centre Azure pour les solutions SAP (ACSS)"
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Module AZ 1006 : Concevoir et implémenter une infrastructure pour prendre en charge les charges de travail SAP sur Azure
# AZ-1006 Labo de cours en une journée : Vue d’ensemble des prérequis pour le déploiement de charges de travail SAP avec le Centre Azure pour les solutions SAP (ACSS)

Temps estimé : 100 minutes

Toutes les tâches de ce labo de cours en une journée AZ-1006 sont effectuées à partir du portail Azure

## Objectifs

À la fin de ce labo, vous serez en mesure de faire ce qui suit :

- Implémenter les prérequis au déploiement des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

## Instructions

### Exercice 1 : Implémenter les prérequis au déploiement des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 60 minutes

Dans cet exercice, vous passez en revue et implémentez les prérequis pour le déploiement des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP. Cela comprend les activités suivantes :

- Création d’une identité managée affectée par l’utilisateur Microsoft Entra devant être utilisée par le Centre Azure pour les solutions SAP pour l’accès de Stockage Azure pendant son déploiement.
- Création du réseau virtuel Azure qui héberge toutes les machines virtuelles Azure incluses dans le déploiement.
- Création d’une ressource Azure Bastion pour sécuriser la connectivité aux machines virtuelles Azure à partir d’Internet.
- Création d’un compte Stockage Azure à usage général v2 associé au Centre Azure pour les solutions SAP utilisé pour le déploiement
- Octroi, à l’identité managée affectée par l’utilisateur Microsoft Entra utilisée pour le déploiement, de l’accès à l’abonnement Azure et au compte Stockage Azure à usage général
- Création d’un compte de partages de fichiers Azure Premium utilisé pour implémenter le répertoire de transport SAP
- Création et configuration d’un groupe de sécurité réseau (NSG) utilisé pour restreindre l’accès sortant à partir de sous-réseaux du réseau virtuel qui héberge le déploiement.
- Création d’une machine virtuelle Azure à utiliser pour l’installation de logiciels SAP dans le cadre d’un déploiement du Centre Azure pour les solutions SAP.
- Connexion à la machine virtuelle Azure à l’aide d’Azure Bastion et configuration de celle-ci pour l’installation de logiciels SAP.
- Suppression de toutes les ressources Azure approvisionnées dans ce labo.

Ces activités correspondent aux tâches suivantes de cet exercice :

- Tâche 1 : Créer une identité managée affectée par l’utilisateur Microsoft Entra
- Tâche 2 : Créer le réseau virtuel Azure
- Tâche 3 : Créer une ressource Azure Bastion
- Tâche 4 : Créer un compte Stockage Azure à usage général v2
- Tâche 5 : Configurer l’autorisation de l’identité managée affectée par l’utilisateur Microsoft Entra
- Tâche 6 : Créer un compte de partages de fichiers Azure Premium
- Tâche 7 : Créer et configurer un groupe de sécurité réseau
- Tâche 8 : Créer une machine virtuelle Azure
- Tâche 9 : Configurer la machine virtuelle Azure
- Tâche 10 : Supprimer les ressources Azure

#### Tâche 1 : Créer une identité managée affectée par l’utilisateur Microsoft Entra

Durant cette tâche, vous créez une identité managée affectée par l’utilisateur Microsoft Entra devant être utilisée par le Centre Azure pour les solutions SAP pour l’accès de Stockage Azure pendant son déploiement.

1. À partir de l’ordinateur de labo, démarrez un navigateur web, naviguez vers le portail Azure à l’adresse `https://portal.azure.com`, et authentifiez-vous à l’aide d’un compte Microsoft ou Microsoft Entra ID disposant du rôle Propriétaire dans l’abonnement Azure que vous utilisez dans ce labo.
1. Dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Identités managées**.
1. Dans la page **Identités managés**, sélectionnez **+ Créer**.
1. Sous l’onglet **De base** de la page **Créer une identité managée affectée par l’utilisateur**, spécifiez les paramètres suivants, puis sélectionnez **Vérifier + Créer** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
`   |Resource group| Choisir ou créer **acss-infra-RG**|
   |Région|Nom de la région Azure que vous utilisez pour le déploiement ACSS|
   |Nom|**acss-infra-MI**|

1. Sous l’onglet **Vérifier**, attendez que le processus de validation se termine et sélectionnez **Créer**.

   >**Remarque** : N’attendez pas que le processus d’approvisionnement se termine, mais passez à la tâche suivante. L’approvisionnement ne devrait prendre que quelques secondes.

   >**Remarque** : Durant l’une des tâches à venir, vous autoriserez l’accès par l’identité managée au compte de stockage hébergeant le support d’installation SAP afin de prendre en charge l’installation de logiciels SAP via le Centre Azure pour les solutions SAP.

#### Tâche 2 : Créer un réseau virtuel

Durant cette tâche, vous créez le réseau virtuel Azure qui héberge toutes les machines virtuelles Azure incluses dans le déploiement. De plus, au sein du réseau virtuel, vous créez les sous-réseaux suivants :

- AzureFirewallSubnet : destiné au déploiement de Pare-feu Azure
- AzureBastionSubnet : destiné au déploiement d’Azure Bastion
- dmz : destiné au déploiement de la machine virtuelle Azure utilisée pour déployer des logiciels SAP
- application : destiné à l’hébergement des serveurs d’instance d’application SAP et SAP Central Services
- db : destiné à l’hébergement de la couche base de données SAP

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Réseaux virtuels**. 
1. Dans la page **Réseaux virtuels**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** du panneau **Créer un réseau virtuel**, spécifier les paramètres suivants et sélectionner **Suivant** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**acss-infra-RG**|
   |Nom du réseau virtuel|**acss-infra-VNET**|
   |Région|Nom de la région Azure que vous avez utilisée dans la tâche précédente de cet exercice|

1. Sous l’onglet **Sécurité**, acceptez les paramètres par défaut et sélectionnez **Suivant**.

   >**Remarque** : Vous pourriez approvisionner à ce stade Azure Bastion et Pare-feu Azure, mais vous les approvisionnerez séparément une fois le réseau virtuel créé.

1. Sous l’onglet **Adresses IP**, spécifiez les paramètres de sous-réseau suivants, puis sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Espace d’adressage IP|**10.0.0.0/16 (65 536 adresses)**|

1. Dans la liste des sous-réseaux, sélectionnez l’icône de corbeille pour supprimer le sous-réseau **par défaut**.
1. Sélectionnez **+ Ajouter un sous-réseau**.
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants puis sélectionnez **Ajouter** (conservez les valeurs par défaut des autres paramètres) :

   |Paramètre|Valeur|
   |---|---|
   |Objectif du sous-réseau|**Pare-feu Azure**|
   |Adresse de début|**10.0.0.0**|

   >**Remarque** : Cela affectera automatiquement au sous-réseau le nom **AzureFirewallSubnet** et définira sa taille sur **/26 (64 adresses)**.

1. Sélectionnez **+ Ajouter un sous-réseau**.
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants puis sélectionnez **Ajouter** (conservez les valeurs par défaut des autres paramètres) :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**dmz**|
   |Adresse de début|**10.0.0.128**|
   |Taille|**/26 (64 adresses)**|

   >**Remarque** : Ce sous-réseau sera utilisé pour héberger la machine virtuelle Azure que vous utiliserez pour installer les logiciels SAP via le Centre Azure pour les solutions SAP.

1. Sélectionnez **+ Ajouter un sous-réseau**.
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants puis sélectionnez **Ajouter** (conservez les valeurs par défaut des autres paramètres) :

   |Paramètre|Valeur|
   |---|---|
   |Objectif du sous-réseau|**Azure Bastion**|
   |Adresse de début|**10.0.1.0**|
   |Taille|**/24 (256 adresses)**|

   >**Remarque** : Cela affectera automatiquement au sous-réseau le nom **AzureBastionSubnet**.

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
1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine, puis sélectionnez **Créer**.

   >**Remarque** : N’attendez pas que le processus d’approvisionnement se termine, mais passez à la tâche suivante. L’approvisionnement ne devrait prendre que quelques secondes.

#### Tâche 3 : Créer une ressource Azure Bastion

Durant cette tâche, vous créez une ressource Azure Bastion afin de sécuriser la connectivité aux machines virtuelles Azure à partir d’Internet.

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Bastions**. 
1. Dans la page **Bastions**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Bastions**, spécifiez les paramètres suivants et sélectionnez **Suivant : Avancé >**  :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**acss-infra-RG**|
   |Nom|**acss-infra-BASTION**|
   |Région|Nom de la région Azure que vous avez utilisée précédemment dans cet exercice|
   |Niveau|**De base**|
   |Nombre d’instances|**2**|
   |Réseau virtuel|**acss-infra-VNET**|
   |Sous-réseau|**AzureBastionSubnet (10.0.1.0/24)**|
   |Adresse IP publique|**Création**|
   |Nom de l’adresse IP publique|**acss-bastion-PIP**|

1. Sous l’onglet **Avancé**, passez en revue les options disponibles sans apporter de modifications, puis sélectionnez **Vérifier + créer**.
1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine, puis sélectionnez **Créer**.

   >**Remarque** : N’attendez pas que l’approvisionnement se termine, mais passez à la tâche suivante. L’approvisionnement peut durer environ cinq minutes.

#### Tâche 4 : Créer un compte Stockage Azure à usage général v2

Durant cette tâche, vous créez un compte Stockage Azure à usage général v2 associé au Centre Azure pour les solutions SAP utilisé pour le déploiement. Ce compte de stockage est utilisé pour héberger le support d’installation SAP afin de prendre en charge l’installation de logiciels SAP par le biais du Centre Azure pour les solutions SAP.

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Comptes de stockage**.
1. Dans la page **Comptes de stockage**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un compte de stockage**, spécifiez les paramètres suivants et sélectionnez **Suivant : Avancé >**.

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**acss-infra-RG**|
   |Nom du compte de stockage|Nom global unique comprenant entre 3 et 24 caractères alphanumériques|
   |Région|Nom de la région Azure que vous avez utilisée précédemment dans cet exercice|
   |Performances|**Standard**|
   |Redondance|**Stockage géo-redondant (GRS)**|
   |Permettre l’accès en lecture aux données en cas d’indisponibilité régionale|Désactivé|

1. Sous l’onglet **Avancé**, passez en revue les options disponibles, acceptez les valeurs par défaut, puis sélectionnez **Suivant : Mise en réseau >**.
1. Sous l’onglet **Mise en réseau**, effectuez les actions suivantes, puis sélectionnez **Vérifier**.

   1. Sélectionnez **Activer l’accès public à partir de réseaux virtuels et d’adresses IP sélectionnés**.
   1. Dans la section **Réseaux virtuels**, vérifiez que la liste déroulante **Abonnement au réseau virtuel** affiche le nom de l’abonnement Azure que vous utilisez dans ce labo.
   1. Dans la section **Réseaux virtuels**, dans la liste déroulante **Réseau virtuel**, sélectionnez **acss-infra-VNET**.
   1. Dans la liste déroulante **Sous-réseaux**, sélectionnez les sous-réseaux **app**, **db** et **dmz**.

1. Sous l’onglet **Vérifier**, attendez que le processus de validation se termine et sélectionnez **Créer**.

   >**Remarque** : Attendez la fin du processus de provisionnement. L’approvisionnement doit prendre moins d’une minute.

1. Dans la page **Votre déploiement a été effectué**, sélectionnez **Accéder à la ressource**.
1. Dans la page du compte de stockage, dans le menu de navigation vertical situé à gauche, dans la section **Stockage de données**, sélectionnez **Conteneurs**.
1. Sélectionnez **+ Conteneur**.
1. Dans le volet **Nouveau conteneur**, dans la zone de texte **Nom**, entrez **sapbits** et sélectionnez **Créer**.

   >**Remarque** : Le conteneur **sapbits** hébergera le support d’installation SAP.

#### Tâche 5 : Configurer l’autorisation de l’identité managée affectée par l’utilisateur Microsoft Entra

Durant cette tâche, vous utilisez une attribution de rôle RBAC (contrôle d’accès en fonction du rôle) Azure pour accorder une autorisation à l’identité managée affectée par l’utilisateur Microsoft Entra. L’identité managée est utilisée pour l’accès au déploiement à l’abonnement Azure et au compte Stockage Azure à usage général v2 créé lors de la tâche précédente.

1. Dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Identités managées**.
1. Dans la page Identités managées, sélectionnez l’entrée **acss-infra-MI**.
1. Dans la page **acss-infra-MI**, dans le menu de navigation vertical sur le côté gauche, sélectionnez **Attributions de rôles Azure**.
1. Dans la page **Attributions de rôles Azure**, sélectionnez **+ Ajouter une attribution de rôle (préversion)**.
1. Dans le volet **+ Ajouter une attribution de rôle (préversion)**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

   |Paramètre|Valeur|
   |---|---|
   |Étendue|**Abonnement**|
   |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Rôle|**Rôle de service Centre Azure pour les solutions SAP**|

1. De retour dans la page **Attributions de rôles Azure**, sélectionnez **+ Ajouter une attribution de rôle (préversion)**.
1. Dans le volet **+ Ajouter une attribution de rôle (préversion)**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

   |Paramètre|Valeur|
   |---|---|
   |Étendue|**Stockage**|
   |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Ressource|Nom du compte Stockage Azure que vous avez créé lors de la tâche précédente|
   |Rôle|**Lecteur et accès aux données**|

#### Tâche 6 : Créer un compte de partages de fichiers Azure Premium

Durant cette tâche, vous créez un compte de partages de fichiers Azure Premium utilisé pour implémenter le répertoire de transport SAP.

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Comptes de stockage**.
1. Dans la page **Comptes de stockage**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un compte de stockage**, spécifiez les paramètres suivants et sélectionnez **Suivant : Avancé >**.

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|**acss-infra-RG**|
   |Nom du compte de stockage|Nom global unique comprenant entre 3 et 24 caractères alphanumériques|
   |Région|Nom de la région Azure que vous avez utilisée précédemment dans cet exercice|
   |Performances|**Premium**|
   |Type de compte Premium|**Partages de fichiers**|
   |Redondance|**Stockage redondant interzone (ZRS)**|

1. Sous l’onglet **Avancé**, désactivez le paramètre **Exiger un transfert sécurisé pour les opérations d’API REST**, puis sélectionnez **Suivant : Mise en réseau >**.

   >**Remarque** : Le protocole NFS ne prend pas en charge le chiffrement, mais s’appuie plutôt sur la sécurité au niveau du réseau. Ce paramètre doit être désactivé pour que le protocole NFS fonctionne.

1. Sous l’onglet **Mise en réseau**, effectuez les actions suivantes, puis sélectionnez **Vérifier**.

   1. Sélectionnez **Activer l’accès public à partir de réseaux virtuels et d’adresses IP sélectionnés**.
   1. Dans la section **Réseaux virtuels**, vérifiez que la liste déroulante **Abonnement au réseau virtuel** affiche le nom de l’abonnement Azure que vous avez utilisé dans ce labo.
   1. Dans la section **Réseaux virtuels**, dans la liste déroulante **Réseau virtuel**, sélectionnez **acss-infra-VNET**.
   1. Dans la liste déroulante **Sous-réseaux**, sélectionnez les sous-réseaux **app**, **db** et **dmz**.

   >**Remarque** : En général, évitez d’autoriser l’accès à vos ressources internes à partir de sous-réseaux de périmètre. En l’occurrence, la seule raison de le faire est d’autoriser la validation de cet accès plus loin dans ce labo.

1. Sous l’onglet **Vérifier**, attendez que le processus de validation se termine et sélectionnez **Créer**.

   >**Remarque** : Attendez la fin du processus de provisionnement. L’approvisionnement doit prendre moins d’une minute.

1. Dans la page **Votre déploiement a été effectué**, sélectionnez **Accéder à la ressource**.
1. Dans la page du compte de stockage, dans le menu de navigation vertical situé à gauche, dans la section **Stockage de données**, sélectionnez **Partages de fichiers**, puis **+ Partage de fichiers**.
1. Sous l’onglet **Informations de base** de la page **Nouveau partage de fichiers**, spécifiez les paramètres suivants, puis sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Nom|**trans**|
   |Capacité allouée|**128**|
   |Protocol|**NFS**|
   |Squash racine|*Pas de Squash racine**|

1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine, puis sélectionnez **Créer**.

   >**Remarque** : Attendez la fin de l’approvisionnement du partage de fichiers. L’approvisionnement ne devrait prendre que quelques secondes.

1. Dans la page **Se connecter à ce partage NFS à partir de Linux**, dans la liste déroulante **Sélectionner votre distribution Linux**, sélectionnez **SUSE** et passez en revue les exemples de commandes pour monter ce partage NFS.

#### Tâche 7 : Créer et configurer un groupe de sécurité réseau

Durant cette tâche, vous créez et configurez un groupe de sécurité réseau (NSG) utilisé pour restreindre l’accès sortant à partir de sous-réseaux du réseau virtuel qui héberge le déploiement. Pour ce faire, vous pouvez bloquer la connectivité Internet tout en autorisant explicitement les connexions aux services suivants :

- Points de terminaison d’infrastructure de mise à jour SUSE ou Red Hat
- Stockage Azure
- Azure Key Vault
- Microsoft Entra ID
- Azure Resource Manager

>**Remarque** : En général, il est préférable d’utiliser Pare-feu Azure plutôt que des groupes de sécurité réseau afin de sécuriser la connectivité réseau pour votre déploiement SAP. Ce labo couvre les deux options.

1. Sur l’ordinateur de labo, dans le navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Groupes de sécurité réseau**.
1. Dans la page **Groupe de sécurité réseau**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un groupe de sécurité réseau**, spécifiez les paramètres suivants, puis sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|**acss-infra-RG**|
   |Nom|**acss-infra-NSG**|
   |Région|Nom de la région Azure que vous avez utilisée précédemment dans cet exercice|

1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.

   >**Remarque** : Attendez la fin du processus de provisionnement. L’approvisionnement doit prendre moins d’une minute.

1. Dans la page **Votre déploiement a été effectué**, sélectionnez **Accéder à la ressource**.

   >**Remarque** : Par défaut, les règles intégrées des groupes de sécurité réseau autorisent tout le trafic sortant, tout le trafic au sein du même réseau virtuel ainsi que tout le trafic entre les réseaux virtuels appairés. Du point de vue de la sécurité, vous devez envisager de restreindre ce comportement par défaut. La configuration proposée limite la connectivité sortante à Internet et Azure. Vous pouvez également utiliser des règles de groupe de sécurité réseau pour restreindre la connectivité au sein d’un réseau virtuel.

1. Dans la page **acss-infra-NSG**, dans le menu de navigation vertical situé à gauche, dans la section **Paramètres**, sélectionnez **Règles de sécurité de trafic sortant**.
1. Dans la page **acss-infra-NSG \| Règles de sécurité de trafic sortant**, sélectionnez **+ Ajouter**.
1. Dans la page **Ajouter une règle de sécurité de trafic sortant**, spécifiez les paramètres suivants et sélectionnez **Ajouter** :

   >**Remarque** : La règle suivante doit être ajoutée pour autoriser explicitement la connectivité aux points de terminaison d’infrastructure de mise à jour Red Hat.

   >**Remarque** : Pour identifier les adresses IP à utiliser pour RHEL, consultez [Préparer le réseau pour le déploiement d’infrastructure](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)

   |Paramètre|Valeur|
   |---|---|
   |Source|**Any**|
   |Source port ranges|*|
   |Destination|**Adresses IP**|
   |Plages d’adresses IP/CIDR de destination|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|
   |Service|**Personnalisée**|
   |Plages de ports de destination|*|
   |Protocol|**Any**|
   |Action|**Autoriser**|
   |Priority|**300**|
   |Nom|**AllowAnyRHELOutbound**|
   |Description|**Autoriser la connectivité sortante aux points de terminaison d’infrastructure de mise à jour RHEL**|

1. Dans la page **acss-infra-NSG \| Règles de sécurité de trafic sortant**, sélectionnez **+ Ajouter**.
1. Dans la page **Ajouter une règle de sécurité de trafic sortant**, spécifiez les paramètres suivants et sélectionnez **Ajouter** :

   >**Remarque** : La règle suivante doit être ajoutée pour autoriser explicitement la connectivité aux points de terminaison d’infrastructure de mise à jour SUSE.

   >**Remarque** : Pour identifier les adresses IP à utiliser pour SUSE, consultez [Préparer le réseau pour le déploiement d’infrastructure](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)

   |Paramètre|Valeur|
   |---|---|
   |Source|**Any**|
   |Source port ranges|*|
   |Destination|**Adresses IP**|
   |Plages d’adresses IP/CIDR de destination|**52.188.224.179,52.186.168.210,52.188.81.163,40.121.202.140**|
   |Service|**Personnalisée**|
   |Plages de ports de destination|*|
   |Protocol|**Any**|
   |Action|**Autoriser**|
   |Priority|**305**|
   |Nom|**AllowAnySUSEOutbound**|
   |Description|**Autoriser la connectivité sortante aux points de terminaison d’infrastructure de mise à jour SUSE**|

   >**Remarque** : La règle suivante doit être ajoutée pour autoriser explicitement la connectivité à Stockage Azure.

1. Dans la page **acss-infra-NSG \| Règles de sécurité de trafic sortant**, sélectionnez **+ Ajouter**.
1. Dans la page **Ajouter une règle de sécurité de trafic sortant**, spécifiez les paramètres suivants et sélectionnez **Ajouter** :

   |Paramètre|Valeur|
   |---|---|
   |Source|**Any**|
   |Source port ranges|*|
   |Destination|**Étiquette du service**|
   |Identification de destination|**Stockage**|
   |Service|**Personnalisée**|
   |Plages de ports de destination|*|
   |Protocol|**Any**|
   |Action|**Autoriser**|
   |Priority|**400**|
   |Nom|**AllowAnyCustomStorageOutbound**|
   |Description|**Autoriser la connectivité sortante à Stockage Azure**|

   >**Remarque** : Vous pouvez remplacer l’étiquette de service **Stockage** par une étiquette propre à la région, telle que **Stockage.EastUS**.

   >**Remarque** : La règle suivante doit être ajoutée pour autoriser explicitement la connectivité à Azure Key Vault.

1. Dans la page **acss-infra-NSG \| Règles de sécurité de trafic sortant**, sélectionnez **+ Ajouter**.
1. Dans la page **Ajouter une règle de sécurité de trafic sortant**, spécifiez les paramètres suivants et sélectionnez **Ajouter** :

   |Paramètre|Valeur|
   |---|---|
   |Source|**Any**|
   |Source port ranges|*|
   |Destination|**Étiquette du service**|
   |Identification de destination|**AzureKeyVault**|
   |Service|**Personnalisée**|
   |Plages de ports de destination|*|
   |Protocol|**Any**|
   |Action|**Autoriser**|
   |Priority|**500**|
   |Nom|**AllowAnyCustomKeyVaultOutbound**|
   |Description|**Autoriser la connectivité sortante à Azure Key Vault**|

   >**Remarque** : La règle suivante doit être ajoutée pour autoriser explicitement la connectivité à Microsoft Entra D.

1. Dans la page **acss-infra-NSG \| Règles de sécurité de trafic sortant**, sélectionnez **+ Ajouter**.
1. Dans la page **Ajouter une règle de sécurité de trafic sortant**, spécifiez les paramètres suivants et sélectionnez **Ajouter** :

   |Paramètre|Valeur|
   |---|---|
   |Source|**Any**|
   |Source port ranges|*|
   |Destination|**Étiquette du service**|
   |Identification de destination|**AzureActiveDirectory**|
   |Service|**Personnalisée**|
   |Plages de ports de destination|*|
   |Protocol|**Any**|
   |Action|**Autoriser**|
   |Priority|**600**|
   |Nom|**AllowAnyCustomEntraIDOutbound**|
   |Description|**Autoriser la connectivité sortante Microsoft Entra ID**|

   >**Remarque** : La règle suivante doit être ajoutée pour autoriser explicitement la connectivité à Azure Resource Manager.

1. Dans la page **acss-infra-NSG \| Règles de sécurité de trafic sortant**, sélectionnez **+ Ajouter**.
1. Dans la page **Ajouter une règle de sécurité de trafic sortant**, spécifiez les paramètres suivants et sélectionnez **Ajouter** :

   |Paramètre|Valeur|
   |---|---|
   |Source|**Any**|
   |Source port ranges|*|
   |Destination|**Étiquette du service**|
   |Identification de destination|**AzureResourceManager**|
   |Service|**Personnalisée**|
   |Plages de ports de destination|*|
   |Protocol|**Any**|
   |Action|**Autoriser**|
   |Priority|**700**|
   |Nom|**AllowAnyCustomARMOutbound**|
   |Description|**Autoriser la connectivité sortante à Azure Resource Manager**|

   >**Remarque** : La dernière règle doit être ajoutée pour bloquer explicitement la connectivité à Internet.

1. Dans la page **acss-infra-NSG \| Règles de sécurité de trafic sortant**, sélectionnez **+ Ajouter**.
1. Dans la page **Ajouter une règle de sécurité de trafic sortant**, spécifiez les paramètres suivants et sélectionnez **Ajouter** :

   |Paramètre|Valeur|
   |---|---|
   |Source|**Any**|
   |Source port ranges|*|
   |Destination|**Étiquette du service**|
   |Identification de destination|**Internet**|
   |Service|**Personnalisée**|
   |Plages de ports de destination|*|
   |Protocol|**Any**|
   |Action|**Deny**|
   |Priority|**1 000**|
   |Nom|**DenyAnyCustomInternetOutbound**|
   |Description|**Refuser la connectivité sortante à Internet**|

   >**Remarque** : Pour finir, vous devez affecter le groupe de sécurité réseau aux sous-réseaux appropriés du réseau virtuel qui hébergera le déploiement SAP.

1. Dans le volet **Ajouter une règle de sécurité de trafic sortant**, dans le menu de navigation vertical situé à gauche, dans la section **Paramètres**, sélectionnez **Sous-réseaux**.
1. Dans la page **acss-infra-NSG \| Sous-réseaux**, sélectionnez **+ Associer**.
1. Dans le volet **Associer un sous-réseau**, dans la liste déroulante **Réseau virtuel**, sélectionnez **acss-intra-VNET (acss-infra-RG)**, dans la liste déroulante **Sous-réseau**, sélectionnez **app**, puis sélectionnez **OK**.
1. Dans le volet **Associer un sous-réseau**, dans la liste déroulante **Réseau virtuel**, sélectionnez **acss-intra-VNET (acss-infra-RG)**, dans la liste déroulante **Sous-réseau**, sélectionnez **db**, puis sélectionnez **OK**.

#### Tâche 8 : Créer une machine virtuelle Azure

Durant cette tâche, vous créez une machine virtuelle Azure utilisée pour l’installation des logiciels SAP dans le cadre d’un déploiement du Centre Azure pour les solutions SAP.

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **Machines virtuelles**.
1. Dans la page **Machines virtuelles**, sélectionnez **+ Créer** et, dans le menu déroulant, sélectionnez **Machine virtuelle Azure**.
1. Sous l’onglet **Informations de base** de la page **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Disques >** (conservez les valeurs par défaut de tous les autres paramètres) :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|**acss-infra-RG**|
   |Nom de la machine virtuelle|**acss-infra-vm0**|
   |Région|Nom de la région Azure que vous avez utilisée précédemment dans cet exercice|
   |Options de disponibilité|**Aucune redondance de l’infrastructure requise**|
   |Type de sécurité|**Machine virtuelle à lancement fiable**|
   |Image|**Ubuntu Server 20.04 LTS – x64 Gen2**|
   |Architecture de machine virtuelle|**x64**|
   |Exécuter avec une remise Azure Spot|disabled|
   |Taille|**Standard_B2ms**|
   |Type d’authentification|**Mot de passe**|
   |Nom d’utilisateur|Tout nom d’utilisateur valide|
   |Mot de passe|tout mot de passe complexe de votre choix|
   |Aucun port d’entrée public|**Aucun**|
   
    > **Remarque** : Veillez à mémoriser le nom d’utilisateur et le mot de passe que vous avez spécifiés. Vous en aurez besoin plus tard dans ce labo.

1. Sous l’onglet **Disques**, acceptez les valeurs par défaut et sélectionnez **Suivant : Mise en réseau >**.
1. Sous l’onglet **Mise en réseau**, spécifiez les paramètres suivants, puis sélectionnez **Suivant : Gestion >** (conservez les valeurs par défaut de tous les autres paramètres) :

   |Paramètre|Valeur |
   |---|---|
   |Réseau virtuel|**acss-infra-VNET**|
   |Sous-réseau|**dmz**|
   |Adresse IP publique|**Aucun**|
   |Groupe de sécurité réseau de la carte réseau|**Aucun**|
   |Supprimer la carte réseau lors de la suppression de la machine virtuelle|enabled|
   |Options d’équilibrage de charge|**Aucun**|

1. Sous l’onglet **Gestion**, conservez les valeurs par défaut de tous les paramètres, puis sélectionnez **Suivant : Surveillance >**
1. Sous l’onglet **Surveillance**, affectez la valeur **Désactiver** à **Diagnostics de démarrage** et sélectionnez **Suivant : Avancé >** (conservez les valeurs par défaut de tous les autres paramètres)
1. Sous l’onglet **Avancé**, sélectionnez **Vérifier + créer** (conservez les valeurs par défaut de tous les paramètres).
1. Sous l’onglet **Vérifier + créer** du menu **Créer une machine virtuelle**, sélectionnez **Créer**.

   > **Remarque** : Attendez la fin du déploiement. L’approvisionnement peut durer environ trois minutes.

#### Tâche 9 : Configurer la machine virtuelle Azure

Durant cette tâche, vous vous connectez à la machine virtuelle Azure à l’aide d’Azure Bastion et vous la configurez pour l’installation des logiciels SAP. 

> **Remarque** : Avant de commencer cette tâche, vérifiez que l’approvisionnement Azure Bastion est terminé.

> **Remarque** : Vérifiez que votre navigateur web ne bloque pas les fenêtres contextuelles et, le cas échéant, désactivez la fonctionnalité de bloqueur de fenêtres contextuelles.

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **Machines virtuelles**. 
1. Dans la page **machines virtuelles**, sélectionnez l’entrée **acss-infra-vm0**.
1. Dans la page **acss-infra-vm0**, dans la barre d’outils, sélectionnez **Se connecter** et, dans le menu déroulant, sélectionnez **Se connecter via Bastion**.
1. Dans la page **acss-infra-vm0 \| Bastion**, vérifiez que **Type d’authentification** est défini sur **Mot de passe de la machine virtuelle**, dans les zones de texte **Nom d’utilisateur** et **Mot de passe**, entrez le nom d’utilisateur et le mot de passe que vous avez définis lors de l’approvisionnement de la machine virtuelle Azure, vérifiez que la case **Ouvrir dans un nouvel onglet de navigateur** est cochée, puis sélectionnez **Se connecter**.

   > **Remarque** : Cela doit ouvrir un autre onglet de fenêtre de navigateur web affichant la session shell en cours d’exécution sur la machine virtuelle Azure.

   > **Remarque** : Pour préparer le serveur Ubuntu pour le chargement du support d’installation SAP, vous allez installer Azure CLI.

1. Sous l’onglet de navigateur nouvellement ouvert, dans la session shell, exécutez la commande suivante pour installer Azure CLI :

   ```bash
   curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
   ```

1. Dans la session shell, exécutez la commande suivante pour installer PIP3 :

   ```bash
   sudo apt install python3-pip
   ```

1. Dans la session shell, exécutez la commande suivante pour installer Ansible 2.13.19 :

   ```bash
   sudo pip3 install ansible-core==2.13.9
   ```

1. Dans la session shell, exécutez la commande suivante pour installer les modules de collection Ansible Galaxy :

   ```bash
   sudo ansible-galaxy collection install ansible.netcommon:==5.0.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.posix:==1.5.1 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.utils:==2.9.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install ansible.windows:==1.13.0 -p /opt/ansible/collections
   sudo ansible-galaxy collection install community.general:==6.4.0 -p /opt/ansible/collections
   ```

1. Dans la session shell, exécutez la commande suivante pour cloner le dépôt d’exemples d’automatisation SAP à partir de GitHub :

   ```bash
   git clone https://github.com/Azure/SAP-automation-samples.git
   ```

1. Dans la session shell, exécutez la commande suivante pour cloner le dépôt d’automatisation SAP à partir de GitHub :

   ```bash
   git clone https://github.com/Azure/sap-automation.git
   ```

1. Dans la session shell, exécutez la commande suivante pour mettre fin à la session :

   ```bash
   logout
   ```

1. Lorsque vous y êtes invité, sélectionnez **Fermer**.

#### Tâche 10 : Supprimer les ressources Azure

Durant cette tâche, vous supprimez toutes les ressources approvisionnées dans ce labo.

1. Sur l’ordinateur de labo, dans la fenêtre de navigateur web affichant le portail Azure, sélectionnez l’icône **Cloud Shell** pour ouvrir le volet Cloud Shell. Si nécessaire, sélectionnez **Bash** pour démarrer une session d’interpréteur de commandes Bash. 

   > **Remarque** : Si c’est la première fois que vous lancez Cloud Shell dans l’abonnement Azure que vous utilisiez dans ce labo, vous êtes invité à créer un partage de fichiers Azure pour conserver les fichiers Cloud Shell. Dans ce cas, acceptez les valeurs par défaut, ce qui entraîne la création d’un compte de stockage dans un groupe de ressources généré automatiquement.

1. Dans le volet Cloud Shell, exécutez la commande suivante pour supprimer le groupe de ressources **acss-infra-RG** et toutes ses ressources.

   ```cli
   az group delete --name 'acss-infra-RG' --no-wait --yes
   ```

   > **Remarque** : La commande s’exécute de manière asynchrone (comme déterminé par le paramètre `--nowait`). Par conséquent, même si l’invite d’interpréteur de commandes s’affiche immédiatement après l’appel, quelques minutes s’écoulent avant que le groupe de ressources et ses ressources ne soient réellement supprimés.

1. Fermez le volet Cloud Shell.
