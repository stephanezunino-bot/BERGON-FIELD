# BERGON FIELD — DOCUMENTATION PROJET COMPLÈTE

> Document consolidé · Avril 2026
> Produit par Claude (Anthropic) · Stéphane Zunino, Bergon SAS
> Source de vérité unique — lire en premier à chaque session

-----

# 00 — MASTER BRIEF

## Contexte produit

**Bergon Field** est un CRM agronomique augmenté, mobile-first, destiné aux technico-commerciaux de Bergon SAS (distributeur d’intrants viticoles, Le Muy, 83).

- **Interlocuteur** : Stéphane Zunino, technico-commercial Bergon SAS
- **Agence dev** : auroracode.fr → `bergon.auroracode.fr`
- **Repo GitHub** : `github.com/stephanezunino-bot/BERGON-FIELD`

> ⚠️ Le repo GitHub DOIT rester **public**. Les JSON sont chargés via URL raw GitHub. Si le repo devient privé → erreur 404 sur toutes les données → appli hors service.

## Objectif

Permettre au technico-commercial de :

- Gérer son portefeuille clients viticulteurs avec vision agronomique instantanée
- Consulter données sol, ZNT, météo, IFT par parcelle
- Générer des recommandations produits adaptées au profil de chaque parcelle
- Documenter visites et interventions
- Accéder à un assistant IA contextuel avec données client injectées automatiquement

Usage **terrain, en mobilité**, connexion parfois dégradée. Lisibilité immédiate, action en 1–2 taps.

## Ce qui n’est PAS dans le périmètre Field

- Application native iOS/Android (WebApp responsive suffit)
- Gestion comptable ou facturation → **Bergon Logistique** (repo futur)
- Gestion RH, planning équipe → **Bergon RH** (repo futur)
- Intégration ERP Bergon

## Stack technique

|Composant        |Technologie                                                   |
|-----------------|--------------------------------------------------------------|
|Frontend         |HTML/JS ou Vue.js                                             |
|Carte            |Leaflet.js + tuiles IGN                                       |
|Backend          |OVH VPS (PHP ou Node)                                         |
|Auth             |Login par commercial                                          |
|Chat IA          |Claude API `claude-sonnet-4-6`, contexte injecté dynamiquement|
|ZNT              |Fichier DDTM83 BCAE (fonctionnel V1)                          |
|Météo            |API Weenat via Cloudflare Worker proxy                        |
|Données statiques|JSON hébergés sur GitHub                                      |

-----

# 01 — ARCHITECTURE PRODUIT

## Vision 3 outils — 3 repos

### 🌿 Bergon Field (repo actuel)

**Périmètre** : terrain, avant et pendant la visite client

Onglets prévus :

- 🗺 Carte — parcelles, ZNT, sol, météo *(existant)*
- 👤 Clients — portefeuille CRM *(existant)*
- 📊 Dashboard — KPIs commerciaux *(existant)*
- 🌿 Programme Phyto — vigne *(existant)*
- 🧪 Analyses Sol — AUREA + scoring Aurora *(en cours)*
- 📋 Compte-rendu — visite terrain → PDF client
- 📦 Commande terrain → export CEGID
- 📅 Planning + alertes BBCH
- 📈 Suivi IFT historique
- 📰 Newsletter technique ciblée
- 💧 Irrigation Weenat + conseil
- 🫒 Programme Olivier *(plus tard)*

### 📦 Bergon Logistique *(repo futur)*

Injection commandes CEGID · stocks dépôts HYE/MUY · facturation entités

### 👥 Bergon RH *(repo futur)*

Objectifs commerciaux · planning tournées · reporting direction

**Règle** : Field = terrain. Logistique = bureau. RH = management. Jamais de comptabilité dans Field.

-----

## Architecture UX — 3 niveaux obligatoires

Navigation **linéaire descendante** : Portefeuille → Exploitation → Parcelle.
Pas de vue parcelle-centrique en premier accès.

```
PORTEFEUILLE
    └── EXPLOITATION (vue client globale)  ← ⚠️ MANQUANT EN V1
            └── PARCELLE (vue détail)
```

Chaque niveau : Chat IA avec contexte de ce niveau injecté automatiquement.

### Niveau 1 — PORTEFEUILLE

