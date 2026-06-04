# Seance 17 - Synthese pedagogique Apify

## Intention de la séance

Je pars de la base enrichie avec data.gouv et je la complète avec une couche contact issue des sites officiels : emails, téléphones, pages contact, pages de réservation, réseaux sociaux et sources.

## Chronologie utile

| Moment | Ce que je fais | Pourquoi c'est important |
|---|---|---|
| 00:00 | Je pose l'objectif Apify après data.gouv | On sépare vérification légale et recherche de contact. |
| 01:23 | J'ouvre la base Google Sheets | Je vérifie que les sites web sont disponibles avant de crawler. |
| 02:24 | J'introduis Apify et les actors | Apify est une bibliothèque de scrapers réutilisables. |
| 03:14 | Je montre d'autres actors comme TikTok | L'élève comprend que la logique s'adapte à d'autres cas clients. |
| 08:03 | Je cherche Website Contact Extractor | Je choisis l'actor adapté aux sites officiels. |
| 09:04 | Je teste une URL manuellement | Je valide le fonctionnement avant d'automatiser. |
| 11:18 | J'ouvre Claude Code / Codex dans le dossier | Le prompt doit piloter le système localement. |
| 12:00 | Je récupère une clé API via Apify | La clé reste une variable d'environnement, jamais un fichier. |
| 14:10 | Je laisse Codex enchaîner audit, batches, test, runs | La méthode devient industrielle et contrôlable. |
| 19:44 | Je surveille les batches de 50 | Je garde un compromis vitesse, coût et stabilité. |
| 23:40 | Je contrôle les batchs et les fails | Les erreurs sont normales, il faut les tracer. |
| 35:22 | Je surveille le budget autour de 4 dollars | On garde le contrôle sur le coût. |
| 38:00 | Je récupère le fichier final | La base contient les emails et contacts exploitables. |

## Prompts et commandes à reprendre

- Prompt maître : `prompt-apify.md`
- Variable d'environnement : `APIFY_TOKEN`
- Actor recommandé : `betterdevsscrape/contact-details-extractor`
- Batch de départ : 50 sites maximum
- Sortie : `leads-apify-enriched.xlsx`, `leads-apify-enriched.csv`, `rapport-apify.md`

## Points de vigilance

- Ne jamais publier un token Apify.
- Ne pas relancer un batch déjà réussi.
- Ne pas crawler des réseaux sociaux comme site principal si un domaine officiel existe.
- Ne pas écraser les colonnes déjà présentes.
- Toujours conserver la page source qui prouve le contact.
