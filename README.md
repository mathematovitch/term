# Terminal web de Mathématovitch (`/term`)

Site de projet GitHub Pages (dépôt `mathematovitch/term`), servi à
https://mathematovitch.github.io/term/, sous le site de cours
`mathematovitch/mathematovitch.github.io` dont il a été extrait avec son
historique (2026-10-28).

Mini terminal interactif pour naviguer dans les cours et ouvrir les fichiers
(PDF surtout) depuis le navigateur, comme dans un shell Unix. JavaScript pur
(modules ES, aucune dépendance). Reprise du projet GitLab Pages `termjs`
(ancien dépôt `colinouterovitch/my-page` sur forge.apps.education.fr) :
l'API GitLab est remplacée par **`/assets/fs.json`**, l'arborescence des
sections générée à chaque build du site principal. Aucune requête d'API, pas de quota,
fonctionne aussi en local.

```
cd /Sixieme/S04-Angles      # Tab complète les noms
ls -l                       # cliquer sur un nom l'ouvre (dossier : cd, fichier : open)
open Cours-6C04-Les-Angles.pdf
find / -iname '*angle*'
ls | grep DS
help ls                     # aide détaillée de chaque commande
```

L'URL suit le dossier courant : `/term/#/Sixieme/S04-Angles` ouvre le
terminal directement dans la séquence (lien à donner aux élèves).

### Commandes

