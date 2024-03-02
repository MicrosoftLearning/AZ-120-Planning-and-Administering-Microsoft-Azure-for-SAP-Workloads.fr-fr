---
lab:
  title: "01b\_: implémentation du clustering Windows sur des machines virtuelles Azure"
  module: Module 01 - Explore the foundations of IaaS for SAP on Azure
---

# AZ 120 Module 1 : Explorer les bases de l’infrastructure as a service (IaaS) pour SAP sur Azure
# Labo 1b : Implémentation du clustering Windows sur des machines virtuelles Azure

Temps estimé : 120 minutes

Toutes les tâches de ce labo sont effectuées à partir du portail Azure (y compris la session Cloud Shell PowerShell)  

   > **Remarque** : Si vous n’utilisez pas Cloud Shell, la machine virtuelle du labo doit avoir le module Az PowerShell d’installé [**https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi**](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps-msi).

Fichiers de lab : aucun

## Scénario
  
En préparation du déploiement de SAP NetWeaver sur Azure, avec SQL Server en tant que système de gestion de base de données, Adatum Corporation souhaite explorer le processus d’implémentation du clustering sur des machines virtuelles Azure exécutant Windows Server 2022.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

-   Approvisionnez les ressources de calcul Azure nécessaires à la prise en charge des déploiements SAP NetWeaver à haut niveau de disponibilité.

-   Configurez le système d’exploitation des machines virtuelles Azure exécutant Windows Server 2022 pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité.

-   Approvisionnez les ressources réseau Azure nécessaires à la prise en charge des déploiements SAP NetWeaver à haut niveau de disponibilité.

## Spécifications

-   Un abonnement Microsoft Azure comportant un nombre suffisant de processeurs virtuels DSv2 et Dsv3 disponibles (une machine virtuelle Standard_DS1_v2 avec un processeur virtuel et quatre machines virtuelles Standard_D4s_v3 avec quatre processeurs virtuels chacune) dans la région Azure que vous prévoyez d’utiliser dans le cadre de ce labo

-   Un ordinateur de labo avec un navigateur web compatible Azure Cloud Shell et un accès à Azure