- Liste clients triable par score / urgence / dernière visite
- Score global exploitation (pondéré par surface) + code couleur
- Badge urgence rouge si alerte critique (analyse expirée, ZNT)
- Date dernière visite + jours écoulés
- Filtre par zone géo / statut · Barre recherche
- Chargement < 2s sur 4G · tri par défaut : alertes critiques en premier

### Niveau 2 — EXPLOITATION *(❌ MANQUANT V1)*

- Carte toutes parcelles, couleur = score
- Score domaine pondéré : `Σ(score_i × surface_i) / Σ(surface_i)`
- Bloc alertes consolidées triées par criticité
- Tableau parcelles (nom, surface, cépage, score, alertes, analyse, action)
- Historique interventions (filtrable : visite / traitement / analyse / note)
- Météo Weenat 7 jours + tensiométrie + indicateur risque mildiou
- Bouton Chat IA permanent avec contexte client injecté

### Niveau 3 — PARCELLE

- Identité : nom, surface, cépage, porte-greffe, densité, âge, cadastre
- Carte Leaflet + ZNT buffers 5m/20m BCAE
- Analyse sol complète + scoring + interprétation par paramètre
- Min. 3 recommandations produits Bergon si profil justifié
- Météo 7 jours glissants + prévisions J+3 + OAD mildiou
- Historique amendements / rendements / traitements
- Actions : note terrain / analysecommandée / compte-rendu PDF / IA

### Règles générales navigation

- Breadcrumb permanent : `Portefeuille > EARL Baude > Parcelle Les Lauriers`
- Bouton retour toujours visible
- Chat IA icône flottante à tous les niveaux
- Pas de modal sur mobile → slide-in panels
- Chargement progressif

-----

# 02 — DESIGN SYSTEM BERGON FIELD

## Palette de couleurs

```css
/* Fonds — thème sombre obligatoire */
--bg-primary:     #0d0f0a;
--bg-surface:     #1f2418;
--bg-elevated:    #2a2f1e;
--border-subtle:  #3a4028;

/* Marque — SACRÉ, ne jamais remplacer */
--green-bergon:   #2E9E3A;   /* ✅ COULEUR OFFICIELLE BERGON */
--green-light:    #4db85a;
--green-dim:      #1a5c21;

/* Statuts */
--status-critical: #E3281E;
--status-warning:  #F59E0B;
--status-ok:       #2E9E3A;
--status-neutral:  #6B7280;

/* Scores agronomiques */
--score-excellent: #2E9E3A;  /* 80–100 */
--score-good:      #84CC16;  /* 60–79 */
--score-medium:    #F59E0B;  /* 40–59 */
--score-poor:      #EF4444;  /* 20–39 */
--score-critical:  #E3281E;  /* 0–19  */

/* Textes */
--text-primary:   #F0F4E8;
--text-secondary: #A8B89A;
--text-muted:     #6B7280;
```

## Typographie

|Usage                            |Police            |Taille |
|---------------------------------|------------------|-------|
|Titres, noms exploitations       |Cormorant Garamond|24–32px|
|Corps, UI, boutons               |Lexend            |14–16px|
|Scores, valeurs numériques, dates|DM Mono           |12–20px|

```html
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:wght@400;600;700&family=Lexend:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
```

## Composants clés

```css
/* Bandeau alerte critique — sticky, non masquable */
.alert-banner-critical {
  background: #E3281E; color: white; padding: 12px 20px;
  font-family: 'Lexend'; font-weight: 600; font-size: 14px;
  text-align: center; position: sticky; top: 0; z-index: 1000;
}

/* Badges */
.badge-critical { background: rgba(227,40,30,0.2); color: #E3281E; border: 1px solid #E3281E; }
.badge-warning  { background: rgba(245,158,11,0.2); color: #F59E0B; border: 1px solid #F59E0B; }
.badge-ok       { background: rgba(46,158,58,0.2);  color: #2E9E3A; border: 1px solid #2E9E3A; }
.badge { padding: 3px 8px; border-radius: 4px; font-family: 'Lexend'; font-size: 11px; font-weight: 600; }

/* Boutons */
.btn-primary   { background: #2E9E3A; color: #0d0f0a; font-family: 'Lexend'; font-weight: 600; padding: 10px 20px; border-radius: 6px; }
.btn-secondary { background: transparent; color: #2E9E3A; border: 1px solid #2E9E3A; padding: 10px 20px; border-radius: 6px; }

/* ZNT mobile — scroll horizontal obligatoire */
.table-znt-wrapper { overflow-x: auto; -webkit-overflow-scrolling: touch; }
```

