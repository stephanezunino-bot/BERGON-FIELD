# ETAT BERGON FIELD

> Uploader ce fichier en début de chaque session Claude

-----

## 📅 Dernière mise à jour

21 avril 2026

## 👤 Contexte

- **Projet** : Bergon Field — CRM terrain augmenté pour technico-commercial viticole
- **Repo** : github.com/stephanezunino-bot/BERGON-FIELD (GitHub Pages)
- **Conseiller** : Stéphane Zunino — Bergon SAS, Le Muy (Var 83)

-----

## 📁 Fichiers actifs

|Fichier                 |Rôle                          |Statut |
|------------------------|------------------------------|-------|
|`bergon-field.html`     |App desktop principale        |✅ Actif|
|`bergon-field-v2.html`  |App mobile                    |✅ Actif|
|`programme-phyto.html`  |Module phytosanitaire         |✅ Actif|
|`catalogue_phyto.json`  |Données produits phyto        |✅ Actif|
|`clients_parcelles.json`|Données clients/parcelles     |✅ Actif|
|`ddtm83_bcae.json`      |Données ZNT cours d’eau DDTM83|✅ Actif|
|`analyses_aurea.json`   |Analyses de sol AUREA         |✅ Actif|
|`weenat_plots.json`     |Stations météo Weenat         |✅ Actif|
|`programmes/`           |PDFs programmes générés       |✅ Actif|
|`journal.md`            |Journal de bord               |✅ Actif|

-----

## 🔧 Règles métier critiques

- **CEGID** : quantités toujours en unités bidons/packs — JAMAIS en litres
- **ZNT** : filtre `bcae='t'` sur ddtm83_bcae.json (rivières permanentes ET intermittentes nommées)
- **Couleurs Bergon** : vert `#2E9B4E`, rouge `#E03020`
- **Retour programme-phyto → Bergon Field** : via paramètre URL `?retour=`

-----

## 🚧 En cours / Bloqué

- [ ] Intégrer `bergon-phyto-link.js` dans `bergon-field.html` ET `bergon-field-v2.html`
- [ ] Bug ZNT client Laroche (Cuers) — problème bbox/geometry margin non résolu
- [ ] Attendre export SQL foliaires/biostimulants de Frédéric Bergon
- [ ] Attendre export clients agent 360 de Frédéric Bergon
- [ ] Migration architecture : extraire logique commune vers `js/bergon-core.js`

-----

## ✅ Dernière session (21 avril 2026)

- Audit du repo BERGON-FIELD effectué (10 fichiers actifs)
- Architecture cible définie : séparation data/ js/ + HTML UI only
- Décision : garder deux fichiers HTML séparés desktop/mobile (voulu)
- Création de ce fichier ETAT.md

-----

## 📌 Prochaine session — dire à Claude

> “Voici mon ETAT.md Bergon Field. On continue sur [sujet].”
