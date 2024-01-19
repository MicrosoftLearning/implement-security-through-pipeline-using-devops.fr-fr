---
lab:
  title: Étendre un pipeline pour utiliser plusieurs modèles
  module: 'Module 5: Extend a pipeline to use multiple templates'
---

# Étendre un pipeline pour utiliser plusieurs modèles

Dans ce module, découvrez pourquoi il est important d’étendre un pipeline à plusieurs modèles et comment le faire avec Azure DevOps. Ce labo couvre les concepts fondamentaux et les bonnes pratiques pour créer un pipeline multiphase, un modèle de variables, un modèle de travail et un modèle de phase.

Ces exercices prennent environ **20** minutes.

## Avant de commencer

Vous aurez besoin d’un abonnement Azure, d’une organisation Azure DevOps et de l’application eShopOnWeb pour suivre les labos.

- Procédez comme suit pour [valider votre environnement de labo](APL2001_M00_Validate_Lab_Environment.md).

## Instructions

### Exercice 1 : Créer un pipeline YAML multiphase

#### Tâche 1 : Créer un pipeline YAML principal multiphase

1. Accédez au Portail Azure DevOps sur `https://dev.azure.com` et ouvrez votre organisation.

1. Ouvrez le projet **eShopOnWeb**.

1. Accédez à **Pipelines > Pipelines**.

1. Sélectionnez **Créer un pipeline**.

1. Sélectionnez **Azure Repos Git (YAML)**.

1. Sélectionnez le référentiel **eShopOnWeb** .

1. Sélectionnez **Pipeline de démarrage**.

1. Remplacez le contenu du fichier **azure-pipelines.yml** par le code suivant :

   ```yaml
   trigger:
   - main

   pool:
     vmImage: 'windows-latest'

   stages:
   - stage: Dev
     jobs:
     - job: Build
       steps:
       - script: echo Build
   - stage: Test
     jobs:
     - job: Test
       steps:
       - script: echo Test
   - stage: Production
     jobs:
     - job: Deploy
       steps:
       - script: echo Deploy
   ```

1. Sélectionnez **Enregistrer et exécuter**. Choisissez de commiter directement dans la branche principale, puis sélectionnez à nouveau **Enregistrer et exécuter**.

1. Vous verrez le pipeline s’exécutant avec les trois phases (Dev, Test et Production) et les travaux correspondants. Attendez que le pipeline se termine et revenez à la page **Pipelines**.

   ![Capture d’écran du pipeline en cours d’exécution avec les trois phases et les travaux correspondants](media/eshoponweb-pipeline-multi-stage.png)

1. Sélectionnez **...** (Autres options) sur le côté droit du pipeline que vous venez de créer, puis sélectionnez **Renommer/déplacer**.

1. Renommez le pipeline **eShopOnWeb-MultiStage-Main**, puis sélectionnez **Enregistrer**.

#### Tâche 2 : Créer un modèle de variables

1. Accédez à **Repos > Fichiers**.

1. Développez le dossier **.ado** et cliquez sur **Nouveau fichier**.

1. Nommez le fichier **eshoponweb-variables.yml**, puis cliquez sur **Créer**.

1. Ajoutez le code suivant au fichier :

   ```yaml
   variables:
     resource-group: 'YOUR-RESOURCE-GROUP-NAME'
     location: 'southcentralus' #name of the Azure region you want to deploy your resources
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
     webappname: 'YOUR-WEB-APP-NAME'
   ```

1. Remplacez les valeurs des variables par les valeurs de votre environnement :

   - Remplacez **YOUR-RESOURCE-GROUP-NAME** par le nom du groupe de ressources que vous souhaitez utiliser dans ce labo, par exemple, **rg-eshoponweb-multi**.
   - Définissez la valeur de la variable de **localisation** sur le nom de la région Azure dans laquelle vous souhaitez déployer vos ressources, par exemple, **southcentralus**.
   - Remplacez **YOUR-SUBSCRIPTION-ID** par votre ID d’abonnement Azure.
   - Remplacez **YOUR-AZURE-SERVICE-CONNECTION-NAME** par **azure subs**
   - Remplacez **YOUR-WEB-APP-NAME** par un nom global unique de l’application web à déployer, par exemple, la chaîne **eshoponweb-lab-perm-** suivie d’un nombre aléatoire à six chiffres.  

