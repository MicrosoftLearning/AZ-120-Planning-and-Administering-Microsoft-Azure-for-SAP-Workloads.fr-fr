---
lab:
  title: 06b - Vue d’ensemble du déploiement et de la maintenance du Centre Azure pour les solutions SAP (ACSS)
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Module AZ 1006 : Concevoir et implémenter une infrastructure pour prendre en charge les charges de travail SAP sur Azure
# AZ-1006 Labo de cours en une journée : Vue d’ensemble du déploiement et de la maintenance du Centre Azure pour les solutions SAP (ACSS)

Temps estimé : 100 minutes

Toutes les tâches de ce labo de cours en une journée AZ-1006 sont effectuées à partir du portail Azure

## Objectifs

À la fin de ce labo, vous serez en mesure de faire ce qui suit :

- Déployer et maintenir l’infrastructure hébergeant des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

## Instructions

### Exercice 1 : Implémenter les prérequis minimaux au déploiement d’évaluation des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 30 minutes

Dans cet exercice, vous implémentez les prérequis minimaux nécessaires au déploiement d’évaluation des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP. Cela comprend les activités suivantes :

>**Important** : Les conditions préalables implémentées dans cet exercice ne sont *pas* destinées à représenter les meilleures pratiques pour le déploiement de charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP. Leur objectif est de réduire le temps, le coût et les ressources nécessaires pour évaluer la mécanique du déploiement de charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP et d’effectuer des tâches de gestion et de maintenance post-déploiement. L’implémentation des prérequis inclut les activités suivantes :

- Création d’une identité managée affectée par l’utilisateur Microsoft Entra devant être utilisée par le Centre Azure pour les solutions SAP pour l’accès de Stockage Azure pendant son déploiement.
- Octroi, à l’identité managée affectée par l’utilisateur Microsoft Entra utilisée pour le déploiement, de l’accès à l’abonnement Azure et au compte Stockage Azure à usage général v2
- Création du réseau virtuel Azure qui héberge toutes les machines virtuelles Azure incluses dans le déploiement.
- Création et configuration d’un groupe de sécurité réseau (NSG) qui est utilisé pour restreindre l’accès sortant à partir de sous-réseaux du réseau virtuel qui héberge le déploiement.
- Création et configuration d’une passerelle NAT qui permettra la connectivité sortante à partir de machines virtuelles Azure qui font partie du déploiement.

Ces activités correspondent aux tâches suivantes de cet exercice :

- Tâche 1 : Créer une identité managée affectée par l’utilisateur Microsoft Entra
- Tâche 2 : Configurez les attributions de rôles RBAC (Contrôle d’accès en fonction du rôle) Azure pour l’identité managée affectée par l'utilisateur Microsoft Entra ID
- Tâche 3 : Créer le réseau virtuel Azure
- Tâche 4 : Créer et configurer un groupe de sécurité réseau
- Tâche 5 : Créer et configurer une passerelle NAT

#### Tâche 1 : Créer une identité managée affectée par l’utilisateur Microsoft Entra

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

1. Sous l’onglet **Vérifier**, attendez que le processus de validation se termine et sélectionnez **Créer**.

   >**Remarque** : N’attendez pas que le processus d’approvisionnement se termine, mais passez à la tâche suivante. L’approvisionnement ne devrait prendre que quelques secondes.

   >**Remarque** : Durant l’une des tâches à venir, vous autoriserez l’accès par l’identité managée au compte de stockage hébergeant le support d’installation SAP afin de prendre en charge l’installation de logiciels SAP via le Centre Azure pour les solutions SAP.

#### Tâche 2 : Configurer les attributions de rôles RBAC (Contrôle d’accès en fonction du rôle) Azure pour le compte d’utilisateur Microsoft Entra ID qui sera utilisé pour effectuer le déploiement

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, rechercher et sélectionner **Abonnements**.
1. Dans la page **Abonnements**, sélectionnez l’entrée représentant l’abonnement Azure que vous utiliserez pour ce labo. 
1. Dans la page affichant les propriétés de l’abonnement Azure, sélectionnez **Contrôle d’accès (IAM)**.
1. Sur la page **Contrôle d’accès (IAM)**, sélectionner **+ Ajouter** puis, dans le menu déroulant, sélectionner **Ajouter une attribution de rôle**.
1. Sous l’onglet **Rôle** de la page **Ajouter une attribution** de rôle, dans la liste des **rôles de fonction de travail**, rechercher et sélectionner l’entrée **Rôle de service du Centre Azure pour les solutions SAP**, puis sélectionner **Suivant**.
1. Sous l’onglet **Membres** de la page **Ajouter une attribution de rôle**, pour **Attribuer l’accès à**, sélectionner **Identité managée**, puis cliquer sur **+ Sélectionner des membres**.
1. Dans le volet **Sélectionner des identités managées**, spécifier les paramètres suivants :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Identité gérée|**Identité managée affectée par l’utilisateur**|
   |Sélectionnez|**acss-infra-MI**|
