# Recette - AD CS

![Bannière CUB](https://cub.bts.loutik.fr/assets/banniere_cub.png)

---

## Informations

- **Auteur :** Louis MEDO
- **Date :** 02/10/2026
- **Domaine :** Cybersécurité

---

## 1. Contexte du test

Cette procédure de recette valide le déploiement de l'Infrastructure à Clés Publiques (AD CS). Les scénarios visent à certifier la bonne propagation de la racine de confiance sur les postes de travail via GPO, l'activation effective du chiffrement LDAPS sur le contrôleur de domaine, le fonctionnement des certificats web (HTTPS) sans erreur de sécurité sur GLPI, ainsi que l'accessibilité de la liste de révocation (CRL).

> [!tip] Conditions préalables
> Les tests côté client doivent être exécutés depuis une machine jointe au domaine Active Directory (`local.agence.cub.sioplc.fr`) et avec une session utilisateur standard pour valider le comportement en conditions réelles.

## 2. Procédures de validation

### 2.1. Présence du certificat racine sur un poste client

**Objectif :** Vérifier que la stratégie de groupe (GPO) d'auto-enrôlement a bien poussé le certificat de l'Autorité de Certification Racine dans le magasin de confiance de la machine cliente.

**Commande utilisée :**

```powershell
Get-ChildItem -Path Cert:\LocalMachine\Root | Where-Object Subject -match "cub"
```

* `Get-ChildItem` : Cmdlet listant le contenu d'un répertoire (ici utilisé sur le fournisseur d'environnement virtuel des certificats).
* `-Path Cert:\LocalMachine\Root` : Cible précisément le magasin système hébergeant les "Autorités de certification racines de confiance" de l'ordinateur local.
* `Where-Object` : Filtre les objets du pipeline en fonction de leurs propriétés.
* `Subject -match "CUB"` : Recherche les certificats dont la propriété "Sujet" contient le nom de l'infrastructure CUB.

**Résultat attendu :**

```powershell


   PSParentPath : Microsoft.PowerShell.Security\Certificate::LocalMachine\Root

Thumbprint                                Subject
----------                                -------
145603B8E3962215E2E82367F6F43C239E4EEEFB  CN=CUB ROOT CERTIFICAT, DC=local, DC=dortmund, DC=cub, DC=sioplc, DC=fr
```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.2. Validation de la connexion sécurisée (HTTPS) interne

**Objectif :** Confirmer que l'accès à site interne (ex. GLPI) s'effectue en HTTPS validé par la PKI, sans lever d'alerte de sécurité "Certificat non valide" dans le navigateur du client.

**Commande utilisée :**

```powershell title="test_https_glpi.ps1"
Invoke-WebRequest -Uri "https://glpi.agence.cub.sioplc.fr" -UseBasicParsing
```

* `Invoke-WebRequest` : Envoie une requête HTTP ou HTTPS à une page web ou un service web. Si le certificat TLS présenté par le serveur n'est pas approuvé par la machine, la cmdlet renverra une exception bloquante.
* `-Uri "https://glpi.agence.cub.sioplc.fr"` : Spécifie l'URL complète cible à atteindre.
* `-UseBasicParsing` : Désactive l'analyse DOM complexe basée sur Internet Explorer pour accélérer l'exécution et la compatibilité.

**Résultat attendu :**

```powershell
StatusCode        : 200
StatusDescription : OK
Content           : <!DOCTYPE html>...
```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................

### 2.3. Validation du protocole LDAPS sur le Contrôleur de Domaine

**Objectif :** S'assurer que le contrôleur de domaine (DC) a bien généré son certificat serveur et écoute les requêtes d'annuaire chiffrées sur le port TCP 636.

**Commande utilisée :**

```bash title="test_ldaps.sh"
openssl s_client -connect local.agence.cub.sioplc.fr:636 -showcerts
```

> [!info] Installer openssl sur Windows
>
> ```bash
> winget install --id Git.Git -e --source winget
> ```
>
> Puis ouvrir **git bash**.

* `openssl s_client` : Outil de diagnostic Linux agissant comme un client SSL/TLS générique, très utile pour tester les connexions sécurisées vers un service.
* `-connect local.agence.cub.sioplc.fr` : Spécifie l'hôte (le contrôleur de domaine) et le port cible (636, port standard du LDAPS) à interroger.
* `-showcerts` : Force l'affichage complet de toute la chaîne de certificats renvoyée par le serveur.

**Résultat attendu :**

```bash
CONNECTED(00000003)
depth=1 DC = local, DC = cub, CN = CUB Enterprise Root CA
verify return:1
depth=0 CN = local.agence.cub.sioplc.fr
verify return:1
---
Certificate chain
 0 s:CN = local.agence.cub.sioplc.fr
...

```

**Statut :**

* [ ] Ok
* [ ] KO

**Commentaire :**

................................................................................................................................................................................................................................................................................................................................................................
