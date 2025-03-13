---
lab:
  title: Configurer une structure de projet et de référentiel pour prendre en charge les pipelines sécurisés
  module: 'Module 1: Configure a project and repository structure to support secure pipelines'
---

# Configurer une structure de projet et de référentiel pour prendre en charge les pipelines sécurisés

Dans ce labo, vous allez découvrir comment configurer une structure de projets et de dépôts dans Azure DevOps pour prendre en charge les pipelines sécurisés. Ce labo décrit les bonnes pratiques d’organisation des projets et des dépôts, l’attribution des autorisations et la gestion des fichiers sécurisés.

Ces exercices prennent environ **30** minutes.

## Avant de commencer

Vous aurez besoin d’un abonnement Azure, d’une organisation Azure DevOps et de l’application eShopOnWeb pour suivre les labos.

- Procédez comme suit pour [valider votre environnement de labo](APL2001_M00_Validate_Lab_Environment.md).

## Instructions

### Exercice 1 : Configurer une structure de projet sécurisée

Dans cet exercice, vous allez configurer une structure de projet sécurisée en créant un projet et en lui attribuant des autorisations de projet. La séparation des responsabilités et des ressources en différents projets ou référentiels avec des autorisations spécifiques prend en charge la sécurité.

#### Tâche 1 : Créer un projet d’équipe

1. Accédez au Portail Azure DevOps sur `https://aex.dev.azure.com` et ouvrez votre organisation.

1. Ouvrez les **paramètres de votre organisation** en bas à gauche du portail, puis **Projets** sous la section Général.

1. Sélectionnez l’option **Nouveau projet** et utilisez les paramètres suivants :

   - nom : **eShopSecurity**
   - visibilité : **Privé**
   - Avancé : Contrôle de version : **Git**
   - Avancé : Processus d’élément de travail : **Scrum**

   ![Capture d’écran de la boîte de dialogue du nouveau projet avec les paramètres spécifiés.](media/new-team-project.png)

1. Sélectionnez **Créer** pour créer le nouveau projet.

1. Vous pouvez maintenant basculer entre les différents projets d’équipe en cliquant sur l’icône Azure DevOps dans le coin supérieur gauche du Portail Azure DevOps.

   ![Capture d’écran des projets d’équipe Azure DevOps eShopOnWeb et eShopSecurity.](media/azure-devops-projects.png)

Vous pouvez gérer les autorisations et les paramètres de chaque projet séparément en accédant au menu Paramètres du projet et en sélectionnant le projet d’équipe approprié. Si vous avez plusieurs utilisateurs ou équipes travaillant sur différents projets, vous pouvez également attribuer des autorisations à chaque projet séparément.

#### Tâche 2 : Créer un référentiel et attribuer des autorisations de projet

1. Sélectionnez le nom de l’organisation dans le coin supérieur gauche du Portail Azure DevOps, puis sélectionnez le nouveau projet **eShopSecurity** .

1. Sélectionnez le menu **Référentiels**.

1. Sélectionnez le bouton **Initialiser** pour initialiser le nouveau référentiel en ajoutant le fichier README.md.

1. Ouvrez le menu **Paramètres du projet** dans le coin inférieur gauche du portail et sélectionnez **Référentiels** sous la section Repos.

1. Sélectionnez le nouveau référentiel **eShopSecurity**, puis l’onglet **Sécurité**.

   > **Remarque** : vérifiez que vous sélectionnez l’onglet Sécurité dans le référentiel spécifique uniquement, et non pour tous les référentiels du projet. Si vous sélectionnez tous les référentiels, vous risquez de perdre l’accès à d’autres référentiels dans le projet.

1. Supprimez les autorisations Hériter du parent en décochant le bouton à bascule **Héritage**.

1. Sélectionnez le groupe **Contributeurs** et sélectionnez la liste déroulante **Refuser** pour toutes les autorisations, à l’exception de **Gestion des autorisations** et **Lecture**. Cela empêche tous les utilisateurs du groupe Contributeurs d’accéder au référentiel.

   > **Remarque** : dans un scénario réel, vous refuserez également les autorisations de gestion au groupe Contributeurs. Pour ce labo, nous permettons au groupe Contributeurs de gérer les autorisations pour vous permettre de terminer le labo.

1. Sélectionnez votre utilisateur sous Utilisateurs, puis le bouton **Autoriser** pour accepter toutes les autorisations.

   > **Remarque** : si vous ne voyez pas votre nom dans la section **Utilisateurs**, entrez-le dans la zone de texte **Rechercher des utilisateurs ou des groupes**, puis sélectionnez-le dans la liste des résultats.

   ![Capture d’écran des paramètres de sécurité du référentiel avec autorisation de lecture et de refus pour toutes les autres autorisations.](media/repository-security.png)

1. Vos modifications seront enregistrées automatiquement.

À présent, seul l’utilisateur auquel vous avez affecté des autorisations et les administrateurs peuvent accéder au référentiel. Cela est utile lorsque vous souhaitez autoriser des utilisateurs spécifiques à accéder au référentiel et à exécuter des pipelines à partir du projet eShopOnWeb.

### Exercice 2 : Configurer une structure de pipeline et de modèle pour prendre en charge des pipelines sécurisés

#### Tâche 1 : importer et exécuter le pipeline CI

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

#### Tâche 2 : importer et exécuter le pipeline CD

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

1. Sélectionnez **Enregistrer et Exécuter**, puis choisissez de valider directement dans la branche primaire.

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

