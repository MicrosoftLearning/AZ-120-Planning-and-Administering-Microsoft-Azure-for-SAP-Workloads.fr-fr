---
lab:
  title: 04b – Implémenter une architecture SAP sur des machines virtuelles Azure exécutant Windows
  module: Module 04 - Deploy SAP on Azure
---

# AZ 120 Module 4 : Déployer SAP sur Azure
# Labo 4b : Implémenter une architecture SAP sur des machines virtuelles Azure exécutant Windows

Temps estimé : 150 minutes

Toutes les tâches de ce labo sont effectuées à partir du portail Azure (y compris une session Cloud Shell PowerShell).  

   > **Remarque** : Si vous n’utilisez pas Cloud Shell, le module Az PowerShell doit être installé [**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi).

Fichiers de lab : aucun

## Scénario
  
En préparation au déploiement de SAP NetWeaver sur Azure, Adatum Corporation souhaite mettre en œuvre une démo qui illustrera l’implémentation à haute disponibilité de SAP NetWeaver sur des machines virtuelles Azure exécutant Windows Server 2016.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

-   Provisionner les ressources Azure nécessaires à la prise en charge d’un déploiement SAP NetWeaver à haute disponibilité

-   Configurer le système d’exploitation des machines virtuelles Azure exécutant Windows pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité

-   Configurer le clustering sur des machines virtuelles Azure exécutant Windows pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité

## Spécifications

-   Un abonnement Microsoft Azure avec le nombre suffisant de processeurs virtuels Dsv3 disponibles (quatre machines virtuelles Standard_D2s_v3 avec 2 processeur virtuel et six machines virtuelles Standard_D4s_v3 avec 4 processeurs virtuels chacune) dans une région Azure prenant en charge les zones de disponibilité

-   Un ordinateur de labo avec un navigateur web compatible Azure Cloud Shell et un accès à Azure


## Exercice 1 : Approvisionner les ressources Azure nécessaires à la prise en charge des déploiements SAP NetWeaver à haut niveau de disponibilité

Durée : 60 minutes

Dans cet exercice, vous allez déployer des composants de calcul d’infrastructure Azure nécessaires pour configurer le clustering Windows. Cela implique la création d’une paire de machines virtuelles Azure exécutant Windows Server 2016 dans le même groupe à haute disponibilité.

### Tâche 1 : Déployer une paire de machines virtuelles Azure exécutant des contrôleurs de domaine Active Directory à haute disponibilité en utilisant un modèle Azure Resource Manager

1. Depuis l’ordinateur de labo, démarrez un navigateur web et accédez au portail Azure à l’adresse **https://portal.azure.com**.

1. Si vous y êtes invité, connectez-vous avec le compte Microsoft professionnel, scolaire ou personnel, avec le rôle de propriétaire ou de contributeur à l’abonnement Azure que vous allez utiliser pour ce labo.

1. Dans le portail Azure, démarrez une session PowerShell dans Cloud Shell. 

    > **Remarque** : Si c’est la première fois que vous lancez Cloud Shell dans l’abonnement Azure actuel, vous êtes invité à créer un partage de fichiers Azure pour conserver les fichiers Cloud Shell. Dans ce cas, acceptez les valeurs par défaut, ce qui entraîne la création d’un compte de stockage dans un groupe de ressources généré automatiquement.

1. Dans le volet Cloud Shell, exécutez les commandes suivantes pour créer un clone superficiel du référentiel hébergeant le modèle Bicep que vous allez utiliser pour le déploiement d’une paire de machines virtuelles Azure exécutant des contrôleurs de domaine Active Directory à haute disponibilité, et pour définir le répertoire actif sur l’emplacement de ce modèle et de son fichier de paramètres :

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/active-directory/active-directory-new-domain-ha-2-dc-zones/
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$rgName` sur `az12001b-ad-RG` :

    ```
    $rgName = 'az12003b-ad-RG'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$location` sur le nom de la région Azure qui prend en charge les zones de disponibilité et où vous envisagez de déployer les machines virtuelles de labo (remplacez l’espace réservé `<Azure_region>` par le nom de cette région) :

    ```
    $location = '<Azure_region>'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer un groupe de ressources nommé **az12001b-ad-RG** dans la région Azure que vous avez choisie :

    ```
    New-AzResourceGroup -Name $rgName -Location $location
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$deploymentName` :

    ```
    $deploymentName = 'az1203b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
    ```

1. Dans le volet Cloud Shell, exécutez les commandes suivantes pour définir le nom du compte d’utilisateur d’administration et son mot de passe (remplacez les espaces réservés `<username>` et `<password>` par le nom du compte d’utilisateur d’administration et par la valeur de son mot de passe, respectivement) :

    ```
    $adminUsername = '<username>'
    $adminPassword = ConvertTo-SecureString '<password>' -AsPlainText -Force
    ```

    > **Remarque** : Veillez à ce que le mot de passe réponde aux exigences de complexité applicables au déploiement de machines virtuelles Azure exécutant Windows (une longueur d’au moins 12 caractères contenant des lettres minuscules et majuscules, des chiffres et des caractères spéciaux).

