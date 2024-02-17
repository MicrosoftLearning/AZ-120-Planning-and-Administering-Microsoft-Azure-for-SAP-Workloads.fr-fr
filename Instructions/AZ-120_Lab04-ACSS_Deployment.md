---
lab:
  title: Automatiser le déploiement avec le Centre Azure pour les solutions SAP
  module: Design and implement an infrastructure to support SAP workloads on Azure
---

# Module AZ 120 : Concevoir et implémenter une infrastructure pour prendre en charge les charges de travail SAP sur Azure
# Labo : Automatiser le déploiement avec le Centre Azure pour les solutions SAP

Temps estimé : 100 minutes

Toutes les tâches de ce labo sont effectuées à partir du portail Azure

## Objectifs

À la fin de ce lab, vous serez en mesure de :

- Implémenter les prérequis au déploiement des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP
- Déployer l’infrastructure qui hébergera les charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

## Exercice 1 : Implémenter les prérequis au déploiement des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 60 minutes

Dans cet exercice, vous allez implémenter les prérequis au déploiement des charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP. Cela inclut les tâches suivantes :
- Réponse aux exigences relatives aux processeurs virtuels dans l’abonnement Azure cible
- Configuration des attributions de rôles RBAC (Contrôle d’accès en fonction du rôle) Azure pour le compte d’utilisateur Microsoft Entra ID qui sera utilisé pour effectuer le déploiement
- Création d’un compte de stockage associé au Centre Azure pour les solutions SAP utilisé pour le déploiement
- Création et configuration d’une identité managée affectée par l’utilisateur à utiliser par le Centre Azure pour les solutions SAP pour l’authentification et l’autorisation de son déploiement automatisé
- Création d’un groupe de sécurité réseau (NSG) à utiliser dans les sous-réseaux du réseau virtuel qui hébergera le déploiement
- Création de tables de routage à utiliser dans les sous-réseaux du réseau virtuel qui hébergera le déploiement
- Création et configuration du réseau virtuel qui hébergera le déploiement
- Déploiement du Pare-feu Azure dans le réseau virtuel qui hébergera le déploiement
- Déploiement d’Azure Bastion dans le réseau virtuel qui hébergera le déploiement

L’exercice se compose des tâches suivantes :

- Tâche 1 : Répondre aux exigences relatives aux processeurs virtuels dans l’abonnement Azure cible
- Tâche 2 : Configurer les attributions de rôles RBAC (Contrôle d’accès en fonction du rôle) Azure pour le compte d’utilisateur Microsoft Entra ID qui sera utilisé pour effectuer le déploiement
- Tâche 3 : Créer un compte de stockage associé au Centre Azure pour les solutions SAP utilisé pour le déploiement
- Tâche 4 : Créer et configurer une identité managée affectée par l’utilisateur à utiliser par le Centre Azure pour les solutions SAP pour l’authentification et l’autorisation de son déploiement automatisé
- Tâche 5 : Créer un groupe de sécurité réseau (NSG) à utiliser dans les sous-réseaux du réseau virtuel qui hébergera le déploiement
- Tâche 6 : Créer des tables de routage à utiliser dans les sous-réseaux du réseau virtuel qui hébergera le déploiement
- Tâche 7 : Créer et configurer le réseau virtuel qui hébergera le déploiement
- Tâche 8 : Déployer le Pare-feu Azure dans le réseau virtuel qui hébergera le déploiement
- Tâche 9 : Déployer Azure Bastion dans le réseau virtuel qui hébergera le déploiement

### Tâche 1 : Répondre aux exigences relatives aux processeurs virtuels dans l’abonnement Azure cible

>**Remarque** : Pour suivre ce labo (comme décrit), vous avez besoin d’un abonnement Microsoft Azure avec les quotas de processeurs virtuels qui prennent en charge le déploiement des machines virtuelles suivantes :

