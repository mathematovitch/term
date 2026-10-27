# CLAUDE.md

Terminal web `/term` de mathematovitch.github.io : émulateur de shell en JS
pur (modules ES, aucune dépendance) pour naviguer dans les cours. Site de
projet GitHub Pages (dépôt `mathematovitch/term`, branche `main`, aucun
build), extrait avec son historique du site principal `~/work/00.site`
(2026-10-28). Reprise du side-project GitLab `termjs` (copie dans
`~/work/00.site/dev/termjs/`), dont la roadmap est entièrement implémentée.
Doc complète : `README.md`. Alias dans `aliases.sh` (syntaxe bash
restreinte, lue par `js/aliases.js`).

- **Dépend du site principal**, servi à côté (`SITE = ".."` de
  `js/config.js`, `../assets/…`) : ne marche pas seul.
- Seule source de données : **`/assets/fs.json`** du site principal, **pas
  dans ce dépôt** : écrit par `build_term_fs()` de
  `~/work/00.site/tools/build.py` dans `assets/fs.json` (dossier → objet,
  fichier → taille ; `HIDDEN_EXT` inclus, que le terminal ne montre qu'en
  root ; listes `hidden`, `prof`, `enc`). Format à changer des deux côtés à
  la fois. Pas d'API GitHub (quota de 60 req/h sans jeton).
- `sudo` : mot de passe du mode prof, vérifié par `unlock()` de
  `/assets/prof.js` du site (voir « Mode élève / mode prof » dans son
  CLAUDE.md) ; `open`/`cat` déchiffrent les chemins de la liste `enc` de
  `fs.json`. Préfixes `prof` (dont `_Archive`) traités comme `HIDDEN_EXT`.
- **Vraie sémantique Unix** (demande de l'utilisateur) : `sudo cmd` = **une**
  commande en root ; `chroot` (ou `su`, `sudo -s`) = shell root
  (`state.root`, invite `root@` rouge) jusqu'à `exit`. La session prof sert
  de cache du mot de passe (2 h) mais ne rend pas root à elle seule ;
  `isRoot()` = `elevated || (root && active())`. Tab complète après `sudo`.
  Toute nouvelle commande suit le comportement du vrai outil.
- Messages d'erreur au format GNU en locale française (`Aucun fichier ou
  dossier de ce type`…). Tout texte affiché passe par `textContent`.
- Aperçu local : lien symbolique `~/work/00.site/term → ~/code/04-web/term`
  (ignoré par git dans le site), `python3 -m http.server` à la racine de
  `00.site`, http://localhost:8000/term/.
- Tests : Chromium ne tourne pas dans le sandbox (sockets interdits) ; la
  logique se teste sous Deno (`--location http://127.0.0.1:PORT/term/`), avec
  le serveur HTTP lancé **dans la même commande** (réseau non partagé entre
  commandes), depuis la racine de `~/work/00.site` (où le lien symbolique
  `term` rend ce dépôt visible avec `/assets/`), et `--reload` (Deno met les
  imports HTTP en cache).
- Ne jamais committer sans demande explicite de l'utilisateur.
