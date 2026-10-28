# bodyvision-contracts

Contrats d'interface du projet **BodyVision** (projet thématique M2 CCM, INSSET).

Ce dépôt est la **source de vérité** des contrats API. Il est volontairement séparé
des dépôts de services : le contrat couvre les trois services (Utilisateurs,
Ingestion, Analytique), et le loger dans l'un d'eux obligerait les deux autres à
dépendre d'un dépôt qui n'est pas le leur.

La version qui fait foi est celle de la branche `main`.

## Contenu

| Fichier | Rôle |
|---|---|
| `openapi.yaml` | Contrat OpenAPI 3.0.3 de l'API BodyVision (v0.3.0) |
| `.spectral.yaml` | Jeu de règles de validation (ruleset `spectral:oas`) |

## Valider le contrat

```bash
npx --yes @stoplight/spectral-cli@6.17.0 lint openapi.yaml
```

Aucune installation préalable : `npx` récupère l'outil à la volée.
`0 errors` signifie que le document est un OpenAPI valide. Les `warnings`
signalent des manques de style (opération sans `operationId`, réponse non
documentée...).

Le workflow `.github/workflows/ci.yml` lance cette validation sur chaque Pull Request et
sur `main`, avec Node.js 24 et Spectral 6.17.0 (SCRUM-42). Les erreurs bloquent le job ;
les avertissements restent informatifs.

## Servir un mock

Le contrat déclare un serveur de mock Prism, ce qui permet au mobile de
développer contre l'API avant que le backend existe :

```bash
npx @stoplight/prism-cli mock openapi.yaml --port 4010
```

## Principes portés par le contrat

- Le mobile n'envoie **jamais** de vidéo : uniquement des landmarks JSON, par série,
  en envoi différé.
- Landmarks encodés en tableaux compacts `[x, y, z, v]`, sous-échantillonnés à
  10 FPS avant envoi (les 30 FPS servent l'alerte posturale locale, pas le Cloud).
- Les landmarks bruts ne sont persistés dans aucune table : bucket GCS, rétention
  30 jours. L'API de lecture n'expose que des résultats calculés.
- Ingestion asynchrone : `202 Accepted`, traitement via RabbitMQ, lecture par polling.
- Idempotence par `batchId` : un rejeu ne crée aucun doublon.
- Les statistiques ne portent que sur les séries filmées, et l'application le dit.

## Références

- SCRUM-20 — Spécifications techniques et contrats OpenAPI/Swagger
- SCRUM-23 — Contrat de données JSON de télémétrie v1.0 (espace Confluence BV)