- 2 machines virtuelles Standard_E4ds_v4 (4 processeurs virtuels et 32 Gio de mémoire chacune) ou 2 machines virtuelles Standard_D4ds_v4 (4 processeurs virtuels et 16 Gio de mémoire chacune) pour la couche ASCS
- 2 machines virtuelles Standard_E4ds_v4 (4 processeurs virtuels et 32 Gio de mémoire chacune) ou 2 machines virtuelles Standard_D4ds_v4 (4 processeurs virtuels et 16 Gio de mémoire chacune) pour la couche Application 
- 2 machines virtuelles Standard_M64ms (64 processeurs virtuels et 1750 Gio de mémoire chacune) pour la couche Base de données

>**Remarque** : Afin de réduire les besoins en processeurs virtuels et en mémoire pour les machines virtuelles de base de données, vous pouvez remplacer la référence SKU de la machine virtuelle par Standard_M32ts (32 processeurs virtuels et 192 Gio de mémoire chacune).

1. À partir de l’ordinateur de labo, démarrez un navigateur web et accédez au portail Azure à l’adresse `https://portal.azure.com`.
1. Dans le portail Azure, sélectionnez l’icône **Cloud Shell** et démarrez une session PowerShell dans Cloud Shell. 

    > **Remarque** : Si c’est la première fois que vous lancez Cloud Shell dans l’abonnement Azure que vous utiliserez dans ce labo, vous êtes invité à créer un partage de fichiers Azure pour conserver les fichiers Cloud Shell. Dans ce cas, acceptez les valeurs par défaut afin de créer un compte de stockage dans un groupe de ressources généré automatiquement.

