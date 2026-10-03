# Installation de AD CS

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 02/10/2026
- **Domaine :** Administration Windows

---

## 1. Sommaire

- [1. Sommaire](#1-sommaire)
- [2. Contexte](#2-contexte)
- [3. Prérequis](#3-prerequis)
- [4. Installation du rôle AD CS](#4-installation-du-rôle-ad-cs)

## 2. Contexte

Le déploiement des services de certificats Active Directory (AD CS) constitue le socle de l'Infrastructure à Clés Publiques (PKI). Dans le cadre de l'infrastructure CUB, ce service permet de garantir l'identité des serveurs, de sécuriser les flux réseau et d'authentifier les clients. L'installation se fait sur un environnement Windows Server Core, privilégié pour réduire la surface d'attaque et minimiser les besoins en ressources.

> [!WARNING]
> **Architecture à deux niveaux :** la PKI nécessite deux serveurs distincts : une CA Racine (isolée, hors domaine et hors-ligne) et une CA Intermédiaire (jointe au domaine et en ligne). Ce document couvre uniquement l'installation des binaires communs aux deux serveurs.

## 3. Prérequis {#3-prerequis}

Avant de provisionner le rôle AD CS, le nœud cible doit valider les pré-requis systèmes et réseaux suivants :

- **OS :** Windows Server Core installé, à jour et sans interface graphique (GUI).
- **Réseau :** Adresse IP statique configurée de manière pérenne et résolution DNS (primaire/secondaire) opérationnelle.
- **Identité du serveur :**
  - **CA Racine :** Maintenu en groupe de travail (Workgroup).
  - **CA Intermédiaire :** Joint au domaine Active Directory.
- **Sécurité et droits :** Exécution des commandes avec un compte disposant des privilèges Administrateur local (ou Administrateur du Domaine pour la CA Intermédiaire).

## 4. Installation du rôle AD CS

4.1. **Déploiement des binaires AD CS.** Installation du rôle serveur d'Autorité de Certification via l'interface en ligne de commande PowerShell. Cette action se limite au déploiement logiciel.

```powershell
Install-WindowsFeature -Name ADCS-Cert-Authority -IncludeManagementTools
```

- `Install-WindowsFeature` : Cmdlet PowerShell permettant d'ajouter des rôles, des services de rôle ou des fonctionnalités sur un serveur Windows.
- `-Name ADCS-Cert-Authority` : Argument spécifiant le nom technique du rôle principal à installer (l'Autorité de certification elle-même).
- `-IncludeManagementTools` : Commutateur ordonnant l'installation conjointe des outils d'administration (modules PowerShell et outils en ligne de commande) nécessaires à la gestion locale ou distante.

4.2. **Validation de l'installation.** Contrôle de l'état du déploiement pour s'assurer que les binaires sont correctement enregistrés dans le système.

```powershell
Get-WindowsFeature -Name ADCS-Cert-Authority
```

- `Get-WindowsFeature` : Cmdlet récupérant l'état d'installation des rôles et fonctionnalités.
- `-Name ADCS-Cert-Authority` : Cible la vérification spécifiquement sur le rôle de la PKI. L'état renvoyé doit être "Installed" (Installé).

**Exemple de retour :**

```powershell
Display Name                                            Name                       Install State
------------                                            ----                       -------------
    [X] Autorité de certification                       ADCS-Cert-Authority            Installed
```

> [!NOTE]
> **Étape suivante :** L'installation des binaires étant terminée, la configuration cryptographique (génération des clés, définition de l'algorithme de hachage, durée de validité) devra être effectuée ultérieurement à l'aide de la commande `Install-AdcsCertificationAuthority` selon le rôle attribué à la machine (Racine ou Subordonnée).
>
> Procédures de configuration : [Configuration AD CS](./configuration-ad-cs.md)