1. Dans le volet Cloud Shell, exécutez la commande suivante pour effectuer le déploiement :

    ```
    New-AzResourceGroupDeployment -Name $deploymentName -ResourceGroupName $rgName -TemplateFile .\main.bicep -TemplateParameterFile .\azuredeploy.parameters.json -adminUsername $adminUsername -adminPassword $adminPassword -c
    ```

1. Passez en revue la sortie de la commande et vérifiez qu’elle n’inclut pas d’erreurs et d’avertissements. Quand vous y êtes invité, appuyez sur la touche **Entrée** pour procéder au déploiement.

    > **Remarque** : Le déploiement doit prendre environ 30 minutes. Attendez la fin du déploiement avant de passer à la tâche suivante.

    > **Remarque** : Si le déploiement échoue avec une erreur incluant l’indication `PowerShell DSC resource MSFT_xADDomainController failed to execute Set-TargetResource functionality with error message: Domain 'adatum.com' could not be found`, procédez comme suit pour corriger ce problème :

    - Dans le portail Azure, accédez au panneau de la machine virtuelle **adBDC** ; dans le menu de navigation vertical situé à gauche, dans la section **Paramètres**, sélectionnez **Extensions + applications** ; dans le volet **Extensions + applications**, sélectionnez **PrepareBDC** ; dans le volet **Préparer BDC**, sélectionnez **Désinstaller**. 

    - Revenez au panneau de la machine virtuelle **adBDC**, puis redémarrez la machine virtuelle Azure.

    - Accédez au panneau **az1203b-ad-RG** puis, dans le menu de navigation vertical situé à gauche, dans la **section Paramètres**, sélectionnez **Déploiements**.

    - Dans le panneau **az1203b-ad-RG \| Déploiements**, sélectionnez le déploiement dont le nom commence par le préfixe **az1203b** puis, dans le panneau de déploiement, sélectionnez **Redéployer**.

    - Dans le panneau **Déploiement personnalisé**, dans la zone de texte **Mot de passe de l’administrateur**, entrez le même mot de passe que celui que vous avez utilisé lors du déploiement d’origine, sélectionnez **Vérifier + créer**, puis sélectionnez **Créer**.

    - N’attendez pas que le déploiement se termine et passez à la tâche suivante. Le déploiement doit prendre environ 3 minutes.

### Tâche 2 : Provisionner des sous-réseaux qui vont héberger des machines virtuelles Azure exécutant un déploiement SAP NetWeaver à haute disponibilité et le cluster S2D.

1. Dans le portail Azure, accédez au panneau du groupe de ressources **az12003b-ad-RG**.

1. Dans le panneau du groupe de ressources **az12003b-ad-RG**, dans la liste des ressources, recherchez le réseau virtuel **adVNET**, puis cliquez sur son entrée pour afficher le panneau **adVNET**.

1. Dans le panneau **adVNET**, accédez à son panneau **adVNET – Sous-réseaux**. 

1. Dans le panneau **adVNET – Sous-réseaux**, créez un sous-réseau avec les paramètres suivants :

    -   Nom : **sapSubnet**

    -   Plage d’adresses (bloc CIDR) : **10.0.1.0/24**

1. Dans le panneau **adVNET – Sous-réseaux**, créez un sous-réseau avec les paramètres suivants :

    -   Nom : **s2dSubnet**

    -   Plage d’adresses (bloc CIDR) : **10.0.2.0/24**