1. Dans le portail Azure, dans le volet **Cloud Shell**, à l’invite PowerShell, exécutez ce qui suit, où `<Azure_region>` désigne la région Azure dans laquelle vous envisagez de déployer les ressources dans ce labo (par ex. : `eastus`) :

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardMSFamily'}
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'cores'}
    ```

    > **Remarque** : Pour identifier les noms des régions Azure, dans **Cloud Shell**, à l’invite Bash, exécutez `(Get-AzLocation).Location`

1. Passez en revue la sortie pour identifier l’utilisation actuelle des processeurs virtuels et la limite des processeurs virtuels. Vérifiez que la différence entre elles est suffisante pour prendre en charge les processeurs virtuels des machines virtuelles Azure que vous allez déployer dans ce labo. Prenez en compte à la fois la quantité totale de processeurs virtuels régionaux et la quantité propre à la famille de machines virtuelles. 
1. Si le nombre de processeurs virtuels n’est pas suffisant, fermez le volet Cloud Shell. Dans le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Quotas**.
1. Dans la page **Quotas**, sélectionnez **Calcul**.
1. Dans la page **Quotas \| calcul**, utilisez le filtre **Région** pour sélectionner la région Azure dans laquelle vous envisagez de déployer les ressources dans ce labo.
1. Dans la colonne **Nom du quota**, recherchez et sélectionnez le nom de la référence SKU de machine virtuelle qui nécessite une augmentation du quota. 
1. Sur la même ligne, vérifiez l’entrée dans la colonne **Ajustable**. L’étape suivante varie selon que la colonne contient l’entrée **Oui** ou **Non**.

   - Si l’entrée est définie sur **Oui**, sélectionnez l’icône **Ajustement de la demande**. Dans **Nouvelle demande de quota**, dans la zone de texte **Nouvelle limite**, entrez la nouvelle limite de quota, puis sélectionnez **Envoyer**.
   - Si l’entrée est définie sur **Non**, sélectionnez l’icône **Demander l’accès ou obtenir des recommandations**. Dans le volet **Recommandations de quota**, sélectionnez l’option **Contacter le service clientèle**, puis **Suivant**. 
1. Sous l’onglet **Description du problème** de la page **Nouvelle demande de support**, spécifiez les paramètres suivants, puis sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |À quoi votre problème est-il lié ?|**Services Azure**|
    |Type de problème|**Limites du service et de l’abonnement (quotas)**|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Type de quota|**Augmentations de la limite d’abonnement de calcul/machine virtuelle (cœurs/processeurs virtuels)**|

1. Sous l’onglet **Détails supplémentaires**, sélectionnez **Entrer les détails**.
1. Sous l’onglet **Détails du quota**, dans la liste déroulante **Modèle de déploiement**, sélectionnez **Resource Manager**. Dans la liste déroulante **Localisations**, sélectionnez la région Azure cible. Dans la liste déroulante **Quotas**, sélectionnez la série de machines virtuelles Azure dont vous devez augmenter les limites de quota. Dans la zone de texte **Nouvelle limite**, entrez la nouvelle limite de quota, puis sélectionnez **Enregistrer et Continuer**.
1. De retour sous l’onglet **Détails supplémentaires**, sous l’onglet **Informations de diagnostic avancées**, sélectionnez **Oui (recommandé)**.
1. Dans la section **Méthode de support**, sélectionnez **E-mail** ou **Téléphone** comme méthode de contact préférée, puis sélectionnez **Suivant**.
1. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.

    > **Remarque** : Attendez que la demande d’augmentation des limites de quota soit terminée avant de passer à la tâche suivante.

### Tâche 2 : Configurer les attributions de rôles RBAC (Contrôle d’accès en fonction du rôle) Azure pour le compte d’utilisateur Microsoft Entra ID qui sera utilisé pour effectuer le déploiement

1. Sur l’ordinateur de labo, démarrez Microsoft Edge et accédez au portail Azure à l’adresse `https://portal.azure.com`.
1. Lorsque vous êtes invité à vous authentifier, connectez-vous à l’aide des informations d’identification Microsoft Entra ID avec le rôle Propriétaire dans l’abonnement Azure que vous utiliserez pour ce labo. 
1. Dans le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Abonnements**.
1. Dans la page **Abonnements**, sélectionnez l’entrée représentant l’abonnement Azure que vous utiliserez pour ce labo. 
1. Dans la page affichant les propriétés de l’abonnement Azure, sélectionnez **Contrôle d’accès (IAM)**.
1. Dans la page **Contrôle d’accès (IAM)**, sélectionnez **+ Ajouter**, puis dans le menu déroulant, sélectionnez **Ajouter une attribution de rôle**.
1. Sous l’onglet **Rôle** de la page **Ajouter une attribution de rôle**, dans la liste des **rôles de fonction de travail**, recherchez et sélectionnez l’entrée **Administrateur Centre Azure pour les solutions SAP**, puis sélectionnez **Suivant**.
1. Sous l’onglet **Membres** de la page **Ajouter une attribution de rôle**, cliquez sur **+ Sélectionner des membres**. 
1. Dans le volet **Sélectionner des membres**, dans la zone de texte **Sélectionner**, entrez le nom du compte d’utilisateur Microsoft Entra ID que vous avez utilisé pour accéder à l’abonnement Azure que vous utilisez pour ce labo. Sélectionnez-le dans la liste des résultats correspondant à votre entrée, puis cliquez sur **Sélectionner**.
1. De retour sous l’onglet **Membres**, sélectionnez **Vérifier + attribuer**.
1. Sous l’onglet **Vérifier + attribuer**, sélectionnez **Vérifier + attribuer**.
1. Répétez les six étapes précédentes pour attribuer le rôle **Opérateur d’identité managée** au compte d’utilisateur que vous utilisez pour ce labo.

### Tâche 3 : Créer un compte de stockage associé au Centre Azure pour les solutions SAP utilisé pour le déploiement

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Comptes de stockage**.
1. Dans la page **Comptes de stockage**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un compte de stockage**, spécifiez les paramètres suivants et sélectionnez **Suivant : Avancé >**.

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un **nouveau** groupe de ressources **ACSS-DEMO**|
    |Nom du compte de stockage|Nom global unique comprenant entre 3 et 24 caractères alphanumériques|
    |Région|**USA Est**|
    |Performances|**Standard**|
    |Redondance|**Stockage géo-redondant (GRS)**|
    |Permettre l’accès en lecture aux données en cas d’indisponibilité régionale|Désactivé|

1. Sous l’onglet **Avancé**, passez en revue les options disponibles, acceptez les valeurs par défaut et sélectionnez **Suivant : Mise en réseau >**.
1. Sous l’onglet **Mise en réseau**, passez en revue les options disponibles, vérifiez que l’option **Activer l’accès public à partir de tous les réseaux** est activée et sélectionnez **Vérifier**.
1. Sous l’onglet **Vérifier**, attendez que le processus de validation se termine et sélectionnez **Créer**.

    >**Remarque** : N’attendez pas la fin de l’approvisionnement du compte Stockage Azure. Au lieu de cela, passez à la tâche suivante. L’approvisionnement peut durer environ 2 minutes.

