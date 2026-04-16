# BERGON FIELD — Journal de bord technique

> **LIRE EN PREMIER à chaque session.**
> Mis à jour après chaque session de travail.
> Dernière mise à jour : 16 avril 2026

-----

## ÉTAT DES FICHIERS — VERSION ACTUELLE

### 🌿 programme-phyto.html

**Base** : version remixed fournie par Stéphane (15 avril 2026)
**Corrections session 16/04** :

- Impression : blob URL → pleine page A4 paysage garantie (plus de demi-page)
- Séparations visuelles Mildiou / Oïdium / Nutrition (barres dégradées colorées)
- IFT : Nutrition exclue, doses AMM officielles intégrées dans catalogue
- CEGID : quantités en **unités bidons/packs** (plus de litres)
- UAB corrigés : KAIZ’N ✅ KAIZ’NK ✅ PANGOLIN DG ❌ REDELI ❌
- Bug syntaxe JS corrigé (updStock guillemets imbriqués, rows.join)
- Bouton retour ← Bergon Field via `?retour=` dans URL

**Fonctionnalités actives :**

- Saisie client + 5 entités juridiques (surfaces CEGID)
- Passages libres (1-20), cadence auto AB/HVE
- Dates automatiques depuis T1 (13j floraison, 10j AB)
- Catalogue depuis GitHub (`catalogue_phyto.json`)
- Filtre UAB automatique en mode AB
- Copie passage précédent (📋)
- Alertes : mélanges interdits (arrêté 12/06/2015), DAR, cuivre SPe1, ZNT, fractions consécutives
- IFT hors biocontrôle + IFT biocontrôle séparés
- Ventilation CEGID en unités bidons/packs par entité + stock N-1 optionnel
- Export CSV CEGID (par passage, par entité, QTE_CEGID_UNITES)
- Export JSON programme complet
- Sauvegarde GitHub (dossier `programmes/`)
- Chargement liste clients depuis GitHub
- Impression calendrier A4 paysage
- Lien → Bergon Field

**Statut** : ✅ À uploader dans repo

-----

### 📦 catalogue_phyto.json

**Version** : v2 — avril 2026
**Contenu** : 57 produits (20 Mildiou, 18 Oïdium, 19 Nutrition)
**Champs clés** :

- `dose_ref_ift` : dose AMM officielle pour calcul IFT
- `calcul_ift` : false pour tous les Nutrition
- `cond_taille` / `cond_label` / `cond_type` : conditionnements CEGID
- `ha_par_unite` : Pack Zeus 1.5ha, Orondis 5ha, Enervin 2ha
- `mentions_h` : matrice mélanges interdits
- `uab` : corrigés avril 2026

**Pour ajouter un produit** : éditer ce JSON sur GitHub uniquement — jamais dans le HTML.

**Statut** : ✅ À uploader dans repo

-----

### 🔗 bergon-phyto-link.js

**Version** : v1 — avril 2026
**Usage** : ajouter `<script src="bergon-phyto-link.js"></script>` avant `</body>` dans bergon-field.html ET bergon-field-v2.html
**Effet** : bouton 🌿 Phyto apparaît dans la nav des deux applis

**Statut** : ✅ À uploader dans repo

-----

### 🗺 bergon-field.html + bergon-field-v2.html

**Statut** : ⏳ Ajouter la ligne `<script src="bergon-phyto-link.js"></script>` avant `</body>`

-----

## TRAVAUX EN COURS — AUTRES CONVERSATIONS

### CEGID — Table de correspondance produits (conversation du 15 avril)

**Ce qui a été fait :**

- Frédéric Bergon a fourni 3 CSV : `F_DOCENTETE` (50 commandes), `F_DOCLIGNE` (100 lignes), `AGRI_base_article.csv` (284 articles)
- Structure CEGID identifiée :
  - `GP_TIERS` = code client (format 6-8 lettres, ex: PEREMAN, CLEMMARI)
  - `GL_ARTICLE` = code article (ex: AGRI10832)
  - `GP_REPRESENTANT` = 360 (Stéphane)
  - Dépôts : HYE (Hyères) ou MUY (Le Muy)
- Table de correspondance construite : **38 produits phyto matchés automatiquement**
- **18 produits non trouvés** : tous les foliaires/biostimulants (Kori, Kaiz’N, Pepton, GeoDyn, Laminaflor…) → Frédéric n’a extrait que les phytos, pas les engrais foliaires

**Ce qu’il reste à faire :**

- [ ] Obtenir export foliaires/biostimulants depuis CEGID (nouvelle requête SQL)
- [ ] Intégrer la table de correspondance dans le HTML (colonne GA_ARTICLE dans CSV export)
- [ ] Ajouter champ “Code client CEGID” dans Step 1 de l’appli
- [ ] Générer CSV au format F_DOCENTETE + F_DOCLIGNE pour injection directe

**Requête SQL pour les foliaires manquants :**

```sql
SELECT GA_ARTICLE, GA_CODEARTICLE, GA_LIBELLE, GA_FAMILLENIV1,
       GA_FAMILLENIV2, GA_FAMILLENIV3, GA_PVHT
FROM F_ARTICLE
WHERE GA_FERME = '-'
  AND GA_INTERDITVENTE = '-'
  AND GA_FAMILLENIV2 IN ('FOL', 'BST', 'EFO', 'NUT', 'ENG')
ORDER BY GA_FAMILLENIV2, GA_LIBELLE
```

