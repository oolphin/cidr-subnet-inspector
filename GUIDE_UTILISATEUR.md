# Guide utilisateur — CIDR Subnet Inspector

Version du guide : 1.2.0

## 1. Installation

### Utilisation ponctuelle / test (aucune signature requise)

1. Ouvrir `about:debugging#/runtime/this-firefox` dans Firefox.
2. Cliquer sur *Charger un module complémentaire temporaire*.
3. Sélectionner le fichier `manifest.json` à la racine du dossier de
   l'extension.
4. L'icône apparaît dans la barre d'outils. Elle reste active jusqu'au
   redémarrage de Firefox.

### Installation permanente

Nécessite un fichier `.xpi` signé par Mozilla (voir la section *Packaging
et signature* du `README.md`). Une fois signé :

1. Ouvrir le fichier `.xpi` avec Firefox (double-clic, ou glisser-déposer
   dans une fenêtre du navigateur).
2. Confirmer l'installation dans la boîte de dialogue qui s'affiche.

## 2. Utilisation sur une page web

### 2.1 Survol automatique

Toute expression réseau reconnue sur une page (CIDR, paire IP/masque,
plage d'adresses) est surlignée en jaune. Passer la souris dessus ouvre un
panneau avec le calcul complet.

Formats reconnus :

| Format | Exemple |
| --- | --- |
| CIDR IPv4 | `192.168.1.0/21` |
| CIDR IPv6 | `2001:db8::/32` |
| IP + masque décimal (espace) | `192.168.1.0 255.255.255.0` |
| IP + masque décimal (barre oblique) | `192.168.7.254/255.255.248.0` |
| Plage d'adresses | `10.0.0.5-10.0.0.20` |

### 2.2 Sélection manuelle

Sélectionner n'importe quel texte correspondant à l'un des formats
ci-dessus (même s'il n'est pas surligné) ouvre le même panneau au
relâchement de la sélection.

### 2.3 Menu contextuel

Sélectionner un texte puis clic droit → **« Calculer le sous-réseau
(CIDR) »**. Utile si le surlignage automatique est désactivé sur le site,
ou si le texte contient des caractères qui empêchent la détection directe.

### 2.4 Lire le panneau de résultat

Le panneau affiche, selon le type d'expression détectée :

- le **titre** (l'expression normalisée, ex. `192.168.1.0/21`) ;
- un **badge de portée** : Privée (RFC 1918), CGNAT (RFC 6598), Publique,
  Lien-local, Loopback, Documentation, Multicast, ULA (IPv6), etc. ;
- pour un CIDR : masque, wildcard, adresse réseau, broadcast, plage
  d'adresses utilisables, nombre d'hôtes utilisables, classe historique ;
- pour une plage : nombre d'adresses, et si elle correspond exactement à
  un bloc CIDR (avec un bouton pour l'afficher directement).

### 2.5 Diviser et regrouper un sous-réseau

Dans le panneau, un champ numérique pré-rempli avec le préfixe actuel
permet de :

- **Diviser** : entrer un préfixe plus grand (ex. passer de `/24` à
  `/26`) puis cliquer sur *Diviser* → la liste des sous-réseaux résultants
  s'affiche, chacun cliquable pour ouvrir son propre détail (navigation
  récursive). Au-delà de 64 sous-réseaux, seuls les 64 premiers sont
  affichés, avec le total indiqué.
- **Regrouper** : entrer un préfixe plus petit (ex. passer de `/24` à
  `/20`) puis cliquer sur *Regrouper* → le bloc parent correspondant
  s'affiche directement, quel que soit l'écart de préfixe.

Fonctionne en IPv4 comme en IPv6.

### 2.6 Copier le résultat

Le bouton *Copier* place le résultat formaté (texte brut, prêt à coller
dans un ticket ou une documentation) dans le presse-papiers.

### 2.7 Fermer le panneau

Cliquer sur *✕*, cliquer en dehors du panneau, ou appuyer sur **Échap**.

## 3. Popup (icône de la barre d'outils)

### 3.1 Calcul manuel

Saisir n'importe laquelle des expressions du tableau de la section 2.1
dans le champ de recherche, puis cliquer sur *Calculer* (ou appuyer sur
**Entrée**). Le résultat s'affiche avec les mêmes actions (diviser,
regrouper, copier) que le panneau de survol.

### 3.2 Historique

Les 20 dernières recherches effectuées depuis le popup ou le menu
contextuel apparaissent sous forme de liste cliquable : cliquer sur une
entrée la recalcule immédiatement. Le bouton *Effacer* vide l'historique.

### 3.3 Activer/désactiver le surlignage sur le site courant

La ligne *Site actuel* affiche le domaine de l'onglet actif et un bouton :

- **Désactiver ici** : le surlignage automatique est coupé sur ce site
  uniquement (les autres sites ne sont pas affectés).
- **Réactiver ici** : réactive le surlignage sur ce site.

Ce raccourci ajuste automatiquement la liste blanche/noire décrite en
section 4, sans avoir besoin d'ouvrir la page d'options pour un usage
ponctuel.

### 3.4 Interrupteur global

La case à cocher en bas du popup active/désactive le surlignage
automatique pour l'ensemble des sites (indépendamment des listes
blanche/noire configurées).

## 4. Page d'options (listes de sites)

Accessible via *Options avancées (listes de sites)…* dans le popup, ou
depuis le gestionnaire de modules complémentaires de Firefox
(`about:addons` → CIDR Subnet Inspector → *Préférences*).

Trois modes :

| Mode | Comportement |
| --- | --- |
| Activé sur tous les sites | Réglage par défaut. |
| Liste noire | Surlignage actif partout, sauf sur les domaines listés. |
| Liste blanche | Surlignage désactivé partout, sauf sur les domaines listés. |

La liste se saisit avec un domaine par ligne (ex. `intranet.example.com`).
Un domaine listé couvre aussi ses sous-domaines (`intranet.example.com`
couvre `admin.intranet.example.com`). Cliquer sur *Enregistrer* pour
appliquer.

Cas d'usage typique en environnement professionnel : mode *Liste
blanche* limité aux outils internes (supervision, documentation réseau,
IPAM) pour éviter tout surlignage sur des sites externes ou sensibles
(banque, portails RH…).

## 5. Limites connues

- Pas de détection des adresses IPv4 mappées en IPv6
  (`::ffff:192.168.1.1`).
- La détection des paires « IP + masque » et des plages suppose une
  syntaxe raisonnablement propre (peu de texte parasite entre les deux
  adresses).
- Aucun surlignage dans les zones éditables (`contenteditable`,
  `<textarea>`) ni dans `<script>`/`<style>`.
- La scission est plafonnée à 64 sous-réseaux affichés simultanément.

## 6. Support

Dépôt du projet et signalement de problèmes :
[github.com/LAB-INF0/.githLAB](https://github.com/LAB-INF0/.githLAB).