### Tâche 4 : Créer et configurer une identité managée affectée par l’utilisateur à utiliser par le Centre Azure pour les solutions SAP pour l’authentification et l’autorisation de son déploiement automatisé

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Identités managées**.
1. Dans la page **Identités managés**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer une identité managée affectée par l’utilisateur**, spécifiez les paramètres suivants, puis sélectionnez **Vérifier + Créer** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**ACSS-DEMO**|
    |Région|**USA Est**|
    |Nom|**Contoso-MSI**|

1. Sous l’onglet **Vérifier**, attendez que le processus de validation se termine et sélectionnez **Créer**.

    >**Remarque** : Attendez la fin de l’approvisionnement de l’identité managée affectée par l’utilisateur. Cela devrait prendre juste quelques secondes.

1. Dans le portail Azure, accédez à la page **Identités managées** et sélectionnez l’entrée **Contoso-MSI**.
1. Dans la page **Contoso-MSI**, sélectionnez **Attributions de rôles Azure**.
1. Dans la page **Attributions de rôles Azure**, sélectionnez **+ Ajouter une attribution de rôle (préversion)**.
1. Dans le volet **+ Ajouter une attribution de rôle (préversion)**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

    |Paramètre|Valeur|
    |---|---|
    |Étendue|**Abonnement**|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Rôle|**Rôle de service Centre Azure pour les solutions SAP**|

2. De retour dans la page **Attributions de rôles Azure**, sélectionnez **+ Ajouter une attribution de rôle (préversion)**.
3. Dans le volet **+ Ajouter une attribution de rôle (préversion)**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

    |Paramètre|Valeur|
    |---|---|
    |Étendue|**Stockage**|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Ressource|Nom du compte Stockage Azure que vous avez créé lors de la tâche précédente|
    |Rôle|**Lecteur et accès aux données**|

### Tâche 5 : Créer un groupe de sécurité réseau (NSG) à utiliser dans les sous-réseaux du réseau virtuel qui hébergera le déploiement

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Groupes de sécurité réseau**.
1. Dans la page **Groupe de sécurité réseau**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un groupe de sécurité réseau**, spécifiez les paramètres suivants, puis sélectionnez **Vérifier + créer** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un **nouveau** groupe de ressources **CONTOSO-VNET-RG**|
    |Nom|**ACSS-DEMO-NSG**|
    |Région|**USA Est**|

1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.

    >**Remarque** : Par défaut, les règles intégrées des groupes de sécurité réseau autorisent tout le trafic sortant, tout le trafic au sein du même réseau virtuel ainsi que tout le trafic entre les réseaux virtuels appairés. Cela suffit pour pouvoir suivre le labo. En fonction de vos exigences de sécurité, vous pouvez envisager de bloquer une partie de ce trafic. Dans ce cas, reportez-vous aux conseils dans la documentation Microsoft Learn [Préparer le réseau pour le déploiement d’infrastructure](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network).

### Tâche 6 : Créer des tables de routage à utiliser dans les sous-réseaux du réseau virtuel qui hébergera le déploiement

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Tables de routage**.
1. Dans la page **Tables de routage**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer une table de routage**, spécifiez les paramètres suivants, puis sélectionnez **Vérifier + créer** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**CONTOSO-VNET-RG**|
    |Région|**USA Est**|
    |Nom|**ACSS-ROUTE**|
    |Propager des itinéraires de passerelle|**Aucun**|

1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.

### Tâche 7 : Créer et configurer le réseau virtuel qui hébergera le déploiement

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Réseaux virtuels**. 
1. Dans la page **Réseaux virtuels**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un réseau virtuel**, spécifiez les paramètres suivants et sélectionnez **Suivant** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**CONTOSO-VNET-RG**|
    |Nom du réseau virtuel|**CONTOSO-VNET**|
    |Région|**(États-Unis) USA Est**|