**Requête SQL clients agent 360 :**

```sql
SELECT CT_NUM, CT_INTITULE, CT_REPRESENTANT
FROM F_COMPTET
WHERE CT_REPRESENTANT = '360'
AND CT_SOMMEIL = 0
ORDER BY CT_INTITULE
```

-----

### Aurora auroracode.fr — Moteur agronomique (conversation du 7 avril)

**Ce qui a été livré :**

- Spec sections 3 et 5 v3.1 (`bergon_spec_moteur_agronomique_sections3et5_v3_1.md`)
- Scoring : Sol vivant 50pts (MO 30 + C/N 20) / Équilibre minéral 30pts / Oligos 20pts
- Matrice produits → paramètres avec doses, UAB, SOL/FOLIAIRE, filtres bio
- **Projet parallèle géré par le dev Aurora — pas lié au HTML phyto**

-----

## EN ATTENTE

|Priorité   |Tâche                                                        |Bloquant sur              |
|-----------|-------------------------------------------------------------|--------------------------|
|🔴 Haute    |Export CEGID foliaires/BST manquants                         |Frédéric Bergon / SQL     |
|🔴 Haute    |Export clients agent 360 (`F_COMPTET`)                       |Frédéric Bergon / SQL     |
|🟡 Normale  |Intégration codes CEGID dans catalogue JSON                  |CSV foliaires reçu        |
|🟡 Normale  |Champ code client CEGID dans appli                           |CSV clients reçu          |
|🟡 Normale  |Vérifier doses AMM IFT officielles Pack Zeus, Yucca, Pangolin|alim.agriculture.gouv.fr  |
|🟢 Plus tard|Injection SQL directe depuis CSV export                      |Structure tables confirmée|
|🟢 Plus tard|Fusion bergon-field.html + v2 en fichier responsive unique   |Stabilité appli phyto     |
|🟢 Plus tard|Backend Aurora remplace GitHub                               |Dev Aurora                |

-----

## DÉCISIONS TECHNIQUES

|Date    |Décision                          |Raison                                  |
|--------|----------------------------------|----------------------------------------|
|Avr 2026|Catalogue JSON externe dans GitHub|Modifications produits sans HTML        |
|Avr 2026|GitHub comme backend sauvegarde   |Gratuit, zéro serveur, iPhone           |
|Avr 2026|Blob URL pour impression          |srcdoc → rendu demi-page sur Chrome     |
|Avr 2026|CEGID en unités bidons/packs      |1 unité CEGID = 1 bidon, jamais litres  |
|Avr 2026|Nutrition exclue du calcul IFT    |Engrais/biostimulants ≠ phyto AMM       |
|Avr 2026|IFT biocontrôle séparé            |Méthode officielle Ministère Agriculture|
|Avr 2026|bergon-phyto-link.js externe      |Lien entre outils sans réécrire le HTML |
|Avr 2026|Pack Zeus = unité par 1.5 ha      |1 pack couvre 1.5 ha — pas en litres    |

-----

## TOKEN GITHUB

- Compte : `stephanezunino-bot`
- Repo : `BERGON-FIELD`
- Scope : `repo` (lecture + écriture Contents)
- Stockage : `localStorage` du navigateur (par appareil)
- Reconfiguration : bouton ⚙️ Token dans écran 3 de programme-phyto.html
- Fonctionne uniquement depuis GitHub Pages (pas depuis file://)
- URL GitHub Pages : `https://stephanezunino-bot.github.io/BERGON-FIELD/`

-----

## VISION PRODUIT — ARCHITECTURE GLOBALE

### Trois outils distincts — trois repos à terme

**🌿 Bergon Field** (repo actuel : BERGON-FIELD)
Outil terrain conseiller. Tout ce qui se passe avant et pendant la visite client.

- Carte parcelles / ZNT / Sol / Météo
- Portefeuille clients (CRM terrain)
- Programme Phyto (vigne, oliver à terme)
- Analyses Sol AUREA + scoring Aurora
- Compte-rendu visite (PDF)
- Commande terrain → export CEGID
- Planning visites / alertes BBCH
- Suivi IFT par client/saison
- Newsletter technique ciblée
- Suivi irrigation (Weenat)

**📦 Bergon Logistique** (repo futur : BERGON-LOGISTIQUE)
Gestion stocks, commandes, CEGID. Ce que fait le bureau.

- Injection commandes CEGID
- Suivi stocks dépôts (HYE, MUY)
- Ventilation facturation entités

**👥 Bergon RH** (repo futur : BERGON-RH)
Équipe, planning, objectifs commerciaux.

### Règle de périmètre

Bergon Field ne fait jamais de comptabilité, de gestion stocks ou de RH.
CEGID gère l’après-visite. Bergon Field gère l’avant et le pendant.

-----

## INSTRUCTIONS POUR LA PROCHAINE SESSION

1. **Lire ce fichier en premier** — `https://raw.githubusercontent.com/stephanezunino-bot/BERGON-FIELD/main/JOURNAL.md`
1. Vérifier l’état des fichiers dans le repo GitHub
1. Reprendre la liste “EN ATTENTE” dans l’ordre de priorité
1. **À la fin de la session** : régénérer ce JOURNAL mis à jour et l’uploader

-----

*Bergon SAS — Le Muy, Var (83) · Commercial : Stéphane Zunino*
*Outil développé avec Claude (Anthropic) — sessions itératives avril 2026*