1. Dans le portail Azure, démarrez une session PowerShell dans Cloud Shell. 

    > **Remarque** : Si c’est la première fois que vous lancez Cloud Shell dans l’abonnement Azure actuel, vous êtes invité à créer un partage de fichiers Azure pour conserver les fichiers Cloud Shell. Dans ce cas, acceptez les valeurs par défaut afin de créer un compte de stockage dans un groupe de ressources généré automatiquement.

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$resourceGroupName` sur le nom du groupe de ressources contenant les ressources que vous avez provisionnées dans la tâche précédente :

    ```
    $resourceGroupName = 'az12003b-ad-RG'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour identifier le réseau virtuel créé dans la tâche précédente :

    ```
    $vNetName = 'adVNet'

    $subnetName = 'sapSubnet'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour identifier l’ID de ressource du sous-réseau nouvellement créé :

    ```
    $vNet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name $vNetName
    
    (Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vNet).Id
    ```

1. Copiez la valeur résultante dans le Presse-papiers. Vous en aurez besoin dans la prochaine tâche.

### Tâche 3 : Déployer un modèle Azure Resource Manager provisionnant des machines virtuelles Azure exécutant Windows Server 2016 qui vont héberger un déploiement SAP NetWeaver à haute disponibilité

1. Sur l’ordinateur de labo, dans le portail Azure, recherchez et sélectionnez **Déploiement de modèle (déployer en utilisant un modèle personnalisé)**.

1. Dans le panneau **Déploiement personnalisé**, dans la liste déroulante **Modèle de démarrage rapide (exclusion),**, tapez **application-workloads/sap/sap-3-tier-marketplace-image-md**, puis cliquez sur **Sélectionner un modèle**.

    > **Remarque** : Veillez à utiliser Microsoft Edge ou un navigateur de tiers. N’utilisez pas Internet Explorer.

1. Dans le panneau **SAP NetWeaver à 3 niveaux (disque managé)**, sélectionnez **Modifier le modèle**.

1. Dans le panneau **Modifier le modèle**, appliquez la modification suivante, puis sélectionnez **Enregistrer** :

    -   Dans la ligne **197**, remplacez `"dbVMSize": "Standard_E8s_v3",` par `"dbVMSize": "Standard_D4s_v3",`

1. De retour dans le panneau **SAP NetWeaver à 3 niveaux (disque managé),**, spécifiez les paramètres suivants, cliquez sur **Vérifier + créer**, puis cliquez sur **Créer** pour lancer le déploiement :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | *nom de votre abonnement Azure*  |
    | **Groupe de ressources** | *le nom d’un nouveau groupe de ressources* **az12003b-23-RG** |
    | **Lieu** | *la même région Azure que celle que vous avez spécifiée dans la première tâche de cet exercice* |
    | **ID du système SAP** | **I20** |
    | **Type de pile** | **ABAP** |
    | **Type de système d’exploitation** | **Windows Server 2016 Datacenter** |
    | **Dbtype** | **SQL** |
    | **Taille du système SAP** | **Démonstration** |
    | **Disponibilité du système** | **HA** |
    | **Nom de l’utilisateur administrateur** | **Étudiant** |
    | **Type d’authentification** | **mot de passe** |
    | **Mot de passe de l’administrateur ou clé** | *le même mot de passe que celui que vous avez spécifié précédemment dans ce labo* |
    | **ID du sous-réseau** | *valeur que vous avez copiée dans le Presse-papiers dans la tâche précédente* |
    | **Zones de disponibilité** | **1,2** |
    | **Lieu** | **[resourceGroup().location]** |
    | **_Emplacement des artefacts** | **https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/sap/sap-3-tier-marketplace-image-md/** |
    | **Jeton de signature d’accès partagé de l’emplacement des artefacts** | *laisser vide* |

1. N’attendez pas que le déploiement se termine, mais passez à la tâche suivante. 

### Tâche 4 : Déployer le cluster du serveur de fichiers avec montée en puissance parallèle (SOFS)

Dans cette tâche, vous allez déployer le cluster SOFS (Scale-out File Server) qui va héberger un partage de fichiers pour les serveurs SAP ASCS en utilisant un modèle de démarrage rapide Azure Resource Manager disponible sur GitHub : [**https://github.com/polichtm/301-storage-spaces-direct-md**](https://github.com/polichtm/301-storage-spaces-direct-md). 

1. Sur l’ordinateur de labo, démarrez un navigateur et accédez à [**https://github.com/polichtm/301-storage-spaces-direct-md**](https://github.com/polichtm/301-storage-spaces-direct-md). 

    > **Remarque** : Veillez à utiliser Microsoft Edge ou un navigateur de tiers. N’utilisez pas Internet Explorer.

1. Dans la page intitulée **Utiliser des disques managés pour créer un cluster SOFS d’espaces de stockage direct (S2D) avec Windows Server 2016**, cliquez sur **Déployer sur Azure**. Ceci redirige automatiquement votre navigateur vers le portail Azure et affiche le panneau **Déploiement personnalisé**.

1. Dans le panneau **Déploiement personnalisé**, spécifiez les paramètres suivants, cliquez sur **Vérifier + créer**, puis sur **Créer** pour lancer le déploiement :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | *nom de votre abonnement Azure*  |
    | **Groupe de ressources** | *le nom d’un nouveau groupe de ressources* **az12003b-s2d-RG** |
    | **Région** | *la même région Azure que celle où vous avez déployé les machines virtuelles Azure dans les tâches précédentes de cet exercice* |
    | **Préfixe de nom** | **i20** |
    | **Taille de la machine virtuelle** | **D4s\_v3 Standard** |
    | **Activer la mise en réseau accélérée** | **true** |
    | **Référence SKU de l’image** | **2016-Datacenter-Server-Core** |
    | **Nombre de machines virtuelles** | **2** |
    | **Taille du disque de la machine virtuelle** | **128** |
    | **Nombre de disques de la machine virtuelle** | **3** |
    | **Compte de domaine existant** | **adatum.com** |
    | **Nom de l’utilisateur administrateur** | **Étudiant** |
    | **Mot de passe administrateur** | *le même mot de passe que celui que vous avez spécifié précédemment dans ce labo* |
    | **Nom du groupe de ressources du réseau virtuel existant** | **az12003b-ad-RG** |
    | **Nom du réseau virtuel existant** | **adVNet** |
    | **Nom du sous-réseau existant** | **s2dSubnet** |
    | **Nom du SOFS** | **sapglobalhost** |
    | **Nom du partage** | **sapmnt** |
    | **Jour de la mise à jour planifiée** | **Dimanche** |
    | **Heure de la mise à jour planifiée** | **3:00** |
    | **Logiciel anti-programme malveillant en temps réel activé** | **false** |
    | **Logiciel anti-programme malveillant planifié activé** | **false** |
    | **Heure du logiciel anti-programme malveillant planifiée** | **120** |
    | **_Emplacement des artefacts** | **https://raw.githubusercontent.com/polichtm/301-storage-spaces-direct-md/master** |
    | **Jeton de signature d’accès partagé de l’emplacement des artefacts** | **Conservez la valeur par défaut** |

1. Le déploiement peut prendre environ 20 minutes. N’attendez pas que le déploiement se termine, mais passez à la tâche suivante.

    > **Remarque** : Si le déploiement échoue avec le message d’erreur **Conflit** lors du déploiement du composant i20-s2d-1/s2dPrep ou i20-s2d-0/s2dPrep, procédez comme suit pour corriger ce problème :

       - Dans le portail Azure, accédez à la machine virtuelle **i20-s2d-0** puis, dans le menu de navigation vertical, dans la section **Opérations**, sélectionnez **Exécuter la commande** ; dans le volet **Exécuter le script de commande**, dans la zone de texte **Script PowerShell**, entrez le script suivant et sélectionnez le bouton **Exécuter** (veillez à remplacer l’espace réservé `<password>` par le mot de passe que vous avez spécifié précédemment dans ce labo) :

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```

       - Accédez au volet de la machine virtuelle **i20-s2d-1** puis, dans le menu de navigation vertical, dans la section **Opérations**, sélectionnez **Exécuter la commande** ; dans le volet **Exécuter le script de commande**, dans la zone de texte **Script PowerShell**, entrez le script suivant et sélectionnez le bouton **Exécuter** (veillez à remplacer l’espace réservé `<password>` par le mot de passe que vous avez spécifié précédemment dans ce labo) :

       ```
       $domain = 'adatum.com'
       $password = '<password>' | ConvertTo-SecureString -asPlainText -Force
       $username = "Student@$domain" 
       $credential = New-Object System.Management.Automation.PSCredential($username,$password)
       Add-Computer -DomainName $domain -Credential $credential -Restart -Force
       ```
       
       - Réexécuter les étapes de la tâche actuelle à partir du début

