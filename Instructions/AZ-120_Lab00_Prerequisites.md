---
lab:
  title: "00\_: Prérequis pour le labo"
  module: Module 00 - Lab prerequisites
---

# AZ 120 : Prérequis pour le labo

## Exigences en matière de cœur de processeur virtuel

> **Important** : Les exigences en matière de cœur de processeur virtuel dépendent des labos que vous envisagez d’implémenter.

- Pour terminer le labo 04b : Implémenter l’architecture SAP sur les machines virtuelles Azure exécutant Windows, vous aurez besoin d’un abonnement Microsoft Azure avec au moins 28 processeurs virtuels disponibles dans la région Azure qui prend en charge les zones de disponibilité où les machines virtuelles Azure déployées dans ce labo résideront.

    - 4 x Standard_DS1_v2 (1 processeur virtuel chacun) = 4
    - 6 x Standard_D4s_v3 (4 processeurs virtuels chacun) = 24

    > **Remarque** : Envisagez d’utiliser les régions **USA Est** ou **USA Est 2** pour le déploiement de vos ressources.

    > **Remarque** : Pour identifier les régions Azure qui prennent en charge les zones de disponibilité, reportez-vous à <https://docs.microsoft.com/en-us/azure/availability-zones/az-overview>

- Pour terminer le labo 05 : Automatiser le déploiement à l’aide du Centre Azure pour les solutions SAP, vous aurez besoin d’un abonnement Microsoft Azure avec la disponibilité de processeurs virtuels suivante dans la région Azure où les machines virtuelles Azure déployées dans ce labo résideront.

    - 4 x Standard_E4ds_v4 (4 processeurs virtuels chacun) ou 4 X Standard_D4ds_v4 (4 processeurs virtuels chacun) = 8
    - 2 x Standard_M64ms (64 processeurs virtuels chacun) = 128

>**Remarque** : Pour réduire les besoins en processeurs virtuels et en mémoire, vous pouvez utiliser Standard_M32ts (32 processeurs virtuels et 192 Gio de mémoire chacun) au lieu de Standard_M64m.

>**Remarque** : Bien que les exigences du processeur virtuel pour les trois premiers laboratoires de ce cours soient inférieures, nous vous recommandons de demander une augmentation des quotas pour répondre aux exigences de tous les laboratoires, car le processus d’augmentation des quotas peut prendre un certain temps (même si les demandes d’augmentation de quota sont généralement traitées au cours du même jour ouvrable).

## Avant le labo pratique

Délai d’exécution : 30 minutes

### Tâche 1 : Valider un nombre suffisant de cœurs de processeurs virtuels pour le labo « Implémenter l’architecture SAP sur des machines virtuelles Azure exécutant Windows »

1. À partir de l’ordinateur de labo, démarrez un navigateur web et accédez au portail Azure à l’adresse `https://portal.azure.com`.
1. Dans le portail Azure, démarrez une session PowerShell dans Cloud Shell. 

    > **Remarque** : Si c’est la première fois que vous lancez Cloud Shell dans l’abonnement Azure actuel, vous êtes invité à créer un partage de fichiers Azure pour conserver les fichiers Cloud Shell. Dans ce cas, acceptez les valeurs par défaut, ce qui entraînera la création d’un compte de stockage dans un groupe de ressources généré automatiquement.