> **Remarque** : Assurez-vous que la région Azure que vous choisissez pour le déploiement de vos ressources prend en charge les zones de disponibilité. Pour obtenir la liste de ces régions, reportez-vous à (https://docs.microsoft.com/en-us/azure/availability-zones/az-overview). Envisagez d’utiliser **USA Est** ou **USA Est2**.

## Exercice 1 : Approvisionner les ressources de calcul Azure nécessaires à la prise en charge des déploiements SAP NetWeaver à haut niveau de disponibilité

Durée : 50 minutes

Dans cet exercice, vous allez déployer des composants de calcul d’infrastructure Azure nécessaires pour configurer le clustering de basculement sur des machines virtuelles Azure exécutant Windows Server 2022. Cela impliquera le déploiement d’une paire de contrôleurs de domaine Active Directory, suivi d’une paire de machines virtuelles Azure exécutant Windows Server 2022. Chaque paire de machines virtuelles sera placée dans des zones de disponibilité distinctes au sein du même réseau virtuel. Pour automatiser le déploiement de contrôleurs de domaine, vous allez utiliser un modèle de démarrage rapide Azure Resource Manager disponible à partir de <https://aka.ms/az120-1bdeploy>

### Tâche 1 : Déployer une paire de machines virtuelles Azure exécutant des contrôleurs de domaine Active Directory à haute disponibilité en utilisant un modèle Bicep

1. Depuis l’ordinateur de labo, démarrez un navigateur web et accédez au portail Azure à partir de **https://portal.azure.com**.

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
    $rgName = 'az12001b-ad-RG'
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
    $deploymentName = 'az1201b-' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
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

    - Accédez au panneau **az1201b-ad-RG** puis, dans le menu de navigation vertical situé à gauche, dans la **section Paramètres**, sélectionnez **Déploiements**.

    - Dans le panneau **az1201b-ad-RG \| Déploiements**, sélectionnez le déploiement dont le nom commence par le préfixe **az1201b** puis, dans le panneau de déploiement, sélectionnez **Redéployer**.

    - Dans le panneau **Déploiement personnalisé**, dans la zone de texte **Mot de passe de l’administrateur**, entrez le même mot de passe que celui que vous avez utilisé lors du déploiement d’origine, sélectionnez **Vérifier + créer**, puis sélectionnez **Créer**.

    - N’attendez pas que le déploiement se termine et passez à la tâche suivante. Le déploiement doit prendre environ 3 minutes.

### Tâche 2 : Déployer une paire de machines virtuelles Azure exécutant Windows Server 2022 dans différentes zones de disponibilité

1. Sur l’ordinateur du labo, sur le portail Azure, naviguer vers me volet **Machines virtuelles**, cliquer sur **+ Créer** et depuis le menu déroulant, sélectionner **+ Machine virtuelle Azure**.

1. À partir du panneau **Créer une machine virtuelle**, lancez l’approvisionnement d’un **Windows Server 2022 Datacenter : Machine virtuelle Azure, Azure Edition - Gen2** avec les paramètres suivants (laissez les autres à leurs valeurs par défaut) :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | *nom de votre abonnement Azure*  |
    | **Groupe de ressources** | *le nom d’un nouveau groupe de ressources* **az12001b-cl-RG** |
    | **Nom de la machine virtuelle** | **az12001b-cl-vm0** |
    | **Région** | *la même région Azure que celle où vous avez déployé les machines virtuelles Azure dans la tâche précédente* |
    | **Options de disponibilité** | **Zone de disponibilité** |
    | **Zone de disponibilité** | **Zone 1** |
    | **Type de sécurité** | **Standard** |
    | **Image** | *sélectionner* **Windows Server 2022 Datacenter : Édition Azure - Gen 2** |
    | **Taille** | **Standard D4s v3** |
    | **Nom d’utilisateur** | *le même nom d’utilisateur que celui que vous avez spécifié lors du déploiement du modèle Bicep précédemment dans cet exercice* |
    | **Mot de passe** | *le même mot de passe que celui que vous avez spécifié lors du déploiement du modèle Bicep précédemment dans cet exercice* |
    | **Ports d’entrée publics** | **Aucun** |
    | **Souhaitez-vous utiliser une licence Windows Server existante ?** | **Aucun** |
    | **Type de disque du système d’exploitation** | **SSD Premium** |
    | **Réseau virtuel** | **adVNET** |
    | **Nom du sous-réseau** | *un nouveau sous-réseau nommé* **clSubnet** |
    | **Plage d’adresses de sous-réseau** | **10.0.1.0/24** |
    | **Adresse IP publique** | **Aucun** |
    | **Groupe de sécurité réseau de la carte réseau** | **Aucun**  |
    | **Activer la mise en réseau accélérée** | **Activé** |
    | **Options d’équilibrage de charge** | **Aucun** |
    | **Activer l’identité managée affectée par le système** | **Désactivé** |
    | **Se connecter avec Azure AD** | **Désactivé** |
    | **Activer l’arrêt automatique** | **Désactivé** |
    | **Option d'orchestration de patch** | **Mises à jour manuelles** |
    | **Diagnostics de démarrage** | **Désactiver** |
    | **Extensions** | *Aucun* |
    | **Balises** | *Aucun* |

1. Ne pas attendre que l’approvisionnement se termine mais passer à la tâche suivante.

1. Approvisionner un autre **Windows Server 2022 Datacenter : Machine virtuelle Azure, Azure Edition - Gen2** avec les paramètres suivants :
     
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | *nom de votre abonnement Azure*  |
    | **Groupe de ressources** | *le nom du groupe de ressources que vous avez utilisé lors du déploiement du premier **Windows Server 2022 Datacenter : Machine virtuelle Azure, Azure Edition - Gen2** dans cette tâche* |
    | **Nom de la machine virtuelle** | **az12001b-cl-vm1** |
    | **Région** | *la même région Azure que celle où vous avez déployé le premier **Centre de données Windows Server 2022 : Machine virtuelle Azure, Azure Edition - Gen2** dans cette tâche* |
    | **Options de disponibilité** | **Zone de disponibilité** |
    | **Zone de disponibilité** | **Zone 2** |
    | **Type de sécurité** | **Standard** |
    | **Image** | *sélectionner* **Windows Server 2022 Datacenter : Édition Azure - Gen 2** |
    | **Taille** | **Standard D4s v3** |
    | **Nom d’utilisateur** | *le même nom d’utilisateur que celui que vous avez spécifié lors du déploiement du modèle Bicep précédemment dans cet exercice* |
    | **Mot de passe** | *le même mot de passe que celui que vous avez spécifié lors du déploiement du modèle Bicep précédemment dans cet exercice* |
    | **Ports d’entrée publics** | **Aucun** |
    | **Souhaitez-vous utiliser une licence Windows Server existante ?** | **Aucun** |
    | **Type de disque du système d’exploitation** | **SSD Premium** |
    | **Réseau virtuel** | **adVNET** |
    | **Nom du sous-réseau** | **clSubnet** |
    | **Adresse IP publique** | **Aucun** |
    | **Groupe de sécurité réseau de la carte réseau** | **Aucun**  |
    | **Activer la mise en réseau accélérée** | **Activé** |
    | **Options d’équilibrage de charge** | **Aucun** |
    | **Se connecter avec Azure AD** | **Désactivé** |
    | **Activer l’arrêt automatique** | **Désactivé** |
    | **Option d'orchestration de patch** | **Mises à jour manuelles** |
    | **Diagnostics de démarrage** | **Désactiver** |
    | **Extensions** | *Aucun* |
    | **Balises** | *Aucun* |

1. Attendez la fin du déploiement. Ce processus devrait prendre quelques minutes.

### Tâche 3 : Créer et configurer des disques de machines virtuelles Azure

1. Dans le portail Azure, démarrez une session PowerShell dans Cloud Shell. 

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$resourceGroupName` sur le nom du groupe de ressources contenant les ressources que vous avez provisionnées dans la tâche précédente :

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer le premier ensemble de 4 disques managés que vous allez attacher à la première machine virtuelle Azure que vous avez déployée dans la tâche précédente :

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location
    
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm0').Zones

    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone

    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm0-DataDisk$i -Disk $diskConfig}
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer le deuxième ensemble de 4 disques managés que vous allez attacher à la deuxième machine virtuelle Azure que vous avez déployée dans la tâche précédente :

    ```
    $zone = (Get-AzVM -ResourceGroupName $resourceGroupName -Name 'az12001b-cl-vm1').Zones
    
    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty -Zone $zone
        
    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm1-DataDisk$i -Disk $diskConfig}
    ```

1. Dans le portail Azure, accédez au panneau de la première machine virtuelle Azure que vous avez provisionnée dans la tâche précédente (**az12001b-cl-vm0**).

1. Depuis le panneau **az12001b-cl-vm0**, accédez au panneau **az12001b-cl-vm0 - Disques**.

1. À partir du panneau **az12001b-cl-vm0 - Disques**, attachez des disques de données avec les paramètres suivants à az12001b-cl-vm0 :
   
   | Paramètre | Valeur |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nom du disque** | **az12001b-cl-vm0-DataDisk0** |
   | **Groupe de ressources** | *le nom du groupe de ressources que vous avez utilisé lors du déploiement de la paire de **machines virtuelles Centre de données Azure Windows Server 2022** dans la tâche précédente* |
   | **MISE EN CACHE DE L’HÔTE** | **Lecture seule** |

1. Répétez l’étape précédente pour attacher les 3 disques restants avec le préfixe **az12001b-cl-vm0-DataDisk** (pour le total des 4). Affectez le numéro d’unité logique (LUN) qui correspond au dernier caractère du nom du disque. Pour le dernier disque (LUN **3**), définir MISE EN CACHE DE L’HÔTE sur **None**.

1. Enregistrez vos modifications. 

1. Dans le portail Azure, accéder au panneau de la deuxième machine virtuelle Azure que vous avez provisionnée dans la tâche précédente (**az12001b-cl-vm1**).

1. Depuis le panneau **az12001b-cl-vm1**, accéder au panneau **az12001b-cl-vm1 - Disques**.

1. À partir du panneau **az12001b-cl-vm1 - Disques**, joindre des disques de données avec les paramètres suivants à az12001b-cl-vm1 :
     
   | Paramètre | Valeur |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nom du disque** | **az12001b-cl-vm1-DataDisk0** |
   | **Groupe de ressources** | *le nom du groupe de ressources que vous avez utilisé lors du déploiement de la paire de **machines virtuelles Centre de données Azure Windows Server 2022** dans la tâche précédente* |
   | **MISE EN CACHE DE L’HÔTE** | **Lecture seule** |

1. Répéter l’étape précédente pour attacher les 3 disques restants avec le préfixe **az12001b-cl-vm1-DataDisk** (pour le total des 4). Affectez le numéro d’unité logique (LUN) qui correspond au dernier caractère du nom du disque. Pour le dernier disque (LUN **3**), définir MISE EN CACHE DE L’HÔTE sur **None**.

1. Enregistrez vos modifications. 

#### Tâche 4 : Provisionner Azure Bastion 

> **Remarque** : Azure Bastion autorise la connexion aux machines virtuelles Azure (que vous avez déployées dans la tâche précédente de cet exercice) sans utiliser de points de terminaison publics, tout en fournissant une protection contre les attaques par force brute qui ciblent les informations d’identification au niveau du système d’exploitation.

> **Remarque** : Pout utiliser Azure Bastion, vérifiez que la fonctionnalité de fenêtre contextuelle de votre navigateur est activée.

1. Dans la fenêtre du navigateur affichant le Portail Azure, ouvrez un autre onglet, et, dans l’onglet du navigateur, accédez au [**Portail Azure**](https://portal.azure.com).
1. Dans le portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de barre d’outils juste à droite de la zone de texte de recherche.
1. Dans la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour ajouter un sous-réseau nommé **AzureBastionSubnet** au réseau virtuel nommé **az12001a-RG-vnet** que vous avez créé précédemment dans cet exercice :

   ```powershell
   $resourceGroupName = 'az12001b-cl-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'adVNET'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 10.0.255.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Fermez le volet Cloud Shell.
