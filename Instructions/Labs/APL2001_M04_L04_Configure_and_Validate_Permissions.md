---
lab:
  title: Configurer et valider des autorisations
  module: 'Module 4: Configure and validate permissions'
---

# Configurer et valider des autorisations

Dans ce labo, vous allez configurer un environnement sécurisé qui respecte le principe du moindre privilège, en veillant à ce que les membres puissent accéder uniquement aux ressources dont ils ont besoin pour effectuer leurs tâches et réduire les risques de sécurité potentiels. Cela implique de configurer et valider des autorisations d’utilisateur et de pipeline, et de configurer des vérifications d’approbation et de branche dans Azure DevOps.

Ces exercices prennent environ **20** minutes.

## Avant de commencer

Vous aurez besoin d’un abonnement Azure, d’une organisation Azure DevOps et de l’application eShopOnWeb pour suivre les labos.

- Procédez comme suit pour [valider votre environnement de labo](APL2001_M00_Validate_Lab_Environment.md).
- Installez un agent auto-hébergé en suivant le labo [Configurer des agents et des pools d’agents pour des pipelines sécurisés](APL2001_M02_L02_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md) ou les étapes de [Installer un agent auto-hébergé](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent).

## Instructions

### Exercice 0 : (ignorer si déjà effectué) importer et exécuter des pipelines CI/CD

Dans cet exercice, vous allez importer et exécuter des pipelines CI/CD dans le projet Azure DevOps.

#### Tâche 1 : (ignorer si déjà effectuée) importer et exécuter le pipeline CI

