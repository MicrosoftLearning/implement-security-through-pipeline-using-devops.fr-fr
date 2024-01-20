---
lab:
  title: Identité gérée pour les projets et les pipelines
  module: 'Module 2: Manage identity for projects, pipelines, and agents'
---

# Identité gérée pour les projets et les pipelines

Les identités managées offrent une méthode sécurisée pour contrôler l’accès aux ressources Azure. Azure gère automatiquement ces identités, ce qui vous permet de vérifier l’accès aux services compatibles avec l’authentification Azure AD. Vous n’avez donc pas besoin d’incorporer des informations d’identification dans votre code, ce qui améliore la sécurité. Dans Azure DevOps, les identités managées peuvent authentifier les ressources Azure au sein de vos agents auto-hébergés, ce qui simplifie le contrôle d’accès sans compromettre la sécurité.

Dans ce labo, vous allez créer une identité managée et l’utiliser dans des pipelines YAML Azure DevOps exécutés sur des agents auto-hébergés pour déployer des ressources Azure.

Le labo prend environ **45** minutes.

## Avant de commencer

Vous aurez besoin d’un abonnement Azure, d’une organisation Azure DevOps et de l’application eShopOnWeb pour suivre les labos.

- Procédez comme suit pour [valider votre environnement de labo](APL2001_M00_Validate_Lab_Environment.md).
- Vérifiez que vous disposez d'un compte Microsoft ou d'un compte Microsoft Entra avec le rôle Contributeur ou Propriétaire dans l'abonnement Azure. Pour plus d’informations, consultez [Répertorier les attributions de rôle Azure à l’aide du portail Azure](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) et [Afficher et attribuer des rôles d’administrateur dans Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).
- Suivez le labo [Configurer des agents et des pools d’agents pour des pipelines sécurisés](APL2001_M03_L02_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md).

## Instructions

### Exercice 1 : Importer et exécuter des pipelines CI/CD

Dans cet exercice, vous allez importer et exécuter le pipeline CI, configurer la connexion de service avec votre abonnement Azure, puis importer et exécuter le pipeline CD.

#### Tâche 1 : importer et exécuter le pipeline CI