1. Sous l’onglet **Sécurité**, acceptez les paramètres par défaut et sélectionnez **Suivant**.

    >**Remarque** : Vous pourriez approvisionner Azure Bastion et le Pare-feu Azure à ce stade, mais vous les approvisionnerez séparément une fois le réseau virtuel créé.

1. Sous l’onglet **Adresses IP**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

    |Paramètre|Valeur|
    |---|---|
    |Espace d’adressage IP|**10.5.0.0/16 (65 536 adresses)**|

    >**Remarque** : Supprimez toutes les entrées de sous-réseau prédéfinies. Vous ajouterez des sous-réseaux une fois le réseau virtuel créé.

1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.
1. Revenez à la page **Réseaux virtuels** et sélectionnez l’entrée **CONTOSO-VNET**. 
1. Dans la page **CONTOSO-VNET**, dans la barre de menus verticale située à gauche de la page, sélectionnez **Sous-réseaux**.
1. Dans la page **CONTOSO-VNET \| Sous-réseaux**, sélectionnez **+ Sous-réseau**. 
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**app**|
    |Plage d’adresses de sous-réseau|**10.5.0.0/24**|
    |Groupe de sécurité réseau|**ACSS-DEMO-NSG**|
    |Table de routage|**ACSS-ROUTE**|

1. De retour dans la page **CONTOSO-VNET \| Sous-réseaux**, sélectionnez **+ Sous-réseau**. 
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**AzureBastionSubnet**|
    |Plage d’adresses de sous-réseau|**10.5.1.0/26**|

1. De retour dans la page **CONTOSO-VNET \| Sous-réseaux**, sélectionnez **+ Sous-réseau**. 
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**db**|
    |Plage d’adresses de sous-réseau|**10.5.2.0/24**|
    |Groupe de sécurité réseau|**ACSS-DEMO-NSG**|
    |Table de routage|**ACSS-ROUTE**|

1. De retour dans la page **CONTOSO-VNET \| Sous-réseaux**, sélectionnez **+ Sous-réseau**. 
1. Dans le volet **Ajouter un sous-réseau**, spécifiez les paramètres suivants et sélectionnez **Enregistrer** :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**AzureFirewallSubnet**|
    |Plage d’adresses de sous-réseau|**10.5.3.0/24**|

### Tâche 8 : Déployer le Pare-feu Azure dans le réseau virtuel qui hébergera le déploiement

>**Remarque** : Avant de déployer une instance Pare-feu Azure, vous allez d’abord créer une stratégie de pare-feu et une adresse IP publique qui sera utilisée par l’instance.

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Stratégies de pare-feu**.
1. Dans la page **Stratégies de pare-feu**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer une stratégie de pare-feu Azure**, spécifiez les paramètres suivants et sélectionnez **Suivant : Paramètres DNS >**  :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**CONTOSO-VNET-RG**|
    |Nom|**FirewallPolicy_contoso-firewall**|
    |Région|**USA Est**|
    |Niveau de stratégie|**Standard**|
    |Stratégie parente|**Aucun**|

1. Sous l’onglet **Paramètres DNS**, acceptez l’option **Désactivé** par défaut, puis sélectionnez **Suivant : Inspection TLS >**.
1. Sous l’onglet **Inspection TLS**, sélectionnez **Suivant : Règles >**.
1. Sous l’onglet **Règles**, sélectionnez **+ Ajouter une collection de règles**.
1. Dans le volet **Ajouter une collection de règles**, spécifiez les paramètres suivants :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**AllowOutbound**|
    |Type de regroupement de règles|**Réseau**|
    |Priority|**101**|
    |Action de regroupement de règles|**Autoriser**|
    |Groupe de regroupement de règles|**DefaultNetworkRuleCollectionGroup**|