### Tâche 5 : Déployer un hôte de saut

   > **Remarque** : Comme les machines virtuelles Azure que vous avez déployées dans la tâche précédente ne sont pas accessibles depuis Internet, vous allez déployer une machine virtuelle Azure exécutant Windows Server 2016 Datacenter qui servira d’hôte de saut. 

1. Depuis l’ordinateur de labo, dans l’interface du portail Azure, cliquez sur **+ Créer une ressource**.

1. Dans le panneau **Nouveau**, lancez la création d’une machine virtuelle Azure basée sur l’image **Windows Server 2019 Datacenter - Gen1**.

1. Provisionnez une machine virtuelle Azure avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | *nom de votre abonnement Azure*  |
    | **Groupe de ressources** | *le nom d’un nouveau groupe de ressources* **az12003b-dmz-RG** |
    | **Nom de la machine virtuelle** | **az12003b-vm0** |
    | **Région** | *même région Azure que celle dans laquelle vous avez déployé les machines virtuelles Azure dans les tâches précédentes de cet exercice* |
    | **Options de disponibilité** | **Aucune redondance de l’infrastructure requise** |
    | **Image** | *sélectionnez* **Windows Server 2019 Datacenter - Gen2** |
    | **Taille** | **D2s_v3 standard** |
    | **Nom d’utilisateur** | **Étudiant** |
    | **Mot de passe** | *le même mot de passe que celui que vous avez spécifié précédemment dans ce labo* |
    | **Ports d’entrée publics** | **Autoriser les ports sélectionnés** |
    | **Ports d’entrée sélectionnés** | **RDP (3389)** |
    | **Souhaitez-vous utiliser une licence Windows Server existante ?** | **Aucun** |
    | **Type de disque du système d’exploitation** | **HDD Standard** |
    | **Réseau virtuel** | **adVNET** |
    | **Nom du sous-réseau** | *un nouveau sous-réseau nommé* **dmzSubnet** |
    | **Plage d’adresses de sous-réseau** | **10.0.255.0/24** |
    | **Adresse IP publique** | *une nouvelle adresse IP nommée* **az12003b-vm0-ip** |
    | **Groupe de sécurité réseau de la carte réseau** | **De base**  |
    | **Ports d’entrée publics** | **Autoriser les ports sélectionnés** |
    | **Ports d’entrée sélectionnés** | **RDP (3389)** |
    | **Activer la mise en réseau accélérée** | **Désactivé** |
    | **Options d’équilibrage de charge** | **Aucun** |
    | **Activer l’identité managée affectée par le système** | **Désactivé** |
    | **Se connecter avec Azure AD** | **Désactivé** |
    | **Activer l’arrêt automatique** | **Désactivé** |
    | **Options d’orchestration des patchs** | **Mises à jour manuelles** |
    | **Diagnostics de démarrage** | **Désactiver** |
    | **Activer les diagnostics du système d’exploitation invité** | **Désactivé** |
    | **Extensions** | *Aucun* |
    | **Balises** | *Aucun* |
   
