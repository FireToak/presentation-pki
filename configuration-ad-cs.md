# Configuration de AD CS - Racine d'Entreprise

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
- [3. Point de distribution de la liste de révocation (IIS)](#3-point-de-distribution-de-la-liste-de-revocation-iis)
- [4. Configuration de l'Autorité de Certification (Racine d'Entreprise)](#4-configuration-de-lautorite-de-certification-racine-dentreprise)
- [5. Déploiement du certificat](#5-deploiement-du-certificat)

## 2. Contexte

> [!NOTE]
> **Certificat racine d'entreprise :** Dans le contexte des certificats X.509, une « racine d'entreprise » (ou Autorité de Certification Racine d'Entreprise) désigne le certificat racine (Root CA) généré et géré en interne par une organisation, par opposition aux racines publiques (comme DigiCert, Let's Encrypt, etc.) utilisées pour les sites web accessibles sur Internet.

Le déploiement d'une PKI complète nécessite normalement une séparation des rôles. Pour des besoins pédagogiques et afin de simplifier l'architecture, les concepts d'Autorité Racine (sommet de la confiance) et d'Autorité Intermédiaire (émission des certificats) sont ici consolidés sur un seul et même serveur Windows.

Ce serveur déploiera le rôle en mode **Racine d'Entreprise** (`EnterpriseRootCA`), lui permettant de générer sa propre ancre de confiance tout en s'intégrant à l'Active Directory pour automatiser la distribution vers les postes clients. Il hébergera également le serveur web IIS pour la Liste de Révocation (CRL).

> [!WARNING]
> **Mise en production :** Dans un environnement de production, cette architecture consolidée est formellement proscrite. L'AC Racine **DOIT** être un serveur autonome (Standalone), non joint au domaine, et maintenu strictement **hors-ligne** (éteint dans un coffre) pour protéger sa clé privée ou utiliser un HSM (modules de sécurité matériels). L'AC Intermédiaire est le seul serveur en ligne habilité à interagir avec le domaine. Gardez ce concept de séparation (Air-Gap) à l'esprit pour vos futures missions en entreprise.

## 3. Point de distribution de la liste de révocation (IIS) {#3-point-de-distribution-de-la-liste-de-revocation-iis}

> [!WARNING]
> **Prérequis :** Vous devez avoir installé le rôle **AD DS** pour le déploiement du certificat racine sur tous les postes du domaine !

3.1. **Déploiement et configuration du rôle Serveur Web (IIS).** Installation du service web pour exposer la liste des certificats révoqués (CRL) aux clients du réseau de manière performante.

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
New-Item -ItemType Directory -Path "C:\inetpub\wwwroot\PKI" -Force
New-WebVirtualDirectory -Site "Default Web Site" -Name "PKI" -PhysicalPath "C:\inetpub\wwwroot\PKI"
```

- `Install-WindowsFeature` : Installe les binaires du serveur web IIS et ses outils de gestion.
- `New-Item` : Crée le dossier physique racine qui hébergera les fichiers de révocation.
- `New-WebVirtualDirectory` : Crée le répertoire virtuel HTTP (ex: `http://pki-serveur.cub.local/PKI/`) lié au dossier physique créé précédemment.

## 4. Configuration de l'Autorité de Certification {#4-configuration-de-lautorite-de-certification-racine-dentreprise}

4.1. **Déploiement de la configuration cryptographique consolidée.** Paramétrage du rôle AD CS combinant la création de la racine de confiance et la liaison avec l'annuaire Active Directory.

```powershell
Install-AdcsCertificationAuthority -CAType EnterpriseRootCA -CryptoProviderName "RSA#Microsoft Software Key Storage Provider" -KeyLength 4096 -HashAlgorithmName SHA256 -ValidityPeriod Years -ValidityPeriodUnits 10 -CACommonName "CUB ROOT CERTIFICAT" -Force
```

- `Install-AdcsCertificationAuthority` : Cmdlet appliquant la configuration au rôle AD CS.
- `-CAType EnterpriseRootCA` : Fusionne les concepts : le serveur est sa propre Racine (Root) mais est lié à l'AD (Enterprise) pour émettre directement les certificats clients.
- `-CryptoProviderName "RSA#Microsoft Software Key Storage Provider"` : Spécifie le fournisseur logiciel pour générer la clé asymétrique RSA.
- `-KeyLength 4096` : Définit une taille de clé très robuste (4096 bits) pour sécuriser l'infrastructure consolidée.
- `-HashAlgorithmName SHA256` : Paramètre l'algorithme de hachage SHA-256 pour prévenir les vulnérabilités liées aux collisions.
- `-ValidityPeriodUnits 10` : Fixe la durée de validité du certificat de l'AC à 10 ans (un compromis entre la longévité d'une racine et la rotation d'une intermédiaire).
- `-Force` : Exécute le script sans interruption, validant l'approche d'Infrastructure as Code.

## 5. Déploiement du certificat {#5-deploiement-du-certificat}

Le déploiement d'une PKI en mode "Racine d'Entreprise" tire parti de l'intégration native avec Active Directory (AD DS). Lors de la configuration du rôle AD CS en mode Entreprise, le certificat public racine est automatiquement publié dans la partition de configuration de l'annuaire AD. Par conséquent, tous les postes clients joints au domaine téléchargent et installent silencieusement ce certificat dans leur magasin local d'autorités de confiance.

5.1. **Forcer l'actualisation sur les postes clients.** L'installation du certificat dépendant du cycle régulier d'actualisation des stratégies de groupe, il est possible d'accélérer ce processus manuellement sur un poste client pour valider le déploiement.

```cmd
gpupdate /force
```

- `gpupdate` : Utilitaire natif en ligne de commande permettant d'actualiser les paramètres de stratégie de groupe locaux et Active Directory.
- `/force` : Paramètre ordonnant de réappliquer toutes les stratégies (et non seulement celles modifiées), forçant ainsi la machine à récupérer immédiatement le nouveau certificat publié dans l'AD.

> [!TIP]
> **Alternative :** Si vous aviez appliqué la bonne pratique avec une **Autorité de Certification Racine Autonome** (Standalone), cette publication automatique n'aurait pas eu lieu car le serveur ne communique pas avec l'AD DS. Il aurait alors fallu exporter manuellement le certificat `.crt`, puis configurer une **Stratégie de Groupe (GPO)**.