1. Dans la liste des identités managées, sélectionner l’entrée **acss-infra-MI**, puis cliquer sur **Sélectionner**.
1. De retour sous l’onglet **Membres**, sélectionnez **Vérifier + attribuer**.  
1. Sous l’onglet **Vérifier + attribuer**, sélectionnez **Vérifier + attribuer**.
   
   ##### Ajoutez : « Administrateur d’identité de Centre Azure pour les solutions SAP »
   
1. De retour sur la page **Contrôle d’accès (IAM)**, sélectionner **+ Ajouter** puis, dans le menu déroulant, sélectionner **Ajouter une attribution de rôle**.
1. Sous l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, dans la liste des **rôles de fonction de travail**, rechercher et sélectionner l’entrée **Administrateur du Centre Azure pour les solutions SAP**, puis sélectionner **Suivant**.
1. Dans l’onglet **Membres**
   - pour **Attribuer l’accès à**, sélectionner **Utilisateur, groupe ou principal du service**
   - Cliquer sur **+ Sélection de Membres**.
1. Dans le volet **Sélectionner des membres**, dans la zone de texte **Sélectionner**, entrer le nom du compte d’utilisateur Microsoft Entra ID que vous avez utilisé pour accéder à l’abonnement Azure que vous utilisez pour ce labo, sélectionnez-le dans la liste des résultats correspondant à votre entrée, puis cliquer sur **Sélectionner**.
1. De retour sous l’onglet **Membres**, sélectionnez **Vérifier + attribuer**.
1. Sous l’onglet **Vérifier + attribuer**, sélectionnez **Vérifier + attribuer**.
   
   ##### Ajoutez : « Opérateur d’identités gérées »
   
1. De retour sur la page **Contrôle d’accès (IAM)**, sélectionner **+ Ajouter** puis, dans le menu déroulant, sélectionner **Ajouter une attribution de rôle**.
1. Sous l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, dans la liste des **rôles de fonction de travail**, rechercher et sélectionner l’entrée **Opérateur d’identités gérées**, puis sélectionner **Suivant**.
1. Dans l’onglet **Membres**
   - pour **Attribuer l’accès à**, sélectionner **Utilisateur, groupe ou principal du service**
   - Cliquer sur **+ Sélection de Membres**.
1. Dans le volet **Sélectionner des membres**, dans la zone de texte **Sélectionner**, entrer le nom du compte d’utilisateur Microsoft Entra ID que vous avez utilisé pour accéder à l’abonnement Azure que vous utilisez pour ce labo, sélectionnez-le dans la liste des résultats correspondant à votre entrée, puis cliquer sur **Sélectionner**.
1. De retour sous l’onglet **Membres**, sélectionnez **Vérifier + attribuer**.
1. Sous l’onglet **Vérifier + attribuer**, sélectionnez **Vérifier + attribuer**.

#### Tâche 3 : Créer un réseau virtuel

Dans cette tâche, vous créez le réseau virtuel Azure qui héberge toutes les machines virtuelles Azure incluses dans le déploiement. De plus, au sein du réseau virtuel, vous créez les sous-réseaux suivants :

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
   |Région|Nom de la région Azure que vous avez utilisée dans la tâche précédente de cet exercice|

1. Sous l’onglet **Sécurité**, acceptez les paramètres par défaut et sélectionnez **Suivant**.

   >**Remarque** : Vous pourriez approvisionner Azure Bastion et le Pare-feu Azure à ce stade, mais vous les approvisionnerez séparément une fois le réseau virtuel créé.

1. Sous l’onglet **Adresses IP**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Espace d’adressage IP|**10.0.0.0/16 (65 536 adresses)**|

1. Dans la liste des sous-réseaux, sélectionnez l’icône de corbeille pour supprimer le sous-réseau **par défaut**.
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

