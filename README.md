# Automatisation du traitement des commentaires et messages privés — Toyota

Projet Bonzai Agency : automatisation de la réponse aux commentaires publiés sous les publications Facebook/Instagram et aux messages privés (Messenger / Instagram Direct) du compte client **Toyota**, conformément au [cahier des charges fonctionnel v1.0](docs/cahier-des-charges.md).

## Vue d'ensemble

Le processus manuel actuel (recherche de la bonne réponse dans un fichier Excel, copier-coller, archivage manuel) est remplacé par deux workflows automatisés bâtis sur **[n8n](https://n8n.io)**, un outil d'automatisation **open-source et gratuit** en self-hosted — ce qui répond à la contrainte budgétaire du CDC (§8). Les workflows s'appuient exclusivement sur les **API officielles Meta** (Graph API / Messenger Send API) et sur une base de connaissances **Google Sheets** (équivalent en ligne, gratuit, du fichier Excel Toyota) éditable directement par le consultant.

```
Facebook / Instagram (Meta)
        │  webhooks (nouveau commentaire / nouveau message)
        ▼
   Workflows n8n  ──lit──▶  Base de connaissances (Google Sheets)
        │
        ├─ catégorie trouvée   → réponse publiée automatiquement (Graph API)
        └─ catégorie inconnue  → alerte email au consultant (BF-12)
        │
        ▼
   Registre de suivi (Google Sheets, consultable à tout moment — BF-13)
```

## Contenu du dépôt

| Dossier | Contenu |
|---|---|
| `workflows/n8n/` | Les 2 workflows n8n prêts à importer (`.json`) |
| `knowledge-base/` | Modèle de base de connaissances catégorisée (équivalent du fichier Excel Toyota) |
| `registre-suivi/` | Modèle du registre d'archivage automatique |
| `docs/` | Cahier des charges, architecture, guide d'installation, guide consultant |

## Les deux workflows

1. **`01-comments-workflow.json`** — Détecte tout nouveau commentaire (BF-01), identifie sa catégorie (BF-02), publie la réponse pré-rédigée correspondante (BF-03, BF-04) et archive l'interaction (BF-05).
2. **`02-messages-workflow.json`** — Fait de même pour les messages privés Messenger / Instagram Direct (BF-06 à BF-09).

Dans les deux cas, si aucune catégorie ne correspond, le commentaire/message n'est **pas traité automatiquement** : il est archivé avec le statut « à traiter manuellement » et le consultant reçoit une alerte email (BF-12). Le registre est consultable à tout moment dans Google Sheets (BF-13).

## Démarrage rapide

Voir le guide détaillé : [docs/setup-guide.md](docs/setup-guide.md).

1. Déployer n8n (gratuit, self-hosted ou n8n Cloud en essai).
2. Créer la base de connaissances et le registre dans Google Sheets à partir des modèles fournis dans `knowledge-base/` et `registre-suivi/`.
3. Créer une app Meta for Developers, obtenir un token de page longue durée, configurer les webhooks (`feed` pour les commentaires, `messages`/`messaging` pour les messages privés).
4. Importer les 2 fichiers `.json` dans n8n, configurer les identifiants (Facebook Graph API, Google Sheets, SMTP), remplacer les `TO_CONFIGURE_*`.
5. Activer les workflows.

## Périmètre (rappel CDC §5)

**Inclus (V1.0) :** commentaires et messages privés Facebook/Instagram, réponses à partir des catégories du fichier Toyota, archivage automatique.
**Exclu (V1.0) :** génération de réponses libres non catégorisées, traitement des demandes complexes/sensibles (redirigées vers le consultant via BF-12).