| Commande | Rôle |
|---|---|
| `help [cmd]`, `man` | liste des commandes, aide détaillée |
| `ls [-l] [-1] [-R] [-d]`, `ll` | contenu d'un dossier, en colonnes, couleurs par type |
| `cd [chemin]` | `..`, `/`, `~`, `-`, chemins relatifs/absolus ; liste le dossier ensuite (alias) |
| `pwd` | dossier courant |
| `tree [-d] [-L n]` | arborescence |
| `open fichier\|dossier`, `pdf` | PDF/HTML : nouvel onglet ; image : aperçu ; texte : affiché ; dossier : sa page du site |
| `cat [-n]`, `head`, `tail`, `wc`, `sort` | lecture et filtres de texte |
| `find [chemin] -name/-iname/-type/-maxdepth` | recherche par nom |
| `grep [-ivnclrw] motif [fichiers]` | recherche dans le contenu (ou filtre d'un pipe) |
| `history`, `!!`, `!n`, `!préfixe` | historique |
| `clock [-s]` | l'heure en grand pour un contrôle : plein écran, police figlet « doh » (comme `termdown -z -f doh`), « : » qui clignotent, écran sans veille ; Échap ou Ctrl+C pour quitter |
| `echo`, `whoami`, `date`, `clear` (`c`), `theme` | divers (`theme vert\|ambre\|gris\|clair`) |
| `alias`, `source` (`s`, `.`), `builtin` | alias de `aliases.sh` (ci-dessous) |
| `chroot`, `sudo cmd`, `sudo -s`, `su`, `exit` | mode professeur (ci-dessous) |

Shell : guillemets `'…'` `"…"`, `\`, variables (`$PWD`, `$?`, `$HOME`,
`$USER`, `$OLDPWD`), jokers `* ? […]`, enchaînements `|`, `;`, `&&`, `||`.
Clavier : Tab, ↑/↓, Ctrl+C, Ctrl+L, Ctrl+U, Ctrl+K, Ctrl+A, Ctrl+E, Ctrl+W.
Messages d'erreur au format GNU en français, codes de retour dans `$?`.

Le système de fichiers est en lecture seule : `rm`, `mkdir`, `touch`, `cp`,
`mv`, `nano`, `vim`, `>`… existent mais échouent avec l'erreur POSIX
`Système de fichiers accessible en lecture seule`.

### Alias (`aliases.sh`)

Joue le rôle du `~/.bash_aliases` (copie de ceux de l'auteur), lu au
démarrage. Syntaxe bash restreinte : `alias nom="commande"` et fonctions
`nom() { … "$@" … }`. Le corps est une vraie ligne du shell (`&&`, `|`…),
sans récursion : `builtin cd` court-circuite la fonction `cd`.

- `cd` ⇒ `builtin cd "$@" && ll` (liste le dossier après chaque `cd`) ;
- `s` ⇒ `source ~/.bashrc` : relit `aliases.sh`, efface l'écran et
  réaffiche l'accueil, sans changer de dossier ;
- `c` ⇒ `clear` ;
- `pdf` ⇒ `open` (`pdf Cours-6C04-Les-Angles.pdf`).

### Mode sudo

Les fichiers masqués sur le site (`.tex`, `.sty`, `.py`, `.sh`, `.md`… :
`HIDDEN_EXT` de `tools/build.py` du site principal), les documents du mode prof (`DS-`,
`Seq-`, `Prep-`, `…-Corr`) et les archives sont invisibles pour l'élève
(`Permission non accordée`). Comme dans un vrai shell :
`sudo commande` lance **une** commande en root (`sudo open DS-…`), et
`chroot` (ou `su`, `sudo -s`) ouvre un **shell root** : invite
`root@mathematovitch#` en rouge, tout visible et complété par Tab, documents
prof déchiffrés à la volée (`open`, `cat`, `grep`), jusqu'à `exit`. Mot de
passe : celui du mode prof (3 essais), pas redemandé tant que le mode prof
du site est actif (2 h, il sert de cache) ; `sudo -k` ou « Quitter » le
coupent. Tab complète aussi après `sudo`, avant le mot de passe. Ex. :
`chroot` puis `grep -ri "angle nul" .` cherche dans les sources LaTeX. Mot de passe :
`python3 tools/profkey.py passwd` dans le site principal (voir son README,
« Mode élève / mode prof »).

### Fichiers

```
term/                 (ce dépôt)
├── README.md, CLAUDE.md
├── index.html        page, touches tactiles (Tab, ↑, ↓, cd .., ls, ^C)
├── term.css          thèmes (variables CSS), couleurs façon dircolors, mobile
├── aliases.sh        alias et fonctions du shell (cd → cd && ll, s, c, pdf)
└── js/
    ├── main.js       affichage, prompt, clavier, historique, démarrage
    ├── shell.js      parseur : guillemets, variables, jokers, | ; && ||, alias
    ├── fs.js         système de fichiers virtuel, chemins, erreurs POSIX
    ├── aliases.js    lecture de aliases.sh
    ├── complete.js   complétion Tab
    ├── state.js      état + sauvegarde localStorage
    ├── config.js     hôte, utilisateur
    └── commands/     une commande = { help, usage, more, run(args, io) }
```

## Dépendances au site principal

Le terminal n'est pas autonome : servi à `/term/`, il charge par chemins
relatifs (`SITE = ".."` dans `js/config.js`, `../assets/…` ailleurs) des
fichiers du site `mathematovitch.github.io` :

- **`/assets/fs.json`** : arborescence des sections, générée par
  `build_term_fs()` de `tools/build.py` (dossier → objet, fichier → taille ;
  listes `hidden`, `prof`, `enc`). Rien à générer dans ce dépôt ;
- **`/assets/prof.js`** (module ES : `unlock()`, `active()`, `quit()`,
  `openEnc()`, `decryptText()`) et **`/assets/prof.json`** (clé chiffrée par
  le mot de passe prof) : mode prof, `sudo`, déchiffrement des `X.enc` ;
- **`/assets/favicon.ico`**, **`/assets/favicon.svg`** ;
- les fichiers de cours eux-mêmes (`open`, `cat`, `grep`) et les pages
  `index.html` des dossiers, via `SITE` ; liens « Retour au site » vers
  `../index.html`.

## Aperçu local

Les modules ES et `fetch` ne marchent pas en `file://`, et le terminal a
besoin du site principal à côté de lui. Lien symbolique dans le site (ignoré
par git là-bas) :

```
ln -s ~/code/04-web/term ~/work/00.site/term
cd ~/work/00.site && python3 -m http.server   # puis http://localhost:8000/term/
```

## Déploiement

Push sur `main` de `mathematovitch/term` : GitHub Pages sert le dépôt tel
quel (aucun build). Le contenu (`/assets/fs.json`) se met à jour avec le
site principal, sans toucher à ce dépôt.

## 🛣️ Roadmap

- [ ] **1. Tutoriel intégré** (`tuto`), plus tard : quelques défis du type
  « trouve le DS sur les angles » pour apprendre les bases du terminal.

<details>
<summary>Déjà fait</summary>

- [x] `ls`, `cd`, `open`, historique clavier, complétion Tab
- [x] `cd ..`, `/`, `~`, `-`, `../../x` ; messages d'erreur POSIX et codes de retour
- [x] `open` intelligent (PDF, images, HTML, texte, dossiers), `tree`, `cat`
- [x] CSS : scroll, sélection, focus, thèmes
- [x] Mode sudo (même mot de passe que le mode prof, vérifié par déchiffrement de la clé)
- [x] Tri des dossiers (dossiers d'abord, `_Archive` en dernier, tri naturel)
- [x] Recherche (`find`, `grep`), pipes et filtres de texte
- [x] Sauvegarde de l'état (localStorage : dossier, historique, thème)
- [x] Support mobile (touches tactiles, clavier virtuel, noms cliquables)
- [x] `clock` plein écran pour les contrôles
- [x] Alias façon `~/.bash_aliases` (`cd && ll`, `s`, `c`, `pdf`)

</details>
