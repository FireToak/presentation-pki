# Présentation - Infrastructure à Clés Publiques (PKI)

![Bannière du projet](./assets/img/banniere-pki.png)

## Contexte

Ce projet est une ressource documentaire détaillée visant à présenter les concepts fondamentaux des Infrastructures à Clés Publiques (PKI). Il explique les mécanismes de la cryptographie asymétrique, le cycle de vie des certificats X.509 (génération, validation, émission), les chaînes de confiance, ainsi que les processus de révocation (CRL, OCSP). Ce dépôt dresse également un panorama technique des outils du marché (Microsoft AD CS, Step-CA, XCA) pour aider au déploiement et à l'automatisation de PKI internes en environnements de production ou DevOps.

[Lien vers la présentation Canva](https://canva.link/lqxtfifh11st9ix)

---

## Structure du dépôt

L’organisation du dépôt suit la logique suivante :

```text
.
├── assets/
│   ├── img/
│   └── schemas/
├── Présentation de la notion de PKI.md
└── README.md
```

- **`assets/img/`** : Contient les visuels, diagrammes de séquence et schémas d'architecture exportés au format PNG.
- **`assets/schemas/`** : Regroupe les fichiers sources (Mermaid.js et Excalidraw) permettant la maintenance et l'évolution des diagrammes.
- **`Présentation de la notion de PKI.md`** : Document principal contenant la théorie, les études de cas (ex: serveur web Apache) et les comparatifs d'outils.

---

## Utilisation du projet

### 1. Cloner le dépôt localement

```bash
git clone https://github.com/FireToak/pki-presentation.git
cd pki-presentation
```

### 2. Lire la documentation

Le cours principal est rédigé en Markdown. Il est recommandé de le lire à l'aide d'un éditeur compatible (VS Code, Obsidian, Typora) ou directement via l'interface web de votre forge logicielle.

### 3. Éditer ou maintenir les schémas

Pour modifier les diagrammes d'obtention ou de vérification, copiez le contenu des fichiers `.md` situés dans `assets/schemas/` et collez-le dans le [Mermaid Live Editor](https://mermaid.live/) ou l'éditeur Excalidraw.

---

## 👨‍💻 Mainteneurs

- **Louis MEDO** | [LinkedIn](https://www.linkedin.com/in/louismedo/) | [Portfolio](https://louis.loutik.fr/) | [GitHub](https://github.com/FireToak) | [louis.medo@loutik.fr](mailto:louis.medo@loutik.fr)

-----

<div align="center">
<br>
<small><i>Dernière mise à jour : 20 septembre 2026</i></small>
</div>
