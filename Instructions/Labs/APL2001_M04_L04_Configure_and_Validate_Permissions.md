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

#### Tâche 1 : (à ignorer si vous l’avez déjà effectuée) Importer et exécuter le pipeline CI

> [!NOTE]
> Ignorez l’importation si vous l’avez déjà effectuée dans un autre labo.

Commencez par importer le pipeline CI nommé [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Accédez au Portail Azure DevOps sur `https://dev.azure.com` et ouvrez votre organisation.

1. Ouvrez le projet **eShopOnWeb**.

1. Accédez à **Pipelines > Pipelines**.

1. Sélectionnez le bouton **Nouveau pipeline**.

1. Sélectionnez **Azure Repos Git (Yaml)**.

1. Sélectionnez le référentiel **eShopOnWeb** .

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-ci.yml**, puis **Continuer**.

1. Sélectionnez le bouton **Exécuter** pour exécuter le pipeline.

1. Votre pipeline choisira un nom en fonction du nom du projet. Renommez-le pour mieux identifier le pipeline.

1. Accédez à **Pipelines > Pipelines**, sélectionnez le pipeline récemment créé, sélectionnez les points de suspension, puis sélectionnez l’option **Renommer/déplacer**.

1. Nommez-le **eshoponweb-ci**, puis sélectionnez **Enregistrer**.

### Tâche 2 : Configurer et exécuter le pipeline avec des autorisations spécifiques

Dans cette tâche, vous allez configurer le pipeline CI pour qu’il s’exécute avec un pool d’agents spécifique et valider les autorisations d’exécution du pipeline. Vous devez disposer des autorisations nécessaires pour modifier le pipeline et ajouter des autorisations au pool d’agents.

1. Accédez à Paramètres du projet, puis sélectionnez **Pools d’agents** sous **Pipelines**.

1. Ouvrez le pool d’agents **Par défaut**.

1. Sélectionnez l’onglet **Sécurité**.

1. S’il n’existe aucune restriction sur le pool d’agents, sélectionnez le bouton **Restreindre les autorisations**.

    ![Capture d’écran de l’onglet de sécurité du pool d’agents sans restrictions.](media/agent-pool-security-no-restriction.png)

1. Sélectionnez le bouton **Ajouter** et sélectionnez le pipeline **eshoponweb-ci** pour l’ajouter à la liste des pipelines ayant accès au pool d’agents.

1. Sélectionnez le bouton **Exécuter** pour exécuter le pipeline.

1. Ouvrez le pipeline en cours. Si vous voyez le message « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse continuer à générer la solution .Net Core », sélectionnez **Afficher**, **Autoriser** et à nouveau **Autoriser**.

Il doit être d’exécuter correctement le pipeline.

#### Tâche 3 : (à ignorer si vous l’avez déjà effectuée) Configurer le pipeline CD et valider les autorisations

> [!NOTE]
> Ignorez l’importation si vous l’avez déjà effectuée dans un autre labo.

> [!IMPORTANT]
> Si vous disposez d’autorisations, vous pourrez **autoriser** l’exécution du pipeline directement à partir du pipeline en cours d’exécution. Si vous n’avez pas d’autorisations, vous devez utiliser un autre compte avec des autorisations d’administration pour permettre à votre pipeline de s’exécuter à l’aide de l’agent spécifique, comme décrit dans la tâche 2 précédente, ou d’ajouter des autorisations utilisateur au pool d’agents.

1. Accédez à **Pipelines > Pipelines**.

1. Sélectionnez le bouton **Nouveau pipeline**.

1. Sélectionnez **Azure Repos Git (Yaml)**.

1. Sélectionnez le référentiel **eShopOnWeb** .

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-cd-webapp-code.yml**, puis **Continuer**.

1. Dans la définition du pipeline YAML, dans la section des variables, personnalisez :
   - **YOUR-SUBSCRIPTION-ID** par votre ID d’abonnement Azure ;
   - **az400eshop-NAME** par un nom d’application web à déployer avec un nom unique global, par exemple, **eshoponweb-lab-YOURNAME** ;
   - **AZ400-EWebShop-NAME** par le nom de votre préférence, par exemple **rg-eshoponweb**.

1. Mettez à jour le fichier YAML pour utiliser l’image **windows-latest** dans le pool d’agents hébergés par Microsoft **Par défaut**. Pour ce faire, définissez la section **pool** sur la valeur suivante :

    ```yaml
    pool: 
      vmImage: windows-latest

    ```

1. Sélectionnez **Enregistrer** puis **Exécuter**.

1. Ouvrez le pipeline et le message « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse continuer à déployer l’application web » s’affiche. Sélectionnez **Afficher**, puis **Autoriser** pour permettre au pipeline de s’exécuter.

    ![Capture d’écran du pipeline avec des boutons d’autorisation.](media/pipeline-permission-permit.png)

### Exercice 2 : Configurer et valider les vérifications d’approbation et de branche

Dans cet exercice, vous allez configurer et valider les vérifications d’approbation et de branche pour le pipeline CD.

#### Tâche 1 : Créer un environnement et ajouter des approbations et des vérifications

1. Accédez à **Pipelines > Environnements**.

1. Sélectionnez le bouton **Créer un environnement**.

1. Nommez l’environnement **Test**, sélectionnez **Aucun** comme ressource, puis sélectionnez **Créer**.

1. Sélectionnez **Nouvel environnement**, créez un environnement **Production**, sélectionnez **Aucun** comme ressource et sélectionnez **Créer**.

1. Ouvrez l’environnement **Test**, sélectionnez ***...*** et sélectionnez **Approbations et vérifications**.

1. Sélectionnez **Approbations**.

1. Dans la zone de texte **Approbateurs**, entrez votre nom d’utilisateur et, si vous avez un autre utilisateur, ajoutez-le pour valider le processus d’approbation.

1. Suivez les instructions de **Veuillez approuver le déploiement à tester** et sélectionnez **Créer**.

    ![Capture d’écran des approbations d’environnement avec des instructions.](media/add-environment-approvals.png)

1. Sélectionnez le bouton**+**, sélectionnez **Contrôle de branche**, puis sélectionnez **Suivant**.

1. Dans le champ **Branches autorisées** , conservez la valeur par défaut et sélectionnez **Créer**. Vous pouvez ajouter des actions supplémentaires si vous le souhaitez.

    ![Capture d’écran du contrôle de branche d’environnement avec la branche principale.](media/add-environment-branch-control.png)

1. Ouvrez l’environnement **Production** et suivez la même procédure pour ajouter des approbations et un contrôle de branche. Pour différencier les environnements, ajoutez les instructions **Veuillez approuver le déploiement en production** et ajouter la branche **refs/heads/main** aux branches autorisées.

1. (Facultatif) Vous pouvez ajouter d’autres environnements et configurer des approbations et un contrôle de branche pour eux. En outre, vous pouvez configurer **Sécurité** pour ajouter des utilisateurs ou des groupes à l’environnement.
    - Ouvrez l’environnement **Test**, sélectionnez ***...*** et sélectionnez **Sécurité**.
    - Sélectionnez **Ajouter** et sélectionner l’utilisateur qui exécute le pipeline, ainsi que le rôle *Utilisateur*, *Créateur* ou *Lecteur*.
    - Sélectionnez **Ajouter**.
    - Sélectionnez **Enregistrer**.

#### Tâche 2 : Configurer le pipeline CD pour utiliser le nouvel environnement

1. Accédez à **Pipelines > Pipelines**.

1. Ouvrez le pipeline **eshoponweb-cd-webapp-code**.

1. Sélectionnez **Modifier**.

1. Au-dessus du commentaire **#télécharger des artefacts**, ajoutez :

    ```yaml
    stages:
    - stage: Test
      displayName: Testing WebApp
      jobs:
      - deployment: Test
        pool:
          vmImage: 'windows-latest'
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
        pool: 
          vmImage: windows-latest
        environment: Production
        strategy:
          runOnce:
            deploy:
              steps:
              - checkout: self
    ```

    > [!NOTE]
    > Vous devez déplacer les lignes qui suivent le code ci-dessus de six espaces vers la droite pour vous assurer que les règles de retrait YAML sont satisfaites.

    Votre pipeline doit se présenter comme suit :

    ![Capture d’écran du pipeline avec le nouveau déploiement.](media/pipeline-add-yaml-deployment.png)

1. Sélectionnez **Enregistrer** et **Exécuter**.

1. Ouvrez le pipeline et le message « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse continuer à tester l’application web » s’affiche. Sélectionnez Afficher****, **Autoriser** et à nouveau **Autoriser**.

    ![Capture d’écran du pipeline avec le bouton Autoriser pour approuver l’exécution du pipeline ».](media/pipeline-environment-permit.png)

1. Ouvrez la phase **Test de l’application web**, et le message **1 approbation a besoin de votre évaluation avant que cette exécution puisse continuer à tester l’application web** s’affiche. Sélectionnez **Vérifier**, puis **Approuver**.

    ![Capture d’écran du pipeline avec la phase de test à approuver ».](media/pipeline-test-environment-approve.png)

1. Attendez que le pipeline se termine, ouvrez le journal du pipeline et vérifiez que la phase **Test de l’application web** a bien été exécutée.

    ![Capture d’écran du journal du pipeline avec l’étape Test de l’application web correctement exécutée.](media/pipeline-test-environment-success.png)

1. Revenez au pipeline et vous verrez l’étape **Déployer sur l’application web** en attente d’approbation. Sélectionnez **Vérifier** et **Approuver** comme vous l’avez fait avant la phase **Test de l’application web**.

1. Attendez que le pipeline se termine et vérifiez que la **Déployer sur l’application web** a bien été exécutée.

    ![Capture d’écran du pipeline avec la phase Déployer sur l’application web à approuver ».](media/pipeline-deploy-environment-success.png)

Vous devriez être en mesure d’exécuter le pipeline avec les vérifications d’approbation et de branche dans les deux environnements, Test et Production.

### Exercice 3 : Supprimer les ressources utilisées dans le labo

1. Dans le Portail Azure, ouvrez le groupe de ressources créé, puis cliquez sur **Supprimer le groupe de ressources** pour toutes les ressources créées dans ce labo.

    ![Capture d’écran du bouton Supprimer le groupe de ressources.](media/delete-resource-group.png)

    > [!WARNING]
    > N’oubliez pas de supprimer toutes les nouvelles ressources Azure que vous n’utilisez plus. La suppression des ressources inutilisées vous évitera d’encourir des frais inattendus.

1. Réinitialisez les autorisations spécifiques que vous avez ajoutées à l’organisation et au projet Azure DevOps dans ce labo.

## Révision

Dans ce labo, vous avez appris à configurer un environnement sécurisé qui respecte le principe du moindre privilège, en veillant à ce que les membres puissent accéder uniquement aux ressources dont ils ont besoin pour effectuer leurs tâches et réduire les risques de sécurité potentiels. Vous avez configuré et validé des autorisations d’utilisateur et de pipeline, et configuré des vérifications d’approbation et de branche dans Azure DevOps.