Commençons par importer le pipeline CI nommé [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Accédez au Portail Azure DevOps sur `https://dev.azure.com` et ouvrez votre organisation.

1. Ouvrez le projet **eShopOnWeb**.

1. Accédez à **Pipelines > Pipelines**.

1. Sélectionnez le bouton **Créer un pipeline**.

1. Sélectionnez **Azure Repos Git (YAML)**.

1. Sélectionnez le référentiel **eShopOnWeb**.

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-ci.yml**, puis cliquez sur **Continuer**.

1. Sélectionnez le bouton **Exécuter** pour exécuter le pipeline.

   > [!NOTE]
   > Votre pipeline choisira un nom en fonction du nom du projet. Renommez-le pour mieux identifier le pipeline.

1. Accédez à **Pipelines > Pipelines**, sélectionnez le pipeline récemment créé, sélectionnez les points de suspension, puis sélectionnez l’option **Renommer/déplacer**.

1. Nommez-le **eshoponweb-ci**, puis sélectionnez **Enregistrer**.

#### Tâche 2 : importer et exécuter le pipeline CD

> [!NOTE]
> Dans cette tâche, vous allez importer et exécuter le pipeline CD appelé [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Dans le volet **Pipelines** du projet **eShopOnWeb**, sélectionnez le bouton **Nouveau pipeline**.

1. Sélectionnez **Azure Repos Git (YAML)**.

1. Sélectionnez le référentiel **eShopOnWeb**.

1. Sélectionnez **Fichier YAML Azure Pipelines existant**.

1. Sélectionnez le fichier **/.ado/eshoponweb-cd-webapp-code.yml**, puis **Continuer**.

1. Dans la définition du pipeline YAML, définissez la section des variables pour effectuer les opérations suivantes :

   ```yaml
   variables:
     resource-group: 'AZ400-EWebShop-NAME'
     location: 'southcentralus'
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs'
     webappname: 'az400-webapp-NAME'
   ```

1. Dans la section des variables, remplacez les espaces réservés par les valeurs suivantes :

   - **AZ400-EWebShop-NAME** par le nom de votre préférence, par exemple **rg-eshoponweb**.
   - **emplacement** par le nom de la région Azure dans laquelle vous souhaitez déployer vos ressources, par exemple, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** par votre ID d’abonnement Azure ;
   - **az400-webapp-NAME**, avec un nom global unique de l’application web à déployer, par exemple, la chaîne **eshoponweb-lab-id-** suivie d’un nombre à six chiffres aléatoire. 

1. Sélectionnez **Enregistrer et exécuter**, puis choisissez de valider directement dans la branche principale.

1. Sélectionnez de nouveau **Enregistrer et exécuter**.

1. Ouvrez le pipeline. Si vous voyez le message « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse poursuivre le déploiement vers l’application web », sélectionnez **Afficher**, **Autoriser** et à nouveau **Autoriser**. Cette opération est nécessaire pour permettre au pipeline de créer la ressource Azure App Service.

   ![Capture d’écran de l’autorisation d’accès à partir du pipeline YAML.](media/pipeline-deploy-permit-resource.png)

1. Le déploiement peut prendre quelques minutes, attendez que le pipeline s’exécute. La définition CD se compose des tâches suivantes :

   - **AzureResourceManagerTemplateDeployment** : Déploie l’application web Azure App Service à l’aide du modèle bicep.
   - **AzureRmWebAppDeployment** : Publie le site web sur l’application web Azure App Service.

   > [!NOTE]
   > Si le déploiement échoue, accédez à la page d’exécution du pipeline et sélectionnez **Réexécuter les travaux non réussis** pour appeler une autre exécution de pipeline.

   > [!NOTE]
   > Votre pipeline est nommé en fonction du nom du projet. **Renommons**-le pour mieux identifier le pipeline.

1. Accédez à **Pipelines > Pipelines**, sélectionnez le pipeline récemment créé, sélectionnez les points de suspension, puis sélectionnez l’option **Renommer/déplacer**.

1. Nommez-le **eshoponweb-cd-webapp-code**, puis sélectionnez **Enregistrer**.

### Exercice 2 : Configurer une identité managée dans des pipelines Azure

Dans cet exercice, vous allez utiliser une identité managée pour configurer une nouvelle connexion de service et l’incorporer dans les pipelines CI/CD.

#### Tâche 1 : Configurer un agent auto-hébergé pour utiliser l’identité managée et mettre à jour le pipeline CI

1. Dans votre navigateur, ouvrez le portail Azure sur `https://portal.azure.com`.

1. Dans le portail Azure, accédez à la page affichant la machine virtuelle Azure **eshoponweb-vm** que vous avez déployée dans ce labo

1. Dans la page de la machine virtuelle Azure **eshoponweb-vm**, dans la barre d’outils, sélectionnez **Démarrer** pour la démarrer.

1. Dans la page de la machine virtuelle Azure **eshoponweb-vm**, dans le menu vertical situé à gauche, dans la section **Sécurité**, sélectionnez **Identité**.

1. Dans la page **eshoponweb-vm /|Identité**, vérifiez que l’**État** est sur **Activé** et sélectionnez **Attributions des rôles Azure**.

1. Sélectionnez le bouton **Ajouter une attribution de rôle** et effectuez les actions suivantes :

   | Setting | Action |
   | -- | -- |
   | Liste déroulante **Portée** | Sélectionnez **Abonnement**. |
   | Liste déroulante  **Abonnement** | Sélectionnez votre abonnement Azure. |
   | Liste déroulante **Rôle** | Sélectionnez le rôle **Contributeur**. |

   > [!NOTE]
   > L’étendue au niveau abonnement est nécessaire pour prendre en charge les déploiements dans les labos suivants.

1. Cliquez sur le bouton **Enregistrer**.

    ![Capture d’écran du volet d’ajout d’attribution de rôle](media/add-role-assignment.png)

#### Tâche 2 : Créer une connexion de service basée sur l’identité managée

1. Basculez vers le navigateur web affichant le projet **eShopOnWeb** dans le portail Azure DevOps sur `https://dev.azure.com`.

1. Dans le projet **eShopOnWeb**, accédez à **Paramètres du projet > Connexions de service**.

1. Sélectionnez le bouton **Nouvelle connexion de service**, puis **Azure Resource Manager**.

1. Comme **méthode d’authentification**, sélectionnez **Identité managée**.

1. Définissez le niveau d’étendue sur **Abonnement** et fournissez les informations collectées pendant la phase de validation de votre environnement lab, notamment l’ID de l’abonnement, le nom de l’abonnement et l’ID du locataire. 

1. Dans **Nom de la connexion de service** tapez **azure subs managed**. Ce nom est référencé dans les pipelines YAML lors de l’accès à votre abonnement Azure.

1. Sélectionnez **Enregistrer**.

#### Tâche 3 : Mettre à jour le pipeline CD

1. Basculez vers la fenêtre de navigateur affichant le projet **eShopOnWeb** dans le portail Azure DevOps.

1. Dans la page du projet **eShopOnWeb**, accédez à **Pipelines > Pipelines**.

1. Sélectionnez le pipeline **eshoponweb-cd-webapp-code**, puis sélectionnez **Modifier**.

1. Dans la section des variables, mettez à jour la variable **serviceConnection** pour utiliser le nom de la connexion de service que vous avez créée dans la tâche précédente, **azure subs managed**.

   ```yaml
     azureserviceconnection: 'azure subs managed'
   ```

1. Dans la sous-section **travaux**de la section **phases**, mettez à jour la valeur de la propriété **pool** pour référencer le pool d’agents auto-hébergé que vous avez créé dans le labo précédent, **eShopOnWebSelfPool** afin de lui donner le format suivant :

   ```yaml
     jobs:
     - job: Deploy
       pool: eShopOnWebSelfPool
       steps:
       #download artifacts
       - download: eshoponweb-ci
   ```

1. Sélectionnez **Enregistrer**, puis choisissez de valider directement dans la branche principale.

1. Sélectionnez **Enregistrer** à nouveau.

1. Sélectionnez **Exécuter** le pipeline, puis cliquez à nouveau sur **Exécuter**.

1. Ouvrez le pipeline. Si vous voyez le message « Ce pipeline a besoin d’une autorisation pour accéder à une ressource avant que cette exécution puisse poursuivre le déploiement vers l’application web », sélectionnez **Afficher**, **Autoriser** et à nouveau **Autoriser**. Cette opération est nécessaire pour permettre au pipeline de créer la ressource Azure App Service.

1. Le déploiement peut prendre quelques minutes, attendez que le pipeline s’exécute.

1. Vous devriez voir dans les journaux de pipeline que le pipeline utilise l’identité managée.

   ![Capture d’écran des journaux de pipeline utilisant l’identité managée.](media/pipeline-logs-managed-identity.png)

   > [!NOTE]
   > Une fois le pipeline terminé, vous pouvez utiliser le portail Azure pour vérifier l’état des ressources de l’application web App Service.

### Exercice 3 : Effectuer le nettoyage des ressources Azure et Azure DevOps

Dans cet exercice, vous allez effectuer le nettoyage post-labo des ressources Azure et Azure DevOps créées dans ce labo.

#### Tâche 1 : Arrêter et libérer la machine virtuelle Azure

1. Accédez à la page affichant le groupe de ressources **rg-eshoponweb** et sélectionnez **Supprimer le groupe de ressources** pour supprimer toutes ses ressources.

   > [!IMPORTANT]
   > Ne supprimez pas le groupe de ressources **rg-eshoponweb-agentpool** qui contient les ressources de l’agent auto-hébergé. Vous en aurez besoin dans le prochain labo.

1. Dans le portail Azure, accédez à la page affichant la machine virtuelle Azure **eshoponweb-vm** que vous avez déployée dans ce labo

1. Dans la page de la machine virtuelle Azure **eshoponweb-vm**, dans la barre d’outils, sélectionnez **Arrêter** pour l’arrêter et la libérer.

#### Tâche 2 : Supprimer les pipelines Azure DevOps

1. Accédez au Portail Azure DevOps sur `https://dev.azure.com` et ouvrez votre organisation.

1. Ouvrez le projet **eShopOnWeb**.

1. Accédez à **Pipelines > Pipelines**.

1. Accédez à **Pipelines > Pipelines** et supprimez les pipelines existants.

   > [!IMPORTANT]
   > Ne supprimez pas la connexion de service ni le pool d’agents que vous avez créés dans ce labo. Vous en aurez besoin dans le prochain.

#### Tâche 3 : Recréer le dépôt Azure DevOps

1. Dans le portail Azure DevOps, dans le projet **eShopOnWeb**, sélectionnez **Paramètres du projet** en bas à gauche.

1. Dans le menu vertical **Paramètres du projet** sur le côté gauche, dans la section ** Dépôts**, sélectionnez **Dépôts**.

1. Dans le volet **Tous les dépôts**, pointez sur l’extrémité droite de l’entrée du dépôt **eShopOnWeb** jusqu’à ce que l’icône des points de suspension **Plus d’options** s’affiche. Sélectionnez-la, puis dans le menu **Plus d’options**, sélectionnez **Renommer**.  

1. Dans la fenêtre **Renommer le dépôt eShopOnWeb**, dans la zone de texte **Nom du dépôt**, entrez **eShopOnWeb_old** et sélectionnez **Renommer**.

1. De retour dans le volet **Tous les dépôts**, sélectionnez **+ Créer**.

1. Dans le volet **Créer un dépôt**, dans la zone de texte **Nom du dépôt**, entrez **eShopOnWeb**, décochez la case **Ajouter un fichier README**, puis sélectionnez **Créer**.

1. De retour dans le volet **Tous les dépôts**, pointez sur l’extrémité droite de l’entrée du dépôt **eShopOnWeb-old** jusqu’à ce que l’icône des points de suspension **Plus d’options** s’affiche. Sélectionnez-la, puis dans le menu **Plus d’options**, sélectionnez **Supprimer**.  

1. Dans la fenêtre **Supprimer le dépôt eShopOnWeb_old**, entrez **eShopOnWeb_old** et sélectionnez **Supprimer**.

1. Dans le menu de navigation de gauche du portail Azure DevOps, sélectionnez **Dépôts**.

1. Dans le volet **eShopOnWeb est vide. Ajoutez du code !**, sélectionnez **Importer un dépôt**.

1. Dans la fenêtre **Importer un référentiel Git**, collez l’URL `https://github.com/MicrosoftLearning/eShopOnWeb` suivante, puis sélectionnez **Importer** :

## Révision

Dans ce labo, vous avez appris à utiliser une identité managée affectée à des agents auto-hébergés dans des pipelines YAML Azure DevOps.