1. Attendez la fin du déploiement. Ce processus devrait prendre quelques minutes.

> **Result** : Une fois cet exercice terminé, vous avez provisionné des ressources Azure nécessaires pour prendre en charge les déploiements SAP NetWeaver à haute disponibilité.


## Exercice 2 : Configurer le système d’exploitation des machines virtuelles Azure exécutant Windows pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité

Durée : 60 minutes

Dans cet exercice, vous allez configurer le système d’exploitation de machines virtuelles Azure exécutant Windows pour l’adapter à un déploiement SAP NetWeaver à haute disponibilité.

### Tâche 1 : Joindre les machines virtuelles Azure Windows Server 2016 au domaine Active Directory

   > **Remarque** : Avant de commencer cette tâche, vérifiez que les déploiements de modèle que vous avez lancés dans l’exercice précédent ont réussi. 

1. Dans le portail Azure, accédez au panneau du réseau virtuel nommé **adVNET**, qui a été provisionné automatiquement dans le premier exercice de ce labo.

1. Affichez le panneau **adVNET – Serveurs DNS** et notez que le réseau virtuel est configuré avec les adresses IP privées affectées aux contrôleurs de domaine déployés dans le premier exercice de ce labo en tant que serveurs DNS.

1. Dans le portail Azure, démarrez une session PowerShell dans Cloud Shell. 

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$resourceGroupName` sur le nom du groupe de ressources contenant les ressources que vous avez provisionnées dans la tâche précédente :

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour joindre les machines virtuelles Azure Windows Server que vous avez déployées dans la troisième tâche de l’exercice précédent au domaine Active Directory **adatum.com** (veillez à remplacer l’espace réservé `<password>` par le mot de passe que vous avez spécifié précédemment dans ce labo) :

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1','i20-di-0','i20-di-1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

### Tâche 2 : Examiner la configuration du stockage des machines virtuelles Azure du niveau de base de données

1. Sur l’ordinateur de labo, dans le portail Azure, accédez au panneau **az12003b-vm0**.

1. Dans le panneau **az12003b-vm0**, connectez-vous à la machine virtuelle Azure az12003b-vm0 via Bureau à distance. Quand vous y êtes invité, fournissez les informations d’identification suivantes :

    -   Se connecter en tant que : **student**

    -   Mot de passe : *le même mot de passe que celui que vous avez spécifié précédemment dans ce labo*

1. Dans la session RDP avec az12003b-vm0, utilisez Bureau à distance pour vous connecter à la machine virtuelle Azure **i20-db-0.adatum.com**. Quand vous y êtes invité, fournissez les informations d’identification suivantes :

    -   Se connecter en tant que : **ADATUM\\Student**

    -   Mot de passe : *le même mot de passe que celui que vous avez spécifié précédemment dans ce labo*

1. Utilisez Bureau à distance pour vous connecter à la machine virtuelle Azure **i20-db-1.adatum.com** avec les mêmes informations d’identification.

1. Dans la session RDP avec i20-db-0.adatum.com, utilisez Services de fichiers et de stockage dans le Gestionnaire de serveur pour examiner la configuration du disque. Notez qu’un seul disque de données a été configuré via des montages de volumes pour fournir un stockage pour les fichiers journaux et de base de données. 

1. Dans la session RDP avec i20-db-1.adatum.com, utilisez Services de fichiers et de stockage dans le Gestionnaire de serveur pour examiner la configuration du disque. Notez qu’un seul disque de données a été configuré via des montages de volumes pour fournir un stockage pour les fichiers journaux et de base de données. 


### Tâche 3 : Préparer la configuration du clustering de basculement sur des machines virtuelles Azure exécutant Windows Server 2016 pour prendre en charge une installation SAP NetWeaver à haute disponibilité

1. Dans la session RDP avec i20-db-0.adatum.com, démarrez une session Windows PowerShell ISE et installez les fonctionnalités des outils de clustering de basculement et d’administration à distance en exécutant les commandes suivantes sur la paire de serveurs ASCS et DB, qui deviendront respectivement des nœuds des clusters ASCS et SQL Server :

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1','i20-db-0','i20-db-1')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **Remarque** : Ceci peut entraîner le redémarrage du système d’exploitation invité des quatre machines virtuelles Azure.

1. Sur l’ordinateur de labo, dans le portail Azure, cliquez sur **+ Créer une ressource**.

1. Dans le panneau **Nouveau**, lancez la création d’un nouveau **Compte de stockage** avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | *nom de votre abonnement Azure* |
    | **Groupe de ressources** | *le nom du groupe de ressources dans lequel vous avez déployé les machines virtuelles Azure qui vont héberger un déploiement SAP NetWeaver à haute disponibilité* |
    | **Nom du compte de stockage** | *chaque nom unique composé de 3 à 24 lettres et chiffres* |
    | **Lieu** | *la même région Azure que celle où vous avez déployé les machines virtuelles Azure dans l’exercice précédent* |
    | **Niveau de performance** | **Standard** |
    | **Redondance** | **Stockage localement redondant (LRS)** |
    | **Méthode de connectivité** | **Point de terminaison public (tous les réseaux)** |
    | **Exiger un transfert sécurisé pour les opérations d’API REST** | **Activé** |
    | **Partages de fichiers volumineux** | **Disabled** |
    | **Suppression réversible pour les objets blob, les conteneurs et les fichiers** | **Disabled** |
    | **Espace de noms hiérarchique** | **Disabled** |


### Tâche 4 : Configurer le clustering de basculement sur des machines virtuelles Azure exécutant Windows Server 2016 pour prendre en charge un niveau de base de données à haute disponibilité de l’installation SAP NetWeaver

1. Si nécessaire, dans la session RDP avec az12003b-vm0, utilisez Bureau à distance pour vous connecter à la machine virtuelle Azure **i20-db-0.adatum.com**. Quand vous y êtes invité, fournissez les informations d’identification suivantes :

    -   Se connecter en tant que : **ADATUM\\Student**

    -   Mot de passe : *le même mot de passe que celui que vous avez spécifié précédemment dans ce labo*

1. Dans la session RDP avec i20-db-0.adatum.com, dans Gestionnaire de serveur, accédez à la vue **Serveur local** et désactivez **Configuration de sécurité renforcée d’Internet Explorer**.

1. Dans la session RDP avec i20-db-0.adatum.com, dans le menu **Outils** du Gestionnaire de serveur, démarrez **Centre d’administration Active Directory**.

1. Dans le Centre d’administration Active Directory, créez une unité d’organisation nommée **Clusters** à la racine du domaine adatum.com.

1. Dans le Centre d’administration Active Directory, déplacez les comptes d’ordinateurs de i20-db-0 et i20-db-1 du conteneur **Ordinateurs** vers l’unité d’organisation **Clusters**.

1. Dans la session RDP avec i20-db-0, démarrez une session Windows PowerShell ISE et créez un cluster en exécutant ceci :

    ```
    $nodes = @('i20-db-0','i20-db-1')

    New-Cluster -Name az12003b-db-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.15
    ```

1. Dans la session RDP avec i20-db-0.adatum.com, passez à la console **Centre d’administration Active Directory**.

1. Dans le Centre d’administration Active Directory, accédez à l’unité d’organisation **Clusters** et affichez sa fenêtre **Propriétés**. 

1. Dans la fenêtre **Propriétés** de l’unité organisationnelle **Clusters**, accédez à la section **Extensions**, puis affichez l’onglet **Sécurité**. 

1. Sous l’onglet **Sécurité**, cliquez sur le bouton **Avancé** pour ouvrir la fenêtre **Paramètres de sécurité avancés pour Clusters**. 

1. Sous l’onglet **Autorisations** de la fenêtre **Paramètres de sécurité avancés pour Clusters**, cliquez sur **Ajouter**.

1. Dans la fenêtre **Entrée d’autorisation pour Clusters**, cliquez sur **Sélectionner le principal**

1. Dans la boîte de dialogue **Sélectionner l’utilisateur, le compte de service ou le groupe**, cliquez sur **Types d’objets**, cochez la case en regard de l’entrée **Ordinateurs**, puis cliquez sur **OK**. 

1. De retour dans la boîte de dialogue **Sélectionner l’utilisateur, l’ordinateur, le compte de service ou le groupe**, dans le champ **Entrer le nom de l’objet à sélectionner**, tapez **az12003b-db-cl0**, puis cliquez sur **OK**.

1. Dans la fenêtre **Entrée d’autorisation pour Clusters**, vérifiez que **Autoriser** apparaît dans la liste déroulante **Type**. Ensuite, dans la liste déroulante **S’applique à**, sélectionnez **Cet objet et tous les objets descendants**. Dans la liste **Autorisations**, cochez les cases **Créer des objets ordinateur** et **Supprimer des objets ordinateur**, puis cliquez sur **OK** deux fois.

1. Dans la session Windows PowerShell ISE, installez le module Az PowerShell en exécutant ceci :

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Dans la session Windows PowerShell ISE, authentifiez-vous en utilisant vos informations d’identification Azure AD en exécutant ceci :

    ```
    Add-AzAccount
    ```

    > **Remarque** : Quand vous y êtes invité, connectez-vous en utilisant le compte Microsoft professionnel, scolaire ou personnel avec le rôle de propriétaire ou de contributeur de l’abonnement Azure que vous utilisez pour ce labo.

1. Dans la session Windows PowerShell ISE, exécutez la commande suivante pour définir la valeur de la variable `$resourceGroupName` sur le nom du groupe de ressources contenant le compte de stockage que vous avez provisionné dans la tâche précédente :

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Dans la session Windows PowerShell ISE, exécutez la commande suivante pour définir le quorum de témoin cloud du nouveau cluster :

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Pour vérifier la configuration résultante, dans la session RDP avec i20-db-0.adatum.com, dans le menu **Outils** du Gestionnaire de serveur, démarrez **Gestionnaire du cluster de basculement**.

1. Dans la console **Gestionnaire du cluster de basculement**, passez en revue la configuration du cluster **az12003b-db-cl0**, y compris ses nœuds ainsi que ses paramètres de témoin et de réseau. Notez que le cluster n’a pas de stockage partagé.


### Tâche 6 : Configurer le clustering de basculement sur des machines virtuelles Azure exécutant Windows Server 2016 pour prendre en charge un niveau ASCS à haute disponibilité de l’installation SAP NetWeaver

> **Remarque** : Vérifiez que le déploiement du cluster S2D que vous avez lancé dans la tâche 4 de l’exercice 1 a réussi avant de démarrer cette tâche.

1. Dans la session RDP avec az12003b-vm0, utilisez Bureau à distance pour vous connecter à la machine virtuelle Azure **i20-ascs-0.adatum.com**. Quand vous y êtes invité, fournissez les informations d’identification suivantes :

    -   Se connecter en tant que : **ADATUM\\Student**

    -   Mot de passe : *le même mot de passe que celui que vous avez spécifié précédemment dans ce labo*

1. Dans la session RDP avec i20-ascs-0.adatum.com, dans Gestionnaire de serveur, accédez à la vue **Serveur local** et désactivez **Configuration de sécurité renforcée d’Internet Explorer**.

1. Dans la session RDP avec i20-ascs-0.adatum.com, dans le menu **Outils** du Gestionnaire de serveur, démarrez **Centre d’administration Active Directory**.

1. Dans le Centre d’administration Active Directory, accédez au conteneur **Ordinateurs**. 

1. Dans le Centre d’administration Active Directory, déplacez les comptes d’ordinateurs de i20-ascs-0 et i20-ascs-1 du conteneur **Ordinateurs** vers l’unité d’organisation **Clusters**.

1. Dans la session RDP avec i20-ascs-0.adatum.com, démarrez une session Windows PowerShell ISE et créez un cluster en exécutant ceci :

    ```
    $nodes = @('i20-ascs-0','i20-ascs-1')

    New-Cluster -Name az12003b-ascs-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.16
    ```

1. Dans la session RDP avec i20-ascs-0.adatum.com, passez à la console **Centre d’administration Active Directory**.

1. Dans le Centre d’administration Active Directory, accédez à l’unité d’organisation **Clusters** et affichez sa fenêtre **Propriétés**. 

1. Dans la fenêtre **Propriétés** de l’unité organisationnelle **Clusters**, accédez à la section **Extensions**, puis affichez l’onglet **Sécurité**. 

1. Sous l’onglet **Sécurité**, cliquez sur le bouton **Avancé** pour ouvrir la fenêtre **Paramètres de sécurité avancés pour Clusters**. 

1. Sous l’onglet **Autorisations** de la fenêtre **Paramètres de sécurité avancés pour les ordinateurs**, cliquez sur **Ajouter**.

1. Dans la fenêtre **Entrée d’autorisation pour Clusters**, cliquez sur **Sélectionner le principal**.

1. Dans la boîte de dialogue **Sélectionner l’utilisateur, le compte de service ou le groupe**, cliquez sur **Types d’objets**, cochez la case en regard de l’entrée **Ordinateurs**, puis cliquez sur **OK**. 

1. De retour dans la boîte de dialogue **Sélectionner l’utilisateur, l’ordinateur, le compte de service ou le groupe**, dans le champ **Entrer le nom de l’objet à sélectionner**, tapez **az12003b-ascs-cl0**, puis cliquez sur **OK**.

1. Dans la fenêtre **Entrée d’autorisation pour Clusters**, vérifiez que **Autoriser** apparaît dans la liste déroulante **Type**. Ensuite, dans la liste déroulante **S’applique à**, sélectionnez **Cet objet et tous les objets descendants**. Dans la liste **Autorisations**, cochez les cases **Créer des objets ordinateur** et **Supprimer des objets ordinateur**, puis cliquez sur **OK** deux fois.

1. Dans la session Windows PowerShell ISE, installez le module Az PowerShell en exécutant ceci :

    ```
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Dans la session Windows PowerShell ISE, authentifiez-vous en utilisant vos informations d’identification Azure AD en exécutant ceci :

    ```
    Add-AzAccount
    ```

    > **Remarque** : Quand vous y êtes invité, connectez-vous en utilisant le compte Microsoft professionnel, scolaire ou personnel avec le rôle de propriétaire ou de contributeur de l’abonnement Azure que vous utilisez pour ce labo.

