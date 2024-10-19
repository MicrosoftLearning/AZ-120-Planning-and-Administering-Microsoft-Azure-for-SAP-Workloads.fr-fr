---
lab:
  title: "02a\_: Implémenter le clustering Linux sur des machines virtuelles Azure"
  module: Module 02 - Explore the foundations of IaaS for SAP on Azure
---

# AZ 120 Module 2 : Explorer les bases de l’infrastructure as a service (IaaS) pour SAP sur Azure
# Labo 2a : Implémenter le clustering Linux sur des machines virtuelles Azure

Temps estimé : 90 minutes

Toutes les tâches de ce labo sont effectuées dans le portail Azure (y compris la session Cloud Shell Bash)  

   > **Remarque** : Si vous n’utilisez pas Cloud Shell, la machine virtuelle du labo doit avoir Azure CLI installé [**https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows) et inclure un client SSH (par ex. Putty) disponible dans [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

Fichiers de lab : aucun

## Scénario
  
En préparation du déploiement de SAP HANA sur Azure, Adatum Corporation souhaite explorer le processus de mise en œuvre du clustering sur des machines virtuelles Azure exécutant la distribution SUSE de Linux.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

- Provisionner les ressources de calcul Azure nécessaires à la prise en charge des déploiements SAP HANA à haut niveau de disponibilité

- Configurer le système d’exploitation des machines virtuelles Azure exécutant Linux pour prendre en charge une installation SAP HANA à haut niveau de disponibilité

- Provisionner les ressources réseau Azure nécessaires à la prise en charge des déploiements SAP HANA à haut niveau de disponibilité

## Spécifications

- Un abonnement Microsoft Azure avec le nombre suffisant de processeurs virtuels DSv3 (2 x 4) et DSv2 (1 x 1) disponibles

- Un ordinateur de labo avec un navigateur web compatible Azure Cloud Shell et un accès à Azure

## Exercice 1 : Approvisionner les ressources de calcul Azure nécessaires à la prise en charge des déploiements SAP HANA à haut niveau de disponibilité

Durée : 30 minutes

Dans cet exercice, vous allez déployer des composants de calcul d’infrastructure Azure nécessaires pour configurer le clustering Windows. Cela implique de créer deux machines virtuelles Azure exécutant Linux SUSE dans le même groupe à haute disponibilité et de provisionner Azure Bastion.

### Tâche 1 : Déployer des machines virtuelles Azure exécutant Linux SUSE

1. Depuis l’ordinateur de labo, démarrez un navigateur web et accédez au portail Azure à l’adresse **https://portal.azure.com**.

1. Si vous y êtes invité, connectez-vous en utilisant le compte Microsoft professionnel, scolaire ou personnel avec le rôle de propriétaire ou de contributeur de l’abonnement Azure que vous allez utiliser pour ce labo.

1. Dans le portail Azure, utilisez la zone de texte **Rechercher dans les ressources, services et documents** en haut de la page du portail Azure pour rechercher et accéder au panneau **Groupes de placement de proximité**. Dans le panneau **Groupes de placement de proximité**, sélectionnez **+ créer**.

1. Sous l’onglet **Informations de base** du panneau **Créer un groupe de placement de proximité**, spécifiez les paramètres suivants et sélectionnez **Examiner et créer** (laissez les autres valeurs par défaut) :

    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | nom de votre abonnement Azure |
    | Section **Groupe de ressources** | Sélectionnez **Créer**, entrez **az12001a-RG**, puis sélectionnez **OK**. |
    | **Nom du groupe de placement de proximité** | **az12001a-ppg** |
    | **Région** | région Azure où vous disposez de quotas de processeurs virtuels suffisants |
    | **Tailles de machine virtuelle** | **Standard D4s v3** |

   > **Remarque** : Envisagez d’utiliser les régions **USA Est** ou **USA Est 2** pour le déploiement de vos ressources.

1. Sous l’onglet **Vérifier + créer** du panneau **Créer des groupes de placement de proximité**, sélectionnez **Créer**.

   > **Remarque** : Attendez la fin du déploiement. L’opération doit prendre moins d’une minute.

1. Dans le portail Azure, utilisez la zone de texte **Rechercher dans les ressources, services et documents** en haut de la page du portail Azure pour rechercher et accéder au panneau **Machines virtuelles**. Dans le panneau **Machines virtuelles**, sélectionnez **+ Créer**, puis dans le menu déroulant, sélectionnez **Machine virtuelle Azure**.

1. Sous l’onglet **Informations de base** du panneau **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Disques >** (laissez tous les autres paramètres avec leur valeur par défaut) :

    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | nom de votre abonnement Azure |
    | **Groupe de ressources** | le nom du groupe de ressources que vous avez créé précédemment dans cette tâche |
    | **Nom de la machine virtuelle** | **az12001a-vm0** |
    | **Région** | le nom de la région Azure que vous avez choisie lorsque vous avez créé le groupe de placement de proximité |
    | **Options de disponibilité** | **Groupe à haute disponibilité** |
    | **Groupe à haute disponibilité** | un nouveau groupe à haute disponibilité nommé **az12001a-avset** avec 2 domaines d’erreur et 5 domaines de mise à jour |
    | **Type de sécurité** | **Standard** |
    | **Image** | **SUSE Enterprise Linux pour SAP 15 SP5 - BYOS - x64 Gen 2** |
    | **Exécuter avec la remise Azure Spot** | disabled |
    | **Taille** | **Standard_D4s_v3** |
    | **Type d’authentification** | **Mot de passe** |
    | **Nom d’utilisateur** | **student** |
    | **Mot de passe** | un mot de passe complexe de votre choix |
   
    > **Remarque** : Veillez à mémoriser le mot de passe que vous avez spécifié pendant le déploiement. Vous en aurez besoin plus tard dans ce labo.

    > **Remarque** : pour localiser l’image, cliquez sur le lien **Voir toutes les images** dans le panneau **Sélectionner une image**. Dans la zone de texte de recherche, tapez **SUSE Enterprise Linux**. Ensuite, dans la liste des résultats, cliquez sur **SUSE Enterprise Linux pour SAP 15 SP5 - BYOS** et sélectionnez **Génération 2**.

1. Sous l’onglet **Disques** du panneau **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Réseau >** (laissez tous les autres paramètres avec leur valeur par défaut) :

    | Paramètre | Value |
    |   --    |  --   |
    | **Type de disque du système d’exploitation** | **SSD Premium (stockage localement redondant)**  |
    | **Gestion des clés** | **Clé gérée par la plateforme** |

1. Sous l’onglet **Mise en réseau** du volet **Créer une machine virtuelle** de la section **Interface réseau**, directement sous la zone de texte **Réseau virtuel**, sélectionnez **Créer**. 
1. Dans le panneau **Créer un réseau virtuel**, attribuez les paramètres suivants, puis sélectionnez **OK** :

    | Paramètre | Valeur |
    |   --    |  --   |
    | **Nom** | **az12001a-RG-vnet** |
    | **Espace d’adressage** | **192.168.0.0/20** |
    | **Nom du sous-réseau** | **subnet-0** |
    | **Plage d’adresses** | **192.168.0.0/24** |

1. Sous l’onglet **Mise en réseau** du panneau **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Gestion >** (laissez les autres valeurs par défaut) :

    | Paramètre | Valeur |
    |   --    |  --   |
    | **Adresse IP publique** | **Aucun** |
    | **Groupe de sécurité réseau de la carte réseau** | **Avancée**  |
    | **Activer la mise en réseau accélérée** | activé |
    | **Options d’équilibrage de charge** | **Aucun** |
    
    > **Remarque** : ce déploiement a des règles NSG préconfigurées.

1. Sous l’onglet **Gestion** du panneau **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Surveillance >** (laissez tous les autres paramètres avec leur valeur par défaut) :
   
   | Paramètre | Value |
   |   --    |  --   |
   | **Activer gratuitement le plan De base** | disabled |
   | **Activer l’identité managée affectée par le système** | disabled |
   | **Activer l’arrêt automatique** | disabled  |

   > **Remarque** : Le paramètre **plan de base gratuit** n’est pas disponible si vous avez déjà activé Microsoft Defender pour le cloud dans votre abonnement.

1. Sous l’onglet **Surveillance** du panneau **Créer une machine virtuelle**, sélectionnez **Suivant : Options avancées >** (laissez tous les paramètres avec leur valeur par défaut)

1. Sous l’onglet **Options avancées** du panneau **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** (laissez tous les autres paramètres avec leur valeur par défaut) :

   | Paramètre | Valeur |
   |   --    |  --   |
   | **Groupe de placement de proximité** | **az12001a-ppg** |

1. Sous l’onglet **Vérifier + créer** du panneau **Créer une machine virtuelle**, sélectionnez **Créer**.

   > **Remarque** : Attendez la fin du déploiement. Cela devrait prendre moins de 3 minutes.

1. Dans le portail Azure, utilisez la zone de texte **Rechercher dans les ressources, services et documents** en haut de la page du portail Azure pour rechercher et accéder au panneau **Machines virtuelles**. Dans le panneau **Machines virtuelles**, sélectionnez **+ Créer**, puis dans le menu déroulant, sélectionnez **Machine virtuelle Azure**.

1. Sous l’onglet **Informations de base** du panneau **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Disques >** (laissez tous les autres paramètres avec leur valeur par défaut) :
   
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | nom de votre abonnement Azure |
    | **Groupe de ressources** | le nom du groupe de ressources que vous avez créé précédemment dans cette tâche |
    | **Nom de la machine virtuelle** | **az12001a-vm1** |
    | **Région** | le nom de la région Azure que vous avez choisie lorsque vous avez créé le groupe de placement de proximité |
    | **Options de disponibilité** | **Groupe à haute disponibilité** |
    | **Groupe à haute disponibilité** | **az12001a-avset** |
    | **Type de sécurité** | **Standard** |
    | **Image** | **SUSE Enterprise Linux pour SAP 15 SP5 - BYOS - x64 Gen 2** |
    | **Exécuter avec la remise Azure Spot** | disabled |
    | **Taille** | **Standard_D4s_v3** |
    | **Type d’authentification** | **Mot de passe** |
    | **Nom d’utilisateur** | **student** |
    | **Mot de passe** | le même mot de passe que vous avez spécifié lors du premier déploiement |
   
    > **Remarque** : pour localiser l’image, cliquez sur le lien **Voir toutes les images** dans le panneau **Sélectionner une image**. Dans la zone de texte de recherche, tapez **SUSE Enterprise Linux**. Ensuite, dans la liste des résultats, cliquez sur **SUSE Enterprise Linux pour SAP 15 SP5 - BYOS** et sélectionnez **Génération 2**.

1. Sous l’onglet **Disques** du panneau **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Réseau >** (laissez tous les autres paramètres avec leur valeur par défaut) :

    | Paramètre | Value |
    |   --    |  --   |
    | **Type de disque du système d’exploitation** | **SSD Premium**  |
    | **Gestion des clés** | **Clé gérée par la plateforme** |

1. Sous l’onglet **Réseau** du panneau **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Gestion >** (laissez tous les autres paramètres avec leur valeur par défaut) :
    
    | Paramètre | Valeur |
    |   --    |  --   |
    | **Réseau virtuel** | **az12001a-RG-vnet** |
    | **Sous-réseau** | **subnet-0 (192.168.0.0/24)** |
    | **Adresse IP publique** | **Aucun** |
    | **Groupe de sécurité réseau de la carte réseau** | **Avancée**  |
    | **Activer la mise en réseau accélérée** | **Activé** |
    | **Options d’équilibrage de charge** | **Aucun** |

1. Sous l’onglet **Gestion** du panneau **Créer une machine virtuelle**, spécifiez les paramètres suivants et sélectionnez **Suivant : Surveillance >** (laissez tous les autres paramètres avec leur valeur par défaut) :
    
   | Paramètre | Value |
   |   --    |  --   |
   | **Activer gratuitement le plan De base** | disabled |
   | **Activer l’identité managée affectée par le système** | disabled |
   | **Activer l’arrêt automatique** | disabled |

   > **Remarque** : Le paramètre **plan de base gratuit** n’est pas disponible si vous avez déjà sélectionné le plan Azure Security Center.

1.  Sous l’onglet **Surveillance** du panneau **Créer une machine virtuelle**, sélectionnez **Suivant : Options avancées >** (laissez tous les paramètres avec leur valeur par défaut)

1.  Sous l’onglet **Avancé** du panneau **Créer une machine virtuelle**, vérifiez que le paramètre suivant est configuré et sélectionnez **Examiner et créer** (laissez tous les autres paramètres avec leur valeur par défaut) :
    
   | Paramètre | Valeur |
   |   --    |  --   |
   | **Groupe de placement de proximité** | **az12001a-ppg** |

1.  Sous l’onglet **Vérifier + créer** du panneau **Créer une machine virtuelle**, sélectionnez **Créer**.

   > **Remarque** : Attendez la fin du déploiement. Cela devrait prendre moins de 3 minutes.


### Tâche 2 : Créer et configurer des disques de machines virtuelles Azure

1. Dans le portail Azure, démarrez une session Bash dans Cloud Shell. 

   > **Remarque** : Si c’est la première fois que vous lancez Cloud Shell dans l’abonnement Azure actuel, vous êtes invité à créer un partage de fichiers Azure pour conserver les fichiers Cloud Shell. Dans ce cas, acceptez les valeurs par défaut afin de créer un compte de stockage dans un groupe de ressources généré automatiquement.

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `RESOURCE_GROUP_NAME` sur le nom du groupe de ressources contenant les ressources que vous avez provisionnées dans la tâche précédente :

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer le premier ensemble de 8 disques managés que vous allez attacher à la première machine virtuelle Azure que vous avez déployée dans la tâche précédente :

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer le deuxième ensemble de 8 disques managés que vous allez attacher à la deuxième machine virtuelle Azure que vous avez déployée dans la tâche précédente :

   ```cli
   for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm1-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
   ```

1. Fermez le volet Cloud Shell.

1. Dans le portail Azure, accédez au panneau de la première machine virtuelle Azure que vous avez provisionnée dans la tâche précédente (**az12001a-vm0**).

1. Dans le panneau **az12001a-vm0**, accédez au panneau **az12001a-vm0 \| Disques**.

1. Dans le panneau **az12001a-vm0 \| Disques**, sélectionnez **Attacher des disques existants** et attachez un disque de données avec les paramètres suivants à az12001a-vm0 :
    
   | Paramètre | Valeur |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nom du disque** | **az12001a-vm0-DataDisk0** |
   | **Groupe de ressources** | le nom du groupe de ressources que vous avez utilisé précédemment dans cette tâche |
   | **MISE EN CACHE DE L’HÔTE** | **Lecture seule** |

2. Répétez l’étape précédente pour attacher les 7 disques restants avec le préfixe **az12001a-vm0-DataDisk** (pour le total des 8). Affectez le numéro d’unité logique (LUN) qui correspond au dernier caractère du nom du disque. Définissez la MISE EN CACHE DE L’HÔTE du disque avec le numéro LUN **1** en **lecture seule**. Pour tous les autres, définissez la MISE EN CACHE DE L’HÔTE sur **Aucune**.

3. Enregistrez vos modifications. 

4. Dans le portail Azure, accédez au panneau de la deuxième machine virtuelle Azure que vous avez provisionnée dans la tâche précédente (**az12001a-vm1**).

5. Dans le panneau **az12001a-vm1**, accédez au panneau **az12001a-vm1 \| Disques**.

6. À partir du panneau **az12001a-vm1 \| Disques**, attachez des disques de données avec les paramètres suivants à az12001a-vm1 :
    
   | Paramètre | Valeur |
   |   --    |  --   |
   | **LUN** | **0** |
   | **Nom du disque** | **az12001a-vm1-DataDisk0** |
   | **Groupe de ressources** | le nom du groupe de ressources que vous avez utilisé précédemment dans cette tâche |
   | **MISE EN CACHE DE L’HÔTE** | **Lecture seule** |

7. Répétez l’étape précédente pour attacher les 7 disques restants avec le préfixe **az12001a-vm1-DataDisk** (pour le total des 8). Affectez le numéro d’unité logique (LUN) qui correspond au dernier caractère du nom du disque. Définissez la MISE EN CACHE DE L’HÔTE du disque avec le numéro LUN **1** en **lecture seule**. Pour tous les autres, définissez la MISE EN CACHE DE L’HÔTE sur **Aucune**.

8. Enregistrez les changements apportés. 

#### Tâche 3 : Provisionner Azure Bastion 

> **Remarque** : Azure Bastion autorise la connexion aux machines virtuelles Azure (que vous avez déployées dans la tâche précédente de cet exercice) sans utiliser de points de terminaison publics, tout en fournissant une protection contre les attaques par force brute qui ciblent les informations d’identification au niveau du système d’exploitation.

> **Remarque** : Pout utiliser Azure Bastion, vérifiez que la fonctionnalité de fenêtre contextuelle de votre navigateur est activée.

1. Dans la fenêtre du navigateur affichant le Portail Azure, ouvrez un autre onglet, et, dans l’onglet du navigateur, accédez au [**Portail Azure**](https://portal.azure.com).
1. Dans le portail Azure, ouvrez le volet **Cloud Shell** en sélectionnant l’icône de barre d’outils juste à droite de la zone de texte de recherche.
1. Dans la session PowerShell du volet Cloud Shell, exécutez la commande suivante pour ajouter un sous-réseau nommé **AzureBastionSubnet** au réseau virtuel nommé **az12001a-RG-vnet** que vous avez créé précédemment dans cet exercice :

   ```powershell
   $resourceGroupName = 'az12001a-RG'
   $vnet = Get-AzVirtualNetwork -ResourceGroupName $resourceGroupName -Name 'az12001a-RG-vnet'
   $subnetConfig = Add-AzVirtualNetworkSubnetConfig `
     -Name 'AzureBastionSubnet' `
     -AddressPrefix 192.168.15.0/24 `
     -VirtualNetwork $vnet
   $vnet | Set-AzVirtualNetwork
   ```

1. Fermez le volet Cloud Shell.
1. Dans le portail Azure, recherchez et sélectionnez **Bastions**, puis dans le panneau **Bastions**, sélectionnez **+ Créer**.
1. Sous l’onglet **De base** du panneau **Créer un bastion**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** :

   |Paramètre|Valeur|
   |---|---|
   |Abonnement|le nom de l’abonnement Azure que vous utilisez dans ce labo|
   |Resource group|**az12001a-RG**|
   |Nom|**az12001a-bastion**|
   |Région|la même région Azure que celle dans laquelle vous avez déployé les ressources lors des tâches précédentes de cet exercice|
   |Niveau|**De base**|
   |Réseau virtuel|**az12001a-RG-vnet**|
   |Sous-réseau|**AzureBastionSubnet (192.168.15.0/24)**|
   |Adresse IP publique|**Création**|
   |Nom de l’IP publique|**az12001a-RG-vnet-ip**|

1. Sous l’onglet **Vérifier + créer** du panneau **Créer un bastion**, sélectionnez **Créer** :

   > **Remarque** : Attendez la fin du déploiement avant de passer à la tâche suivante de cet exercice. Le déploiement peut prendre environ 5 minutes.

> **Result** : Une fois que vous avez terminé cet exercice, vous avez provisionné les ressources de calcul Azure nécessaires pour prendre en charge les déploiements SAP HANA à haute disponibilité.


## Exercice 2 : Configurer le système d’exploitation des machines virtuelles Azure exécutant Linux pour prendre en charge une installation SAP HANA à haut niveau de disponibilité

Durée : 30 minutes

Dans cet exercice, vous allez configurer le système d’exploitation et le stockage sur des machines virtuelles Azure exécutant SUSE Linux Enterprise Server pour prendre en charge les installations en cluster de SAP HANA.

### Tâche 1 : Se connecter à des machines virtuelles Azure Linux

1. À partir de votre ordinateur de labo, dans le portail Azure, recherchez et sélectionnez **Machines virtuelles**, puis dans le panneau **Machines virtuelles**, sélectionnez l’entrée **az12001a-vm0**. Le panneau **az12001a-vm0** s’ouvre.

1. Dans le panneau **az12001a-vm0**, sélectionnez **Se connecter**. Dans le menu déroulant, sélectionnez **Se connecter via Bastion**. Sous l’onglet **Bastion** d’**az12001a-vm0**, laissez le **Type d’authentification** défini sur **Mot de passe de la machine virtuelle**, indiquez les informations d’identification que vous avez définies lors du déploiement de la machine virtuelle **az12001a-vm0**, laissez la case **Ouvrir dans un nouvel onglet de navigateur** cochée, puis sélectionnez **Se connecter**.

1. Répétez les deux étapes précédentes pour vous connecter via Bastion à la machine virtuelle Azure **az12001a-vm1**.

### Tâche 2 : Configurer le stockage des machines virtuelles Azure exécutant Linux

1. Dans la session Bastion sur la machine virtuelle Azure **az12001a-vm0**, exécutez la commande suivante pour élever les privilèges : 

   ```sh
   sudo su -
   ```

1. Exécutez la commande suivante pour identifier les correspondances entre les appareils nouvellement attachés et leur numéro d’unité logique :
   
   ```sh
   lsscsi
   ```

1. Créez des volumes physiques pour 6 disques de données (sur 8) en exécutant :
   
   ```sh
   pvcreate /dev/sdc
   pvcreate /dev/sdd
   pvcreate /dev/sde
   pvcreate /dev/sdf
   pvcreate /dev/sdg
   pvcreate /dev/sdh
   ```

1. Créez des groupes de volumes en exécutant :
   
   ```sh
   vgcreate vg_hana_data /dev/sdc /dev/sdd
   vgcreate vg_hana_log /dev/sde /dev/sdf
   vgcreate vg_hana_backup /dev/sdg /dev/sdh
   ```

1. Créez des volumes logiques en exécutant :

   ```sh
   lvcreate -l 100%FREE -n hana_data vg_hana_data
   lvcreate -l 100%FREE -n hana_log vg_hana_log
   lvcreate -l 100%FREE -n hana_backup vg_hana_backup
   ```

   > **Remarque** : Nous créons un volume logique unique par groupe de volumes.

1. Formatez les volumes logiques en exécutant :

   ```sh
   mkfs.xfs /dev/vg_hana_data/hana_data -m crc=1
   mkfs.xfs /dev/vg_hana_log/hana_log -m crc=1
   mkfs.xfs /dev/vg_hana_backup/hana_backup -m crc=1
   ```

   > **Remarque** : À compter de SUSE Linux Enterprise Server 12, vous avez la possibilité d’utiliser le nouveau format sur disque (v5) du système de fichiers XFS, qui offre des sommes automatiques de métadonnées XFS, une prise en charge de type de fichier et une limite plus élevée du nombre de listes de contrôle d’accès par fichier. Le nouveau format s’applique automatiquement lors de l’utilisation de YaST pour créer des systèmes de fichiers XFS. Pour créer un système de fichiers XFS dans l’ancien format pour des raisons de compatibilité, utilisez la commande mkfs.xfs sans l’option `-m crc=1`. 

1. Partitionnez le disque **/dev/sdi** en exécutant :

   ```sh
   fdisk /dev/sdi
   ```

1. Lorsque vous y êtes invité, tapez successivement `n`, `p`, `1` (suivis de la touche **Entrée** à chaque fois). Appuyez deux fois sur la touche **Entrée** et tapez `w` pour terminer l’écriture.

1. Partitionnez le disque **/dev/sdj** en exécutant :

   ```sh
   fdisk /dev/sdj
   ```

1. Lorsque vous y êtes invité, tapez successivement `n`, `p`, `1` (suivis de la touche **Entrée** à chaque fois). Appuyez deux fois sur la touche **Entrée** et tapez `w` pour terminer l’écriture.

1. Formatez la partition nouvellement créée en exécutant (tapez `y` et appuyez sur la touche **Entrée** lorsque vous êtes invité à confirmer) :

   ```sh
   mkfs.xfs /dev/sdi -m crc=1 -f
   mkfs.xfs /dev/sdj -m crc=1 -f
   ```

1. Créez les répertoires qui serviront de points de montage en exécutant :

   ```sh
   mkdir -p /hana/data
   mkdir -p /hana/log
   mkdir -p /hana/backup
   mkdir -p /hana/shared
   mkdir -p /usr/sap
   ```

1. Affichez les ID des volumes logiques en exécutant :

   ```sh
   blkid
   ```

   > **Remarque** : Identifiez les valeurs **UUID** associées aux groupes de volumes et partitions nouvellement créés, notamment **/dev/sdi** (à utiliser pour **/hana/shared**) et **dev/sdj** (à utiliser pour **/usr/sap**).


1. Ouvrez **/etc/fstab** dans l’éditeur vi (vous êtes libre d’utiliser un autre éditeur) en exécutant :

   ```sh
   vi /etc/fstab
   ```

   > **Remarque** : si vous utilisez l’éditeur vi, appuyez sur i **** pour entrer dans le mode INSERT.

   ![indicateur de mode INSERT de l’éditeur vi](../media/az120-lab01-vieditor-insert.png)

1. Dans l’éditeur, ajoutez les entrées suivantes à **/etc/fstab** (où `\<UUID of /dev/vg\_hana\_data-hana\_data\>`, `\<UUID of /dev/vg\_hana\_log-hana\_log\>`, `\<UUID of /dev/vg\_hana\_backup-hana\_backup\>`, `\<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)\>` et `\<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)\>` représentent les ID que vous avez identifiées à l’étape précédente) :

   ```sh
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_data-hana_data> /hana/data xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_log-hana_log> /hana/log xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_backup-hana_backup> /hana/backup xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)> /hana/shared xfs  defaults,nofail  0  2
   /dev/disk/by-uuid/<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)> /usr/sap xfs  defaults,nofail  0  2
   ```

1. Enregistrez les modifications et fermez l'éditeur.

1. Montez les nouveaux volumes en exécutant :

   ```sh
   mount -a
   ```

1. Vérifiez que le montage a réussi en exécutant :

   ```sh
   df -h
   ```

   ![sortie df-h](../media/az120-lab01-df-output.png)
1. Quittez le mode privilégié en exécutant :

   ```sh
   exit
   ```

1. Basculez vers la session Bastion sur **az12001a-vm1** et répétez toutes les étapes de cette tâche pour configurer le stockage sur **az12001a-vm1**.


### Tâche 3 : Activer l’accès SSH sans mot de passe inter-nœuds

1. Dans la session Bastion sur **az12001a-vm1**, générez une paire de clés SSH sans phrase secrète en exécutant :

      ```sh
   ssh-keygen -trsa
   ```

1. Lorsque vous y êtes invité, appuyez trois fois sur la touche **Entrée**.

1. Copiez la clé publique de la paire de clés nouvellement générée sur **az12001a-vm0** en exécutant :

      ```sh
   ssh-copy-id -i /home/student/.ssh/id_rsa.pub student@az12001a-vm0
   ```

1. Lorsque vous êtes invité à confirmer si vous souhaitez continuer à vous connecter, entrez **oui** et appuyez sur la touche **Entrée**.

1. Lorsque vous êtes invité à vous authentifier, entrez le mot de passe que vous avez défini lors de l’approvisionnement d’**az12001a-vm0** précédemment dans ce labo.

1. Basculez vers la session Bastion sur la machine virtuelle Azure **az12001a-vm0**.

1. Dans la session Bastion sur **az12001a-vm0**, générez une paire de clés SSH sans phrase secrète en exécutant :

      ```sh
   ssh-keygen -trsa
   ```

1. Lorsque vous y êtes invité, appuyez trois fois sur la touche **Entrée**.

1. Copiez la clé publique de la paire de clés nouvellement générée sur **az12001a-vm1** en exécutant :

      ```sh
   ssh-copy-id -i /home/student/.ssh/id_rsa.pub student@az12001a-vm1
   ```

1. Lorsque vous êtes invité à confirmer si vous souhaitez continuer à vous connecter, entrez **oui** et appuyez sur la touche **Entrée**.

1. Lorsque vous êtes invité à vous authentifier, entrez le mot de passe que vous avez défini lors de l’approvisionnement d’**az12001a-vm1** précédemment dans ce labo.

1. Pour vérifier que la configuration a réussi, dans la session Bastion sur la machine virtuelle **az12001a-vm0**, établissez une session SSH en tant qu’**étudiant** sur **az12001a-vm1** en exécutant : 

   ```sh
   ssh student@az12001a-vm1
   ```

1. Vérifiez que vous n’êtes pas invité à entrer le mot de passe.

1. Fermez la session de **az12001a-vm0** vers **az12001a-vm1** en exécutant : 

   ```sh
   exit
   ```

1. Basculez vers la session Bastion sur la machine virtuelle **az12001a-vm1**.

1. Dans la session Bastion sur **az12001a-vm1**, établissez une session SSH en tant qu’**étudiant ** sur **az12001a-vm0** en exécutant : 

   ```sh
   ssh student@az12001a-vm0
   ```

1. Vérifiez que vous n’êtes pas invité à entrer le mot de passe.

1. Fermez la session de **az12001a-vm1** vers **az12001a-vm0** en exécutant : 

   ```sh
   exit
   ```

> **Result** : Une fois que vous avez terminé cet exercice, vous avez configuré le système d’exploitation des machines virtuelles Azure exécutant Linux pour prendre en charge une installation SAP HANA à haut niveau de disponibilité


## Exercice 3 : Approvisionner les ressources réseau Azure nécessaires à la prise en charge des déploiements SAP HANA à haut niveau de disponibilité

Durée : 30 minutes

Dans cet exercice, vous allez implémenter des équilibreurs de charge Azure pour prendre en charge les installations en cluster de SAP HANA.


### Tâche 1 : Configurer des machines virtuelles Azure pour faciliter la configuration de l’équilibrage de charge.

1. Dans le portail Azure, accédez au panneau de la machine virtuelle Azure **az12001a-vm0**.

1. Dans le panneau **az12001a-vm0**, accédez au panneau **az12001a-vm0 \| Réseau**. 

1. Dans le panneau **az12001a-vm0 \| Réseau**, sélectionnez l’entrée représentant l’interface réseau de la machine virtuelle az12001a-vm0. 

1. Dans le panneau de l’interface réseau de la machine virtuelle az12001a-vm0, accédez à son panneau des configurations IP pour afficher son panneau **ipconfig1**.

1. Dans le panneau **ipconfig1**, définissez l’attribution d’adresse IP privée sur **Statique** et enregistrez la modification.

1. Dans le portail Azure, accédez au panneau de la machine virtuelle Azure **az12001a-vm1**.

1. Dans le panneau **az12001a-vm1**, accédez au panneau **az12001a-vm1 \| Réseau**. 

1. Dans le panneau **az12001a-vm1 \| Réseau**, accédez à l’interface réseau de la machine virtuelle az12001a-vm1. 

1. Dans le panneau de l’interface réseau de la machine virtuelle az12001a-vm1, accédez à son panneau des configurations IP pour afficher son panneau **ipconfig1**.

1. Dans le panneau **ipconfig1**, définissez l’attribution d’adresse IP privée sur **Statique** et enregistrez la modification.


### Tâche 2 : Créer et configurer des équilibreurs de charge Azure qui gèrent le trafic entrant

1. Dans le portail Azure, utilisez la zone de texte **Rechercher dans les ressources, services et documents** en haut de la page du portail Azure pour rechercher et accéder au panneau **Équilibreurs de charge**, puis dans le panneau **Équilibreurs de charge**, sélectionnez **+ Créer**.

1. Sous l’onglet **Informations de base**, du panneau **Créer un équilibreur de charge**, spécifiez les paramètres suivants et sélectionnez **Vérifier + créer** (laissez les autres avec leur valeur par défaut) :
    
   | Paramètre | Valeur |
   |   --    |  --   |
   | **Abonnement** | nom de votre abonnement Azure |
   | **Groupe de ressources** | **az12001a-RG** |
   | **Nom** | **az12001a-lb0** |
   | **Région** | la même région Azure que celle où vous avez déployé les machines virtuelles Azure dans le premier exercice de ce labo |
   | **Référence (SKU)** | **Standard** |
   | **Type** | **Interne** |

1. Cliquez sur **Suivant : Configuration IP frontale**. À l’écran **Configuration IP frontale**, cliquez sur **Ajouter une configuration IP frontale**, puis sur **Ajouter**.
    
   | Paramètre | Valeur |
   |   --    |  --   |
   | **Nom** | **frontend1** |
   | **Réseau virtuel** | **az12001a-RG-vnet** |
   | **Sous-réseau** | **subnet-0** |
   | **Affectation d’adresses IP** | **Statique** |
   | **Adresse IP** | **192.168.0.240** |
   | **Zone de disponibilité** | **Redondant dans une zone** |

1. Sélectionnez **Examiner + créer**, puis sélectionnez **Créer**.

   > **Remarque** : Attendez que l’équilibreur de charge soit provisionné. L’opération doit prendre moins d’une minute. 

1. Dans le portail Azure, accédez au panneau affichant les propriétés de l’équilibreur de charge **az12001a-lb0** qui vient d’être provisionné. 

1. Dans le panneau **az12001a-lb0**, sélectionnez **Pools principaux**, sélectionnez **+ Ajouter**, puis dans**Ajouter un pool principal**, spécifiez les paramètres suivants (laissez les autres avec leur valeur par défaut) :
    
   | Paramètre | Valeur |
   |   --    |  --   |
   | **Nom** | **az12001a-lb0-bepool** |
   | **Réseau virtuel** | **az12001a-RG-vnet** |
   | **Configuration du pool de back-ends** | **Adresse IP** |
   | **Adresse IP** | **192.168.0.4** Nom de la ressource : **az12001a-vm0** |
   | **Adresse IP** | **192.168.0.5** Nom de la ressource : **az12001a-vm1** |

1. Dans le panneau **az12001a-lb0**, sélectionnez **Sondes d’intégrité**, sélectionnez **+ Ajouter**, puis dans le panneau **Ajouter une sonde d’intégrité**, spécifiez les paramètres suivants (laissez les autres avec leur valeur par défaut) :
    
   | Paramètre | Valeur |
   |   --    |  --   |
   | **Nom** | **az12001a-lb0-hprobe** |
   | **Protocole** | **TCP** |
   | **Port** | **62500** |
   | **Intervalle** | **5** *secondes* |

1. Dans le **panneau az12001a-lb0** , sélectionnez **Règles** d’équilibrage de charge, sélectionnez **+ Ajouter** et, dans le **panneau Ajouter une règle** d’équilibrage de charge, spécifiez les paramètres suivants (laissez d’autres personnes avec leurs valeurs par défaut) :
    
   | Paramètre | Valeur |
   |   --    |  --   |
   | **Nom** | **az12001a-lb0-lbruleAll** |
   | **Version de l’adresse IP** | **IPv4** |
   | **Frontend IP address (Adresse IP frontale)** | **192.168.0.240 (LoadBalancerFrontEnd)** |
   | **Ports HA** | activé |
   | **Pool back-end** | **az12001a-lb0-bepool (2 machines virtuelles)** |
   | **Sonde d’intégrité** | **az12001a-lb0-hprobe (TCP:62500)** |
   | **Persistance de session** | **Aucun** |
   | **Délai d’inactivité (minutes).** | **4** |
   | **Réinitialisation du protocole TCP** | disabled |
   | **Adresse IP flottante (retour direct du serveur)** | activé |

### Tâche 3 : Créer et configurer des équilibreurs de charge Azure qui gèrent le trafic sortant

1. Dans le portail Azure, démarrez une session Bash dans Cloud Shell. 

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `RESOURCE_GROUP_NAME` sur le nom du groupe de ressources contenant les ressources que vous avez provisionnées dans le premier exercice de ce labo :

   ```cli
   RESOURCE_GROUP_NAME='az12001a-RG'
   ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer l’adresse IP publique à utiliser par le deuxième équilibreur de charge :

   ```cli
   LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

   PIP_NAME='az12001a-lb1-pip'

   az network public-ip create --resource-group $RESOURCE_GROUP_NAME --name $PIP_NAME --sku Standard --location $LOCATION
   ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer le deuxième équilibreur de charge :

   ```cli
   LB_NAME='az12001a-lb1'

   LB_BE_POOL_NAME='az12001a-lb1-bepool'

   LB_FE_IP_NAME='az12001a-lb1-fe'

   az network lb create --resource-group $RESOURCE_GROUP_NAME --name $LB_NAME --sku Standard --backend-pool-name $LB_BE_POOL_NAME --frontend-ip-name $LB_FE_IP_NAME --location $LOCATION --public-ip-address $PIP_NAME
   ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer la règle de trafic sortant du deuxième équilibreur de charge :

   ```cli
   LB_RULE_OUTBOUND='az12001a-lb1-ruleoutbound'

   az network lb outbound-rule create --resource-group $RESOURCE_GROUP_NAME --lb-name $LB_NAME --name $LB_RULE_OUTBOUND --frontend-ip-configs $LB_FE_IP_NAME --protocol All --idle-timeout 4 --outbound-ports 1000 --address-pool $LB_BE_POOL_NAME
   ```