```javascript
// Couleurs parcelles Leaflet par score
score > 60  → { fillColor: '#2E9E3A', color: '#1a5c21', weight: 2, fillOpacity: 0.4 }
score 40–60 → { fillColor: '#F59E0B', color: '#b45309', weight: 2, fillOpacity: 0.4 }
score < 40  → { fillColor: '#E3281E', color: '#991b1b', weight: 2, fillOpacity: 0.4 }
ZNT buffer  → { fillColor: '#3B82F6', color: '#1D4ED8', weight: 1.5, fillOpacity: 0.25, dashArray: '4 4' }
```

-----

# 03 — RÈGLES MÉTIER — Non négociables

## Calcul IFT

```
IFT traitement = dose appliquée / dose AMM de référence officielle
```

- **Nutrition exclue** — engrais foliaires et biostimulants ≠ phyto AMM
- **IFT biocontrôle séparé** de l’IFT hors biocontrôle
- Dose de référence = champ `dose_ref_ift` dans `catalogue_phyto.json`

## Cuivre SPe1

- Maximum **4 kg Cu métal/ha/an** — alerte obligatoire si dépassement

## Mélanges extemporanés interdits (arrêté 12/06/2015)

Toujours interdits avec H340, H350, H360, H370, H372.

|    |H351|H341|H371|H373|H361|H362|
|----|:--:|:--:|:--:|:--:|:--:|:--:|
|H351|❌   |❌   |❌   |    |    |    |
|H341|❌   |❌   |❌   |    |    |    |
|H371|❌   |❌   |❌   |❌   |    |    |
|H373|    |    |❌   |❌   |❌   |❌   |
|H361|    |    |    |❌   |❌   |❌   |
|H362|    |    |    |❌   |❌   |❌   |

## UAB — Statuts corrects (avril 2026)

|Produit    |UAB                            |
|-----------|:-----------------------------:|
|KAIZ’N     |✅                              |
|KAIZ’NK    |✅                              |
|PANGOLIN DG|❌                              |
|REDELI     |❌                              |
|YARIS      |❌ (H362 ≠ CMR HVE mais non UAB)|

## CMR et HVE V4

- H341, H351, H361 → CMR2 — pénalisé HVE V4
- H362 → neutre HVE (ne pas confondre avec CMR2)

## CEGID — Quantités

- **Toujours en unités** (bidons, sacs, packs) — jamais en litres
- Pack Zeus = 1 pack / 1.5 ha · Pack Orondis = 1 pack / 5 ha

## Oxathiapiproline

- Max 2 applications/saison · pas 2 consécutives · alerte si non respecté

## Bore — Var

- Seuil critique : 0.5 mg/kg eau chaude · Fenêtre : BBCH 55–68
- Produits : Laminaflor, Kori Force, STICoP Innov, HomeoPLante 172

## Analyse sol

- 3 ans → badge “À renouveler” (orange)
- ≥ 5 ans → **bandeau rouge bloquant sticky** — non masquable
- Score domaine toujours **pondéré par surface** : `Σ(score_i × surface_i) / Σ(surface_i)`

-----

# 04 — STACK FICHIERS & OPÉRATIONNEL

## Fichiers du repo BERGON-FIELD

|Fichier                 |Rôle                                |Qui modifie                |
|------------------------|------------------------------------|---------------------------|
|`programme-phyto.html`  |Outil programme phytosanitaire vigne|Claude → uploader          |
|`catalogue_phyto.json`  |Source unique produits phyto        |GitHub directement         |
|`bergon-field.html`     |Cartographie desktop                |Dev Aurora                 |
|`bergon-field-v2.html`  |Cartographie mobile                 |Dev Aurora                 |
|`bergon-phyto-link.js`  |Bouton lien Field → Phyto           |Rarement                   |
|`clients_parcelles.json`|Données clients + polygones         |Dev Aurora                 |
|`analyses_aurea.json`   |Analyses sol AUREA                  |Dev Aurora                 |
|`weenat_plots.json`     |Stations météo Weenat               |Dev Aurora                 |
|`ddtm83_bcae.json`      |Cours d’eau BCAE ZNT                |Figé                       |
|`BERGON_PROJECT_DOCS.md`|Ce fichier — source de vérité       |Claude après chaque session|
|`programmes/`           |Programmes sauvegardés              |Automatique via appli      |