1. Dans le portail Azure, recherchez et sélectionnez **Bastions**, puis dans le panneau **Bastions**, sélectionnez **+ Créer**.
1. Sous l’onglet **De base** du panneau **Créer un bastion**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az12001b-cl-RG**|
   |Nom|**az12001b-bastion**|
   |Région|la même région Azure dans laquelle vous avez déployé les ressources dans les tâches précédentes de cet exercice|
   |Niveau|**De base**|
   |Réseau virtuel|**adVNET**|
   |Sous-réseau|**AzureBastionSubnet (10.0.255.0/24)**|
   |Adresse IP publique|**Création**|
   |Nom de l’IP publique|**adVNET-ip**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un bastion**, sélectionnez **Créer** :

   > **Remarque** : N’attendez pas la fin du déploiement. Vous pouvez passer à la tâche suivante de cet exercice. Le déploiement peut prendre environ 5 minutes.

> **Result** : Une fois cet exercice terminé, vous avez provisionné des ressources de calcul Azure nécessaires pour prendre en charge les déploiements SAP NetWeaver à haute disponibilité.

## Exercice 2 : Configurer le système d’exploitation des machines virtuelles Azure exécutant le centre de données Windows Server 2022 pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité

Durée : 40 minutes

### Tâche 1 : Joindre les machines virtuelles du centre de données Azure Windows Server 2022 au domaine Active Directory.

   > **Remarque** : Avant de commencer cette tâche, vérifiez que le déploiement de modèle que vous avez lancé lors de la tâche précédente a réussi. 

