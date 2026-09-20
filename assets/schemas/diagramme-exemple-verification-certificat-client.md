https://mermaid.ai/live/edit

```mermaid
sequenceDiagram
    participant Étudiant as [ Navigateur Étudiant ]
    participant Apache as [ Serveur Apache ]
    participant LetEncrypt as [ Let's Encrypt ]

    Note over Étudiant,Apache: www.projet-sio.fr

    Étudiant->>Apache: 1. Requête de connexion HTTPS
    Apache-->>Étudiant: 2. Envoi du Certificat X.509 (Signé par Let's Encrypt)

    Note over Étudiant: Contrôles Locaux
    Note over Étudiant: • Signature valide ? (Calcul mathématique avec l'AC Racine Let's Encrypt)
    Note over Étudiant: • Domaine correct ? (L'URL demandée correspond bien au certificat)
    Note over Étudiant : • Dates valides ? (La date du jour est comprise dans la période de validité)

    Étudiant->>LetEncrypt: 3. Requête OCSP : "Ce certificat est-il révoqué ?"
    LetEncrypt-->>Étudiant: 4. Réponse OCSP : "Statut Bon"
    Étudiant->>Apache: 5. Création du tunnel chiffré (TLS)
```