1. Sélectionnez **Commiter**, dans la zone de texte de commentaire de commit, entrez `[skip ci]`, puis sélectionnez **Commiter**.

   > [!NOTE]
   > En ajoutant le commentaire `[skip ci]` au commit, vous empêchez l’exécution automatique du pipeline qui, à ce stade, s’exécute par défaut après chaque changement effectué dans le référentiel. 

#### Tâche 3 : Préparer le pipeline pour utiliser des modèles

1. Dans le portail Azure DevOps, dans la page du projet **eShopOnWeb**, accédez à **Référentiels**. 

1. Dans le répertoire racine du dépôt, sélectionnez **azure-pipelines.yml**, qui contient la définition du pipeline **eShopOnWeb-MultiStage-Main**.

1. Sélectionnez **Modifier**.

1. Remplacez le contenu du fichier **azure-pipelines.yml** par le code suivant :

   ```yaml
   trigger:
   - main
   variables:
   - template: .ado/eshoponweb-variables.yml
   
   stages:
   - stage: Dev
     jobs:
     - template: .ado/eshoponweb-ci.yml
   - stage: Test
     jobs:
     - template: .ado/eshoponweb-cd-webapp-code.yml
   - stage: Production
     jobs:
     - job: Deploy
       steps:
       - script: echo Deploy to Production or Swap
   ```

1. Sélectionnez **Commiter**, dans la zone de texte de commentaire de commit, entrez `[skip ci]`, puis sélectionnez **Commiter**.

#### Tâche 4 : Mise à jour des modèles CI/CD

1. Dans les **référentiels** du projet **eShopOnWeb**, sélectionnez le répertoire **.ado** et sélectionnez le fichier ** eshoponweb-ci.yml**. 

1. Supprimez tout ce qui précède la section **Travaux**.

   ```yaml
   #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
   # trigger:
   # - main
   
   resources:
     repositories:
       - repository: self
         trigger: none
   
   stages:
   - stage: Build
     displayName: Build .Net Core Solution
   ```

1. Sélectionnez **Commiter**, dans la zone de texte de commentaire de commit, entrez `[skip ci]`, puis sélectionnez **Commiter**.

1. Dans les **référentiels** du projet **eShopOnWeb**, sélectionnez le répertoire **.ado** et sélectionnez le fichier **eshoponweb-cd-webapp-code.yml**.