1. Dans le volet **Ajouter une collection de règles**, dans la section **Règles**, ajoutez une règle avec les paramètres suivants :

    |Paramètre|Valeur|
    |---|---|
    |Nom|**RHEL**|
    |Type de source|**Adresse IP**|
    |Source|*|
    |Protocol|**N’importe lequel/laquelle**|
    |Ports de destination|*|
    |Type de destination|**Adresse IP**|
    |Destination|**13.91.47.76,40.85.190.91,52.187.75.218,52.174.163.213,52.237.203.198**|

    >**Remarque** : Pour identifier les adresses IP à utiliser pour RHEL, consultez [Préparer le réseau pour le déploiement d’infrastructure](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

    |Paramètre|Valeur|
    |---|---|
    |Nom|**ServiceTags**|
    |Type de source|**Adresse IP**|
    |Source|*|
    |Protocol|**N’importe lequel/laquelle**|
    |Ports de destination|*|
    |Type de destination|**Étiquette du service**|
    |Destination|**AzureActiveDirectory,AzureKeyVault,Storage**|

    >**Remarque** : Si vous le souhaitez, vous pouvez utiliser des étiquettes de service avec des étendues régionales. 

    |Paramètre|Valeur|
    |---|---|
    |Nom|**SUSE**|
    |Type de source|**Adresse IP**|
    |Source|*|
    |Protocol|**N’importe lequel/laquelle**|
    |Ports de destination|*|
    |Type de destination|**Adresse IP**|
    |Destination|**52.188.224.179,52.186.168.210,52.188.81.163,40.121.202.140**|

    >**Remarque** : Pour identifier les adresses IP à utiliser pour SUSE, consultez [Préparer le réseau pour le déploiement d’infrastructure](https://learn.microsoft.com/en-us/azure/sap/center-sap-solutions/prepare-network)

    |Paramètre|Valeur|
    |---|---|
    |Nom|**AllowOutbound**|
    |Type de source|**Adresse IP**|
    |Source|*|
    |Protocol|**TCP,UDP,ICMP,Any**|
    |Ports de destination|*|
    |Type de destination|**Adresse IP**|
    |Destination|*|

1. Sélectionnez le bouton **Ajouter** pour enregistrer toutes les règles.
1. De retour sous l’onglet **Règles**, sélectionnez **Suivant : IDPS >**.
1. Sous l’onglet **IDPS**, sélectionnez **Suivant : Renseignement sur les menaces >**.

    >**Remarque** : La fonctionnalité IDPS nécessite la référence SKU Premium.

1. Sous l’onglet **Renseignement sur les menaces**, passez en revue les paramètres disponibles sans apporter de modifications, puis sélectionnez **Vérifier + créer**.
1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.

    >**Remarque** : Attendez la fin de l’approvisionnement de la stratégie de pare-feu. L’approvisionnement devrait prendre environ 1 minute.

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Adresses IP publiques**.
1. Dans la page **Adresses IP publiques**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer une adresse IP publique**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**CONTOSO-VNET-RG**|
    |Région|**(États-Unis) USA Est**|
    |Nom|**contoso-firewal-pip**|
    |Version de l’adresse IP|**IPv4**|
    |Référence (SKU)|**Standard**|
    |Zone de disponibilité|**Aucune zone**|
    |Niveau|**Regional**|
    |Préférence de routage|**Réseau Microsoft**|
    |Délai d’inactivité (minutes).|**4**|
    |Étiquette du nom DNS|non défini|

1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.

    >**Remarque** : Attendez la fin de l’approvisionnement de l’adresse IP publique. L’approvisionnement devrait prendre quelques secondes.

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Pare-feu**.
1. Dans la page **Pare-feu**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Créer un pare-feu**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**CONTOSO-VNET-RG**|
    |Nom|**contoso-firewall**|
    |Région|**USA Est**|
    |Zone de disponibilité|**Aucun**|
    |Référence SKU de pare-feu|**Standard**|
    |Gestion de pare-feu|**Utiliser une stratégie de pare-feu pour gérer ce pare-feu**|
    |Stratégie de pare-feu|**FirewallPolicy_contoso-firewall**|
    |Choisir un réseau virtuel|**Utiliser existant**|
    |Réseau virtuel|**CONTOSO-VNET**|
    |Adresse IP publique|**contoso-firewall-pip**|
    |Tunneling forcé|**Disabled**|

    >**Remarque** : Attendez la fin de l’approvisionnement du Pare-feu Azure. L’approvisionnement peut prendre environ 3 minutes.

1. Dans le portail Azure, revenez à la page **Pare-feu**.
1. Dans la page **Pare-feu**, sélectionnez l’entrée **contoso-firewall**.
1. Dans la page **contoso-firewall**, notez l’entrée **IP privée** définie sur **10.5.3.4** représentant l’adresse IP privée de l’instance Pare-feu Azure.

    >**Remarque** : Pour que le trafic réseau soit routé via le Pare-feu Azure, vous devez ajouter des routes définies par l’utilisateur aux tables de routage associées à l’application et aux sous-réseaux de base de données du réseau virtuel qui hébergera le déploiement SAP.

1. Dans le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Tables de routage**.
1. Dans la page **Tables de routage**, sélectionnez l’entrée **ACSS-ROUTE**.
1. Dans la page **ACSS-ROUTE**, sélectionnez **Routes**.
1. Dans la page **ACSS-ROUTE \| Routes**, sélectionnez **+ Ajouter**.
1. Dans le volet **Ajouter une route**, spécifiez les paramètres suivants et sélectionnez **Ajouter** :

    |Paramètre|Valeur|
    |---|---|
    |Nom de l’itinéraire|**Pare-feu**|
    |Type de destination|**Adresses IP**|
    |Plages d’adresses IP/CIDR de destination|**0.0.0.0/0**|
    |Type de tronçon suivant|**Appliance virtuelle**|
    |adresse de tronçon suivant|**10.5.3.4**|

### Tâche 9 : Déployer Azure Bastion dans le réseau virtuel qui hébergera le déploiement

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Bastions**. 
1. Dans la page **Bastions**, sélectionnez **+ Créer**.
1. Sous l’onglet **Informations de base** de la page **Bastions**, spécifiez les paramètres suivants et sélectionnez **Suivant : Étiquettes >**  :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|**CONTOSO-VNET-RG**|
    |Nom|**ACSS-BASTION**|
    |Région|**USA Est**|
    |Niveau|**De base**|
    |Nombre d’instances|**2**|
    |Réseau virtuel|**CONTOSO-VNET**|
    |Sous-réseau|**AzureBastionSubnet**|
    |Adresse IP publique|**Création**|
    |Nom de l’adresse IP publique|**ACSS-BASTION-PIP**|

1. Sous l’onglet **Étiquettes**, sélectionnez **Suivant : Avancé >**
1. Sous l’onglet **Avancé**, passez en revue les paramètres disponibles sans apporter de modifications, puis sélectionnez **Suivant : Vérifier + créer >**
1. Sous l’onglet **Vérifier + créer**, attendez que le processus de validation se termine et sélectionnez **Créer**.

    >**Remarque** : N’attendez pas la fin de l’approvisionnement de l’hôte Bastion. Au lieu de cela, passez à la tâche suivante. L’approvisionnement peut prendre environ 15 minutes.

## Exercice 2 : Déployer l’infrastructure qui hébergera les charges de travail SAP dans Azure à l’aide du Centre Azure pour les solutions SAP

Durée : 40 minutes

Dans cet exercice, vous allez utiliser le Centre Azure pour les solutions SAP afin de déployer l’infrastructure qui hébergera les charges de travail SAP dans l’abonnement Azure que vous avez utilisé dans l’exercice précédent.
L’exercice se compose des tâches suivantes :

- Tâche 1 : Créer une instance virtuelle pour les solutions SAP

### Tâche 1 : Créer une instance virtuelle pour les solutions SAP

1. Sur l’ordinateur de labo, dans la fenêtre Microsoft Edge affichant le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Centre Azure pour les solutions SAP**. 
1. Dans la page **Centre Azure pour les solutions SAP\| Vue d’ensemble**, sélectionnez **Créer un système SAP**.
1. Sous l’onglet **Informations de base** de la page **Créer une instance virtuelle pour les solutions SAP**, spécifiez les paramètres suivants et sélectionnez **Suivant : Machines virtuelles**

    |Paramètre|Valeur|
    |---|---|
    |Abonnement|Nom de l’abonnement Azure que vous utilisez dans ce labo|
    |Resource group|Nom d’un **nouveau** groupe de ressources **Contoso-SAP-C1S**|
    |Nom (SID)|**C1S**|
    |Région|**(États-Unis) USA Est**|
    |Type d’environnement|**Production**|
    |Produit SAP|**S/4HANA**|
    |Base de données|**HANA**|
    |Méthode de mise à l’échelle HANA|**Effectuer un scale-up (recommandé)**|
    |Type de déploiement|**Distribué avec haute disponibilité (HA)**|
    |Disponibilité du calcul|**99,95 % (groupe à haute disponibilité)**|
    |Réseau virtuel|**CONTOSO-VNET**|
    |Sous-réseau d’application|**app (10.5.0.0/24)**|
    |Sous-réseau de base de données|**db (10.5.2.0/24)**|
    |Image du système d’exploitation d’application|**Red Hat Enterprise Linux 8.2 pour applications SAP – x64 Gen2, version la plus récente**|
    |Image du système d’exploitation de base de données|**Red Hat Enterprise Linux 8.2 pour applications SAP – x64 Gen2, version la plus récente**|
    |Options de transport SAP|**Créer un répertoire de transport SAP**|
    |Groupe de ressources de transport|**ACSS-DEMO**|
    |Nom du compte de stockage|Aucune entrée|
    |Type d'authentification|**SSH public**|
    |Nom d’utilisateur|**contososapadmin**|
    |Source de la clé publique SSH|**Générer une nouvelle paire de clés**|
    |Nom de la paire de clés|**contosoc1skey**|
    |Nom de domaine complet SQP|**sap.contoso.com**|
    |Source d’identité managée|**Utiliser une identité managée affectée par l’utilisateur existante**|
    |Nom de l’identité managée|**Contoso-MSI**|

1. Sous l’onglet **Machines Virtuelles**, spécifiez les paramètres suivants :

    |Paramètre|Valeur|
    |---|---|
    |Générer une recommandation basée sur|**SAP Application Performance Standard (SAPS) : sélectionnez cette option pour fournir une valeur SAPS pour la couche Application et la taille de la mémoire de base de données, puis cliquez sur Générer des recommandations**|
    |SAPS pour la couche Application|**10000**|
    |Taille de la mémoire pour la base de données (Gio)|**1024**|

1. Sélectionnez **Générer une recommandation**.
1. Passez en revue la taille et le nombre de machines virtuelles pour les machines virtuelles ASCS, d’application et de base de données. 

    >**Remarque** : Si nécessaire, adaptez les tailles recommandées en sélectionnant le lien **Voir toutes les tailles** pour chaque ensemble de machines virtuelles et en choisissant une autre taille. Par défaut, le type de déploiement distribué avec haute disponibilité ainsi que la valeur SAPS de couche Application et la taille de mémoire de base de données spécifiées ci-dessus aboutissent aux recommandations de référence SKU de machine virtuelle suivantes :
    - 2 machines virtuelles Standard_E4ds_v4 pour la couche ASCS (4 processeurs virtuels et 32 Gio de mémoire chacune)
    - 2 machines virtuelles Standard_E4ds_v4 pour la couche Application (4 processeurs virtuels et 32 Gio de mémoire chacune)
    - 2 machines virtuelles Standard_M64ms pour la couche Base de données (64 processeurs virtuels et 1750 Gio de mémoire chacune)

    >**Remarque** : Afin de réduire les besoins en processeurs virtuels et en mémoire pour les machines virtuelles de base de données, vous pouvez remplacer la référence SKU de machine virtuelle par Standard_M32ts (32 processeurs virtuels et 192 Gio de mémoire chacune).

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

>**Remarque** : Après le déploiement, passez à l’étape suivante qui consiste à installer le logiciel SAP à l’aide du Centre Azure pour les solutions SAP.

>**Important** : Le coût des ressources que vous avez déployées est important, donc veillez à désapprovisionner le labo si vous n’avez plus l’intention de l’utiliser. Notez que la suppression de l’instance virtuelle pour les solutions SAP ne supprime pas les ressources d’infrastructure sous-jacentes. Pour supprimer les ressources, supprimez les groupes de ressources suivants (dans cet ordre) qui ont été créés au cours de ce labo :

- **Contoso-SAP-C1S**
- **CONTOSO-VNET-RG**
- **ACSS-DEMO**