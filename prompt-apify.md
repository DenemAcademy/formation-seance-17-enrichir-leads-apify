# Seance 17 - Prompt enrichissement Apify

```text
Tu es un growth data engineer senior, spécialiste Apify, crawling propre de sites B2B, enrichissement de contacts publics, déduplication et préparation de bases commerciales.

MISSION
Tu dois reprendre le fichier enrichi avec data.gouv :
@~/Desktop/mon-business/07-enrichissement-data-gouv/leads-data-gouv-enriched.xlsx

Tu dois créer une nouvelle version enrichie avec les contacts visibles publiquement sur les sites web officiels des entreprises.
Le but est d'ajouter :
- emails publics
- téléphones publics
- pages contact
- formulaires
- pages réservation
- pages privatisation
- Instagram
- Facebook
- LinkedIn
- TikTok
- WhatsApp
- sources exactes où les données ont été trouvées

Tu ne dois pas recréer une nouvelle base.
Tu dois conserver toutes les colonnes existantes et ajouter seulement des colonnes apify_.

SÉCURITÉ TOKEN
Lis le token Apify depuis :
process.env.APIFY_TOKEN

Si APIFY_TOKEN est absent, stoppe et dis :
"APIFY_TOKEN absent. Ajoute le token en variable d'environnement puis relance."

Ne stocke jamais le token dans un fichier.
Ne l'affiche jamais dans les logs.
Ne le mets jamais dans le XLSX.

ACTOR APIFY RECOMMANDÉ
Utilise en priorité :
betterdevsscrape/contact-details-extractor

Si cet actor n'est pas accessible, cherche un actor équivalent capable de :
- recevoir une liste d'URLs
- crawler quelques pages du même domaine
- extraire emails
- extraire téléphones
- extraire réseaux sociaux
- retourner les URLs sources

FICHIER D'ENTRÉE
Lis :
~/Desktop/mon-business/07-enrichissement-data-gouv/leads-data-gouv-enriched.xlsx

Colonnes importantes :
- lead_id
- company_name
- website
- domain
- email
- phone
- source_urls
- dg_siren
- dg_siret
- dg_raison_sociale
- dg_nom_commercial
- dg_match_status
- dg_match_score
- priority_score
- priority_class
- personalized_angle
- outreach_message

RÈGLE DE PRIORISATION
Traite d'abord :
1. dg_match_status = HIGH_CONFIDENCE
2. puis MEDIUM_CONFIDENCE
3. puis les leads avec website/domain présent
4. puis les leads sans email
5. puis les leads sans téléphone
6. puis les leads A+ / A

Ne traite pas automatiquement :
- LOW_CONFIDENCE
- NOT_FOUND
- ADDRESS_ONLY_CHECK
sauf si l'utilisateur demande explicitement un enrichissement exploratoire.

STRUCTURE DE SORTIE
Crée ou complète :
~/Desktop/mon-business/08-enrichissement-apify/

Crée :
- leads-apify-enriched.xlsx
- leads-apify-enriched.csv
- rapport-apify.md
- apify-input-batches/
- apify-raw-runs/
- apify-normalized/
- apify-ledger.csv

COLONNES À AJOUTER
Ajoute au minimum :
- apify_run_id
- apify_actor
- apify_start_url
- apify_domain
- apify_pages_crawled
- apify_email_found
- apify_email_all
- apify_email_type
- apify_email_confidence
- apify_email_source_page
- apify_phone_found
- apify_phone_all
- apify_phone_type
- apify_phone_confidence
- apify_phone_source_page
- apify_instagram
- apify_facebook
- apify_linkedin
- apify_tiktok
- apify_youtube
- apify_whatsapp
- apify_contact_page
- apify_booking_page
- apify_privatisation_page
- apify_mentions_legales_page
- apify_form_detected
- apify_source_pages
- apify_contact_score
- apify_enrichment_status
- apify_merge_status
- apify_notes
- recommended_contact_channel
- recommended_next_action

PHASE 0 — AUDIT DE LA BASE
Avant tout appel Apify :
1. Compte le nombre total de leads.
2. Compte les leads avec website.
3. Compte les leads sans email.
4. Compte les leads sans téléphone.
5. Compte HIGH_CONFIDENCE / MEDIUM_CONFIDENCE / ADDRESS_ONLY_CHECK / LOW_CONFIDENCE / NOT_FOUND.
6. Estime le nombre de sites à crawler.
7. Crée rapport-apify.md avec ce diagnostic initial.

PHASE 1 — CONSTRUIRE LES BATCHES
Crée des lots de 50 sites maximum pour commencer.
Chaque entrée Apify doit contenir :
- lead_id
- company_name
- website
- domain
- dg_siren
- dg_siret
- dg_match_status

Normalise les URLs :
- ajoute https:// si nécessaire
- retire les paramètres inutiles
- garde le domaine officiel du lead
- ignore les réseaux sociaux comme site principal sauf si aucun autre site n'existe

Crée :
apify-input-batches/batch-001.json
apify-input-batches/batch-002.json
etc.

PHASE 2 — TEST SUR ÉCHANTILLON
Lance d'abord Apify sur 50 sites maximum.
Configuration recommandée :
- maxPagesPerStartUrl : 8
- maxDepth : 2
- sameDomain : true
- mergeContacts : true
- browserMode : auto
- useProxy : false au départ
- timeout raisonnable

Pages à prioriser :
- /
- /contact
- /contactez-nous
- /a-propos
- /about
- /team
- /equipe
- /mentions-legales
- /mentions-légales
- /reservation
- /réservation
- /privatisation
- /groupes
- /evenements
- /recrutement

Après l'échantillon, mesure :
- emails trouvés
- téléphones trouvés
- réseaux trouvés
- no contact
- sites bloqués
- coût estimé
- temps moyen

Si le taux email + téléphone est inférieur à 10 %, fais un diagnostic avant de continuer.
Si le taux est acceptable, continue par lots.

PHASE 3 — LANCER LES BATCHES
Pour chaque batch :
1. Lance l'actor.
2. Sauvegarde l'input.
3. Sauvegarde l'output brut dans apify-raw-runs/.
4. Normalise les résultats dans apify-normalized/.
5. Écris apify-ledger.csv :
   timestamp, batch_id, actor, run_id, input_count, pages_crawled, emails_found, phones_found, socials_found, cost_estimate, status.

Ne relance pas un batch déjà réussi.
Si un site échoue, note l'échec dans apify_notes.

PHASE 4 — NORMALISER LES CONTACTS
Emails :
- nettoie les espaces
- retire mailto:
- convertis contact [at] domaine.fr vers contact@domaine.fr
- retire les emails techniques inutiles si non pertinents
- garde plusieurs emails dans apify_email_all
- choisis le meilleur dans apify_email_found

Classe apify_email_type :
- nominatif_pro
- contact_general
- reservation
- commercial
- recrutement
- free_email
- technical
- unknown

Priorité email :
1. email sur le domaine officiel
2. email contact général
3. email réservation si restaurant
4. email nominatif pro si cohérent
5. email free provider
6. email recrutement seulement si l'offre est RH

Téléphones :
- nettoie les formats
- conserve le format international si possible
- distingue fixe / mobile si possible
- garde tous les numéros dans apify_phone_all
- choisis le meilleur dans apify_phone_found

Réseaux sociaux :
- garde seulement les liens plausibles liés à l'entreprise
- évite les icônes génériques ou liens de partage
- garde la source page.

PHASE 5 — MERGE AVEC LES DONNÉES EXISTANTES
Ne supprime jamais email ou phone existants.
Compare :
- email d'origine vs apify_email_found
- phone d'origine vs apify_phone_found

apify_merge_status :
- confirmed_existing : Apify confirme une donnée déjà présente
- new_contact_added : Apify ajoute une donnée absente
- conflict_to_review : Apify trouve une donnée différente
- no_contact_found : rien trouvé
- blocked_or_failed : site inaccessible ou actor échoué

PHASE 6 — SCORE DE CONTACT
Calcule apify_contact_score sur 100 :
- +30 email trouvé sur domaine officiel
- +20 email général ou réservation exploitable
- +25 téléphone trouvé
- +15 Instagram/Facebook/LinkedIn trouvé
- +10 page contact trouvée
- +10 page réservation ou privatisation trouvée
- +5 formulaire détecté
- -20 email free provider seulement
- -25 source hors domaine officiel
- -30 conflit avec données existantes sans confirmation

Statuts :
- CONTACT_COMPLETE : email + téléphone
- EMAIL_ONLY : email sans téléphone
- PHONE_ONLY : téléphone sans email
- SOCIAL_ONLY : seulement réseaux sociaux
- FORM_ONLY : seulement formulaire
- NO_CONTACT_FOUND : rien
- NEED_MANUAL_CHECK : ambigu, bloqué, conflit ou site cassé

PHASE 7 — ACTION RECOMMANDÉE
Crée recommended_contact_channel :
- email
- phone
- linkedin
- instagram
- form
- manual_review
- do_not_contact

Crée recommended_next_action :
- Envoyer email personnalisé
- Appeler puis envoyer email
- Contacter Instagram
- Utiliser formulaire
- Vérifier manuellement
- Exclure temporairement

Règles :
- Si CONTACT_COMPLETE : email personnalisé + appel possible
- Si EMAIL_ONLY : email personnalisé
- Si PHONE_ONLY : appel court puis demande du bon email
- Si SOCIAL_ONLY : contact social léger ou recherche manuelle
- Si NEED_MANUAL_CHECK : ne pas automatiser

PHASE 8 — CLASSEUR EXCEL
Crée leads-apify-enriched.xlsx avec :

1. APIFY_DASHBOARD
KPIs :
- leads traités
- sites crawlés
- emails trouvés
- téléphones trouvés
- Instagram trouvés
- LinkedIn trouvés
- CONTACT_COMPLETE
- EMAIL_ONLY
- PHONE_ONLY
- NO_CONTACT_FOUND
- NEED_MANUAL_CHECK
- taux de joignabilité
- coût estimé Apify

Graphiques :
- statut de contact
- email vs téléphone
- top domaines avec contact
- no contact par segment

2. ALL_LEADS_APIFY
Toutes les colonnes d'origine + dg_ + apify_.

3. READY_FOR_OUTREACH
Seulement les leads :
- dg_match_status HIGH_CONFIDENCE ou MEDIUM_CONFIDENCE
- apify_enrichment_status CONTACT_COMPLETE ou EMAIL_ONLY ou PHONE_ONLY
- contact_score suffisant

4. EMAIL_READY
Leads avec email exploitable.

5. PHONE_READY
Leads avec téléphone exploitable.

6. SOCIAL_READY
Leads avec réseaux mais pas email.

7. MANUAL_REVIEW
Conflits, sites bloqués, contacts ambigus.

8. NO_CONTACT_FOUND
Sites sans contact.

9. APIFY_LEDGER
Tous les runs Apify.

10. SOURCE_PAGES
Une ligne par page source de contact :
lead_id, company_name, contact_type, contact_value, source_page.

11. SCORING_RULES
Méthode de scoring contact.

12. NEXT_ACTIONS
Vue opérationnelle triée par priorité.

MISE EN FORME
- filtres
- lignes figées
- couleurs par apify_enrichment_status
- largeur lisible
- dashboard synthétique
- aucun token Apify

RAPPORT
Crée rapport-apify.md :
- actor utilisé
- configuration
- nombre de sites traités
- coût estimé
- taux email
- taux téléphone
- taux contacts complets
- limites
- recommandations pour la séance nettoyage/scoring

CONFORMITÉ
Utilise uniquement des contacts publiés publiquement par l'entreprise ou ses pages officielles.
Ne contourne aucun login, captcha, paywall ou restriction technique.
Ne collecte aucune donnée sensible.
Conserve la source de chaque contact.
La prospection devra être B2B, pertinente et inclure une possibilité d'opposition.

CONTRÔLE FINAL
Avant de finir :
- vérifie que le XLSX s'ouvre
- vérifie que les colonnes apify_ existent
- vérifie que READY_FOR_OUTREACH ne contient pas LOW_CONFIDENCE ou NOT_FOUND
- vérifie qu'aucun token n'apparaît dans les fichiers
- vérifie que chaque contact a une source ou une note
- vérifie que les statuts sont cohérents

MESSAGE FINAL
Réponds seulement :
"Enrichissement Apify terminé.
Fichiers :
- leads-apify-enriched.xlsx
- leads-apify-enriched.csv
- rapport-apify.md

Sites traités : X
Emails trouvés : Y
Téléphones trouvés : Z
Contacts complets : A
Email only : B
Phone only : C
À vérifier : D
Prêts pour outreach : E
Coût estimé Apify : F"
```