1. Dans la session Windows PowerShell ISE, exécutez la commande suivante pour définir la valeur de la variable `$resourceGroupName` sur le nom du groupe de ressources contenant le compte de stockage que vous avez provisionné dans cet exercice :

    ```
    $resourceGroupName = 'az12003b-sap-RG'
    ```

1. Dans la session Windows PowerShell ISE, exécutez ceci pour définir le quorum de témoin cloud du cluster :

    ```
    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Pour vérifier la configuration résultante, dans la session RDP avec i20-ascs-0.adatum.com, dans le menu **Outils** du Gestionnaire de serveur, démarrez **Gestionnaire du cluster de basculement**.

1. Dans la console **Gestionnaire du cluster de basculement**, passez en revue la configuration du cluster **az12003b-ascs-cl0**, y compris ses nœuds ainsi que ses paramètres de témoin et de réseau. Notez que le cluster n’a pas de stockage partagé.


### Tâche 7 : Définir des autorisations sur le partage \\\\GLOBALHOST\\sapmnt

Dans cette tâche, vous allez définir des autorisations de niveau partage sur le partage **\\\\GLOBALHOST\\sapmnt**.

> **Remarque** : Par défaut, les autorisations Contrôle total sont accordées seulement au compte ADATUM\Student. 

1. Dans la session Bureau à distance pour i20-ascs-0.adatum.com, depuis la fenêtre **Windows PowerShell ISE**, exécutez ceci :

    ```
    $remoteSession = New-CimSession -ComputerName SAPGLOBALHOST

    Grant-SmbShareAccess -Name sapmnt -AccountName 'ADATUM\Domain Admins' -AccessRight Full -CimSession $remoteSession -Force
   
    ```

### Tâche 8 : Configurer les prérequis du système d’exploitation pour l’installation des composants ASCS et de base de données de SAP NetWeaver

1. Dans la session Bureau à distance pour i20-ascs-0.adatum.com, depuis la session Windows PowerShell ISE, exécutez ce qui suit pour configurer les entrées de Registre nécessaires pour faciliter l’installation des composants ASCS de SAP et l’utilisation de noms virtuels :

    ```
    $nodes = ('i20-db-0','i20-db-1')

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanworkstation\parameters'
        $registryEntry = 'DisableCARetryOnInitialConnect'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Control\LSA'
        $registryEntry = 'DisableLoopbackCheck'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }

    Invoke-Command $nodes {
        $registryPath = 'HKLM:\SYSTEM\CurrentControlSet\Services\lanmanserver\parameters'
        $registryEntry = 'DisableStrictNameChecking'
        $registryValue = 1
        New-ItemProperty -Path $registryPath -Name $registryEntry -Value $registryValue -PropertyType DWORD -Force
    }
    ```

> **Result** : Une fois cet exercice terminé, vous aurez configuré le système d’exploitation de machines virtuelles Azure exécutant Windows pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité.


## Exercice 3 : Supprimer des ressources de lab

Durée : 10 minutes

Dans cet exercice, vous allez supprimer les ressources approvisionnées dans ce labo.

#### Tâche 1 : Ouvrir Cloud Shell

1. En haut du portail, cliquez sur l’icône **Cloud Shell** pour ouvrir le volet Cloud Shell, puis choisissez PowerShell comme shell.

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$resourceGroupName` sur le nom du groupe de ressources contenant la paire de machines virtuelles Azure **Windows Server 2019 Datacenter** que vous avez provisionnées dans le premier exercice de ce labo :

    ```
    $resourceGroupNamePrefix = 'az12003b-'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour répertorier tous les groupes de ressources que vous avez créés dans ce labo :

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Select-Object ResourceGroupName
    ```

1. Vérifiez que la sortie contient seulement les groupes de ressources que vous avez créés dans ce labo. Ces groupes seront supprimés dans la tâche suivante.

#### Tâche 2 : Supprimer des groupes de ressources

1. Dans le volet Cloud Shell, exécutez la commande suivante pour supprimer les groupes de ressources que vous avez créés dans ce labo

    ```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like "$resourceGroupNamePrefix*"} | Remove-AzResourceGroup -Force  
    ```

1. Fermez le volet Cloud Shell.

> **Result** : Une fois cet exercice terminé, vous avez supprimé les ressources utilisées dans ce labo.