1. Supprimez tout ce qui précède la section **Travaux**.

   ```yaml
   #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
   
   # Trigger CD when CI executed successfully
   resources:
     pipelines:
       - pipeline: eshoponweb-ci
         source: eshoponweb-ci # given pipeline name
         trigger: true

   variables:
     resource-group: 'rg-eshoponweb'
     location: 'southcentralus'
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: ''
     azureserviceconnection: 'azure subs'
     webappname: 'eshoponweb-lab'
     # webappname: 'webapp-windows-eshop'
   
   stages:
   - stage: Deploy
     displayName: Deploy to WebApp`
   ```

1. Remplacez le contenu existant de l’étape **#download artifacts** par :

   ```yaml
       - download: current
         artifact: Website
       - download: current
         artifact: Bicep
   ```

1. Sélectionnez **Commiter**, dans la zone de texte de commentaire de commit, entrez `[skip ci]`, puis sélectionnez **Commiter**.

#### Tâche 5 : Exécuter le pipeline principal

1. Accédez à **Pipelines > Pipelines**.

1. Ouvrez le pipeline **eShopOnWeb-MultiStage-Main**.

1. Sélectionnez **Exécuter le pipeline**.

1. Une fois que le pipeline atteint la phase de **déploiement** dans l’environnement **Test**, ouvrez le pipeline et notez le message « Ce pipeline a besoin d’une autorisation d’accès à une ressource pour que cette exécution puisse se poursuivre jusqu’à Test ». Sélectionnez **Afficher**, puis **Autoriser** pour permettre au pipeline de s’exécuter.

   > [!NOTE]
   > Si des travaux de la phase de déploiement échouent, accédez à la page d’exécution du pipeline et sélectionnez **Réexécuter les travaux ayant échoué***.

1. Une fois que le pipeline atteint la phase de **déploiement** dans l’environnement **Production**, ouvrez le pipeline et notez le message « Ce pipeline a besoin d’une autorisation d’accès à une ressource pour que cette exécution puisse se poursuivre jusqu’à Production ». Sélectionnez **Afficher**, puis **Autoriser** pour permettre au pipeline de s’exécuter.

1. Attendez que le pipeline se termine et vérifiez les résultats.

   ![Capture d’écran du pipeline en cours d’exécution avec les trois phases et les travaux correspondants](media/multi-stage-completed.png)

### Exercice 2 : Effectuerle nettoyage des ressources Azure et Azure DevOps

Dans cet exercice, vous allez supprimer les ressources Azure et Azure DevOps créées dans ce labo.

#### Tâche 1 : Supprimer les ressources Azure

1. Dans le portail Microsoft Azure, accédez au groupe de ressources **rg-eshoponweb-multi** contenant les ressources déployées et sélectionnez **Supprimer le groupe de ressources** pour supprimer toutes les ressources créées dans ce labo.

#### Tâche 2 : Supprimer les pipelines Azure DevOps

1. Accédez au Portail Azure DevOps sur `https://dev.azure.com` et ouvrez votre organisation.

1. Ouvrez le projet **eShopOnWeb**.

1. Accédez à **Pipelines > Pipelines**.

1. Accédez à **Pipelines > Pipelines** et supprimez les pipelines existants.

#### Tâche 3 : Recréer le référentiel Azure DevOps

1. Dans le portail Azure DevOps, dans le projet **eShopOnWeb**, sélectionnez **Paramètres du projet** en bas à gauche.

1. Dans le menu vertical **Paramètres du projet** sur le côté gauche, dans la section **Référentiels**, sélectionnez **Référentiels**.

1. Dans le volet **Tous les référentiels**, pointez sur l’extrémité droite de l’entrée de référentiel **eShopOnWeb** jusqu’à ce que l’icône **Plus d’options** s’affiche, sélectionnez-la et, dans le menu **Plus d’options**, sélectionnez **Renommer**.  

1. Dans la fenêtre **Renommer le référentiel eShopOnWeb**, dans la zone de texte **Nom du référentiel**, entrez **eShopOnWeb_old**, puis sélectionnez**Renommer**.

1. De retour dans le volet **Tous les référentiels**, sélectionnez **+ Créer**.

1. Dans le volet **Créer un référentiel**, dans la zone de texte **Nom du référentiel**, entrez **eShopOnWeb**, décochez la case **Ajouter un fichier README** et sélectionnez **Créer**.

1. De retour dans le volet **Tous les référentiels**, pointez sur l’extrémité droite de l’entrée de référentiel **eShopOnWeb_old** jusqu’à ce que l’icône de points de suspension **Plus d’options** s’affiche, sélectionnez-la et, dans le menu **Plus d’options**, sélectionnez **Supprimer**.  

1. Dans la fenêtre **Supprimer le référentiel eShopOnWeb_old**, entrez **eShopOnWeb_old** et sélectionnez **Supprimer**.

1. Dans le menu de navigation de gauche du portail Azure DevOps, sélectionnez **Référentiels**.

1. Dans le volet **eShopOnWeb est vide. Ajouter du code !**, sélectionnez **Importer un référentiel**.

1. Dans la fenêtre **Importer un référentiel Git**, collez l’URL `https://github.com/MicrosoftLearning/eShopOnWeb` suivante, puis sélectionnez **Importer** :

## Révision

Dans ce labo, vous avez appris à étendre un pipeline dans plusieurs modèles à l’aide d’Azure DevOps. Ce labo a couvert les concepts fondamentaux et les meilleures pratiques pour créer un pipeline multiphase, un modèle de variables, un modèle de travail et un modèle de phase.