Vous devez maintenant avoir deux pipelines en cours d’exécution dans votre projet eShopOnWeb.

![Capture d’écran des pipelines CI/CD correctement exécutés.](media/pipeline-successful-executed.png)

#### Tâche 3 : Déplacer les variables de pipeline CD vers un modèle YAML

Dans cette tâche, vous allez créer un modèle YAML pour stocker les variables utilisées dans le pipeline CD. Cela vous permet de réutiliser le modèle dans d’autres pipelines.

1. Accédez à **Repos**, puis à **Fichiers**.

1. Développez le dossier **.ado** et sélectionnez **Nouveau fichier**.

1. Nommez le fichier **eshoponweb-secure-variables.yml**, puis sélectionnez **Créer**.

1. Ajoutez la section des variables utilisée dans le pipeline CD au nouveau fichier. Le fichier doit se présenter comme suit :

   ```yaml
   variables:
     resource-group: 'rg-eshoponweb-secure'
     location: 'southcentralus' #the name of the Azure region you want to deploy your resources
     templateFile: 'infra/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs' #the name of the service connection to your Azure subscription
     webappname: 'eshoponweb-lab-secure-XXXXXX' #the globally unique name of the web app
   ```

   > **Important** : remplacez les valeurs des variables par les valeurs de votre environnement (groupe de ressources, emplacement, ID d’abonnement, connexion de service Azure et nom de l’application web).

1. Sélectionnez **Commiter**, dans la zone de texte de commentaire de commit, entrez `[skip ci]`, puis sélectionnez **Commiter**.

   > **Remarque** : en ajoutant le commentaire `[skip ci]` au commit, vous empêchez l’exécution automatique du pipeline qui, à ce stade, s’exécute par défaut après chaque modification du référentiel.

1. À partir de la liste des fichiers du dépôt, ouvrez la définition de pipeline **eshoponweb-cd-webapp-code.yml**, et remplacez la section des variables par les éléments suivants :

   ```yaml
   variables:
     - template: eshoponweb-secure-variables.yml
   ```

1. Sélectionnez **Commiter**, acceptez le commentaire par défaut, puis sélectionnez **Commiter** pour réexécuter le pipeline.

1. Vérifiez que l’exécution de pipeline s’effectue correctement.

Vous disposez maintenant d’un modèle YAML avec les variables utilisées dans le pipeline CD. Vous pouvez réutiliser ce modèle dans d’autres pipelines dans des scénarios où vous devez déployer les mêmes ressources. Par ailleurs, votre équipe d’opérations peut contrôler le groupe de ressources et l’emplacement où les ressources sont déployées ainsi que d’autres informations de vos valeurs de modèle, et vous n’avez pas besoin de faire des changements dans votre définition de pipeline.

#### Tâche 4 : Déplacer les modèles YAML vers un dépôt et un projet distincts

Dans cette tâche, vous allez déplacer les modèles YAML vers un référentiel et un projet distincts.

1. Dans votre projet eShopSecurity, accédez à **Repos > Fichiers**.

1. Créez un fichier nommé **eshoponweb-secure-variables.yml**.

1. Copiez le contenu du fichier **.ado/eshoponweb-secure-variables.yml** depuis le référentiel eShopOnWeb vers le nouveau fichier.

1. Validez les modifications :

1. Ouvrez la définition de pipeline **eshoponweb-cd-webapp-code.yml** dans le dépôt eShopOnWeb.

1. Ajoutez les éléments suivants à la section resources avant la section variables dans la définition du pipeline :

   ```yaml
     repositories:
       - repository: eShopSecurity
         type: git
         name: eShopSecurity/eShopSecurity #name of the project and repository
   ```

1. Remplacez la section des variables par ce qui suit :

   ```yaml
   variables:
     - template: eshoponweb-secure-variables.yml@eShopSecurity #name of the template and repository
   ```

   ![Capture d’écran de la définition de pipeline avec les nouvelles sections de variables et de ressources.](media/pipeline-variables-resource-section.png)

1. Sélectionnez **Commiter**, acceptez le commentaire par défaut, puis sélectionnez **Commiter** pour réexécuter le pipeline.

1. Accédez à l’exécution de pipeline et vérifiez que le pipeline utilise le fichier YAML du dépôt eShopSecurity.

   ![Capture d’écran de l’exécution du pipeline à l’aide du modèle YAML à partir du référentiel eShopSecurity.](media/pipeline-execution-using-template.png)

Vous avez maintenant le fichier YAML dans un dépôt et un projet distincts. Vous pouvez réutiliser ce fichier dans d’autres pipelines pour les scénarios où vous devez déployer les mêmes ressources. Par ailleurs, votre équipe d’opérations peut contrôler le groupe de ressources, la sécurité et l’emplacement où les ressources sont déployées ainsi que d’autres informations en modifiant les valeurs du fichier YAML, et vous n’avez pas besoin de faire des changements dans votre définition de pipeline.

> [!IMPORTANT]
> N’oubliez pas de supprimer les ressources créées dans le portail Azure pour éviter les frais inutiles.

## Révision

Dans ce labo, vous avez découvert la façon de configurer et d’organiser une structure de projet et de référentiel sécurisée dans Azure DevOps. En gérant efficacement les autorisations, vous pouvez vous assurer que les utilisateurs appropriés ont accès aux ressources dont ils ont besoin tout en maintenant la sécurité et l’intégrité de vos pipelines et processus DevOps.
