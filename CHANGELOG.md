# Changelog

Toutes les modifications notables de ce projet sont documentées ici.
Format inspiré de [Keep a Changelog](https://keepachangelog.com/fr/1.0.0/),
versionnage [SemVer](https://semver.org/lang/fr/).

## [1.2.0] — 2026-09-10

### Ajouté
- Clé `browser_specific_settings.gecko.data_collection_permissions` requise
  par Mozilla pour toute nouvelle extension soumise sur AMO depuis le
  3 novembre 2025 (valeur `"none"` : aucune donnée collectée ni transmise).

### Modifié
- `strict_min_version` remonté de `109.0` à `140.0` (desktop) et ajout de
  `gecko_android.strict_min_version = 142.0` — versions minimales requises
  par Firefox pour prendre en charge `data_collection_permissions`.
- `lib/render.js`, `content/content.js` et `popup/popup.js` ne construisent
  plus le panneau de résultat via `innerHTML` : rendu entièrement en DOM
  (`createElement`/`textContent`/`appendChild`), conformément aux
  recommandations de sécurité du validateur AMO. Les sauts de ligne passent
  par `white-space: pre-line` en CSS plutôt que par des `<br>` injectés.

### Corrigé
- Élimine les 6 avertissements *« Unsafe assignment to innerHTML »* et les
  2 avertissements de compatibilité de version relevés par le validateur
  addons.mozilla.org.

## [1.1.1] — 2026-09-10

### Ajouté
- Détection du format `ip/masque décimal` (ex. `192.168.7.254/255.255.248.0`),
  en plus du CIDR classique (`ip/xx`), des paires espacées
  (`ip masque`) et des plages (`ip1-ip2`). Les quatre formats sont reconnus
  indépendamment ou mélangés dans un même texte.

## [1.1.0] — 2026-09-10

### Ajouté
- **IPv6** : détection et calcul complet (`2001:db8::/32`, etc.) — réseau,
  dernière adresse, nombre total d'adresses (BigInt), classification de
  portée (ULA, lien-local, documentation, multicast…).
- **Paires IP + masque** : `192.168.1.0 255.255.255.0` détecté et calculé
  comme un `/24`.
- **Plages d'adresses** : `10.0.0.5-10.0.0.20` avec détection automatique
  d'alignement sur un bloc CIDR exact.
- **Scission / agrégation interactives** : champ de préfixe cible + boutons
  *Diviser* / *Regrouper* dans le panneau de résultat, avec navigation
  récursive dans les sous-réseaux générés (IPv4 et IPv6).
- **Portées RFC** : badge Privée (RFC 1918), CGNAT (RFC 6598), loopback,
  lien-local, documentation, multicast, ULA IPv6, etc.
- **Bouton Copier** : copie le résultat formaté dans le presse-papiers.
- **Historique** : 20 dernières recherches (popup et menu contextuel),
  ré-exécutables en un clic, avec bouton d'effacement.
- **Liste blanche / liste noire de sites** : page d'options dédiée +
  bascule rapide « Désactiver ici / Réactiver ici » dans le popup pour le
  site courant (correspondance sur les sous-domaines).
- **Logo** : icône dédiée (grille d'adresses + loupe) en 16/32/48/96/128 px,
  déclinée depuis un SVG source.
- **Signature LAB-INF0** : `author`/`homepage_url` dans le manifeste, footer
  et section « Auteur » dans le README, pointant vers
  github.com/LAB-INF0 et github.com/LAB-INF0/.githLAB.

## [1.0.0] — 2026-09-10

### Ajouté
- Version initiale : détection et surlignage automatique des notations
  CIDR IPv4 (`ip/xx`) sur toute page web, infobulle au survol, calcul via
  menu contextuel sur une sélection, popup de calcul manuel.