#### Tâche 4 : Créer et configurer un groupe de sécurité réseau

Durant cette tâche, vous créez et configurez un groupe de sécurité réseau (NSG) qui est utilisé pour restreindre l’accès sortant à partir de sous-réseaux du réseau virtuel qui héberge le déploiement. Pour ce faire, vous pouvez bloquer la connectivité Internet tout en autorisant explicitement les connexions aux services suivants :

- Points de terminaison d’infrastructure de mise à jour SUSE ou Red Hat
- Stockage Azure
- Azure Key Vault
- Microsoft Entra ID
- Azure Resource Manager

>**Remarque** : En général, il est préférable d’utiliser Pare-feu Azure plutôt que des groupes de sécurité réseau afin de sécuriser la connectivité réseau pour votre déploiement SAP. 

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

1. Sur la page **acss-infra-NSG**, dans le menu de navigation, dans la section **Paramètres**, sélectionner **Règles de sécurité de trafic sortant**.
1. Dans la page **acss-infra-NSG \| Règles de sécurité de trafic sortant**, sélectionnez **+ Ajouter**.
1. Dans la page **Ajouter une règle de sécurité de trafic sortant**, spécifiez les paramètres suivants et sélectionnez **Ajouter** :

   >**Remarque** : La règle suivante doit être ajoutée pour autoriser explicitement la connectivité aux points de terminaison d’infrastructure de mise à jour Red Hat.

   >**Remarque** : Pour identifier les adresses IP à utiliser pour RHEL, consultez [Préparer le réseau pour le déploiement d’infrastructure](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network#allowlist-suse-or-red-hat-endpoints)

   |Paramètre|Valeur|
   |---|---|
   |Source|**Any**|
   |Source port ranges|*|
   |Destination|**Adresses IP**|
   |Plages d’adresses IP/CIDR de destination|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198,52.136.197.163,20.225.226.182,52.142.4.99,20.248.180.252,20.24.186.80**|
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

1. Dans le volet **Ajouter une règle de sécurité de trafic sortant**, dans le menu de navigation, dans la section **Paramètres**, sélectionnez **Sous-réseaux**.
1. Dans la page **acss-infra-NSG \| Sous-réseaux**, sélectionnez **+ Associer**.
1. Dans le volet **Associer un sous-réseau**, dans la liste déroulante **Réseau virtuel**, sélectionnez **acss-intra-VNET (acss-infra-RG)**, dans la liste déroulante **Sous-réseau**, sélectionnez **app**, puis sélectionnez **OK**.
1. Dans le volet **Associer un sous-réseau**, dans la liste déroulante **Réseau virtuel**, sélectionnez **acss-intra-VNET (acss-infra-RG)**, dans la liste déroulante **Sous-réseau**, sélectionnez **db**, puis sélectionnez **OK**.

#### Tâche 5 : Créer et configurer une passerelle NAT

Dans cette tâche, vous créez et configurez une passerelle NAT (Network Address Translation) qui permettra aux machines virtuelles Azure incluses dans le déploiement d’atteindre des services externes, tels que les points de terminaison SUSE et RHEL.

1. Sur l’ordinateur de labo, dans la fenêtre du navigateur web affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **NAT Gateway**.
1. Sur la page **Passerelles applicatives**, sélectionnez **+ Créer**.
1. Sous l’onglet **Bases** de la page **Créer une passerelle de traduction d’adresses réseau (NAT)**, spécifier les paramètres suivants, puis sélectionnez **Suivant : Adresse IP sortante>**  :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|**acss-infra-RG**|
   |Nom de la passerelle NAT|**acss-infra-NATGW**|
   |Région|Nom de la région Azure que vous avez utilisée précédemment dans cet exercice|
   |Zone de disponibilité|**Aucune zone**|
   |Délai d’inactivité TCP (minutes)|**4**|

1. Sous l’onglet **Adresse IP sortante**, sous la liste déroulante **Adresses IP publiques**, sélectionner le lien **Créer une nouvelle adresse IP publique**, dans le volet **Ajouter une adresse IP publique**, dans la zone de texte **Nom**, entrer **acss-infra-NATWG-PIP**, puis sélectionner **OK**.
1. De retour sous l’onglet **IP sortante**, sélectionner **Suivant : Sous-réseau >**.
1. Sous l’onglet **Sous-réseau**, dans la liste déroulante **Réseau virtuel**, sélectionnez **acss-infra-VNET**, dans la liste des sous-réseaux, sélectionner la case à cocher à côté de l’**application** et des entrées de **base de données**, puis sélectionner **Vérifier + créer** :
1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.

   >**Remarque** : Attendez la fin du processus de provisionnement. L’approvisionnement devrait prendre environ 1 minute.

### Exercice 2 : Déployer l’infrastructure qui héberge des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 40 minutes

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
   |Resource group|Le nom d’un **nouveau** groupe de ressources **acss-vi-RG**|
   |Nom (SID)|**VI1**|
   |Région|le nom de la région Azure hébergeant le déploiement SAP inscrit par ACSS ou une autre région dans la même zone géographique|
   |Type d’environnement|**Hors production**|
   |Produit SAP|**S/4HANA**|
   |Base de données|**HANA**|
   |Méthode de mise à l’échelle HANA|**Effectuer un scale-up (recommandé)**|
   |Type de déploiement|**Distribué avec haute disponibilité (HA)**|
   |Disponibilité du calcul|**99,95 % (groupe à haute disponibilité)**|
   |Réseau virtuel|**acss-infra-VNET**|
   |Sous-réseau d’application|**app (10.0.2.0/24)**|
   |Sous-réseau de base de données|**db (10.0.3.0/24)**|
   |Options d’image du système d’exploitation d’application|**Utiliser une image de marketplace**|
   |Image du système d’exploitation d’application|**Red Hat Enterprise Linux 8.2 pour applications SAP – x64 Gen2 version la plus récente**|
   |Options d’image du système d’exploitation de base de données|**Utiliser une image de marketplace**|
   |Image du système d’exploitation de base de données|**Red Hat Enterprise Linux 8.2 pour applications SAP – x64 Gen2 version la plus récente**|
   |Options de transport SAP|**Ne pas inclure le répertoire de transport SAP**|
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
   - 2 x Standard_D4ds_v5 pour les machines virtuelles ASCS (4 processeurs virtuels et 16 Gio de mémoire chacun)
   - 2 x Standard_D4ds_v5 pour les machines virtuelles d’application (4 processeurs virtuels et 16 Gio de mémoire chacun)
   - 2 x Standard_E16ds_v5 pour les machines virtuelles de base de données (16 processeurs virtuels et 128 Gio de mémoire chacun)

   >**Remarque** : Si nécessaire, vous pouvez demander une augmentation du quota en sélectionnant le lien **Demander un quota** pour une référence SKU spécifique des machines virtuelles et en faisant une demande d’augmentation de quota. Le traitement d’une demande prend généralement quelques minutes.

   >**Remarque** : Le Centre Azure pour les solutions SAP applique l’utilisation des références SKU de machine virtuelle prises en charge par SAP pendant le déploiement.

1. Sous l’onglet **Machines virtuelles**, dans la section **Disques de données**, sélectionnez le lien **Afficher et personnaliser la configuration**.
1. Dans la page **Configuration du disque de base de données**, passez en revue la configuration recommandée sans apporter de modifications, puis sélectionnez **Fermer**.
1. De retour sous l’onglet **Machines virtuelles**, sélectionnez **Suivant : Visualiser l’architecture**.
1. Sous l’onglet **Visualiser l’architecture**, passez en revue le diagramme illustrant l’architecture recommandée et sélectionnez **Vérifier + créer**.
1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine, cochez la case confirmant que vous disposez d’un quota suffisant dans la région de déploiement pour éviter de rencontrer une erreur « Quota insuffisant », puis sélectionnez **Créer**.
1. À l’invite, dans la fenêtre **Générer une nouvelle paire de clés**, sélectionnez **Télécharger la clé privée et créer une ressource**.

   >**Remarque** : La clé privée requise pour se connecter aux machines virtuelles Azure incluses dans le déploiement sera téléchargée sur l’ordinateur à partir duquel vous exécutez ce labo.

   >**Remarque** : Attendez la fin du déploiement. Cela peut prendre environ 25 minutes.

   >**Remarque** : Après le déploiement, vous pourrez procéder à l’installation de logiciels SAP à l’aide du Centre Azure pour les solutions SAP. Dans ce labo, vous allez explorer les capacités du Centre Azure pour les solutions SAP sans installer de logiciels SAP.

### Exercice 3 : Maintenir les charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 60 minutes

Dans cet exercice, vous passez en revue la gestion post-déploiement et la surveillance des charges de travail SAP à l’aide du Centre Azure pour les solutions SAP. Cela comprend les activités suivantes :

- Implémentation des conditions préalables à la sauvegarde des charges de travail SAP gérées par le Centre Azure pour les solutions SAP 
- Implémentation des conditions préalables à la récupération d'urgence des charges de travail SAP gérées par le Centre Azure pour les solutions SAP 
- Examen des options de supervision disponibles pour les charges de travail SAP gérées par le Centre Azure pour les solutions SAP 
- Suppression de toutes les ressources Azure approvisionnées dans ce labo.

Ces activités correspondent aux tâches suivantes de cet exercice :

- Tâche 1 : Implémentation des conditions préalables à la sauvegarde des charges de travail SAP gérées par le Centre Azure pour les solutions SAP 
- Tâche 2 : Implémentation des conditions préalables à la récupération d'urgence des charges de travail SAP gérées par le Centre Azure pour les solutions SAP 
- Tâche 3 : Examen des options de supervision pour les charges de travail SAP gérées par le Centre Azure pour les solutions SAP 
- Tâche 4 : Supprimer les ressources Azure approvisionnées dans ce labo

#### Tâche 1 : Implémentation des conditions préalables à la sauvegarde des charges de travail SAP gérées par le Centre Azure pour les solutions SAP

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
1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.

   >**Remarque** : Attendez la fin du processus de provisionnement. L’approvisionnement peut durer environ 2 minutes.

1. Dans la page **Votre déploiement a été effectué**, sélectionnez **Accéder à la ressource**.
1. Sur la page **acss-backup-RSV**, dans le menu de navigation vertical situé à gauche, dans la section **Gérer**, sélectionner **Stratégies de sauvegarde Azure**.
1. Dans la **page des stratégies de sauvegarde \| acss-backup-RSV**, sélectionner **+ Ajouter**.
1. Dans la page **Sélectionner un type de stratégie**, passer en revue les types de stratégie disponibles.

   >**Remarque** : Vous allez commencer par configurer la stratégie de sauvegarde pour les machines virtuelles Azure hébergeant la base de données, les serveurs d’applications et l’instance des services centraux SAP.

1. Sur la page **Sélectionner le type de stratégie**, sélectionner **Machine virtuelle Azure**.
1. Sur la page **Créer une stratégie**, passer en revue les différences entre les sous-types de stratégie **Standard** et **Amélioré**, puis sélectionner **Amélioration**.

   >**Remarque** : Vous devez baser votre choix sur vos besoins de résilience. De même, vous devez ajuster les valeurs listées suivantes en fonction de vos besoins.

1. Dans la zone de texte **Nom de la stratégie**, entrer **acss-vm-enhanced-backup-policy**.
1. Définissez la fréquence de planification de sauvegarde sur **Toutes les heures**, heure de début sur **6h00**, planification sur **Toutes les 6 heures**, durée sur **12 heures** et fuseau horaire sur votre fuseau horaire local.
1. Définir la valeur de **Conservation des instantanés de récupération d’instance pour** sur **5** jours.
1. Définir la **rétention des points de sauvegarde quotidiens** sur **60** jours.
1. Définir la période de rétention des points de sauvegarde hebdomadaires, mensuels et annuels en fonction de vos préférences.

   >**Remarque** : Pour activer la hiérarchisation, vous devez activer les points de sauvegarde mensuels ou annuels.

   >**Remarque** : Vous avez également la possibilité de concevoir la convention d’affectation de noms du groupe de ressources généré automatiquement par Sauvegarde Azure.

1. Sélectionnez **Créer**.
1. Dans la page **Votre déploiement a été effectué**, sélectionnez **Accéder à la ressource**.
1. Sur la page **acss-backup-RSV**, dans le menu de navigation vertical situé à gauche, dans la section **Gérer**, sélectionner **Stratégies de sauvegarde Azure**.
1. Dans la **page des stratégies de sauvegarde \| acss-backup-RSV**, sélectionner **+ Ajouter**.

   >**Remarque** : Ensuite, vous allez configurer les stratégies de sauvegarde pour la base de données HANA. La configuration présentée ici suit les instructions décrites dans l’article MS Learn [Sauvegarder l’instance de base de données SAP HANA instantané s sur des machines virtuelles Azure](https://learn.microsoft.com/en-us/azure/backup/sap-hana-database-instances-backup).

1. Dans la page **Sélectionner un type de stratégie**, sélectionner **SAP HANA dans une machine virtuelle Azure (Base de données via Backint)**
1. Dans la page **Créer une stratégie**, dans la zone de texte **Nom de la stratégie**, entrer **acss-hanadb-backint-backup-policy**.
1. Accepter les paramètres par défaut pour les sauvegardes complètes et de fichiers journaux. Conservation des sauvegardes différentielles et incrémentielles désactivé.

   >**Remarque** : Vous avez la possibilité d’activer la compression de sauvegarde HANA et de déplacer les points de récupération éligibles vers l’archivage du coffre. 

1. Sélectionnez **Créer**.

1. Dans la page **Votre déploiement a été effectué**, sélectionnez **Accéder à la ressource**.
1. Sur la page **acss-backup-RSV**, dans le menu de navigation vertical situé à gauche, dans la section **Gérer**, sélectionner **Stratégies de sauvegarde Azure**.
1. Dans la **page des stratégies de sauvegarde \| acss-backup-RSV**, sélectionner **+ Ajouter**.
1. Dans la page **Sélectionner un type de stratégie**, sélectionner **SAP HANA dans une machine virtuelle Azure (Instance de base de données via capture instantanée)**
1. Sur la page **Créer une stratégie**, dans la zone de texte **Nom de la stratégie**, entrer **acss-hanadb-backint-backup-policy**.
1. Définir la fréquence de la sauvegarde de capture instantané à 13h30 de votre fuseau horaire actuel.
1. Configurer pour conserver la récupération instantanée de capture instantanée pendant **5** jours.
1. Conservez la sélection par défaut du groupe de ressources instantané, qui doit être défini sur **acss-mgmt-RG**.
1. Dans la section **Identité Managée**, sélectionner **Créer une identité managée**. Cela ouvre automatiquement un autre onglet dans la fenêtre du navigateur web, affichant la page **Identité managée** dans le Portail Azure.
1. Sur la page **Identités Managés**, sélectionner **+ Créer**.
1. Sous l’onglet **De base** de la page **Créer une identité managée affectée par l’utilisateur**, spécifiez les paramètres suivants, puis sélectionnez **Vérifier + Créer** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|Le nom de l’abonnement Azure utilisé dans ce labo|
   |Resource group|**acss-mgmt-RG**|
   |Région|le nom de la région Azure hébergeant le déploiement SAP inscrit par ACSS|
   |Nom|**acss-mgmt-MI**|

1. Sous l’onglet **Vérifier**, attendez que le processus de validation se termine et sélectionnez **Créer**.

   >**Remarque** : Attendez la fin du processus de provisionnement. L’approvisionnement ne devrait prendre que quelques secondes.

1. Fermez l’onglet du navigateur actuel et revenez à celui qui affiche la page **Créer une stratégie**.
1. Dans la section **Identité managée**, dans la liste déroulante **Groupe de ressources**, sélectionner **acss-mgmt-RG** puis dans la liste déroulante **Identité managée**, sélectionner **acss-mgmt-MI**.
1. Sélectionnez **Créer**.

   >**Remarque** : Lors de la configuration de la sauvegarde au niveau du VIS de l’interface du Centre Azure pour les solutions SAP, vous pourrez tirer parti du coffre existant et de ses stratégies.

   >**Remarque** : Une fois la sauvegarde configurée au niveau du VIS, vous pouvez surveiller l’état des travaux de sauvegarde des machines virtuelles Azure et de la base de données HANA à partir de l’interface VIS dans le Portail Azure.

#### Tâche 2 : Implémentation des conditions préalables à la récupération d'urgence des charges de travail SAP gérées par le Centre Azure pour les solutions SAP 

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

   >**Important** : Notez que les espaces d’adressage IP diffèrent entre le réseau virtuel dans les régions primaires et secondaires. Cela est intentionnel, car il permettra de connecter les deux réseaux virtuels ensemble, ce qui est nécessaire pour configurer la réplication entre les serveurs de base de données hébergés dans les deux régions. Cette connexion peut être établie à l’aide de l’appairage de réseaux virtuels.

1. De retour sur l’onglet **Paramètres de réplication** de la page **Activer la réplication**, sélectionner **Suivant**.
1. Sous l’onglet **Gérer** de la page **Activer la réplication**, effectuer les actions suivantes :

   1. Sous la liste déroulante **Stratégie de réplication**, sélectionner **Créer nouveau**.
   1. Dans le volet **Créer une stratégie de réplication**, dans la zone de texte **Nom**, entrer **stratégie de rétention de 2 jours**, dans la zone de texte **Période de rétention (en jours)**, entrer **2**, puis sélectionner **OK**.
   1. Vous avez la possibilité de créer des groupes de réplication. Cette option n’est pas applicable à notre cas d’usage.
   1. Laissez l’option **Paramètres de mise à jour** des **paramètres d’extension**configurée pour **autoriser la gestion par ASR**.
   1. Acceptez le nom par défaut attribué automatiquement au **compte Automation**.

1. Sous l’onglet **Gérer** de la page **Activer la réplication**, sélectionner **Suivant**.
1. Sous l’onglet **Évaluation** de la page **Activer la réplication**, sélectionner **Activer la réplication**.

   >**Remarque** : Vous allez maintenant activer la réplication pour les deux machines virtuelles Azure restantes hébergeant les serveurs de base de données.

1. De retour sur la page **acss-dr-RSV \| Site Recovery**, dans la section **machines virtuelles Azure**, sélectionner **1. Activer la réplication**. 
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

   >**Remarque** : Vous devez configurer la réplication pour les deux machines virtuelles Azure restantes séparément, car elles sont connectées à un sous-réseau différent.

1. Sous l’onglet **Paramètres de réplication**, effectuez les actions suivantes :

   1. Si nécessaire, dans la liste déroulante **Emplacement cible**, sélectionner la région Azure dans laquelle vous avez créé le coffre Recovery Services **acss-dr-RSV**.
   1. Vérifiez que le nom de l’abonnement Azure utilisé dans ce labo apparaît dans la liste déroulante **Abonnement cible**.
   1. Dans la liste déroulante **Groupe de ressources cible**, sélectionner **acss-dr-RG**.
   1. Dans la liste déroulante **Réseau virtuel de basculement**, sélectionner **CONTOSO-VNET**.
   1. Dans la liste déroulante **Sous-réseau de basculement**, sélectionner **l’application (10.10.0.0/24)**.
   1. Dans la section **Stockage**, sélectionner le lien **configuration d’affichage/modification du stockage**.
   1. Dans la page **Personnaliser les paramètres cibles**, passer en revue la configuration résultante sans apporter de modifications, puis sélectionner **Annuler**.
   1. Dans la section **Options de disponibilité**, sélectionner le lien **Afficher/modifier les options de disponibilité**.
   1. Dans la page **Options de disponibilité**, vous avez la possibilité d’implémenter des groupes de placement de proximité pour les ressources cibles, mais n’apportez aucune modification et sélectionnez **Annuler**.

   >**Remarque** : Vous avez également la possibilité de configurer la réservation de capacité.

1. De retour sur l’onglet **Paramètres de réplication** de la page **Activer la réplication**, sélectionner **Suivant**.
1. Sous l’onglet **Gérer** de la page **Activer la réplication**, effectuer les actions suivantes :

   1. Sous la liste déroulante **Stratégie de réplication**, sélectionner **Créer nouveau**.
   1. Dans le volet **Créer une stratégie de réplication**, dans la zone de texte **Nom**, entrer **stratégie de rétention de 2 jours**, dans la zone de texte **Période de rétention (en jours)**, entrer **2**, puis sélectionner **OK**.
   1. Vous avez la possibilité de créer des groupes de réplication. Cette option n’est pas applicable à notre cas d’usage.
   1. Laissez l’option **Paramètres de mise à jour** des **paramètres d’extension**configurée pour **autoriser la gestion par ASR**.
   1. Acceptez le nom par défaut attribué automatiquement au **compte Automation**.

1. Sous l’onglet **Gérer** de la page **Activer la réplication**, sélectionner **Suivant**.
1. Sous l’onglet **Évaluation** de la page **Activer la réplication**, sélectionner **Activer la réplication**.

   >**Remarque** : La réplication initiale peut prendre beaucoup de temps à terminer. Compte tenu du temps limité alloué à ce laboratoire, se reporter à l’instructeur en ce qui concerne les étapes supplémentaires à effectuer dans le cadre de cette tâche. En l’absence de conseils spécifiques, passer directement à la tâche suivante.

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
