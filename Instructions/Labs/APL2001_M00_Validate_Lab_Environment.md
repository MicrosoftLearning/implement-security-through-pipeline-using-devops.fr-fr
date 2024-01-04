---
lab:
  title: Valider votre environnement de labo
  module: 'Module 0: Welcome'
---

# Valider votre environnement de labo

En préparation des labos, il est essentiel que votre environnement soit correctement configuré. Cette page vous guide tout au long du processus de configuration, en vous assurant que toutes les conditions préalables sont remplies.

- Les labos nécessitent **Microsoft Edge** ou un [navigateur pris en charge par Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurer un abonnement Azure :** si vous n’avez pas encore d’abonnement Azure, créez-en un en suivant les instructions de cette page ou consultez [https://azure.microsoft.com/free](https://azure.microsoft.com/free) pour vous abonner gratuitement.

- **Configurez une organisation Azure DevOps :** si vous ne disposez pas encore d’une organisation Azure DevOps que vous pouvez utiliser pour les labos, créez-en une conformément aux instructions sur cette page ou sur [Créer une organisation ou une collection de projets](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).
  
- Page de téléchargement [Git pour Windows](https://gitforwindows.org/). Cela sera installé dans le cadre des prérequis de ce labo.

- [Visual Studio Code](https://code.visualstudio.com/). Cela sera installé dans le cadre des prérequis de ce labo.

- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli). Installez Azure CLI sur les machines d’agent auto-hébergés.

## Instructions pour créer une organisation Azure DevOps (vous ne devez effectuer cette opération qu’une seule fois)

> **Remarque** : commencez à l’étape 3, si vous avez déjà configuré un **compte Microsoft personnel** et un abonnement Azure actif lié à ce compte.

1. Utilisez une session de navigateur privée pour obtenir un nouveau **compte Microsoft personnel (MSA)** à l’adresse `https://account.microsoft.com`.

1. À l’aide de la même session de navigateur, abonnez-vous gratuitement à Azure à l’adresse `https://azure.microsoft.com/free`.

1. Ouvrez un navigateur et accédez au Portail Azure sur `https://portal.azure.com`, puis recherchez **Azure DevOps** en haut de l’écran du Portail Azure. Dans la page qui s’affiche, cliquez sur **Organisations Azure DevOps**.

1. Ensuite, cliquez sur le lien intitulé **Mes organisations Azure DevOps** ou accédez directement à `https://aex.dev.azure.com`.

1. Sur la page **Nous avons besoin de quelques détails supplémentaires**, sélectionnez **Continuer**.

1. Dans la zone de liste déroulante de gauche, choisissez **Répertoire par défaut**, au lieu de **Compte Microsoft**.

1. Si vous recevez l’invite (*« Nous avons besoin de quelques détails supplémentaires »*, indiquez votre nom, votre adresse de messagerie et votre emplacement, puis cliquez sur **Continuer**.

1. En revenant à `https://aex.dev.azure.com` à l’aide du **répertoire par défaut** sélectionné, cliquez sur le bouton bleu **Créer une organisation**.

1. Acceptez les *conditions d’utilisation du service* en cliquant sur **Continuer**.

1. Si vous recevez l’invite (*« Vous avez presque terminé »)*, laissez le nom de l’organisation Azure DevOps par défaut (il doit s’agir d’un nom global unique) et choisissez un emplacement d’hébergement proche de vous dans la liste.

1. Une fois que l’organisation nouvellement créée s’ouvre dans **Azure DevOps**, sélectionnez **Paramètres de l’organisation** dans le coin inférieur gauche.

1. Dans l’écran **Paramètres de l’organisation**, sélectionnez **Facturation** (l’ouverture de cet écran prend quelques secondes).

1. Sélectionnez **Configurer la facturation** et, sur le côté droit de l’écran, sélectionnez votre **abonnement Azure**, puis sélectionnez **Enregistrer** pour lier l’abonnement à l’organisation.

1. Une fois que l’écran affiche l’ID d’abonnement Azure lié sur la partie supérieure, modifiez le nombre de **travaux parallèles payés** pour **CI/CD hébergé par MS** de 0 à **1**. Sélectionnez ensuite le bouton **ENREGISTRER** au bas de l’écran.

1. Vous pouvez **attendre au moins 3 heures avant d’utiliser les capacités CI/CD** afin que les nouveaux paramètres soient reflétés dans le serveur principal. Sinon, vous verrez toujours le message *« Aucun parallélisme hébergé n’a été acheté ou accordé ».*

## Instructions pour créer l’échantillon de projet Azure DevOps (vous ne devez effectuer cette opération qu’une seule fois)

> **Remarque** : vérifiez que vous avez suivi la procédure de création de votre organisation Azure DevOps avant de poursuivre.

Pour suivre toutes les instructions du labo, vous devez configurer un nouveau projet Azure DevOps et créer un référentiel basé sur l’application [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb).

### Créer et configurer le projet d’équipe

Tout d’abord, vous allez créer un projet Azure DevOps **eShopOnWeb** à utiliser par plusieurs labos.

1. Ouvrez votre navigateur et accédez à votre organisation Azure DevOps.

1. Sélectionnez l’option **Nouveau projet** et utilisez les paramètres suivants :
   - nom : **eShopOnWeb**
   - visibilité : **Privé**
   - avancé : Contrôle de version : **Git**
   - Avancé : Processus d’élément de travail : **Scrum**

1. Sélectionnez **Créer**.

    ![Création d’un projet](media/create-project.png)

### Importer un référentiel Git eShopOnWeb

À présent, vous allez importer eShopOnWeb dans votre référentiel Git.

1. Ouvrez votre navigateur et accédez à votre organisation Azure DevOps.

1. Ouvrez le projet **eShopOnWeb** que nous avons créé précédemment.

1. Sélectionnez **Repos > Fichiers**, **Importer un référentiel**, puis sélectionnez **Importer**.

1. Dans la fenêtre **Importer un référentiel Git**, collez l’URL `https://github.com/MicrosoftLearning/eShopOnWeb` suivante, puis sélectionnez **Importer** :

    ![Importer un référentiel](media/import-repo.png)

1. Le référentiel est organisé de la manière suivante :
    
    - Le dossier **.ado** contient des pipelines YAML Azure DevOps.
    - Conteneur de dossiers **.devcontainer** configuré pour le développement à l’aide de conteneurs (localement dans VS Code ou GitHub Codespaces).
    - Le dossier **.azure** contient l’infrastructure de modèle ARM et Bicep en tant que modèles de code.
    - **Définitions de workflow GitHub YAML du conteneur de dossiers .github**.
    - Le dossier **src** contient le site web .NET 6 utilisé dans les scénarios de labo.

Vous avez maintenant effectué les étapes préalables nécessaires pour poursuivre les labos.