## URLs

- Phyto : `https://stephanezunino-bot.github.io/BERGON-FIELD/programme-phyto.html`
- Field desktop : `https://stephanezunino-bot.github.io/BERGON-FIELD/bergon-field.html`
- Field mobile : `https://stephanezunino-bot.github.io/BERGON-FIELD/bergon-field-v2.html`
- Catalogue brut : `https://raw.githubusercontent.com/stephanezunino-bot/BERGON-FIELD/main/catalogue_phyto.json`

## Token GitHub

- Compte : `stephanezunino-bot` · Scope : `repo`
- Saisie : une fois par appareil dans l’écran config de programme-phyto.html
- Stockage : localStorage (perdu si cache vidé)
- Reconfiguration : bouton ⚙️ Token dans écran 3
- Fonctionne uniquement depuis GitHub Pages — **pas depuis file://**

## Ajouter un produit au catalogue

1. Ouvrir `catalogue_phyto.json` sur GitHub
1. Copier une entrée existante de même cible
1. Remplir : `nom`, `cible`, `famille`, `dose`, `unite`, `cmr`, `mentions_h`, `uab`, `dar`, `cu_kg_ha`, `calcul_ift`, `dose_ref_ift`, `cond_taille`, `cond_label`, `cond_type`
1. Committer — disponible immédiatement

-----

# 05 — ROADMAP

## ✅ Livré — programme-phyto.html (avril 2026)

- Saisie client + 5 entités juridiques · passages libres 1-20 · dates auto
- Catalogue depuis GitHub · filtre UAB · copie passage précédent
- Alertes mélanges / DAR / cuivre SPe1 / ZNT / fractions consécutives
- IFT hors biocontrôle + IFT biocontrôle séparés
- Ventilation CEGID en unités par entité + stock N-1
- Export CSV CEGID · export JSON · sauvegarde GitHub
- Impression calendrier A4 paysage · séparations Mildiou/Oïdium/Nutrition
- Bouton retour ← Bergon Field · bouton ⚙️ Token

## 🔴 Priorité haute — en attente

- Codes CEGID articles foliaires (export SQL Frédéric Bergon)
- Codes clients CEGID (export F_COMPTET agent 360)
- Ajouter `<script src="bergon-phyto-link.js"></script>` dans bergon-field.html et v2
- Vérifier doses AMM IFT sur alim.agriculture.gouv.fr (Pack Zeus, Yucca, Pangolin)

## 🔴 Priorité haute — Aurora (Bergon Field)

- Vue exploitation (niveau 2) — ❌ MANQUANT V1
- Score domaine pondéré par surface — ❌ MANQUANT
- Chat IA avec contexte client — ❌ MANQUANT
- Moteur scoring connecté aux vues — ⚠️ DÉFECTUEUX
- Recommandations sol complètes (min. 3) — ⚠️ DÉFECTUEUX

## 🟡 Planifié

- Injection SQL directe F_DOCENTETE + F_DOCLIGNE
- Champ code client CEGID dans Step 1
- Onglet Compte-rendu visite → PDF
- Onglet Analyses Sol (Aurora)
- ZNT domaine toutes parcelles en 1 clic
- Météo OAD 7 jours glissants + indicateur risque mildiou

## 🟢 Plus tard

- Programme Olivier
- Fusion bergon-field.html + v2 → fichier responsive
- Bouton “Ajouter une note terrain”
- Historique amendements / rendements
- Cache offline (Service Worker)
- Backend Aurora remplace GitHub

-----

# 06 — BRIEF CORRECTIF AURORA (ACTIF)

> Transmis à auroracode.fr. Toute livraison doit contenir : URL staging · changelog avec références · screenshots iPhone Safari · note technique si écart spec.

## P1-A : Vue exploitation

**Attendu** : carte parcelles colorées par score · score pondéré · alertes consolidées · tableau parcelles · historique · météo Weenat · bouton Chat IA

## P1-B : Score domaine

