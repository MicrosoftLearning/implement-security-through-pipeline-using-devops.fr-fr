---
lab:
  title: Configurer des agents et des pools d’agents pour des pipelines sécurisés
  module: 'Module 2: Configure secure access to pipeline resources'
---

# Configurer des agents et des pools d’agents pour des pipelines sécurisés

Dans ce labo, vous allez découvrir comment configurer des agents et des pools d’agents Azure DevOps, et gérer les autorisations pour ces pools. Les pools d’agents Azure DevOps fournissent les ressources pour exécuter vos pipelines de build et de mise en production.

Ces exercices prennent environ **25** minutes.

## Avant de commencer

Vous aurez besoin d’un abonnement Azure, d’une organisation Azure DevOps et de l’application eShopOnWeb pour suivre les labos.

- Procédez comme suit pour [valider votre environnement de labo](APL2001_M00_Validate_Lab_Environment.md).

## Instructions

Vous allez créer des agents et configurer des agents auto-hébergés à l’aide de Windows. Si vous souhaitez configurer des agents sur Linux ou MacOS, suivez les instructions de la documentation [Azure DevOps](https://docs.microsoft.com/azure/devops/pipelines/agents/v2-linux).

Pendant la configuration, gardez à l’esprit les éléments suivants :

- **Conservez des agents distincts pour chaque projet** : chaque agent ne peut être lié qu’à un seul pool. Si le partage des pools d’agents entre les projets permet d’enregistrer des économies sur les coûts d’infrastructure, il crée également un risque de mouvement latéral. Par conséquent, il est préférable d’avoir des pools d’agents distincts avec des agents dédiés pour chaque projet afin d’éviter la contamination croisée.
- **Utilisez des comptes à faible privilège pour exécuter des agents** : l’exécution d’un agent sous une identité disposant d’un accès direct aux ressources Azure DevOps peut poser des menaces pour la sécurité. L’exploitation de l’agent sous un compte local non privilégié tel que le service réseau est conseillé, ce qui réduit le risque.
- **Attention aux noms de groupes trompeurs** : le groupe « Comptes de service de collection de projets » dans Azure DevOps est un risque de sécurité potentiel. L’exécution d’agents à l’aide d’une identité qui fait partie de ce groupe et soutenue par Azure AD peut compromettre la sécurité de l’ensemble de votre organisation Azure DevOps.
- **Évitez les comptes à privilèges élevés pour les agents auto-hébergés** : l’utilisation de comptes à privilèges élevés pour exécuter des agents auto-hébergés, en particulier pour accéder aux secrets ou aux environnements de production, peut exposer votre système à de graves menaces si un pipeline est compromis.
- **Prioriser la sécurité** : pour protéger vos systèmes, utilisez le compte le moins privilégié afin d’exécuter des agents auto-hébergés. Par exemple, envisagez d’utiliser votre compte d’ordinateur ou une identité de service managé. Il est également conseillé d’autoriser Azure Pipelines à gérer l’accès aux secrets et aux environnements.

### Exercice 1 : Créer des agents et configurer des pools d’agents

Dans cet exercice, vous allez créer une machine virtuelle Azure et l’utiliser pour créer un agent et configurer des pools d’agents.

#### Tâche 1 : Créer une machine virtuelle Azure et s’y connecter

1. Dans votre navigateur, ouvrez le portail Azure sur `https://portal.azure.com`. Si vous y êtes invité, connectez-vous avec un compte qui a le rôle Propriétaire dans votre abonnement Azure.

1. Dans la zone **Rechercher des ressources, des services et des documents (G+/),** tapez **Machine virtuelles** et sélectionnez-le dans la liste déroulante.

1. Cliquez sur le bouton **Créer**.

1. Sélectionnez la **machine virtuelle Azure avec une configuration prédéfinie**.

    ![Capture d’écran de la création d’une machine virtuelle avec une configuration prédéfinie.](media/create-virtual-machine-preset.png)

1. Sélectionnez **Dev/Test** comme environnement de charge de travail et **usage général** comme type de charge de travail.

1. Sélectionnez le bouton **Continuer à créer une machine virtuelle**. Sous l’onglet **Informations de base**, effectuez les actions suivantes et sélectionnez **Gestion** :

   | Setting | Action |
   | -- | -- |
   | Liste déroulante  **Abonnement** | Sélectionnez votre abonnement Azure. |
   | Section **Groupe de ressources** | Créez un groupe de ressources nommé **rg-eshoponweb-agentpool**. |
   | Zone de texte **Nom de la machine virtuelle**  | Entrez le nom de votre préférence, par exemple **eshoponweb-vm**. |
   | Liste déroulante **Région** | Vous pouvez choisir la région [Azure](https://azure.microsoft.com/explore/global-infrastructure/geographies) la plus proche de vous. Par exemple, « westeurope », « eastasia » ou « eastus », etc. |
   | Liste déroulante **Options de disponibilité** | Sélectionnez **Aucune redondance d’infrastructure requise**. |
   | Liste déroulante **Type de sécurité** | Sélectionnez avec l’option **Lancement fiable de machines virtuelles**. |
   | Liste déroulante **Image** | Sélectionnez l’image **Windows Server 2022 Datacenter : Édition Azure - x64 Gen2**. |
   | Liste déroulante **Taille** | Sélectionnez la taille **Standard** la moins chère à des fins de test. |
   | Zone de texte **Nom d’utilisateur** | Entrez le nom d’utilisateur de votre préférence. |
   | Zone de texte **Mot de passe**. | Entrez le mot de passe de votre préférence. |
   | Section **Ports d’entrée publics** | Sélectionnez **Autoriser les ports sélectionnés**. |
   | Liste déroulante **Sélectionner les ports entrants** | Sélectionnez **RDP (3389)**. |

1. Sous l’onglet **Gestion**, dans la section **Identité**, cochez la case **Activer l’identité managée affectée par le système**, puis sélectionnez **Vérifier + créer** :

1. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.

   > **Remarque** : Attendez la fin du processus de provisionnement. Ce processus prend environ 2 minutes.

1. Dans le portail Azure, accédez à la page affichant la configuration de la machine virtuelle Azure nouvellement créée.

1. Sur la page de machine virtuelle Azure, sélectionnez **Se connecter**. Dans le menu déroulant, sélectionnez **Se connecter**, puis **Télécharger le fichier RDP**.

1. Utilisez le fichier RDP téléchargé pour établir une session Bureau à distance sur le système d’exploitation en cours d’exécution sur la machine virtuelle Azure.

#### Tâche 2 : Créer un pool d’agents

1. Dans la session Bureau à distance sur la machine virtuelle Azure, démarrez le navigateur web Microsoft Edge.

1. Dans le navigateur web, accédez au portail Azure DevOps sur `https://aex.dev.azure.com` et connectez-vous pour accéder à votre organisation.

   > **Remarque** : s’il s’agit de votre première accès au portail Azure DevOps, vous devrez peut-être créer votre profil.

1. Ouvrez le projet **eShopOnWeb**, puis sélectionnez **Paramètres du projet** dans le menu inférieur gauche.

1. Dans **Pipelines > Pools d’agents**, sélectionnez le bouton **Ajouter un pool**.

1. Choisissez le type de pool **auto-hébergé**.

1. Donnez un nom au pool d’agents, tel que **eShopOnWebSelfPool**, et ajoutez une description facultative.

1. Laissez l’option **Accorder une autorisation d’accès à tous les pipelines** décochée.

   ![Capture d’écran montrant l’ajout d’options de pool d’agents avec un type auto-hébergé.](media/create-new-agent-pool-self-hosted-agent.png)

   > **Remarque** : il n’est pas recommandé d’accorder une autorisation d’accès à tous les pipelines pour les environnements de production. Elle est utilisée uniquement dans ce labo pour simplifier la configuration du pipeline.

1. Sélectionnez le bouton **Créer** pour créer le pool d’agents.

#### Tâche 3 : Télécharger et extraire les fichiers d’installation de l’agent

1. Dans le portail Azure DevOps, sélectionnez le pool d’agents qui vient d’être créé, puis sélectionnez l’onglet **Agents**.

1. Sélectionnez le bouton **Nouvel agent**, puis le bouton **Télécharger** à partir de **Télécharger l’agent** dans la nouvelle fenêtre contextuelle.

   > **Remarque** : suivez les instructions d’installation pour installer l’agent et notez la version téléchargée dans le nom de fichier (par exemple, vsts-agent-win-x64-3.246.0.zip).

1. Démarrez une session PowerShell et exécutez les commandes suivantes pour créer un dossier nommé **agent**.

   ```powershell
   mkdir agent ; cd agent        
   ```

   > **Remarque** : vérifiez que vous êtes dans le dossier dans lequel vous souhaitez installer l’agent, par exemple, C:\agent.

1. Exécutez la commande suivante pour extraire le contenu des fichiers du programme d’installation de l’agent que vous avez téléchargés :

   ```powershell
   Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-3.246.0.zip", "$PWD")
   ```

   > **Remarque** : si vous avez téléchargé l’agent dans un autre emplacement (ou si la version téléchargée est différente), ajustez la commande ci-dessus en conséquence.

#### Tâche 4 : Créer un jeton PAT

> **Remarque** : avant de configurer l’agent, vous devez créer un jeton PAT (sauf si vous en avez déjà un). Pour créer un jeton PAT, suivez les étapes ci-dessous :

1. Dans la session Bureau à distance sur la machine virtuelle Azure, ouvrez une autre fenêtre de navigateur, accédez au portail Azure DevOps à l’adresse `https://aex.dev.azure.com`, puis accédez à votre organisation et à votre projet.

1. Sélectionnez **Paramètres utilisateur** dans le menu supérieur droit (tout de suite à gauche de l’icône d’avatar de votre utilisateur).

1. Sélectionnez l’élément de menu **Jetons d’accès personnels**.

   ![Capture d’écran montrant le menu des jetons d’accès personnel.](media/personal-access-token-menu.png)

1. Sélectionnez le bouton **Nouveau jeton**.

1. Donnez un nom au jeton, tel que **eShopOnWebToken**.

1. Sélectionnez l’organisation Azure DevOps pour vous permettre d’utiliser le jeton.

1. Définissez la date d’expiration du jeton (utilisée uniquement pour configurer l’agent).

1. Sélectionnez l’étendue définie personnalisée.

1. Sélectionnez cette option pour afficher toutes les étendues.

1. Sélectionnez l’étendue **Pools d’agents (Lecture et gestion)**.

1. Sélectionnez le bouton **Créer** pour créer le jeton.

1. Copiez la valeur du jeton et enregistrez-la en lieu sûr (vous ne pourrez pas la voir à nouveau. Vous ne pouvez régénérer que le jeton).

   ![Capture d’écran montrant le configuration des jetons d’accès personnel.](media/personal-access-token-configuration.png)

   > [!IMPORTANT]
   > Utilisez l’option de privilège minimum, **Pools d’agents (lire, gérer)**, uniquement pour la configuration de l’agent. Veillez également à définir la date d’expiration la plus courte pour le jeton si c’est son seul objectif. Vous pouvez créer un autre jeton avec les mêmes privilèges si vous devez configurer l’agent une nouvelle fois.

#### Tâche 5 : Configurer l’agent

1. Dans la session Bureau à distance sur la machine virtuelle Azure, revenez à la fenêtre PowerShell. Si nécessaire, remplacez le répertoire actif par celui dans lequel vous avez extrait les fichiers d’installation de l’agent précédemment dans cet exercice.

1. Pour configurer votre agent pour qu’il s’exécute sans assistance, appelez la commande suivante :

   ```powershell
   .\config.cmd
   ```

   > **Remarque** : si vous souhaitez exécuter l’agent de manière interactive, utilisez `.\run.cmd` à la place.

1. Pour configurer l’agent, effectuez les actions suivantes lorsque vous y êtes invité :

   - Entrez l’URL de l’organisation Azure DevOps (**URL du serveur**) au format `https://aex.dev.azure.com`{nom de votre organisation}.
   - Acceptez le type d’authentification par défaut (**PAT**).
   - Entrez la valeur du jeton PAT que vous avez créé à l’étape précédente.
   - Entrez le nom du pool d’agents **`eShopOnWebSelfPool`** que vous avez créé précédemment dans cet exercice.
   - Entrez le nom de l'agent **`eShopOnWebSelfAgent`**.
   - Acceptez le dossier de travail de l’agent par défaut (_work).
   - Entrez **Y** pour configurer l’agent pour qu’il s’exécute en tant que service.
   - Entrez **Y** pour activer SERVICE_SID_TYPE_UNRESTRICTED pour le service de l’agent.
   - Entrez **NT AUTHORITY\SYSTEM** pour définir le contexte de sécurité du service.

   > [!IMPORTANT]
   > En général, vous devez suivre le principe du privilège minimum lors de la configuration du contexte de sécurité du service.

   - Acceptez l’option par défaut (**N**) pour autoriser le service à démarrer immédiatement après la fin de la configuration.

   ![Capture d’écran montrant la configuration de l’agent.](media/agent-configuration.png)

   > **Remarque** : le processus de configuration de l’agent prend quelques minutes. Une fois celui-ci terminé, un message s’affiche pour indiquer que l’agent est en cours d’exécution en tant que service.

   > [!IMPORTANT] Si vous voyez un message d’erreur indiquant que l’agent n’est pas en cours d’exécution, vous devrez peut-être démarrer le service manuellement. Pour le démarrer manuellement, ouvrez l’applet **Services** dans le Panneau de configuration Windows, recherchez le service nommé **Azure DevOps Agent (eShopOnWebSelfAgent)** et démarrez-le.

   > [!IMPORTANT] Si le démarrage de votre agent échoue, vous devrez peut-être choisir un autre dossier pour le répertoire de travail de l’agent. Pour ce faire, exécutez à nouveau le script de configuration de l’agent et choisissez un autre dossier.

1. Vérifiez l’état de l’agent en basculant vers le navigateur web affichant le portail Azure DevOps, en accédant au pool d’agents et en cliquant sur l’onglet **Agents**. Vous devriez voir le nouvel agent dans la liste.

   ![Capture d’écran montrant l’état de l’agent.](media/agent-status.png)

   > **Remarque** : pour plus d’informations sur les agents Windows, consultez : [Agents Windows auto-hébergés](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent).

   > [!IMPORTANT]
   > Pour que l’agent puisse créer et déployer des ressources Azure à partir des pipelines Azure DevOps (que vous allez passer en revue dans les labos à venir), vous devez installer Azure CLI dans le système d’exploitation de la machine virtuelle Azure qui héberge l’agent.

1. Démarrez un navigateur web et accédez à la page [Installer Azure CLI sur Windows](https://learn.microsoft.com/cli/azure/install-azure-cli-windows?tabs=azure-cli#install-or-update).

1. Téléchargez et installez Azure CLI.

1. (Facultatif) Si vous le souhaitez, exécutez la commande PowerShell suivante pour installer Azure CLI :

   ```powershell
   $ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; Remove-Item .\AzureCLI.msi
   ```

   > **Remarque** : si vous utilisez une autre version d’Azure CLI, vous devrez peut-être ajuster la commande ci-dessus en conséquence.

1. Dans le navigateur web, accédez à la page du programme d’installation du SDK Microsoft .NET 8.0 à l’adresse `https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-8.0.403-windows-x64-installer`.

   > [!IMPORTANT]
   > Vous devez installer le kit de développement logiciel (SDK) .NET 8.0 (ou version ultérieure) sur la machine virtuelle Azure qui héberge l’agent. Cela est nécessaire pour générer l’application eShopOnWeb dans les laboratoires suivants. Tous les autres outils ou kits de développement logiciel requis pour la création de l’application doivent également être installés sur la machine virtuelle Azure.

1. Téléchargez et installez le kit de développement logiciel (SDK) Microsoft .NET 8.0.

### Exercice 2 : Créer et configurer la sécurité du pool d’agents

Dans cet exercice, vous allez configurer la sécurité du pool d’agents.

#### Tâche 1 : Créer un groupe de sécurité

1. Dans la session Bureau à distance sur la machine virtuelle Azure, dans le navigateur web affichant le portail Azure DevOps, dans le volet **Paramètres du projet**, dans la section **Général**, sélectionnez **Autorisations**.

1. Sélectionnez le bouton **Nouveau groupe**.

1. Donnez un nom au groupe, tel que **Groupe de sécurité eShopOnWeb**.

1. Sélectionnez le bouton **Créer** pour créer le groupe.

   ![Capture d’écran montrant la création du groupe de sécurité.](media/create-security-group.png)

#### Tâche 2 : Configurer le groupe de sécurité

1. Dans la fenêtre du navigateur web affichant le portail Azure DevOps, sélectionnez le nouveau groupe pour afficher son onglet **Autorisations**.

1. Refusez les autorisations inutiles pour le groupe, telles que **Renommer le projet d’équipe**, **Supprimer définitivement le ou les éléments de travail** ou toute autre autorisation que vous ne souhaitez pas accorder au groupe, car nous partons du principe qu’il sera utilisé uniquement pour le pool d’agents.

   ![Capture d’écran montrant les paramètres du groupe de sécurité.](media/security-group-settings.png)

   > [!IMPORTANT]
   > Si vous laissez les autorisations que vous ne souhaitez pas que le groupe possède, les scripts ou les tâches qui s’exécutent sur l’agent peuvent utiliser les autorisations de groupe pour réaliser des actions que vous ne souhaitez pas effectuer.

#### Tâche 3 : Gérer les autorisations du pool d’agents

Dans cette tâche, vous allez gérer les autorisations du pool d’agents.

1. Dans la fenêtre du navigateur web affichant le portail Azure DevOps, dans les **Paramètres du projet** du projet **eShopOnWeb**, dans la section **Pipelines**, sélectionnez **Pools d’agents**.

1. Sélectionnez le pool d’agents **eShopOnWebSelfPool**.

1. Dans la vue des détails du pool d’agents, sélectionnez l’onglet **Sécurité**.

1. Sélectionnez le bouton **Ajouter** et ajoutez le nouveau groupe **Groupe de sécurité eShopOnWeb Security** aux autorisations utilisateur du pool d’agents.

1. Choisissez le rôle approprié pour l’utilisateur ou le groupe, tel que Lecteur du pool d’agents, Utilisateur ou Administrateur. Dans ce cas, choisissez **Utilisateur**.

1. Sélectionnez **Ajouter** pour appliquer les autorisations.

   ![Capture d’écran montrant la configuration de la sécurité du pool d’agents.](media/agent-pool-security.png)

Vous êtes maintenant prêt à utiliser le pool d’agents dans vos pipelines en toute sécurité. Vous pouvez utiliser le nouveau groupe pour ajouter des utilisateurs et gérer des autorisations pour le pool d’agents. Vous pouvez reconfigurer l’agent auto-hébergé installé à l’aide du nouveau groupe pour vous assurer que l’agent dispose des autorisations nécessaires pour exécuter les pipelines et plus encore. Par exemple, vous pouvez ajouter un utilisateur au groupe et configurer l’agent pour qu’il s’exécute en tant qu’utilisateur.

Pour plus d’informations sur les pools d’agents, consultez : [Pools d’agents](https://learn.microsoft.com/azure/devops/pipelines/agents/pools-queues).

> [!IMPORTANT]
> N’oubliez pas de supprimer les ressources créées dans le portail Azure pour éviter les frais inutiles.

## Révision

Dans ce labo, vous avez découvert la façon de configurer un agent auto-hébergé et des pools d’agents Azure DevOps, et de gérer les autorisations pour ces pools. En gérant efficacement les autorisations, vous pouvez vous assurer que les utilisateurs appropriés ont accès aux ressources dont ils ont besoin tout en maintenant la sécurité et l’intégrité de vos processus DevOps.