Commençons par importer le pipeline CI nommé [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Accédez au Portail Azure DevOps sur `https://aex.dev.azure.com` et ouvrez votre organisation.

1. Ouvrez le projet **eShopOnWeb** dans Azure DevOps.

1. Accédez à **Pipelines > Pipelines**.

1. Sélectionnez le bouton **Créer un pipeline**.

1. Sélectionnez **Azure Repos Git (Yaml)**.

1. Sélectionnez le référentiel **eShopOnWeb**.

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-ci.yml**, puis cliquez sur **Continuer**.

1. Sélectionnez le bouton **Exécuter** pour exécuter le pipeline.

   > **Remarque** : votre pipeline est nommé en fonction du nom du projet. Vous le renommez pour identifier plus facilement le pipeline.

1. Accédez à **Pipelines > Pipelines** et sélectionnez le pipeline récemment créé. Sélectionnez les points de suspension puis **Renommer/déplacer**.

1. Nommez-le **eshoponweb-ci**, puis sélectionnez **Enregistrer**.

#### Tâche 2 : (ignorer si déjà effectuée) importer et exécuter le pipeline CD

> **Remarque** : dans cette tâche, vous allez importer et exécuter le pipeline CD appelé [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Accédez à **Pipelines > Pipelines**.

1. Sélectionnez le bouton **Nouveau pipeline**.

1. Sélectionnez **Azure Repos Git (Yaml)**.

1. Sélectionnez le référentiel **eShopOnWeb**.

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-cd-webapp-code.yml**, puis **Continuer**.

1. Dans la définition du pipeline YAML, définissez la section des variables pour effectuer les opérations suivantes :

   ```yaml
   variables:
     resource-group: 'YOUR-RESOURCE-GROUP-NAME'
     location: 'centralus'
     templateFile: 'infra/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
     webappname: 'YOUR-WEB-APP-NAME'
   ```

1. Remplacez les valeurs des variables par les valeurs de votre environnement :

   - Remplacez **YOUR-RESOURCE-GROUP-NAME** par le nom du groupe de ressources que vous souhaitez utiliser dans ce labo, par exemple, **rg-eshoponweb-secure**.
   - Définissez la valeur de la variable d’**emplacement** sur le nom de la région Azure dans laquelle vous souhaitez déployer vos ressources, par exemple, **centralus**.
   - Remplacez **YOUR-SUBSCRIPTION-ID** par votre ID d’abonnement Azure.
   - Remplacez **YOUR-AZURE-SERVICE-CONNECTION-NAME** par **azure subs**.
   - Remplacez **YOUR-WEB-APP-NAME** par un nom unique au monde de l’application web à déployer, par exemple, la chaîne **eshoponweb-lab-multi-123456** suivie d’un nombre aléatoire à six chiffres.

1. Sélectionnez **Enregistrer et exécuter**, puis choisissez de valider directement dans la branche principale.

1. Sélectionnez de nouveau **Enregistrer et exécuter**.

1. Ouvrez l’exécution de pipeline. Si vous voyez le message « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse poursuivre le déploiement vers l’application web », sélectionnez **Afficher**, **Autoriser** et à nouveau **Autoriser**. Cette opération est nécessaire pour permettre au pipeline de créer la ressource Azure App Service.

   ![Capture d’écran de l’autorisation d’accès à partir du pipeline YAML.](media/pipeline-deploy-permit-resource.png)

1. Le déploiement peut prendre quelques minutes, attendez que le pipeline s’exécute. Le pipeline est déclenché après l’achèvement du pipeline CI et comprend les tâches suivantes :

   - **AzureResourceManagerTemplateDeployment** : Déploie l’application web Azure App Service à partir d’un modèle bicep.
   - **AzureRmWebAppDeployment** : Publie le site web sur l’application web Azure App Service.

   > **Remarque** : si le déploiement échoue, accédez à la page d’exécution du pipeline et sélectionnez **Réexécuter les travaux ayant échoué** pour appeler une autre exécution de pipeline.

   > **Remarque** : votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline.

1. Accédez à **Pipelines > Pipelines** et sélectionnez le pipeline récemment créé. Sélectionnez les points de suspension puis **Renommer/déplacer**.

1. Nommez-le **eshoponweb-cd-webapp-code**, puis cliquez sur **Enregistrer**.

### Exercice 1 : configurer et valider les vérifications d’approbation et de branche

Dans cet exercice, vous allez configurer et valider les vérifications d’approbation et de branche pour le pipeline CD.

#### Tâche 1 : Créer un environnement et ajouter des approbations et des vérifications

1. Dans le portail Azure DevOps, dans la page du projet **eShopOnWeb**, sélectionnez **Pipelines > Environnements**.

1. Sélectionnez **Créer un environnement**.

1. Nommez l’environnement **Test**, sélectionnez **Aucun** comme ressource, puis sélectionnez **Créer**.

1. Dans l’environnement **Test**, sélectionnez l’onglet **Approbations et vérifications**.

1. Sélectionnez **Approbations**.

1. Dans la zone de texte **Approbateurs**, entrez votre nom d’utilisateur.

1. Si ce n’est pas fait, cochez la case intitulée « Autoriser les approbateurs à approuver leurs propres exécutions ».

1. Donnez les instructions **Approuver le déploiement sur Test** et sélectionnez **Créer**.

   ![Capture d’écran des approbations d’environnement avec des instructions.](media/add-environment-approvals.png)

1. Cliquez sur le bouton **+ Ajouter**, sélectionnez **Contrôle de branche**, puis sélectionnez **Suivant**.

1. Dans le champ **Branches autorisées** , conservez la valeur par défaut et sélectionnez **Créer**. Vous pouvez ajouter des actions supplémentaires si vous le souhaitez.

   ![Capture d’écran du contrôle de branche d’environnement avec la branche principale.](media/add-environment-branch-control.png)

1. Créez un autre environnement nommé **Production** et effectuez les mêmes étapes pour ajouter des approbations et un contrôle de branche. Pour différencier les environnements, ajoutez les instructions **Approuver le déploiement sur Production** et définissez les branches autorisées sur **refs/heads/main**.

> **Remarque** : vous pouvez ajouter d’autres environnements et configurer des approbations et un contrôle de branche pour eux. En outre, vous pouvez configurer **Sécurité** pour ajouter des utilisateurs ou des groupes à l’environnement avec des rôles tels que *Utilisateur*, *Créateur* ou *Lecteur*.

#### Tâche 2 : Configurer le pipeline CD pour utiliser le nouvel environnement

1. Dans le portail Azure DevOps, dans la page du projet **eShopOnWeb**, sélectionnez **Pipelines > Pipelines**.

1. Ouvrez le pipeline **eshoponweb-cd-webapp-code**.

1. Sélectionnez **Modifier**.

1. Sélectionnez la ligne située au-dessus du commentaire ** #download artifacts**, jusqu’à la ligne **stages:** dans le fichier YAML du pipeline et remplacez le contenu par le code suivant :

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

   > **Remarque** : vous devrez décaler toutes les lignes suivant le code ci-dessus de six espaces vers la droite pour vous assurer que les règles de mise en retrait de YAML sont respectées.

   Votre pipeline doit se présenter comme suit :

   ![Capture d’écran du pipeline avec le nouveau déploiement.](media/pipeline-add-yaml-deployment.png)

   > [!IMPORTANT]
   > Vérifiez que le nom du **pool** est identique à celui que vous avez créé dans le labo précédent.

1. Cliquez sur **Valider et enregistrer**, choisissez de valider directement sur la branche primaire, puis cliquez sur **Enregistrer**.

1. Votre pipeline se déclenche automatiquement. Ouvrez l’exécution de pipeline.

   > **Remarque** : si vous recevez le message « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse poursuivre le test de l’application web », sélectionnez **Afficher**, **Autoriser** et à nouveau **Autoriser**.

1. Ouvrez l’étape **Test WebApp** du pipeline et prêtez attention au message **Une approbation a besoin de votre révision avant que cette exécution puisse continuer à tester WebApp**. Sélectionnez **Vérifier**, puis **Approuver**.

   ![Capture d’écran du pipeline avec la phase de test à approuver ».](media/pipeline-test-environment-approve.png)

1. Attendez que le pipeline se termine, ouvrez le journal du pipeline et vérifiez que la phase **Test de l’application web** a bien été exécutée.

   ![Capture d’écran du journal du pipeline avec l’étape Test de l’application web correctement exécutée.](media/pipeline-test-environment-success.png)

1. Revenez au pipeline et vous verrez l’étape **Déployer sur l’application web** en attente d’approbation. Sélectionnez **Vérifier** et **Approuver** comme vous l’avez fait avant la phase **Test de l’application web**.

   > **Remarque** : si vous recevez le message « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse poursuivre le déploiement vers l’application web », sélectionnez **Afficher**, **Autoriser** et à nouveau **Autoriser**.

1. Attendez que le pipeline se termine et vérifiez que l’étape **Déployer sur WebApp** a été exécutée avec succès.

   ![Capture d’écran du pipeline avec la phase Déployer sur l’application web à approuver ».](media/pipeline-deploy-environment-success.png)

> **Remarque** : vous devriez être en mesure d’exécuter le pipeline avec les vérifications d’approbation et de branche dans les deux environnements, Test et Production.

> [!IMPORTANT]
> N’oubliez pas de supprimer les ressources créées dans le portail Azure pour éviter les frais inutiles.

## Révision

Dans ce labo, vous avez appris à configurer un environnement sécurisé qui respecte le principe du moindre privilège, en veillant à ce que les membres puissent accéder uniquement aux ressources dont ils ont besoin pour effectuer leurs tâches et réduire les risques de sécurité potentiels. Vous avez configuré et validé des autorisations d’utilisateur et de pipeline, et configuré des vérifications d’approbation et de branche dans Azure DevOps.
