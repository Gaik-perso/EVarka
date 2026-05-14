# Rapport d'Audit — `EVarka`

*Date : 13 mai 2026 | Auditeur : Claude (claude-sonnet-4-6) | Branch : `claude/repo-code-review-Fz0bx`*

---

## Résumé exécutif

Landing page statique déployée via GitHub Pages sur `evarka.com`.
Design professionnel, responsive, avec carte Google Maps, formulaire
pré-inscription Google Forms et GA4. Le problème critique était une
clé API Google Maps non restreinte exposée publiquement dans le HTML.
La fonction de changement de langue renvoyait un simple `alert()`.

**Score global : 6 / 10**

---

## Forces

| Critère | Détail |
|---|---|
| Design professionnel | Gradient vert cohérent, responsive mobile, animations |
| CNAME configuré | `evarka.com` actif via GitHub Pages |
| GA4 intégré | Suivi des événements CTA et formulaire |
| Formulaire pré-inscription | Intégration Google Forms — pas de backend requis |
| Favicon SVG inline | Pas de dépendance externe pour l'icône |
| CNAME | Déploiement propre |

---

## Problèmes identifiés

### CRITIQUE — Sécurité

#### 1. Clé API Google Maps exposée sans restriction

**Problème :** La clé `AIzaSyBO7fMvnB...91wYNuGk` est visible
dans le code source HTML public. N'importe qui peut :
- L'utiliser pour des milliers de requêtes Maps **à vos frais**
- L'utiliser dans ses propres projets
- Déclencher des coûts qui atteignent votre quota mensuel

**Note :** Il est inévitable qu'une clé Google Maps soit visible côté client
dans une page statique. La solution n'est **pas** de la cacher (impossible),
mais de la **restreindre à votre domaine uniquement**.

**Correction appliquée dans ce PR :**
Ajout d'un commentaire HTML explicite dans `index.html` indiquant la procédure
de restriction avec le lien direct vers Google Cloud Console.

**Action manuelle URGENTE requise (voir ci-dessous) :**
Restreindre la clé dans la console Google Cloud.

> **Note :** Pour les projets disposant d'un backend, fournir la clé via une
> variable d'environnement ou un gestionnaire de secrets (ex. GitHub Secrets,
> Google Secret Manager) plutôt qu'en clair dans le code source ou la
> documentation versionnée.

---

### Important

#### 2. `toggleLanguage()` non implémentée

**Problème initial :** La fonction renvoyait un simple `alert()` :
```javascript
function toggleLanguage() { alert('Fonction multilingue à implémenter avec i18n'); }
```
Visible des utilisateurs qui cliquent sur le bouton "EN".

**Correction appliquée dans ce PR :**
Implémentation complète PL/EN avec :
- Objet `i18n` contenant les traductions pour les deux langues
- Attributs `data-i18n` sur tous les éléments traduisibles
- `data-i18n-placeholder` sur les champs `email` et `city` pour traduire les placeholders
- Fonction `applyLanguage(lang)` pour appliquer une langue sans effet de bord
- Fonction `toggleLanguage()` qui délègue à `applyLanguage()` et persiste dans `localStorage`
- Mise à jour de `document.documentElement.lang` pour l'accessibilité
- Persistance via `localStorage` (clé `evarka_lang`) avec validation de la valeur stockée

---

#### 3. README vide

**Problème :** `README.md` contient seulement "Coming soon".
Un README doit au minimum décrire le projet.

**Correction manuelle recommandée (voir ci-dessous).**

---

### Mineur

#### 4. CSS entièrement inline et minifié

**Problème :** Le CSS est dans un seul bloc `<style>` minifié dans le `<head>`.
Difficile à maintenir, impossible à cacher dans les navigateurs (pas de cache CSS).

**Impact actuel :** Faible (1 seul fichier). Deviendra problématique si le site grandit.

---

#### 5. Données de démo hardcodées dans le JS

**Problème :** `demoChargers` est un tableau JS avec 6 bornes fictives.
Quand de vraies données existeront, il faudra refactoriser.

---

#### 6. Image Unsplash externe

**Problème :**
```html
<img src="https://images.unsplash.com/photo-1593941707882-a5bba14938c7?w=600&h=400&fit=crop">
```
Si Unsplash change l'URL ou la supprime, l'image disparaît sans avertissement.

---

#### 7. Pas de Content Security Policy

**Problème :** Aucun header CSP configuré. Expose la page aux injections XSS
via les scripts tiers (Google Maps, GA4, Font Awesome, Google Fonts).

---

## Corrections appliquées dans ce PR

| Fichier | Action | Détail |
|---|---|---|
| `index.html` | **Modifié** | Commentaire sécurité bilingue avant la clé Maps + `data-i18n` sur les éléments traduisibles + `data-i18n-placeholder` sur les inputs + `applyLanguage()` helper + `toggleLanguage()` implémentée (PL/EN) + `localStorage` persistence + `document.documentElement.lang` mis à jour |

---

## Corrections manuelles requises

### A. URGENT — Restreindre la clé Google Maps API

**Pourquoi :** Risque financier réel. Sans restriction, n'importe qui peut
utiliser votre quota à votre place.

**Durée estimée :** 5 minutes

**Étapes détaillées :**