1. Dans le portail Azure, dans le volet **Cloud Shell**, à l’invite PowerShell, exécutez ce qui suit, où `<Azure_region>` désigne la région Azure cible que vous envisagez d’utiliser dans ce labo (par ex. : `eastus`) :

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv3Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'StandardDSv2Family'}
    ``` 

    > **Remarque** : Pour identifier les noms des régions Azure, dans **Cloud Shell**, à l’invite Bash, exécutez `(Get-AzLocation).Location`
   
1. Passez en revue la valeur actuelle et les entrées de limite dans la sortie des commandes exécutées à l’étape précédente et vérifiez que vous disposez d’un nombre suffisant de processeurs virtuels dans la région Azure cible.
1. Si le nombre de processeurs virtuels n’est pas suffisant, revenez au panneau de l’abonnement dans le portail Azure, puis cliquez sur **Utilisation + quotas**. 
1. Dans le panneau **Utilisation + quotas** de l’abonnement, cliquez sur **Demander une augmentation**.
1. Dans le panneau **Description du problème**, spécifiez les éléments suivants :

    -   Type de problème : **Limites du service et de l’abonnement (quotas)**
    -   Abonnement : Nom de l’abonnement Azure que vous utilisez dans ce labo
    -   Type de quota : **Augmentations de la limite d’abonnement de calcul/machine virtuelle (cœurs/processeurs virtuels)**

1. Cliquez sur **Gérer le quota**.
1. Utilisez le menu déroulant d’emplacement pour filtrer les résultats en fonction de la région Azure que vous prévoyez d’utiliser. Il est recommandé d’utiliser **USA Est** ou **USA Est2**.
1. Recherchez le type de quota **Processeurs virtuels de famille DSv3 standard** et sélectionnez le crayon de modification.
1. Dans le champ Nouvelle limite, spécifiez **40**, puis cliquez sur **Enregistrer et Continuer**.
1. Recherchez le type de quota **Total de processeurs virtuels régionaux** et sélectionnez le crayon de modification.
1. Dans le champ Nouvelle limite, spécifiez **40**, puis cliquez sur **Enregistrer et Continuer**.

   > **Remarque** : La demande d’augmentation de quota devrait être approuvée automatiquement. Si la demande est refusée, ouvrez un ticket de support pour demander l’augmentation du quota.

### Tâche 2 : Valider un nombre suffisant de cœurs de processeurs virtuels pour le labo « Automatiser le déploiement avec le Centre Azure pour les solutions SAP »

> **Remarque** : Pour suivre ce labo (comme décrit), vous avez besoin d’un abonnement Microsoft Azure avec les quotas de processeurs virtuels qui prennent en charge le déploiement des machines virtuelles suivantes :

- 2 machines virtuelles Standard_E4ds_v4 (4 processeurs virtuels et 32 Gio de mémoire chacune) ou 2 machines virtuelles Standard_D4ds_v4 (4 processeurs virtuels et 16 Gio de mémoire chacune) pour la couche ASCS
- 2 machines virtuelles Standard_E4ds_v4 (4 processeurs virtuels et 32 Gio de mémoire chacune) ou 2 machines virtuelles Standard_D4ds_v4 (4 processeurs virtuels et 16 Gio de mémoire chacune) pour la couche Application 
- 2 machines virtuelles Standard_M64ms (64 processeurs virtuels et 1750 Gio de mémoire chacune) pour la couche Base de données

1. À partir de l’ordinateur de labo, démarrez un navigateur web et accédez au portail Azure à l’adresse `https://portal.azure.com`.
1. Dans le portail Azure, sélectionnez l’icône **Cloud Shell** et démarrez une session PowerShell dans Cloud Shell. 
1. Dans le portail Azure, dans le volet **Cloud Shell**, à l’invite PowerShell, exécutez ce qui suit, où `<Azure_region>` désigne la région Azure dans laquelle vous envisagez de déployer les ressources dans ce labo (par ex. : `eastus`) :

    ```powershell
    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardEDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardDSv4Family'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'standardMSFamily'}

    Get-AzVMUsage -Location '<Azure_region>' | Where-Object {$_.Name.Value -eq 'cores'}
    ```

    > **Remarque** : Pour identifier les noms des régions Azure, dans **Cloud Shell**, à l’invite Bash, exécutez `(Get-AzLocation).Location`

1. Passez en revue la sortie pour identifier l’utilisation actuelle des processeurs virtuels et la limite des processeurs virtuels. Vérifiez que la différence entre elles est suffisante pour prendre en charge les processeurs virtuels des machines virtuelles Azure que vous allez déployer dans ce labo. Prenez en compte à la fois la quantité totale de processeurs virtuels régionaux et la quantité propre à la famille de machines virtuelles. 
1. Si le nombre de processeurs virtuels n’est pas suffisant, fermez le volet Cloud Shell. Dans le portail Azure, dans la zone de texte **Rechercher**, recherchez et sélectionnez **Quotas**.
1. Dans la page **Quotas**, sélectionnez **Calcul**.
1. Dans la page **Quotas \| Calcul**, utilisez le filtre **Région** pour sélectionner la région Azure dans laquelle vous envisagez de déployer les ressources dans ce labo.
1. Dans la colonne **Nom du quota**, recherchez et sélectionnez le nom de la référence SKU de machine virtuelle qui nécessite une augmentation du quota. 
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

    > **Remarque** : Attendez que la demande d’augmentation des limites de quota ait abouti avant de démarrer le labo « Automatiser le déploiement avec le Centre Azure pour les solutions SAP ».
