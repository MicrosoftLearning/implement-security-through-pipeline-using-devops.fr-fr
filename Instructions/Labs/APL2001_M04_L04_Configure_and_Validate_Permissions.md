---
lab:
  title: Configurer et valider des autorisations
  module: 'Module 4: Configure and validate permissions'
---

# Configurer et valider des autorisations

Dans ce labo, vous allez configurer un environnement sécurisé qui respecte le principe du moindre privilège, en veillant à ce que les membres puissent accéder uniquement aux ressources dont ils ont besoin pour effectuer leurs tâches et réduire les risques de sécurité potentiels. Cela implique de configurer et valider des autorisations d’utilisateur et de pipeline, et de configurer des vérifications d’approbation et de branche dans Azure DevOps.

Ces exercices prennent environ **30** minutes.

## Avant de commencer

Vous aurez besoin d’un abonnement Azure, d’une organisation Azure DevOps et de l’application eShopOnWeb pour suivre les labos.

- Procédez comme suit pour [valider votre environnement de labo](APL2001_M00_Validate_Lab_Environment.md).
- Installez un agent auto-hébergé en suivant le labo [Configurer des agents et des pools d’agents pour des pipelines sécurisés](/Instructions/Labs/APL2001_M03_L03_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md) ou les étapes de [Installer un agent auto-hébergé](https://docs.microsoft.com/azure/devops/pipelines/agents/v2-windows?view=azure-devops#install).

## Instructions

### Exercice 1 : Importer un pipeline CI et configurer des autorisations propres au pipeline

Dans cet exercice, vous allez importer et exécuter le pipeline CI pour l’application eShopOnWeb, puis configurer des autorisations propres au pipeline.

#### Tâche 1 :  Importer et exécuter le pipeline CI

> [!NOTE]
> Commencez par importer le pipeline CI nommé [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Accédez au Portail Azure DevOps sur `https://dev.azure.com` et ouvrez votre organisation.

1. Ouvrez le projet **eShopOnWeb**.

1. Accédez à **Pipelines > Pipelines**.

1. Sélectionnez **Nouveau pipeline**.

1. Sélectionnez **Azure Repos Git (YAML)**.

1. Sélectionnez le référentiel **eShopOnWeb**.

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-ci.yml**, puis **Continuer**.

1. Sélectionnez le bouton **Exécuter** pour exécuter le pipeline.

   > [!NOTE]
   > Votre pipeline choisira un nom en fonction du nom du projet. Renommez-le pour mieux identifier le pipeline.

1. Accédez à **Pipelines > Pipelines**, sélectionnez le pipeline récemment créé, sélectionnez les points de suspension, puis sélectionnez l’option **Renommer/déplacer**.

1. Nommez-le **eshoponweb-ci**, puis sélectionnez **Enregistrer**.

#### Tâche 2 : Configurer et exécuter le pipeline avec des autorisations spécifiques

> [!NOTE]
> Pour utiliser le pool d’agents configuré dans cette tâche, vous devez d’abord démarrer la machine virtuelle Azure hébergeant l’agent. 

1. Dans votre navigateur, ouvrez le portail Azure sur `https://portal.azure.com`.

1. Dans le portail Microsoft Azure, accédez à la page affichant la machine virtuelle Azure **eshoponweb-vm** que vous avez déployée dans ce labo

1. Dans la page de la machine virtuelle Azure **eshoponweb-vm**, dans la barre d’outils, sélectionnez **Démarrer** pour la démarrer.

   > [!NOTE]
   > Ensuite, vous allez configurer le pipeline CI pour qu’il s’exécute avec le pool d’agents correspondant et valider les autorisations d’exécution du pipeline. Vous devez disposer des autorisations nécessaires pour modifier le pipeline et ajouter des autorisations au pool d’agents.

1. Accédez à Paramètres du projet, puis sélectionnez **Pools d’agents** sous **Pipelines**.

1. Ouvrez le pool d’agents **eShopOnWebSelfPool**.

1. Sélectionnez l’onglet **Sécurité**.

1. Dans la section **Autorisations du pipeline**, sélectionnez le bouton**+**, puis sélectionnez le pipeline **eshoponweb-ci** pour l’ajouter à la liste des pipelines ayant accès au pool d’agents.

1. Accédez à la page du projet **eShopOnWeb**.

1. Dans la page du projet **eShopOnWeb**, accédez à **Pipelines > Pipelines**.

1. Sélectionnez le pipeline **eshoponweb-ci**, puis sélectionnez **Modifier**.

1. Dans la sous-section **travaux** de la section **étapes**, mettez à jour la valeur de la propriété **pool** pour référencer le pool d’agents auto-hébergé **eShopOnWebSelfPool** que vous avez configuré dans cette tâche, afin qu’il ait le format suivant :

   ```yaml
     jobs:
     - job: Build
       pool: eShopOnWebSelfPool
       steps:
       - task: DotNetCoreCLI@2
   ```

1. Sélectionnez **Enregistrer**, puis choisissez de valider directement dans la branche principale.

1. Sélectionnez **Enregistrer** à nouveau.

1. Sélectionnez **Exécuter** le pipeline, puis cliquez à nouveau sur **Exécuter**.

1. Vérifiez que le travail de génération s’exécute sur l’agent **eShopOnWebSelfAgent** et qu’il s’exécute correctement.

#### Tâche 3 : Configurer le pipeline CD et valider les autorisations

1. Dans le portail Azure DevOps, dans la page du projet **eShopOnWeb**, accédez à **Pipelines > Pipelines**.

1. Sélectionnez **Nouveau pipeline**.

1. Sélectionnez **Azure Repos Git (YAML)**.

1. Sélectionnez le référentiel **eShopOnWeb**.

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-cd-webapp-code.yml**, puis **Continuer**.

1. Dans la définition du pipeline YAML, dans la section des variables, personnalisez :

   - **AZ400-EWebShop-NAME** avec le nom de votre préférence, par exemple, **rg-eshoponweb-perm**.
   - **Emplacement** par le nom de la région Azure dans laquelle vous souhaitez déployer vos ressources, par exemple, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** par votre ID d’abonnement Azure ;
   - **sous-réseaux azure** avec **sous-réseaux azure managés**
   - **az400-webapp-NAME** avec un nom global unique de l’application web à déployer, par exemple, la chaîne **eshoponweb-lab-perm-** suivie d’un nombre à six chiffres aléatoire. 

1. Mettez à jour le fichier YAML pour utiliser le pool d’agents **eShopOnWebSelfPool**. Pour ce faire, définissez la section du **pool** sur la valeur suivante :

   ```yaml
     jobs:
     - job: Deploy
       pool: eShopOnWebSelfPool
       steps:
       #download artifacts
       - download: eshoponweb-ci
   ```

1. Sélectionnez **Enregistrer et exécuter**, puis sélectionnez **Enregistrer et réexécuter**.

1. Ouvrez le pipeline et notez le message « Ce pipeline a besoin d’une autorisation pour accéder à deux ressources avant que cette exécution puisse continuer à déployer sur WebApp ». Sélectionnez **Afficher**, puis **Autoriser** pour permettre au pipeline de s’exécuter.

   ![Capture d’écran du pipeline avec des boutons d’autorisation.](media/pipeline-permission-permit.png)

1. Renommez le pipeline en **eshoponweb-cd-webapp-code**.

### Exercice 2 : Configurer et valider les vérifications d’approbation et de branche

Dans cet exercice, vous allez configurer et valider les vérifications d’approbation et de branche pour le pipeline CD.

#### Tâche 1 : Créer un environnement et ajouter des approbations et des vérifications

1. Dans le portail Azure DevOps, dans la page du projet **eShopOnWeb**, sélectionnez **Pipelines > Environnements**.

1. Sélectionnez **Créer un environnement**.

1. Nommez l’environnement **Test**, sélectionnez **Aucun** comme ressource, puis sélectionnez **Créer**.

1. Sélectionnez **Nouvel environnement**, créez un environnement **Production**, vérifiez que **Aucun** est sélectionné comme ressource et sélectionnez **Créer**.

1. Ouvrez l’environnement **Test**, sélectionnez l’onglet **Approbations et vérifications**.

1. Sélectionnez **Approbations**.

1. Dans la zone de texte **Approbateurs**, entrez votre nom d’utilisateur.

1. Donnez les instructions **Approuver le déploiement sur Test** et sélectionnez **Créer**.

   ![Capture d’écran des approbations d’environnement avec des instructions.](media/add-environment-approvals.png)

1. Sélectionnez le bouton**+**, sélectionnez **Contrôle de branche**, puis sélectionnez **Suivant**.

1. Dans le champ **Branches autorisées** , conservez la valeur par défaut et sélectionnez **Créer**. Vous pouvez ajouter des actions supplémentaires si vous le souhaitez.

   ![Capture d’écran du contrôle de branche d’environnement avec la branche principale.](media/add-environment-branch-control.png)

1. Créez un autre environnement nommé **Production** et effectuez les mêmes étapes pour ajouter des approbations et un contrôle de branche. Pour différencier les environnements, ajoutez les instructions **Approuver le déploiement sur Production** et définissez les branches autorisées sur **refs/heads/main**.

> [!NOTE]
> Vous pouvez ajouter d’autres environnements et configurer des approbations et un contrôle de branche pour eux. En outre, vous pouvez configurer **Sécurité** pour ajouter des utilisateurs ou des groupes à l’environnement avec des rôles tels que *Utilisateur*, *Créateur* ou *Lecteur*.

#### Tâche 2 : Configurer le pipeline CD pour utiliser le nouvel environnement

1. Dans le portail Azure DevOps, dans la page du projet **eShopOnWeb**, sélectionnez **Pipelines > Pipelines**.

1. Ouvrez le pipeline **eshoponweb-cd-webapp-code**.

1. Sélectionnez **Modifier**.

1. Remplacez les lignes 21 à 27 (directement au-dessus du commentaire **#télécharger des artefacts**) par le contenu suivant :

   ```yaml
   stages:
   - stage: Test
     displayName: Testing WebApp
     jobs:
     - deployment: Test
       pool: eShopOnWebSelfPool
       environment: Test
       strategy:
         runOnce:
           deploy:
             steps:
             - script: echo Hello world! Testing environments!
   - stage: Deploy
     displayName: Deploy to WebApp
     jobs:
     - deployment: Deploy
       pool: eShopOnWebSelfPool
       environment: Production
       strategy:
         runOnce:
           deploy:
             steps:
             - checkout: self
   ```

   > [!NOTE]
   > Vous devrez décaler toutes les lignes suivant le code ci-dessus de six espaces vers la droite pour vous assurer que les règles d’indentation de YAML sont respectées.

   Votre pipeline doit se présenter comme suit :

   ![Capture d’écran du pipeline avec le nouveau déploiement.](media/pipeline-add-yaml-deployment.png)

1. Sélectionnez **Enregistrer** (deux fois) et **Exécuter** (deux fois).

1. Ouvrez l’étape **Test WebApp** du pipeline et prêtez attention au message **Une approbation a besoin de votre révision avant que cette exécution puisse continuer à tester WebApp**. Sélectionnez **Vérifier**, puis **Approuver**.

   ![Capture d’écran du pipeline avec la phase de test à approuver ».](media/pipeline-test-environment-approve.png)

1. Attendez que le pipeline se termine, ouvrez le journal du pipeline et vérifiez que la phase **Test de l’application web** a bien été exécutée.

   ![Capture d’écran du journal du pipeline avec l’étape Test de l’application web correctement exécutée.](media/pipeline-test-environment-success.png)

1. Revenez au pipeline et vous verrez l’étape **Déployer sur l’application web** en attente d’approbation. Sélectionnez **Vérifier** et **Approuver** comme vous l’avez fait avant la phase **Test de l’application web**.

1. Attendez que le pipeline se termine et vérifiez que l’étape **Déployer sur WebApp** a été exécutée avec succès.

   ![Capture d’écran du pipeline avec la phase Déployer sur l’application web à approuver ».](media/pipeline-deploy-environment-success.png)

> [!NOTE]
> Vous devriez être en mesure d’exécuter le pipeline avec les vérifications d’approbation et de branche dans les deux environnements, Test et Production.

### Exercice 3 : Effectuer le nettoyage des ressources Azure et Azure DevOps

Dans cet exercice, vous allez supprimer les ressources Azure et Azure DevOps créées dans ce labo.

#### Tâche 1 : Supprimer les ressources Azure

1. Dans le portail Microsoft Azure, accédez au groupe de ressources **rg-eshoponweb-perm** contenant les ressources déployées et sélectionnez **Supprimer le groupe de ressources** pour supprimer toutes les ressources créées dans ce labo.

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

Dans ce labo, vous avez appris à configurer un environnement sécurisé qui respecte le principe du moindre privilège, en veillant à ce que les membres puissent accéder uniquement aux ressources dont ils ont besoin pour effectuer leurs tâches et réduire les risques de sécurité potentiels. Vous avez configuré et validé des autorisations d’utilisateur et de pipeline, et configuré des vérifications d’approbation et de branche dans Azure DevOps.