1. Aller sur **Google Cloud Console** :
   [https://console.cloud.google.com/apis/credentials](https://console.cloud.google.com/apis/credentials)

2. Sélectionner le projet associé à la clé Maps utilisée en production (ex. `AIzaSyBO7fMvnB...91wYNuGk`)

3. Cliquer sur la clé pour l'éditer

4. Dans la section **"Restrictions d'application"**, sélectionner **"Référents HTTP (sites web)"**

5. Dans **"Référents HTTP autorisés"**, ajouter :
   ```text
   https://evarka.com/*
   https://www.evarka.com/*
   ```
   Si vous voulez aussi tester en local :
   ```text
   http://localhost:*/*
   ```

6. Dans la section **"Restrictions d'API"**, sélectionner **"Restreindre la clé"**
   et cocher uniquement :
   - Maps JavaScript API
   - Places API (si utilisée)

7. Cliquer **Enregistrer**

8. Vérifier que la carte fonctionne encore sur `evarka.com`

9. **Optionnel mais recommandé :** Activer les alertes de quota dans Google Cloud
   pour être notifié si des requêtes anormales apparaissent.

---

### B. Mettre à jour `README.md`

**Durée estimée :** 15 min

**Contenu minimal suggéré :**
```markdown
# EVarka

Landing page de pré-inscription pour EVarka — plateforme P2P de partage de
borne de recharge pour véhicules électriques en Pologne.

## Déploiement

GitHub Pages via branche `main`. Domaine : `evarka.com` (CNAME configuré).

## Structure

- `index.html` — landing page principale
- `chargeshare-maps.html` — page carte étendue

## Technologies

- HTML/CSS/JS statique
- Google Maps JavaScript API
- Google Forms (pré-inscription)
- Google Analytics 4

## Développement local

Ouvrir `index.html` dans un navigateur. Aucun build requis.
```

---

### C. Héberger l'image hero en local

**Pourquoi :** Éviter la dépendance à Unsplash.

**Durée estimée :** 5 min

**Étapes :**
1. Télécharger l'image ou utiliser une image libre de droits propre au projet.
2. Placer dans `assets/images/ev-charging.jpg`.
3. Remplacer dans `index.html` :
   ```html
   <img src="assets/images/ev-charging.jpg" alt="EV Charging" ...>
   ```

---

### D. Ajouter un Content Security Policy

**Pourquoi :** Réduire le risque XSS via les CDN externes.

**Durée estimée :** 30 min

> **⚠️ Important — ne jamais utiliser `'unsafe-inline'` dans `script-src`** : cela
> annule entièrement la protection XSS que CSP est censée apporter. Les scripts
> inline doivent être autorisés via des **nonces** ou des **hashes**, jamais via
> `'unsafe-inline'`. De même, évitez `*` ou les domaines externes non nécessaires
> dans vos directives.

**Étape préalable — externaliser le JS inline :**

Le JS de `index.html` est actuellement dans un bloc `<script>` embarqué. Pour
activer une CSP stricte, déplacez-le dans `app.js` externe :
```html
<script src="app.js" defer></script>
```

**Configuration CSP stricte avec nonce (si backend disponible) :**

Générer un token aléatoire côté serveur à chaque requête et l'injecter :
```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'self';
               script-src 'self' 'nonce-RANDOM_PER_REQUEST' https://maps.googleapis.com https://www.googletagmanager.com;
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdnjs.cloudflare.com;
               img-src 'self' https://images.unsplash.com data:;
               font-src 'self' https://fonts.gstatic.com https://cdnjs.cloudflare.com;
               connect-src 'self' https://www.google-analytics.com;">
```

Ajouter l'attribut `nonce` sur chaque `<script>` :
```html
<script nonce="RANDOM_PER_REQUEST" src="app.js"></script>
```

**Alternative pour site 100 % statique — hashes SHA-256 :**

1. Calculer le hash du contenu de chaque script inline :
   ```bash
   echo -n 'contenu du script' | openssl dgst -sha256 -binary | base64
   ```
2. Ajouter le hash à `script-src` : `'sha256-xxxx...'`

**Option Netlify (recommandée — headers HTTP réels) :**

Migrer vers Netlify et ajouter un fichier `netlify.toml` :
```toml
[[headers]]
  for = "/*"
  [headers.values]
    Content-Security-Policy = "default-src 'self'; script-src 'self' https://maps.googleapis.com https://www.googletagmanager.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdnjs.cloudflare.com; img-src 'self' https://images.unsplash.com data:; font-src 'self' https://fonts.gstatic.com https://cdnjs.cloudflare.com"
```

Note : `style-src 'unsafe-inline'` reste nécessaire pour les styles inline des
info-bulles Google Maps — acceptable pour une landing page statique où le risque
d'injection CSS est faible.

---

## Roadmap suggérée

| Priorité | Tâche | Effort |
|---|---|---|
| **P0** | **Restreindre la clé Google Maps dans Cloud Console** | **5 min — URGENT** |
| P1 | Mettre à jour README.md | 15 min |
| P2 | Héberger l'image hero en local | 5 min |
| P3 | Séparer CSS dans `style.css` | 30 min |
| P4 | Remplacer les données de démo par une API ou un JSON | selon architecture |
| P5 | Ajouter une CSP (migrer Netlify + externaliser JS) | 2h |

---

*implémentation PL/EN i18n complète*
