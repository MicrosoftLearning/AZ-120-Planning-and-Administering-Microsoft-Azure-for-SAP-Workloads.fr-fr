---
lab:
  title: 04a - Implémenter une architecture SAP sur des machines virtuelles Azure exécutant Linux
  module: Module 04 - Deploy SAP on Azure
---

# AZ 120 Module 4 : Déployer SAP sur Azure
# Labo 4a : Implémenter une architecture SAP sur des machines virtuelles Azure exécutant Linux

Temps estimé : 100 minutes

Toutes les tâches de ce labo sont effectuées depuis le portail Azure (y compris la session Bash Cloud Shell)  

   > **Remarque** : Si vous n’utilisez pas Cloud Shell, Azure CLI doit être installé sur la machine virtuelle du labo [**https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows**](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows).

Fichiers de lab : aucun

## Scénario
  
En préparation au déploiement de SAP NetWeaver sur Azure, Adatum Corporation souhaite mettre en œuvre une démo qui illustrera l’implémentation à haute disponibilité de SAP NetWeaver sur des machines virtuelles Azure exécutant la distribution SUSE de Linux.

## Objectifs
  
À la fin de ce lab, vous serez en mesure de :

-   Approvisionner les ressources Azure nécessaires à la prise en charge d’un déploiement SAP NetWeaver à haute disponibilité

-   Configurer le système d’exploitation des machines virtuelles Azure exécutant Linux pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité

-   Configurer un clustering sur des machines virtuelles Azure exécutant Linux pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité

## Spécifications

-   Un abonnement Microsoft Azure avec le nombre suffisant de processeurs virtuels Dsv3 disponibles (quatre machines virtuelles Standard_D2s_v3 avec 2 processeurs virtuels chacune et deux machines virtuelles Standard_D4s_v3 avec 4 processeurs virtuels chacune) dans une région Azure prenant en charge les zones de disponibilité

-   Un ordinateur de labo avec un navigateur web compatible Azure Cloud Shell et un accès à Azure


## Exercice 1 : Approvisionner les ressources Azure nécessaires à la prise en charge des déploiements SAP NetWeaver à haut niveau de disponibilité

Durée : 30 minutes

Dans cet exercice, vous allez déployer les composants de calcul d’infrastructure Azure nécessaires à la configuration du clustering Linux. Cela impliquera la création d’une paire de machines virtuelles Azure exécutant Linux SUSE dans le même groupe à haute disponibilité.

### Tâche 1 : Créer un réseau virtuel qui hébergera un déploiement SAP NetWeaver à haute disponibilité.

1. Depuis l’ordinateur du labo, démarrez un navigateur web et accédez au portail Azure à l’adresse **https://portal.azure.com**.

1. Si vous y êtes invité, connectez-vous au compte Microsoft professionnel, scolaire ou personnel avec le rôle de propriétaire ou contributeur dans l’abonnement Azure que vous utiliserez pour ce labo et avec le rôle d’administrateur général dans le locataire Azure AD associé à votre abonnement.

1. Dans le portail Azure, démarrez une session Bash dans Cloud Shell. 

    > **Remarque** : Si c’est la première fois que vous lancez Cloud Shell dans l’abonnement Azure actuel, vous êtes invité à créer un partage de fichiers Azure pour conserver les fichiers Cloud Shell. Dans ce cas, acceptez les valeurs par défaut, ce qui entraînera la création d’un compte de stockage dans un groupe de ressources généré automatiquement.