1. Dans le portail Azure, accédez au panneau du réseau virtuel **adVNET**, qui a été provisionné automatiquement dans le premier exercice de ce labo.

1. Affichez le panneau **adVNET – Serveurs DNS** et notez que le réseau virtuel est configuré avec les adresses IP privées affectées aux contrôleurs de domaine déployés dans le premier exercice de ce labo en tant que serveurs DNS.

1. Dans le portail Azure, démarrez une session PowerShell dans Cloud Shell. 

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$resourceGroupName` sur le nom du groupe de ressources contenant la paire de **Centre de données Windows Server 2022 : Machines virtuelles Azure, Azure Edition - Gen2** que vous avez configurées dans l’exercice précédent :

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. Dans le volet Cloud Shell, exécuter la commande suivante pour joindre les machines virtuelles Azure Windows Server 2022 que vous avez déployées lors de la deuxième tâche de l’exercice précédent au domaine Active Directory **domaine adatum.com** (remplacer les espaces réservés `<username>` et `<password>` par le nom et le mot de passe du compte d’utilisateur administratif que vous avez spécifié lors du déploiement du modèle Bicep dans le premier exercice de ce labo) :

    ```
    $location = (Get-AzureRmResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\<username>", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "<password>"}'

    $vmNames = @('az12001b-cl-vm0','az12001b-cl-vm1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
    ```

1. Attendre que le script se termine avant de passer à la tâche suivante.

### Tâche 2 : Configurer le stockage des machines virtuelles Azure exécutant Windows Server 2022 pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité.

1. Dans le portail Azure, naviguer vers le panneau de la machine virtuelle **az12001b-cl-vm0** que vous avez approvisionné dans le premier exercice de ce labo.

1. Dans le panneau **az12001b-cl-vm0**, sélectionnez **Se connecter**. Dans le menu déroulant, sélectionnez **Se connecter via Bastion**. Sous l’onglet **Bastion** d’**az12001b-cl-vm0**, laissez le **Type d’authentification** défini sur **Mot de passe de la machine virtuelle**, indiquez les informations d’identification que vous avez définies lors du déploiement de la machine virtuelle **az12001b-cl-vm0**, laissez la case **Ouvrir dans un nouvel onglet de navigateur** cochée, puis sélectionnez **Se connecter**.

    > **Remarque** : Veillez à vous connecter à l’aide du compte de domaine **ADATUM**, plutôt que du compte de niveau système d’exploitation (par exemple, assurez-vous que le nom d’utilisateur est précédé du préfixe **ADATUM\\**.

1. Dans la session Bastion vers az12001b-cl-vm0, dans Gestionnaire de serveur, naviguez vers le nœud **Fichiers et services de stockage** -> **Serveurs**. 

1. Accéder à la vue **Pools de Stockage** et vérifier que vous voyez tous les disques que vous avez joints à la machine virtuelle Azure dans l’exercice précédent.

1. Utiliser le **Nouvel Assistant de Pools de Stockage** pour créer un pool de stockage avec les paramètres suivants :
     
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom** | **Pool de stockage de Données** |
    | **Disques Physiques** | *sélectionnez les 3 disques avec des numéros de disque correspondant aux trois premiers numéros d’unité logique (0-2) et définissez leur allocation sur* **Automatique** |

    > **Remarque** : Utiliser l’entrée dans la colonne **Châssis** pour identifier le numéro **numéro d'unité logique**.

1. Utiliser le **Nouvel Assistant de Disque Virtuel** pour créer un disque virtuel avec les paramètres suivants :
     
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom du Disque virtuel** | **Disque de Données Virtuel** |
    | **Disposition du Stockage** | **Simple** |
    | **Approvisionnement** | **Fixe** |
    | **Taille** | **Taille maximale** |

1. Utiliser l’**Assistant de Nouveau Volume** pour créer un volume avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Serveur et Disque** | *accepter les valeurs par défaut* |
    | **Taille** | *accepter les valeurs par défaut* |
    | **Lettre de lecteur** | **M** |
    | **Système de fichiers** | **ReFS** |
    | **Taille d'unité d'allocation** | **Par défaut** |
    | **Étiquette de volume** | **Données** |

1. De retour dans l’affichage **Pools de Stockage**, utiliser le **Nouvel Assistant de Pools de Stockage** pour créer un pool de stockage avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom** | **Journaliser du Pool de Stockage** |
    | **Disques Physiques** | *sélectionnez le dernier des 4 disques et définissez son allocation sur* **Automatique** |

1. Utiliser le **Nouvel Assistant de Disque Virtuel** pour créer un disque virtuel avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom du Disque virtuel** | **Journaliser le Disque virtuel** |
    | **Disposition du Stockage** | **Simple** |
    | **Approvisionnement** | **Fixe** |
    | **Taille** | **Taille maximale** |

1. Utiliser l’**Assistant de Nouveau Volume** pour créer un volume avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Serveur et Disque** | *accepter les valeurs par défaut* |
    | **Taille** | *accepter les valeurs par défaut* |
    | **Lettre de lecteur** | **L** |
    | **Système de fichiers** | **ReFS** |
    | **Taille d'unité d'allocation** | **Par défaut** |
    | **Étiquette de volume** | **Journal** |

1. Répétez l’étape précédente de cette tâche pour configurer le stockage sur az12001b-cl-vm1.

### Tâche 3 : Préparer la configuration du clustering de basculement sur des machines virtuelles Azure exécutant Windows Server 2022 pour prendre en charge une installation SAP NetWeaver à haute disponibilité.

1. Dans la session Bastion vers az12001b-cl-vm0, démarrez une session Windows PowerShell ISE et installez les fonctionnalités d’outils de clustering de basculement et d’administration à distance sur az12001b-cl-vm0 et az12001b-cl-vm1 en exécutant les opérations suivantes :

    ```
    $nodes = @('az12001b-cl-vm1', 'az12001b-cl-vm0')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
    ```

    > **Remarque** : Ceci entraînera le redémarrage du système d’exploitation invité des deux machines virtuelles Azure.

1. Sur l’ordinateur de labo, dans le portail Azure, cliquez sur **+ Créer une ressource**.

1. Dans le panneau **Nouveau**, lancez la création d’un nouveau **Compte de stockage** avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | *nom de votre abonnement Azure* |
    | **Groupe de ressources** | *le nom du groupe de ressources contenant la paire de machines virtuelles ** Azure Windows Server 2022 Datacenter** que vous avez provisionnées dans l’exercice précédent* |
    | **Nom du compte de stockage** | *chaque nom unique composé de 3 à 24 lettres et chiffres* |
    | **Lieu** | *la même région Azure que celle où vous avez déployé les machines virtuelles Azure dans l’exercice précédent* |
    | **Niveau de performance** | **Standard** |
    | **Redondance** | **Stockage localement redondant (LRS)** |
    | **Méthode de connectivité** | **Point de terminaison public (tous les réseaux)** |
    | **Exiger un transfert sécurisé pour les opérations d’API REST** | **Activé** |
    | **Partages de fichiers volumineux** | **Disabled** |
    | **Suppression réversible pour les objets blob, les conteneurs et les fichiers** | **Disabled** |
    | **Espace de noms hiérarchique** | **Disabled** |

### Tâche 4 : Configurer le clustering de basculement sur des machines virtuelles Azure exécutant Windows Server 2022 pour prendre en charge l’installation SAP NetWeaver à haute disponibilité.

1. Dans le portail Azure, naviguer vers le panneau de la machine virtuelle **az12001b-cl-vm0** que vous avez approvisionné dans le premier exercice de ce labo.

1. À partir du panneau **az12001b-cl-vm0**, connectez-vous au système d’exploitation invité de la machine virtuelle à l’aide d’Azure Bastion. Lorsque vous êtes invité à vous authentifier, fournissez les informations d’identification du compte d’utilisateur administratif que vous avez spécifié lors du déploiement du modèle Bicep dans le premier exercice de ce labo.

1. Dans la session Bastion avec az12001b-cl-vm0, depuis le menu **Outils** du Gestionnaire de serveur, démarrez **Centre d’administration Active Directory**.

1. Dans le Centre d’administration Active Directory, créez une unité d’organisation nommée **Clusters** à la racine du domaine adatum.com.

1. Dans le Centre d’administration Active Directory, déplacez les comptes d’ordinateurs de **az12001b-cl-vm0** et **az12001b-cl-vm1** du conteneur **Ordinateurs** vers l’unité d’organisation **Clusters**.

1. Dans la session Bastion avec az12001b-cl-vm0, démarrez une session Windows PowerShell ISE et créez un cluster en exécutant les opérations suivantes :

    ```
    $nodes = @('az12001b-cl-vm0','az12001b-cl-vm1')

    New-Cluster -Name az12001b-cl-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
    ```

1. Dans la session Bastion vers az12001b-cl-vm0, basculez vers la console **Centre d’administration Active Directory**.

1. Dans le Centre d’administration Active Directory, accédez à l’unité d’organisation **Clusters** et affichez sa fenêtre **Propriétés**. 

1. Dans la fenêtre **Propriétés** de l’unité organisationnelle **Clusters**, accédez à la section **Extensions**, puis affichez l’onglet **Sécurité**. 

1. Sous l’onglet **Sécurité**, cliquez sur le bouton **Avancé** pour ouvrir la fenêtre **Paramètres de sécurité avancés pour Clusters**. 

1. Sous l’onglet **Autorisations** de la fenêtre **Paramètres de sécurité avancés pour Clusters**, cliquez sur **Ajouter**.

1. Dans la fenêtre **Entrée d’autorisation pour Clusters**, cliquez sur **Sélectionner le principal**

1. Dans la boîte de dialogue **Sélectionner l’utilisateur, le compte de service ou le groupe**, cliquez sur **Types d’objets**, cochez la case en regard de l’entrée **Ordinateurs**, puis cliquez sur **OK**. 

1. De retour dans la boîte de dialogue **Sélectionner l’utilisateur, l’ordinateur, le compte de service ou le groupe**, dans le champ **Entrer le nom de l’objet à sélectionner**, taper **az12001b-cl-cl0**, puis cliquer sur **OK**.

1. Dans la fenêtre **Entrée d’autorisation pour Clusters**, vérifiez que **Autoriser** apparaît dans la liste déroulante **Type**. Ensuite, dans la liste déroulante **S’applique à**, sélectionnez **Cet objet et tous les objets descendants**. Dans la liste **Autorisations**, cochez les cases **Créer des objets ordinateur** et **Supprimer des objets ordinateur**, puis cliquez sur **OK** deux fois.

1. Dans la session Windows PowerShell ISE, installez le module Az PowerShell en exécutant ceci :

    ```
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
    ```

1. Dans la session Windows PowerShell ISE, authentifiez-vous en utilisant vos informations d’identification Azure AD en exécutant ceci :

    ```
    Add-AzAccount
    ```

    > **Remarque** : Quand vous y êtes invité, connectez-vous en utilisant le compte Microsoft professionnel, scolaire ou personnel avec le rôle de propriétaire ou de contributeur de l’abonnement Azure que vous utilisez pour ce labo.

1. Dans la session Windows PowerShell ISE, définir le quorum de témoin cloud du nouveau cluster en exécutant la commande suivante :

    ```
    $resourceGroupName = 'az12001b-cl-RG'

    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
    ```

1. Pour vérifier la configuration résultante, dans la session Bastion avec az12001b-cl-vm0, dans le menu **Outils** du Gestionnaire de serveur, démarrez **Gestionnaire du cluster de basculement**.

1. Dans la console **Gestionnaire du cluster de basculement**, passez en revue la configuration du cluster **az12001b-cl-cl0**, y compris ses nœuds ainsi que ses paramètres de témoin et de réseau. Notez que le cluster n’a pas de stockage partagé.

1. Terminez la session Bastion sur az12001b-cl-vm0.

> **Result** : Une fois cet exercice terminé, vous aurez configuré le système d’exploitation de machines virtuelles Azure exécutant Serveur Windows 2022 pour prendre en charge une installation SAP NetWeaver à haute disponibilité


## Exercice 3 : Approvisionner les ressources réseau Azure nécessaires à la prise en charge des déploiements SAP NetWeaver à haut niveau de disponibilité

Durée : 30 minutes

Dans cet exercice, vous allez implémenter des équilibreurs de charge Azure pour prendre en charge les installations en cluster de SAP NetWeaver.

### Tâche 1 : Configurer des machines virtuelles Azure pour faciliter la configuration de l’équilibrage de charge.

   > **Remarque** : Étant donné que vous allez configurer une paire d’équilibreur de charge Azure de la référence SKU Stardard, vous devez d’abord supprimer les adresses IP publiques associées aux cartes réseau de deux machines virtuelles Azure qui serviront de pool principal à charge équilibrée.

1. Sur l’ordinateur du labo, dans le Portail Azure, naviguer vers le panneau de la machine virtuelle **Azure az12001b-cl-vm0**. 

1. Dans le panneau **az12001b-cl-vm0** , naviguer vers le panneau de l’adresse IP publique **az12001b-cl-vm0-ip** associée à sa carte réseau.

1. À partir du panneau **az12001b-cl-vm0-ip** , dissocier d’abord l’adresse IP publique de l’interface réseau, puis la supprimer.

1. Dans le portail Azure, accédez au panneau de la machine virtuelle Azure **az12001b-cl-vm1**. 

1. Depuis le panneau **az12001b-cl-vm1**, naviguer vers le panneau de l’adresse IP publique **az12001b-cl-vm1-ip** associée à sa carte réseau.

1. À partir du panneau **az12001b-cl-vm1-ip**, dissocier d’abord l’adresse IP publique de l’interface réseau, puis la supprimer.

1. Sur le Portail Azure, accédez au panneau de la machine virtuelle Azure **az12001b-cl-vm0**.

1. Depuis le panneau **az12001b-cl-vm0**, naviguez vers son panneau **Réseau**. 

1. Depuis le panneau **az12001b-cl-vm0 : Mise en réseau**, naviguez vers l’interface réseau de la machine virtuelle az12001b-cl-vm0. 

1. Dans le panneau de l’interface réseau de la machine virtuelle az12001b-cl-vm0, accédez à son panneau des configurations IP pour afficher son panneau **ipconfig1**.

1. Dans le panneau **ipconfig1**, définissez l’attribution d’adresse IP privée sur **Statique** et enregistrez la modification.

1. Sur le Portail Azure, accédez au panneau de la machine virtuelle Azure **az12001b-cl-vm1**.

1. Depuis le panneau **az12001b-cl-vm1**, naviguez vers son panneau **Réseau**. 

1. Depuis le panneau **az12001b-cl-vm1 : Mise en réseau**, naviguez vers l’interface réseau de la machine virtuelle az12001b-cl-vm1. 

1. Dans le panneau de l’interface réseau de la machine virtuelle az12001b-cl-vm1, accédez à son panneau des configurations IP pour afficher son panneau **ipconfig1**.

1. Dans le panneau **ipconfig1**, définissez l’attribution d’adresse IP privée sur **Statique** et enregistrez la modification.

### Tâche 2 : Créer et configurer des équilibreurs de charge Azure qui gèrent le trafic entrant

1. Dans le portail Azure, cliquez sur **+ Créer une ressource**.

1. Depuis le panneau **Nouveau**, lancer la création d’un nouveau Azure Load Balancer avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | *nom de votre abonnement Azure* |
    | **Groupe de ressources** | *le nom du groupe de ressources contenant la paire de machines virtuelles ** Azure Windows Server 2022 Datacenter** que vous avez provisionnées dans le premier exercice de ce labo* |
    | **Nom** | **az12001b-cl-lb0** |
    | **Région** | la même région Azure que celle où vous avez déployé les machines virtuelles Azure dans le premier exercice de ce labo |
    | **Référence (SKU)** | **Standard** |
    | **Type** | **Interne** |
    | **Nom de l'adresse IP de front-end** | **frontend-ip1** |
    | **Réseau virtuel** | **adVNET** |
    | **Sous-réseau** | **clSubnet** |
    | **Affectation d’adresses IP** | **Statique** |
    | **Adresse IP** | **10.0.1.240** |
    | **Zone de disponibilité** | **Redondant interzone** |

1. Attendez que l’équilibreur de charge soit approvisionné, puis accédez à son panneau dans la Portail Azure.

1. Dans le panneau **az12001b-cl-lb0**, ajouter un pool backend avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom** | **az12001b-cl-lb0-bepool** |
    | **Réseau virtuel** | **adVNET** |
    | **Configuration du pool de back-ends** | **Adresse IP** |
    | **Adresse IP** | **10.0.1.4** Nom de la Ressource **az1201b-cl-vm0** |
    | **Adresse IP** | **10.0.1.5** Nom de la Ressource **az1201b-cl-vm1** |

1. Depuis le panneau **az12001b-cl-lb0**, ajouter une sonde d'intégrité avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom** | **az12001b-cl-lb0-hprobe** |
    | **Protocole** | **TCP** |
    | **Port** | **59999** |
    | **Intervalle** | **5** *secondes* |
    | **Seuil de défaillance sur le plan de l’intégrité** | **2** *défaillances consécutives* |

1. À partir du panneau **az12001b-cl-lb0**, ajouter une règle d’équilibrage de charge réseau avec les paramètres suivants :
     
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom** | **az12001b-cl-lb0-lbruletcp1433** |
    | **Version de l’adresse IP** | **IPv4** |
    | **Frontend IP address (Adresse IP frontale)** | **10.0.1.240 (LoadBalancerFrontEnd)** |
    | **Ports HA** | **Disabled** |
    | **Protocole** | **TCP** |
    | **Port** | **1433** |
    | **Port principal** | **1433** |
    | **Pool back-end** | **az12001b-cl-lb0-bepool (2 machines virtuelles)** |
    | **Sonde d’intégrité** | **az12001b-cl-lb0-hprobe (TCP:59999)** |
    | **Persistance de session** | **Aucun** |
    | **Délai d’inactivité (minutes).** | **4** |
    | **Réinitialisation du protocole TCP** | **Disabled** |
    | **Adresse IP flottante (retour direct du serveur)** | **Activé** |

### Tâche 3 : Créer et configurer des équilibreurs de charge Azure qui gèrent le trafic sortant

1. Depuis le portail Azure, démarrer une session PowerShell dans Cloud Shell. 

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$resourceGroupName` sur le nom du groupe de ressources contenant la paire de machines virtuelles Azure **Windows Server 2022 Datacenter** que vous avez provisionnées dans le premier exercice de ce labo :

    ```
    $resourceGroupName = 'az12001b-cl-RG'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer l’adresse IP publique à utiliser par le deuxième équilibreur de charge :

    ```
    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $pipName = 'az12001b-cl-lb0-pip'

    az network public-ip create --resource-group $resourceGroupName --name $pipName --sku Standard --location $location
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer le deuxième équilibreur de charge :

    ```
    $lbName = 'az12001b-cl-lb1'

    $lbFeName = 'az12001b-cl-lb1-fe'

    $lbBePoolName = 'az12001b-cl-lb1-bepool'
   
    $pip = Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $pipName

    $feIpconfiguration = New-AzLoadBalancerFrontendIpConfig -Name $lbFeName -PublicIpAddress $pip

    $bePoolConfiguration = New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBePoolName

    New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Location $location -Name $lbName -Sku Standard -BackendAddressPool $bePoolConfiguration -FrontendIpConfiguration $feIpconfiguration
    ```

1. Fermez le volet Cloud Shell.

1. Dans le portail Azure, accédez au panneau affichant les propriétés de l’équilibreur de charge Azure **az12001b-cl-lb1**.

1. Dans le panneau **az12001b-cl-lb1**, cliquer sur **Pools principaux**.

1. Dans le panneau **az12001b-cl-lb1 - Pools Backend**, cliquer sur **az12001b-cl-lb1-bepool**.

1. Dans le panneau **az12001b-cl-lb1-bepool**, spécifier les paramètres suivants et cliquez sur **Enregistrer** :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Réseau virtuel** | **adVNET (4 VM)** |
    | **Machine virtuelle** | **az12001b-cl-vm0** ADDRESSE IP : **ipconfig1** |
    | **Machine virtuelle** | **az12001b-cl-vm1** ADDRESSE IP : **ipconfig1** |

1. Dans le panneau **az12001b-cl-lb1**, cliquer sur **Sondes d'intégrité**.

1. Depuis le panneau **az12001b-cl-lb1 - Sondes d’intégrité**, ajouter une sonde d'intégrité avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom** | **az12001b-cl-lb1-hprobe** |
    | **Protocole** | **TCP** |
    | **Port** | **80** |
    | **Intervalle** | **5** *secondes* |
    | **Seuil de défaillance sur le plan de l’intégrité** | **2** *défaillances consécutives* |

1. Dans le panneau **az12001b-cl-lb1**, cliquer sur **Règles d’équilibrage de charge**.

1. À partir du panneau **az12001b-cl-lb1 - Règles d’équilibrage de charge**, ajouter une règle d’équilibrage de charge réseau avec les paramètres suivants :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom** | **az12001b-cl-lb1-lbharule** |
    | **Version de l’adresse IP** | **IPv4** |
    | **Frontend IP address (Adresse IP frontale)** | *accepter la valeur par défaut* |
    | **Ports HA** | **Disabled** |
    | **Protocole** | **TCP** |
    | **Port** | **80** |
    | **Port principal** | **80** |
    | **Pool back-end** | **az12001b-cl-lb1-bepool (2 machines virtuelles)** |
    | **Sonde d’intégrité** | **az12001b-cl-lb1-hprobe (TCP:80)** |
    | **Persistance de session** | **Aucun** |
    | **Délai d’inactivité (minutes).** | **4** |
    | **Réinitialisation du protocole TCP** | **Disabled** |
    | **Adresse IP flottante (retour direct du serveur)** | **Disabled** |

> **Result** : Une fois cet exercice terminé, vous avez provisionné des ressources réseaux Azure nécessaires pour prendre en charge les déploiements SAP NetWeaver à haute disponibilité

## Exercice 4 : Supprimer des ressources de lab

Durée : 10 minutes

Dans cet exercice, vous allez supprimer les ressources approvisionnées dans ce labo.

#### Tâche 1 : Ouvrir Cloud Shell

1. En haut du portail, cliquez sur l’icône **Cloud Shell** pour ouvrir le volet Cloud Shell, puis choisissez PowerShell comme shell.

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `$resourceGroupName` sur le nom du groupe de ressources contenant la paire de machines virtuelles Azure **Windows Server 2022 Datacenter** que vous avez provisionnées dans le premier exercice de ce labo :

    ```
    $resourceGroupNamePrefix = 'az12001b-'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour lister tous les groupes de ressources que vous avez créés dans ce labo :

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