1. Fermez le volet Cloud Shell.

1. Dans le portail Azure, accédez au panneau affichant les propriétés de l’équilibreur de charge Azure **az12001a-lb1** qui vient d’être créé.

1. Dans le panneau **az12001a-lb1**, cliquez sur **Pools principaux**.

1. Dans le panneau **az12001a-lb1\| Pools principaux**, cliquez sur **az12001a-lb1-bepool**.

1. Dans le panneau **az12001a-lb1-bepool**, spécifiez les paramètres suivants et cliquez sur **Enregistrer** :
   
   | Paramètre | Valeur |
   |   --    |  --   |
   | **Réseau virtuel** | **az12001a-rg-vnet (2 machines virtuelles)** |
   | **Machine virtuelle** | **az12001a-vm0**  Configuration IP : **ipconfig1 (192.168.0.4)** |
   | **Machine virtuelle** | **az12001a-vm1**  Configuration IP : **ipconfig1 (192.168.0.5)** |

> **Result** : Une fois que vous avez terminé cet exercice, vous avez provisionné les ressources réseau Azure nécessaires pour prendre en charge les déploiements SAP HANA à haute disponibilité.


## Exercice 4 : Supprimer des ressources de lab

Durée : 10 minutes

Dans cet exercice, vous allez supprimer les ressources provisionnées dans ce labo.

#### Tâche 1 : Lister les groupes de ressources à supprimer

1. En haut du portail, cliquez sur l’icône **Cloud Shell** pour ouvrir le volet Cloud Shell, puis choisissez Bash comme interpréteur de commandes.

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `RESOURCE_GROUP_PREFIX` sur le préfixe du nom du groupe de ressources contenant les ressources que vous avez approvisionnées dans ce labo :

   ```cli
   RESOURCE_GROUP_PREFIX='az12001a-'
   ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour lister les groupes de ressources créés dans ce labo :

   ```cli
   az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv
   ```

1. Vérifiez que la sortie contient uniquement le groupe de ressources créé dans ce labo. Ce groupe de ressources et toutes ses ressources sera supprimé dans la prochaine tâche.

#### Tâche 2 : Supprimer des groupes de ressources

1. Dans le volet Cloud Shell, exécutez la commande suivante pour supprimer le groupe de ressources et ses ressources.

   ```cli
   az group list --query "[?starts_with(name,'$RESOURCE_GROUP_PREFIX')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. Fermez le volet Cloud Shell.

> **Result** : Une fois cet exercice terminé, vous avez supprimé les ressources utilisées dans ce labo.