**Attendu** : affiché en grand (DM Mono, 64px, coloré) dans vue parcelle ET exploitation
Formule : `Σ(score_i × surface_i) / Σ(surface_i)`

## P1-C : Chat IA

```
Prompt système :
"Tu es l'assistant agronomique de Bergon SAS. Voici le contexte client :
- Exploitation : [nom], [commune], [surface totale]
- Parcelles : [liste avec scores et alertes]
- Dernière analyse sol : [date, statuts par paramètre]
- Alertes actives : [liste]
- Météo 7 jours : [cumuls, températures]
Réponds en français. Sois précis et opérationnel."
```

API : `claude-sonnet-4-6` · clé côté serveur uniquement

## P1-D : Recommandations sol

Min. 3 recommandations si le profil le justifie. Chaque recommandation : produit Bergon, dose, justification, période.

## P2-A : ZNT domaine

Bouton “Voir toutes les ZNT” → carte + tableau récapitulatif toutes parcelles

## P2-B : Bandeau alerte analyse expirée

Texte : `"⚠️ ANALYSE SOL EXPIRÉE — [Parcelle] — [X] ans. Renouvellement urgent."`
CSS sticky, non masquable — voir Design System section 02

## P2-C : Tableau ZNT mobile

`overflow-x: auto; -webkit-overflow-scrolling: touch;` — aucune colonne tronquée

## P2-D : Typographie et vert `#2E9E3A`

Voir Design System section 02 — partout, sans exception

## P3-A : Météo OAD 7 jours

Cumul pluie 7j · temp min/max/jour · prévisions J+3 · indicateur mildiou Goidanich simplifié

-----

# 07 — JOURNAL — État session 16 avril 2026

## Fichiers à uploader dans le repo

|Fichier                 |Statut                                                                  |
|------------------------|------------------------------------------------------------------------|
|`programme-phyto.html`  |✅ Livré — corrections impression + IFT + CEGID unités                   |
|`catalogue_phyto.json`  |✅ Livré — UAB corrects + conditionnements                               |
|`bergon-phyto-link.js`  |✅ Livré — bouton Phyto dans nav Field                                   |
|`BERGON_PROJECT_DOCS.md`|✅ Ce fichier                                                            |
|`bergon-field.html`     |⏳ Ajouter `<script src="bergon-phyto-link.js"></script>` avant `</body>`|
|`bergon-field-v2.html`  |⏳ Idem                                                                  |

## Travaux CEGID en attente (Frédéric Bergon)

```sql
-- Foliaires/biostimulants manquants
SELECT GA_ARTICLE, GA_CODEARTICLE, GA_LIBELLE, GA_FAMILLENIV2, GA_PVHT
FROM F_ARTICLE WHERE GA_FERME='-' AND GA_INTERDITVENTE='-'
AND GA_FAMILLENIV2 IN ('FOL','BST','EFO','NUT','ENG')

-- Clients agent 360
SELECT CT_NUM, CT_INTITULE FROM F_COMPTET
WHERE CT_REPRESENTANT='360' AND CT_SOMMEIL=0
```

38 produits phyto déjà matchés dans la précédente session (15 avril).

## Décisions techniques actées

|Date    |Décision                    |Raison                   |
|--------|----------------------------|-------------------------|
|Avr 2026|Repo GitHub public permanent|JSON chargés via URL raw |
|Avr 2026|Catalogue JSON externe      |Modifications sans HTML  |
|Avr 2026|CEGID en unités bidons/packs|1 unité = 1 bidon        |
|Avr 2026|Nutrition exclue IFT        |Engrais ≠ phyto AMM      |
|Avr 2026|IFT biocontrôle séparé      |Méthode officielle       |
|Avr 2026|Blob URL impression         |srcdoc → demi-page Chrome|
|Avr 2026|bergon-phyto-link.js externe|Lien sans réécrire HTML  |
|Avr 2026|3 outils séparés à terme    |Field / Logistique / RH  |

## Instructions pour Claude — chaque session

1. Lire ce fichier en premier
1. Vérifier état repo GitHub
1. Reprendre Roadmap dans l’ordre de priorité
1. Appliquer les règles métier section 03 systématiquement
1. Mettre à jour section 07 (Journal) à la fin de la session

-----

*Bergon SAS — Le Muy, Var (83) · Stéphane Zunino*
*Développé avec Claude (Anthropic) — avril 2026*