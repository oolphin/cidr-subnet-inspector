# CIDR Subnet Inspector

Extension Firefox (Manifest V3) qui détecte les expressions réseau IPv4/IPv6
présentes dans une page web et affiche leur calcul complet — réseau,
broadcast, masque, wildcard, plage d'adresses utilisables, portée
(privée/publique/CGNAT/lien-local…), avec scission et agrégation de
sous-réseaux en un clic.

## Fonctionnalités

- **Survol automatique** : chaque expression réseau détectée sur la page est
  surlignée ; passer la souris dessus ouvre un panneau avec le calcul complet.
- **Sélection manuelle** : sélectionner n'importe quel texte reconnu (même
  hors surlignage) ouvre le même panneau.
- **Menu contextuel** : clic droit sur une sélection → *« Calculer le
  sous-réseau (CIDR) »*.
- **IPv4 et IPv6** : `192.168.1.0/21`, `2001:db8::/32`, etc.
- **Paires IP + masque** : `192.168.1.0 255.255.255.0` (espace) et
  `192.168.7.254/255.255.248.0` (masque décimal après le `/`, au lieu d'un
  préfixe numérique) détectés et calculés comme un `/24`/`/21` classique.
- **Plages d'adresses** : `10.0.0.5-10.0.0.20` → nombre d'adresses, et
  détection automatique si la plage correspond exactement à un bloc CIDR.
- **Scission / agrégation interactives** : dans le panneau de résultat, un
  champ de préfixe + boutons *Diviser* / *Regrouper* permettent de passer de
  `/24` à `/26` (liste des 4 sous-réseaux, cliquables) ou de `/24` à `/20`
  (agrégat parent), en IPv4 comme en IPv6.
- **Portée RFC** : badge indiquant si l'adresse est privée (RFC 1918),
  CGNAT (RFC 6598), lien-local, loopback, documentation, ULA IPv6, etc.
- **Copier** : bouton pour copier le résultat formaté dans le presse-papiers.
- **Historique** : les 20 dernières recherches (popup + menu contextuel)
  sont conservées et ré-exécutables en un clic, avec bouton pour l'effacer.
- **Liste blanche / liste noire de sites** : page d'options pour activer le
  surlignage automatique partout sauf sur certains sites, ou seulement sur
  une liste choisie (intranet, outils internes…) ; bascule rapide « Désactiver
  ici / Réactiver ici » disponible directement dans le popup pour le site
  courant.
- **Popup de calcul manuel** : icône de la barre d'outils → champ de saisie
  libre acceptant tous les formats ci-dessus.

## Architecture

```
manifest.json
lib/cidr.js               → calculs purs IPv4/IPv6 (aucune dépendance DOM) :
                             CIDR, paires masque, plages, scission/agrégation,
                             classification de portée. Partagé par tous les
                             contextes de l'extension.
lib/render.js              → rendu HTML interactif d'un résultat (panneau
                             diviser/regrouper/copier), partagé par le
                             content script et le popup.
background/background.js  → menu contextuel + journal d'historique
content/content.js        → scan du DOM (TreeWalker), surlignage, panneau
                             interactif, filtre par domaine, MutationObserver
content/content.css       → style du surlignage et du panneau
popup/popup.{html,js,css} → calculateur manuel, historique, bascule rapide
                             par site
options/options.{html,js,css} → gestion de la liste blanche/noire de sites
icons/                    → logo (grille d'adresses + loupe), SVG source
                             et PNG 16/32/48/96/128
```

`lib/cidr.js` et `lib/render.js` exposent des objets globaux (`CidrCalc`,
`CidrUI`) — pas de modules ES, pour rester chargeables tels quels à la fois
en `background.scripts`, `content_scripts.js` et dans les pages HTML de
l'extension (popup, options).

### Limites connues

- Pas de détection des adresses IPv4 mappées en IPv6 (`::ffff:192.168.1.1`).
- La détection des paires « IP + masque » et des plages suppose une syntaxe
  raisonnablement propre (espacement limité, pas de texte parasite entre les
  deux adresses).
- Le surlignage ne s'applique pas dans les zones `contenteditable` ni les
  `<textarea>`/`<script>`/`<style>`.
- La scission est plafonnée à 64 sous-réseaux affichés (avertissement affiché
  au-delà) pour éviter de bloquer l'interface sur un `/8 → /30`.

## Installation en développement

1. Ouvrir `about:debugging#/runtime/this-firefox` dans Firefox.
2. *Charger un module complémentaire temporaire* → sélectionner
   `manifest.json` dans ce dossier.
3. L'extension reste active jusqu'au redémarrage de Firefox (rechargement
   à faire manuellement après chaque modification, ou via `web-ext run`).

Avec [`web-ext`](https://github.com/mozilla/web-ext) installé :

```bash
npm install -g web-ext
web-ext run --source-dir=./cidr-subnet-inspector
```

## Packaging et signature

```bash
web-ext build --source-dir=./cidr-subnet-inspector --artifacts-dir=./dist
```

produit un `.zip` installable en mode développement (`about:debugging`),
mais **Firefox release refuse d'installer un paquet non signé** de façon
permanente. Pour une installation définitive (glisser-déposer, déploiement
en équipe), il faut un `.xpi` signé par Mozilla :

```bash
web-ext sign \
  --source-dir=./cidr-subnet-inspector \
  --api-key=$AMO_JWT_ISSUER \
  --api-secret=$AMO_JWT_SECRET
```

`AMO_JWT_ISSUER`/`AMO_JWT_SECRET` s'obtiennent depuis un compte développeur
sur [addons.mozilla.org](https://addons.mozilla.org/developers/addon/api/key/) —
gratuit, y compris pour une extension non listée (usage privé/interne,
non publiée sur le store public). Cette étape nécessite des identifiants
personnels et ne peut pas être automatisée depuis ce dépôt.

Alternative pour du test interne sans signature : Firefox Developer Edition
ou Nightly avec `xpinstall.signatures.required = false` dans `about:config`.

# Autrement, la version signée par Firefox est disponible dans l'Add-ons Manager de FIREFOX
# [CIDR Subnet Inspector](https://addons.mozilla.org/en-US/firefox/addon/cidr-subnet-inspector/)

---

## 👨‍💻 Auteur

**Guy SOW**
- GitHub : [@oolphin](https://github.com/oolphin)
- Email : info@thetekitpro.fr

---

<div align="center">

**⭐ If you find this project useful, feel free to give it a star on GitHub ! ⭐**

Made with ❤️ by <a href="https://github.com/LAB-INF0/.githLAB" target="_blank">LAB-INFO</a>

</div>