1. Dans le volet Cloud Shell, exécutez la commande suivante pour spécifier la région Azure qui prend en charge les zones de disponibilité et dans laquelle vous souhaitez créer des ressources pour ce labo (remplacez `<region>` par le nom de la région Azure qui prend en charge les zones de disponibilité) :

    ```cli
    LOCATION='<region>'
    ```

    > **Remarque** : Envisagez d’utiliser les régions **USA Est** ou **USA Est 2** pour le déploiement de vos ressources. 

    > **Remarque** : Veillez à utiliser la notation appropriée pour la région Azure (nom abrégé sans espace, par exemple **usaest** plutôt que **USA Est**)

    > **Remarque** : Pour identifier les régions Azure qui prennent en charge des zones de disponibilité, reportez-vous à [https://docs.microsoft.com/en-us/azure/availability-zones/az-region](https://docs.microsoft.com/en-us/azure/availability-zones/az-region)

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `RESOURCE_GROUP_NAME` sur le nom du groupe de ressources contenant les ressources que vous avez approvisionnées dans la tâche précédente :

    ```cli
    RESOURCE_GROUP_NAME='az12003a-sap-RG'
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer un groupe de ressources dans la région que vous avez spécifiée :

    ```cli
    az group create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION
    ```

1. Dans le volet Cloud Shell, exécutez la commande suivante pour créer un réseau virtuel avec un seul sous-réseau dans le groupe de ressources que vous avez créé :

    ```cli
    VNET_NAME='az12003a-sap-vnet'

    VNET_PREFIX='10.3.0.0/16'

    SUBNET_NAME='sapSubnet'

    SUBNET_PREFIX='10.3.0.0/24'

    az network vnet create --resource-group $RESOURCE_GROUP_NAME --location $LOCATION --name $VNET_NAME --address-prefixes $VNET_PREFIX --subnet-name $SUBNET_NAME --subnet-prefixes $SUBNET_PREFIX
    ```

1.  Dans le volet Cloud Shell, exécutez la commande suivante pour identifier l’ID de ressource du sous-réseau du réseau virtuel nouvellement créé et le stocker dans la variable SUBNET_ID :

    ```cli
    SUBNET_ID=$(az network vnet subnet list --resource-group $RESOURCE_GROUP_NAME --vnet-name $VNET_NAME --query "[?name == '$SUBNET_NAME'].id" --output tsv)
    ```

### Tâche 2 : Déployer un modèle Bicep qui approvisionne des machines virtuelles Azure exécutant Linux SUSE qui hébergera un déploiement SAP NetWeaver à haute disponibilité

1.  Sur l’ordinateur du labo, dans le volet Cloud Shell, exécutez les commandes suivantes pour créer un clone superficiel du référentiel hébergeant le modèle Bicep que vous allez utiliser pour le déploiement de deux machines virtuelles Azure qui hébergeront une installation à haute disponibilité de SAP HANA, et définissez le répertoire actif sur l’emplacement de ce modèle et de son fichier de paramètres :

    ```
    cd $HOME
    rm ./azure-quickstart-templates -rf
    git clone --depth 1 https://github.com/polichtm/azure-quickstart-templates
    cd ./azure-quickstart-templates/application-workloads/sap/sap-3-tier-marketplace-image-md/
    ```

1.  Dans le volet Cloud Shell, exécutez les commandes suivantes pour définir le nom du compte d’utilisateur administratif et son mot de passe :

    ```
    ADMINUSERNAME='student'
    ADMINPASSWORD='Pa55w.rd1234'
    ```

1.  Dans le volet Cloud Shell, exécutez la commande suivante pour identifier les ressources qui seront incluses dans le déploiement à venir :

    ```
    DEPLOYMENT_NAME='az1203a-'$RANDOM
    az deployment group what-if --name $DEPLOYMENT_NAME --resource-group $RESOURCE_GROUP_NAME --template-file ./main.bicep --parameters ./azuredeploy.parameters03a.json --parameters adminUsername=$ADMINUSERNAME adminPasswordOrKey=$ADMINPASSWORD subnetId=$SUBNET_ID
    ```

1.  Évaluez la sortie de la commande et vérifiez qu’elle n’inclut pas d’erreurs (ignorez les avertissements). Ensuite, dans le volet Cloud Shell, exécutez la commande suivante pour démarrer le déploiement :

    ```
    DEPLOYMENT_NAME='az1203a-'$RANDOM
    az deployment group create --name $DEPLOYMENT_NAME --resource-group $RESOURCE_GROUP_NAME --template-file ./main.bicep --parameters ./azuredeploy.parameters.json --parameters adminUsername=$ADMINUSERNAME adminPasswordOrKey=$ADMINPASSWORD subnetId=$SUBNET_ID
    ```

1.  Évaluez la sortie de la commande et vérifiez qu’elle n’inclut pas d’erreurs et d’avertissements. Quand vous y êtes invité, appuyez sur la touche **Entrée** pour procéder au déploiement.

1.  N’attendez pas que le déploiement se termine, mais passez à la tâche suivante. 

    > **Remarque** : Si le déploiement échoue avec le message d’erreur **Conflit** lors du déploiement du composant CustomScriptExtension, procédez comme suit pour corriger ce problème :

       - dans le portail Azure, dans le panneau **Déploiement**, évaluez les détails du déploiement et identifiez les machines virtuelles sur lesquelles l’installation de CustomScriptExtension a échoué

       - dans le Portail Azure, accédez au panneau des machines virtuelles que vous avez identifiées à l’étape précédente, sélectionnez **Extensions** et, depuis le panneau **Extensions**, supprimez l’extension CustomScript

       - Réexécutez l’étape précédente de cette tâche.

### Tâche 3 : Déployer un hôte de saut

   > **Remarque** : Les machines virtuelles Azure déployées dans la tâche précédente n’étant pas accessibles depuis Internet, vous allez déployer une machine virtuelle Azure exécutant Windows Server 2019 Datacenter qui servira d’hôte de saut. 

1. Depuis l’ordinateur du labo, dans le portail Azure, cliquez sur **+ Créer une ressource**.

1. Dans le panneau **Nouveau**, lancez la création d’une machine virtuelle Azure basée sur l’image **Windows Server 2019 Datacenter**.

1. Configurez une machine virtuelle Azure avec les paramètres suivants (laissez les autres à leurs valeurs par défaut) :

    | Paramètre | Valeur |
    |   --    |  --   |
    | **Abonnement** | *nom de votre abonnement Azure*  |
    | **Groupe de ressources** | *nom d’un nouveau groupe de ressources* **az12003a-dmz-RG** |
    | **Nom de la machine virtuelle** | **az12003a-vm0** |
    | **Région** | *même région Azure que celle dans laquelle vous avez déployé les machines virtuelles Azure dans les tâches précédentes de cet exercice* |
    | **Options de disponibilité** | **Aucune redondance de l’infrastructure requise** |
    | **Image** | *sélectionnez* **Windows Server 2019 Datacenter - Gen2** |
    | **Taille** | **D2s_v3 standard** ou similaire |
    | **Nom d’utilisateur** | **Étudiant** |
    | **Mot de passe** | tout mot de passe complexe de votre choix |
    | **Ports d’entrée publics** | **Autoriser les ports sélectionnés** |
    | **Ports d’entrée sélectionnés** | **RDP (3389)** |
    | **Souhaitez-vous utiliser une licence Windows Server existante ?** | **Aucun** |
    | **Type de disque du système d’exploitation** | **HDD Standard** |
    | **Réseau virtuel** | **az12003a-sap-vnet** |
    | **Nom du sous-réseau** | *nouveau sous-réseau nommé* **bastionSubnet** |
    | **Plage d’adresses de sous-réseau** | **10.3.255.0/24** |
    | **Adresse IP publique** | *nouvelle adresse IP nommée* **az12003a-vm0-ip** |
    | **Groupe de sécurité réseau de la carte réseau** | **De base**  |
    | **Ports d’entrée publics** | **Autoriser les ports sélectionnés** |
    | **Ports d’entrée sélectionnés** | **RDP (3389)** |
    | **Activer la mise en réseau accélérée** | **Activé** |
    | **Options d’équilibrage de charge** | **Aucun** |
    | **Activer l’identité managée affectée par le système** | **Désactivé** |
    | **Se connecter avec Azure AD** | **Désactivé** |
    | **Activer l’arrêt automatique** | **Désactivé** |
    | **Options d’orchestration des patchs** | **Mises à jour manuelles** |
    | **Diagnostics de démarrage** | **Désactiver** |
    | **Activer les diagnostics du système d’exploitation invité** | **Désactivé** |
    | **Extensions** | *Aucun* |
    | **Balises** | *Aucun* |

   > **Remarque** : Veillez à mémoriser le mot de passe que vous avez spécifié pendant le déploiement. Vous en aurez besoin plus tard dans ce labo.

1. Attendez la fin du déploiement. Ce processus devrait prendre quelques minutes.

> **Result** : Une fois cet exercice terminé, vous avez approvisionné les ressources Azure nécessaires à la prise en charge des déploiements SAP NetWeaver à haute disponibilité


## Exercice 2 : Configurer des machines virtuelles Azure exécutant Linux pour prendre en charge un déploiement SAP NetWeaver à haut niveau de disponibilité

Durée : 30 minutes

Dans cet exercice, vous allez configurer des machines virtuelles Azure exécutant SUSE Linux Enterprise Server pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité.

### Tâche 1 : Configurez la mise en réseau des machines virtuelles Azure de la couche Base de données.

   > **Remarque** : Avant de commencer cette tâche, vérifiez que les déploiements de modèle que vous avez lancés dans l’exercice précédent se sont bien terminés. 

1. À partir de l’ordinateur du labo, dans le portail Azure, accédez au panneau de la machine virtuelle Azure **i20-db-0**.

1. Depuis le panneau ** i20-db-0**, accédez à son panneau **Mise en réseau**. 

1. Depuis le panneau **i20-db-0 - Mise en réseau**, accédez à l’interface réseau de l’i20-db-0. 

1. Depuis le panneau de l’interface réseau de l’i20-db-0, accédez au panneau de ses configurations IP pour afficher son panneau **ipconfig1**.

1. Sur le panneau **ipconfig1**, définissez l’adresse IP privée sur **10.3.0.20**, remplacez son attribution par **Statique** et enregistrez la modification.

1. Dans le portail Azure, accédez au panneau de la machine virtuelle Azure **i20-db-1**.

1. Depuis le panneau ** i20-db-1**, accédez à son panneau **Mise en réseau**. 

1. Depuis le panneau **i20-db-1 - Mise en réseau**, accédez à l’interface réseau de l’i20-db-1. 

1. Depuis le panneau de l’interface réseau de l’i20-db-1, accédez au panneau de ses configurations IP pour afficher son panneau **ipconfig1**.

1. Sur le panneau **ipconfig1**, définissez l’adresse IP privée sur **10.3.0.21**, remplacez son attribution par **Statique** et enregistrez la modification.


### Tâche 2 : Connectez-vous aux machines virtuelles Azure de la couche Base de données.

1. Depuis l’ordinateur du labo, dans le portail Azure, accédez au panneau **az12003a-vm0**.

1. Depuis le panneau **az12003a-vm0**, connectez-vous à la machine virtuelle Azure az12003a-vm0 via le Bureau à distance. Lorsque vous êtes invité à vous authentifier, entrez le nom d’utilisateur et le mot de passe définis lors du déploiement de cette machine virtuelle.

1. Dans la session RDP sur az12003a-vm0, dans le Gestionnaire de serveur, accédez à la vue **Serveur local** et désactivez **Configuration de sécurité renforcée d’Internet Explorer**.

1. Dans la session RDP sur az12003a-vm0, téléchargez et installez PuTTY à partir de [**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

1. Utilisez PuTTY pour vous connecter via SSH à la machine virtuelle Azure **i20-db-0**. Accusez réception de l’alerte de sécurité et, lorsque vous y êtes invité, fournissez les informations d’identification suivantes :

    -   Connectez-vous en tant que : **étudiant**

    -   Mot de passe : **Pa55w.rd1234**

1. Utilisez PuTTY pour vous connecter via SSH à la machine virtuelle Azure **i20-db-1** avec les mêmes informations d’identification.


### Tâche 3 : Examiner la configuration de stockage des machines virtuelles Azure de la couche Base de données.

1. Depuis la session SSH PuTTY sur la machine virtuelle Azure i20-db-0, exécutez la commande suivante pour élever les privilèges : 

    ```
    sudo su -
    ```

1. Si vous y êtes invité, entrez le mot de passe **Pa55w.rd1234** et appuyez sur la touche **Entrée**. 

1. Dans la session SSH sur i20-db-0, vérifiez que les volumes associés à SAP HANA (notamment **/usr/sap**, **/hana/shared**, **/hana/backup**, **/hana/data** et **/hana/logs**) sont correctement montés en exécutant :

    ```
    df -h
    ```

1. Répétez les étapes précédentes sur la machine virtuelle Azure i20-db-1.


### Tâche 4 : Activer l’accès SSH sans mot de passe inter-nœuds

1. Dans la session SSH sur i20-db-0, générez une clé SSH sans phrase secrète en exécutant :

    ```
    ssh-keygen
    ```

1. Lorsque vous y êtes invité, appuyez trois fois sur **Entrée**, puis affichez la clé en exécutant : 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1. Copiez la valeur de la clé dans le Presse-papiers.

1. Dans la session SSH sur i20-db-1, créez le fichier **/root/.ssh/authorized\_keys** dans l’éditeur vi en exécutant :

    ```
    vi /root/.ssh/authorized_keys
    ```

1. Dans l’éditeur vi, collez la clé que vous avez générée sur i20-db-0.

1. Enregistrez les modifications et fermez l'éditeur.

1. Dans la session SSH sur i20-db-1, générez une clé SSH sans phrase secrète en exécutant :

    ```
    ssh-keygen
    ```

1. Lorsque vous y êtes invité, appuyez trois fois sur **Entrée**, puis affichez la clé en exécutant : 

    ```
    cat /root/.ssh/id_rsa.pub
    ```

1. Copiez la valeur de la clé dans le Presse-papiers.

1. Dans la session SSH sur i20-db-0, créez le fichier **/root/.ssh/authorized\_keys** dans l’éditeur vi en exécutant :

    ```
    vi /root/.ssh/authorized_keys
    ```

1. Dans l’éditeur vi, collez la clé que vous avez générée sur i20-db-1 à partir d’une nouvelle ligne.

1. Enregistrez les modifications et fermez l'éditeur.

1. Pour vérifier que la configuration a réussi, dans la session SSH sur i20-db-0, établissez une session SSH en **root** de i20-db-0 sur i20-db-1 en exécutant : 

    ```
    ssh root@i20-db-1
    ```

1. Lorsque vous êtes invité à confirmer que vous voulez continuer à vous connecter, entrez `yes` et appuyez sur la touche **Entrée**. 

1. Vérifiez que vous n’êtes pas invité à entrer le mot de passe.

1. Fermez la session SSH de i20-db-0 sur i20-db-1 en exécutant : 

    ```
    exit
    ```

1. Dans la session SSH sur i20-db-1, établissez une session SSH en **root** de i20-db-1 sur i20-db-0 en exécutant : 

    ```
    ssh root@i20-db-0
    ```

1. Lorsque vous êtes invité à confirmer que vous voulez continuer à vous connecter, entrez `yes` et appuyez sur la touche **Entrée**. 

1. Vérifiez que vous n’êtes pas invité à entrer le mot de passe.

1. Fermez la session SSH de i20-db-1 sur i20-db-0 en exécutant : 

    ```
    exit
    ```

### Tâche 5 : Ajouter des packages YaST, mettre à jour le système d’exploitation Linux et installer des extensions à haute disponibilité

1. Dans la session SSH sur i20-db-0, exécutez ce qui suit pour lancer YaST :

    ```
    yast
    ```

1. Dans le **Centre de contrôle YaST**, sélectionnez **Logiciels -\> Produits complémentaires**, puis appuyez sur **Entrée**. Cela chargera le **Gestionnaire de package**.

1. Sur l’écran **Produits complémentaires installés**, vérifiez que le **Module cloud public** est déjà installé. Ensuite, appuyez deux fois sur **F9** pour revenir à l’invite de l’interpréteur de commandes.

1. Dans la session SSH sur i20-db-0, exécutez ce qui suit pour mettre à jour le système d’exploitation (lorsque vous y êtes invité, entrez **y** et appuyez sur la touche **Entrée**) :

    ```
    zypper update
    ```

1. Dans la session SSH sur i20-db-0, exécutez la commande suivante pour installer les packages requis par les ressources de cluster (lorsque vous y êtes invité, entrez **y** et appuyez sur la touche **Entrée**) :

    ```
    zypper in socat
    ```

1. Dans la session SSH sur i20-db-0, exécutez la commande suivante pour installer le composant azure-lb requis par les ressources de cluster :

    ```
    zypper in resource-agents
    ```

1. Dans la session SSH sur i20-db-0, ouvrez le fichier **/etc/systemd/system.conf** dans l’éditeur vi en exécutant :

    ```
    vi /etc/systemd/system.conf
    ```

1. Dans l’éditeur vi, remplacez `#DefaultTasksMax=512` par `DefaultTasksMax=4096`. 

    > **Remarque** : Dans certains cas, Pacemaker peut créer de nombreux processus, atteignant la limite par défaut imposée à leur nombre et déclenchant un basculement. Cette modification augmente le nombre maximal de processus autorisés.

1. Enregistrez les modifications et fermez l'éditeur.

1. Dans la session SSH sur i20-db-0, exécutez la commande suivante pour activer la modification de configuration :

    ```
    systemctl daemon-reload
    ```

1. Dans la session SSH sur i20-db-0, exécutez la commande suivante pour installer le package des agents de clôture :

    ```
    zypper install fence-agents
    ```

1. Dans la session SSH sur i20-db-0, exécutez la commande suivante pour installer les kits de développement logiciel (SDK) Azure Python requis par l’agent de clôture (lorsque vous y êtes invité, entrez **y** et appuyez sur la touche **Entrée**) :

    ```
    SUSEConnect -p sle-module-public-cloud/12/x86_64
    zypper install python-azure-mgmt-compute
    ```

1. Répétez les étapes précédentes de cette tâche sur i20-db-1.

> **Result** : Une fois cet exercice terminé, vous avez configuré un système d’exploitation de machines virtuelles Azure exécutant Linux pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité

## Exercice 3 : Configurer le clustering pour des machines virtuelles Azure exécutant Linux pour prendre en charge un déploiement SAP NetWeaver à haut niveau de disponibilité

Durée : 30 minutes

Dans cet exercice, vous allez configurer le clustering pour des machines virtuelles Azure exécutant Linux afin de prendre en charge un déploiement SAP NetWeaver à haute disponibilité.

### Tâche 1 : Configurer le clustering

1. Dans la session RDP sur az12003a-vm0, dans la session SSH basée sur PuTTY sur i20-db-0, exécutez ce qui suit pour lancer la configuration d’un cluster à haute disponibilité sur i20-db-0 :

    ```
    ha-cluster-init -u
    ```

1. Lorsque vous y êtes invité, fournissez les réponses suivantes :

    -   Voulez-vous continuer (y/n) ? : **y**

    -   Adresse de ring0 [10.3.0.20] : **ENTRÉE**

    -   Port de ring0 [5405] : **ENTRÉE**

    -   Voulez-vous utiliser SBD (y/n) ?: **n**

    -   Voulez-vous configurer une adresse IP virtuelle (y/n) ? : **n**

    > **Remarque** : La configuration du clustering génère un compte **hacluster** dont le mot de passe est défini sur **linux**. Vous le modifierez plus tard dans cette tâche.

1. Dans la session RDP sur az12003a-vm0, dans la session SSH basée sur PuTTY sur i20-db-1, exécutez ce qui suit pour joindre le cluster à haute disponibilité sur i20-db-0 depuis i20-db-1 :

    ```
    ha-cluster-join
    ```

1. Lorsque vous y êtes invité, fournissez les réponses suivantes :

    -   Voulez-vous continuer (y/n) ? **y**

    -   Adresse IP ou nom d’hôte du nœud existant (par exemple : 192.168.1.1) \[\] : **i20-db-0**

    -   Adresse de ring0 [10.3.0.21] : **ENTRÉE**

1. Dans la session SSH basée sur PuTTY sur i20-db-0, exécutez la commande suivante pour définir le mot de passe du compte **hacluster** sur **Pa55w.rd1234** (entrez le nouveau mot de passe lorsque vous y êtes invité) : 

    ```
    passwd hacluster
    ```

1. Répétez l’étape précédente sur i20-db-1.

### Tâche 2 : Passer en revue la configuration corosync

1. Dans la session RDP sur az12003a-vm0, dans la session SSH basée sur PuTTY sur i20-db-0, ouvrez le fichier **/etc/corosync/corosync.conf** en exécutant :

    ```
    vi /etc/corosync/corosync.conf
    ```

1. Dans l’éditeur vi, notez l’entrée `transport: udpu` et la section `nodelist` :
    ```
    [...]
       interface { 
           [...] 
       }
       transport:      udpu
    } 
    nodelist {
       node {
         ring0_addr:     10.3.0.20
         nodeid:     1
       }
       node {
         ring0_addr:     10.3.0.21
         nodeid:     2
       } 
    }
    logging {
        [...]
    ```

1. Dans l’éditeur vi, remplacez l’entrée `token: 5000` par `token: 30000`.

    > **Remarque** : Cette modification permet une maintenance avec préservation de la mémoire. Pour plus d’informations, consultez la [documentation Microsoft sur la maintenance des machines virtuelles dans Azure](https://docs.microsoft.com/en-us/azure/virtual-machines/maintenance-and-updates#maintenance-that-doesnt-require-a-reboot)

1. Enregistrez les modifications et fermez l'éditeur.

1. Répétez les étapes précédentes sur i20-db-1.


### Tâche 3 : Identifier la valeur de l’ID d’abonnement Azure et de l’ID de locataire Azure AD

1. Depuis l’ordinateur du labo, dans la fenêtre du navigateur, dans le portail Azure à l’adresse **https://portal.azure.com**, vérifiez que vous êtes connecté avec le compte d’utilisateur disposant du rôle d’administrateur général dans le locataire Azure AD associé à votre abonnement.

1. Dans le portail Azure, démarrez une session Bash dans Cloud Shell. 

1. Dans le volet Cloud Shell, exécutez la commande suivante pour identifier l’ID de votre abonnement Azure et l’ID du locataire Azure AD correspondant :

    ```cli
    az account show --query '{id:id, tenantId:tenantId}' --output json
    ```

1. Copiez les valeurs obtenues dans le Bloc-notes. Vous en aurez besoin dans la prochaine tâche.


### Tâche 4 : Créer une application Azure AD pour l’appareil STONITH

1. Dans le portail Azure, accédez au volet Azure Active Directory.

1. Depuis le panneau Azure Active Directory, accédez au panneau **Inscriptions d’applications**, puis cliquez sur **+ Nouvelle inscription** :

1. Sur le panneau **Inscrire une application**, spécifiez les paramètres suivants, puis cliquez sur **Inscrire** :

    -   Nom : **Application Stonith**

    -   Types de comptes pris en charge : **Comptes dans cet annuaire organisationnel uniquement**

1. Sur le panneau de l’**application Stonith**, copiez la valeur de l’**ID d’application (client)** dans le Bloc-notes. Cette opération sera appelée **login_id** plus tard dans cet exercice :

1. Sur le panneau de l’**application Stonith**, cliquez sur **Certificats et secrets**.

1. Sur le panneau **Application Stonith - Certificats et secrets**, cliquez sur **+Nouvelle clé secrète client**.

1. Dans le volet **Ajouter une clé secrète client**, dans la zone de texte **Description**, entrez **Clé de l’application STONITH**, dans la section **Expire**, laissez la valeur par défaut **Recommandé : 6 mois**, puis cliquez sur **Ajouter**.

1. Copiez la **Valeur** obtenue dans le Bloc-notes (cette entrée n’est affichée qu’une seule fois, après avoir cliqué sur **Ajouter**). Cette opération sera appelée **mot de passe** plus tard dans cet exercice :


### Tâche 5 : Octroyer des autorisations sur les machines virtuelles Azure au principal de service de l’application STONITH 

1. Dans le portail Azure, accédez au panneau de la machine virtuelle Azure **i20-db-0**

1. Dans le panneau **i20-db-0**, affichez le panneau **i20-db-0 - Contrôle d’accès (IAM)**.

1. Depuis le **panneau i20-db-0 - Contrôle d’accès (IAM)**, ajoutez une attribution de rôle avec les paramètres suivants :

    -   Rôle : **Contributeur de machine virtuelle**

    -   Attribuer l’accès à : **Utilisateur, groupe ou principal de service Azure AD**

    -   Sélectionnez : **Application Stonith**

1. Répétez les étapes précédentes pour attribuer le rôle de contributeur de machine virtuelle à l’application Stonith sur la machine virtuelle Azure **i20-db-1**


### Tâche 6 : Configurer le périphérique de cluster STONITH 

1. Dans la session RDP sur az12003a-vm0, basculez vers la session SSH basée sur PuTTY sur i20-db-0.

1. Dans la session RDP sur az12003a-vm0, dans la session SSH basée sur PuTTY sur i20-db-0, exécutez les commandes suivantes (veillez à remplacer les espaces réservés `subscription_id`, `tenant_id`, `login_id,` et `password` par les valeurs identifiées dans la Tâche 4 de l’Exercice 3 :

    ```
    crm configure property stonith-enabled=true

    crm configure property concurrent-fencing=true

    crm configure primitive rsc_st_azure stonith:fence_azure_arm \
      params subscriptionId="subscription_id" resourceGroup="az12003a-sap-RG" tenantId="tenant_id" login="login_id" passwd="password" \
      pcmk_monitor_retries=4 pcmk_action_limit=3 power_timeout=240 pcmk_reboot_timeout=900 \
      op monitor interval=3600 timeout=120

    sudo crm configure property stonith-timeout=900
    ```

### Tâche 7 : Passer en revue la configuration du clustering sur des machines virtuelles Azure exécutant Linux à l’aide de Hawk

1. Dans la session RDP sur az12003a-vm0, démarrez Internet Explorer et accédez à **https://i20-db-0:7630**. Cela doit afficher la page de connexion SUSE Hawk.

   > **Remarque** : Ignorez le message **Ce site n’est pas sécurisé**.

1. Sur la page de connexion SUSE Hawk, connectez-vous à l’aide des informations d’identification suivantes :

    -   Nom d’utilisateur : **hacluster**

    -   Mot de passe : **Pa55w.rd1234**

1. Vérifiez que l’état du cluster est sain. Si vous voyez un message indiquant qu’un des deux nœuds de cluster n’est pas propre, redémarrez ce nœud depuis le portail Azure.

> **Result** : Une fois cet exercice terminé, vous avez configuré le clustering de machines virtuelles Azure exécutant Linux pour prendre en charge un déploiement SAP NetWeaver à haute disponibilité


## Exercice 4 : Supprimer des ressources de lab

Durée : 10 minutes

Dans cet exercice, vous allez supprimer les ressources approvisionnées dans ce labo.

#### Tâche 1 : Ouvrir Cloud Shell

1. En haut du portail, cliquez sur l’icône **Cloud Shell** pour ouvrir le volet Cloud Shell et choisissez Bash comme interpréteur de commandes.

1. Dans le volet Cloud Shell, exécutez la commande suivante pour définir la valeur de la variable `RESOURCE_GROUP_PREFIX` sur le préfixe du nom du groupe de ressources contenant les ressources que vous avez approvisionnées dans ce labo :

    ```cli
    RESOURCE_GROUP_PREFIX='az12003a-'
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
