# Git et GitHub sous Windows : guide pratique pour le mini-projet

> IUT de Cachan, Université Paris-Saclay. Module Réseaux, mini-projet « Radar à ultrasons piloté par réseau ».

Ce guide explique comment versionner votre projet (Keil µVision pour la carte, Qt pour l'application PC) avec Git, depuis votre poste Windows, et le synchroniser avec un dépôt GitHub partagé par votre binôme.

Il suppose que la commande `git` est déjà installée (Git for Windows). Les commandes se tapent dans **Git Bash** (installé avec Git) ou dans PowerShell ; elles sont identiques.

---

## 1. Principe en 30 secondes

```
   Votre PC (copie locale)                 GitHub (dépôt distant, « origin »)
 ┌─────────────────────────┐             ┌───────────────────────────┐
 │ dossier de travail      │             │                           │
 │   ↓ git add             │             │   branche main            │
 │ zone d'index (staging)  │             │                           │
 │   ↓ git commit          │  git push   │                           │
 │ historique local ───────┼───────────► │ historique partagé        │
 │                         │ ◄───────────┼                           │
 └─────────────────────────┘  git pull   └───────────────────────────┘
```

- **commit** : une « photo » de votre projet à un instant donné, avec un message. Il reste sur votre PC.
- **push** : envoie vos commits vers GitHub.
- **pull** : récupère sur votre PC les commits de votre binôme.

Votre enseignant consulte l'**historique GitHub** pour l'évaluation : des commits réguliers, répartis sur toutes les séances, sont mieux valorisés qu'un seul envoi la veille du rendu.

---

## 2. Configuration initiale (une seule fois par PC)

Dans Git Bash :

```bash
git config --global user.name  "Prénom Nom"
git config --global user.email "votre.adresse@universite-paris-saclay.fr"
git config --global core.autocrlf true      # gère les fins de ligne Windows / Linux
git config --global init.defaultBranch main
```

Vérification :

```bash
git config --global --list
```

> **Adresse e-mail et dépôt public :** l'adresse saisie dans `user.email` apparaît dans chaque commit, donc publiquement. Si vous ne souhaitez pas l'exposer, utilisez l'adresse « noreply » fournie par GitHub (*Settings → Emails → Keep my email addresses private*), de la forme `ID+identifiant@users.noreply.github.com`.

> **Poste partagé de l'IUT :** si plusieurs étudiants utilisent le même compte Windows, configurez plutôt l'identité **dans le dépôt seulement** (même commandes, sans `--global`, depuis le dossier du projet). Sinon vos commits seront signés par le mauvais nom.

### Authentification à GitHub

Git for Windows fournit *Git Credential Manager*. Au premier `git push` (ou `git clone` d'un dépôt privé), une fenêtre de navigateur s'ouvre : connectez-vous à GitHub et autorisez l'accès. Il n'y a rien d'autre à configurer.

> **Poste partagé :** à la fin de la séance, supprimez vos identifiants mémorisés : *Panneau de configuration → Gestionnaire d'identification → Informations d'identification Windows*, puis supprimez les lignes `git:https://github.com`.

---

## 3. Mise en place du dépôt (une fois par binôme)

### 3.1 Créer le dépôt (un seul étudiant du binôme)

1. Sur github.com : **New repository**, nom explicite (par exemple `radar-ultrasons`), visibilité **Public** (recommandé : les enseignants peuvent ainsi consulter l'historique sans invitation).
2. Cochez **Add a README** et choisissez un modèle `.gitignore` vide pour l'instant (nous le remplaçons ci-dessous).
3. **Donner les droits d'écriture à votre binôme** : *Settings → Collaborators → Add people*, en saisissant son identifiant GitHub. Il doit accepter l'invitation reçue par e-mail. Un dépôt public est lisible par tous, mais seuls les collaborateurs peuvent y pousser.
4. **Communiquer le lien du dépôt aux enseignants.** Si vous choisissez malgré tout un dépôt privé, ajoutez aussi les enseignants comme collaborateurs.

> **Dépôt public, donc visible de tout le monde :** n'y mettez jamais de mot de passe, d'identifiant de connexion ni de données personnelles (voir les sections 2 et 4).

### 3.2 Cloner le dépôt (chaque étudiant, sur son poste)

Choisissez un dossier **court et sans espace ni accent** (les chemins trop longs posent problème à Keil et à Qt), par exemple `C:\Projets` :

```bash
cd C:\Projets
git clone https://github.com/VOTRE_COMPTE/radar-ultrasons.git
cd radar-ultrasons
```

> Évitez de cloner dans un dossier synchronisé (OneDrive, Dropbox) : les synchronisations perturbent Git.

### 3.3 Déposer le template Keil, en commit séparé

1. Copiez le contenu du dossier template du lecteur réseau dans un sous-dossier `carte/` du dépôt.
2. Ajoutez le `.gitignore` de la section 4 à la racine.
3. **Commitez le template tel quel**, sans aucune modification :

```bash
git add .
git commit -m "Ajout du projet template Keil (état d'origine)"
git push
```

Ce commit sert de référence : on voit ensuite précisément ce que vous avez changé.

---

## 4. Fichiers à versionner, fichiers à ignorer

On versionne les **sources** et la configuration du projet ; on ignore tout ce que l'outil **régénère** (compilation, fichiers temporaires, réglages propres à une machine).

Créez un fichier `.gitignore` à la racine du dépôt :

```gitignore
# --- Keil µVision : fichiers générés ---
**/Objects/
**/Listings/
**/DebugConfig/
*.o
*.d
*.crf
*.axf
*.hex
*.bin
*.map
*.lst
*.htm
*.build_log.htm
*.dep
*.lnp
*.sct.bak
*.uvguix.*
*.bak
*.scvd

# --- Qt / Qt Creator ---
*.user
*.user.*
build-*/
build/
*.autosave
CMakeLists.txt.user*
*.pro.user*
Makefile*
*.o
*.obj
moc_*.cpp
ui_*.h
qrc_*.cpp
debug/
release/

# --- Windows / éditeurs ---
Thumbs.db
Desktop.ini
.vscode/
*.swp
~$*

# --- Secrets : ne jamais versionner ---
*.pem
*.key
secrets*
```

À **versionner** (liste indicative) : `*.c`, `*.h`, `*.cpp`, `*.ui`, `*.pro` ou `CMakeLists.txt`, `*.uvprojx`, le dossier `RTE/`, les fichiers de ressources de l'interface de la carte, `docs/`.

À propos du fichier **`*.uvoptx`** (options de débogage de Keil) : il change souvent d'un poste à l'autre et provoque des conflits inutiles. Si vous constatez que vous le modifiez sans le vouloir à chaque ouverture du projet, ajoutez-le au `.gitignore`.

> **Ne versionnez jamais de mot de passe** (routeur, compte, etc.). Dans le compte rendu et le plan d'adressage, indiquez « identifiants fournis par l'enseignant » plutôt que leur valeur réelle.

Si un fichier a été commité par erreur alors qu'il est ignoré, retirez-le de l'index (sans le supprimer de votre disque) :

```bash
git rm -r --cached carte/Objects
git commit -m "Retrait des fichiers générés"
```

---

## 5. Travailler au quotidien : le cycle de base

À chaque séance, toujours dans cet ordre :

```bash
git pull                     # 1. récupérer le travail du binôme
# ... vous travaillez dans Keil / Qt ...
git status                   # 2. voir ce qui a changé
git add carte/main.c         # 3. choisir les fichiers à inclure
git commit -m "Servo : fonction servo_set_angle() avec bornes calibrées"
git push                     # 4. envoyer sur GitHub
```

Commandes utiles :

| Commande | Rôle |
|---|---|
| `git status` | Fichiers modifiés, ajoutés, non suivis |
| `git diff` | Détail des modifications non encore ajoutées |
| `git log --oneline --graph --all` | Historique résumé, avec les branches |
| `git add -p` | Choisir morceau par morceau ce qu'on commite (facultatif, voir ci-dessous) |
| `git restore NOM_DU_FICHIER` | **Annuler** les modifications d'un fichier (irréversible) |
| `git restore --staged NOM_DU_FICHIER` | Retirer un fichier de l'index, sans perdre ses modifications |

### Faut-il toujours utiliser `git add -p` ?

Non, c'est un outil **facultatif**. Au quotidien, `git add` suivi du nom du fichier suffit.

- **`git add NOM_DU_FICHIER`** : ajoute **tout** le fichier. C'est le cas courant, quand toutes vos modifications dans ce fichier font partie du même commit.
- **`git add -p`** : Git vous présente chaque modification, et pour chacune vous répondez `y` (oui), `n` (non), `s` (découper) ou `q` (quitter). C'est utile quand un même fichier contient **deux modifications sans rapport** (par exemple le servo et un essai temporaire) et que vous voulez les commiter séparément, ou quand vous voulez **relire** vos changements avant de les valider.

Dans tous les cas, lancez `git status` avant `git commit` pour vérifier ce qui sera commité. Évitez `git add .` sans y avoir regardé : il ajoute tout, y compris des fichiers générés ou des essais que vous ne vouliez pas publier.

### Bien écrire un message de commit

Un message utile dit **ce qui a changé et pourquoi**, en une ligne :

- ✅ `Capteur : distance_cm() avec délai maximal ECHO de 30 ms`
- ✅ `Client C : nouvel essai après timeout de 1 s`
- ❌ `modif`, `ça marche`, `test2`, `dernier fichier`

Faites des commits **petits et fréquents** (une fonctionnalité, un correctif) : au moins un par étape de travail, et au minimum à la fin de chaque séance.

---

## 6. Utiliser Git avec Keil µVision et Qt Creator

Keil µVision n'a pas d'intégration Git native : on utilise **Git Bash à côté de l'IDE**. Qt Creator, lui, propose un menu Git (*Outils → Git*), mais les commandes restent les mêmes.

### Les règles à respecter avec Keil

1. **Fermez le projet Keil avant un `git pull`** (ou au moins avant de basculer de branche). Keil garde le projet en mémoire : si Git remplace `*.uvprojx` en arrière-plan, vous risquez d'écraser les modifications reçues au prochain enregistrement.
2. **Rouvrez le projet après le `pull`**, puis recompilez entièrement (*Project → Rebuild all target files*).
3. **Compilez avant de commiter** : ne poussez pas un projet qui ne compile pas, votre binôme le récupérera.
4. **Ajoutez un nouveau fichier source dans Keil ET dans Git** : un fichier créé dans Keil n'est pas versionné tant que vous n'avez pas fait `git add`. `git status` vous le montre dans « Untracked files ».
5. **Chemins relatifs** : gardez le projet dans le dépôt, avec des chemins relatifs. Un chemin absolu (`C:\Users\...`) fonctionnera sur votre PC et pas sur celui de votre binôme.

### Fichiers de l'interface de la carte (AppWizard)

Les fichiers générés par AppWizard (par exemple `ID_SCREEN_00_Slots.c`) sont éditables à la main, uniquement **aux emplacements signalés** (« code à exécuter à mettre ici »). Si vous régénérez l'écran avec AppWizard, relisez `git diff` avant de commiter pour vérifier que votre code n'a pas été écrasé.

### Les fichiers binaires

Les images, polices et ressources (`*.png`, `*.dta`, `*.xbf`, `*.pdf`) ne se fusionnent pas : si vous et votre binôme les modifiez en même temps, il faudra choisir une version. Évitez donc de les modifier à deux simultanément.

Pour que Git ne touche pas à leurs fins de ligne, créez à la racine un fichier `.gitattributes` :

```gitattributes
* text=auto
*.png  binary
*.dta  binary
*.xbf  binary
*.pdf  binary
*.emf  binary
*.axf  binary
```

---

## 7. Travailler à deux sans se marcher dessus

Chaque étudiant **travaille de préférence sur des fichiers différents** (A : réseau et application PC ; B : carte), ce qui évite presque tous les conflits. Pour aller plus loin, utilisez des **branches**.

### 7.1 Une branche par fonctionnalité

```bash
git switch -c feature/servo        # créer et se placer sur la branche
# ... travail, commits ...
git push -u origin feature/servo   # première publication de la branche
```

Quand la fonctionnalité marche, fusionnez-la dans `main` :

```bash
git switch main
git pull
git merge feature/servo
git push
```

(Vous pouvez aussi passer par une *Pull Request* sur GitHub, ce qui permet à votre binôme de relire votre code avant fusion.)

### 7.2 Résoudre un conflit

Si un `git pull` ou un `git merge` annonce `CONFLICT`, c'est que vous avez modifié les mêmes lignes qu'un autre. Git marque les zones concernées dans le fichier :

```
<<<<<<< HEAD
  servo_set_angle(90);
=======
  servo_set_angle(45);
>>>>>>> origin/main
```

1. Ouvrez le fichier, **gardez la bonne version** (ou combinez les deux), et supprimez les lignes `<<<<<<<`, `=======`, `>>>>>>>`.
2. Vérifiez que le projet compile.
3. Terminez :

```bash
git add carte/main.c
git commit                       # le message de fusion est proposé automatiquement
git push
```

Si vous êtes perdus : `git merge --abort` annule la fusion et vous ramène à l'état précédent. Appelez votre binôme, ou l'enseignant, avant d'essayer d'autres commandes.

---

## 8. Problèmes fréquents

| Symptôme | Cause probable | Solution |
|---|---|---|
| `rejected ... fetch first` au `push` | Votre binôme a poussé avant vous | `git pull`, résolvez les éventuels conflits, puis `git push` |
| `Authentication failed` / `403` | Pas d'accès au dépôt, ou ancien identifiant mémorisé | Vérifiez l'invitation acceptée, supprimez l'entrée dans le Gestionnaire d'identification, réessayez |
| `fatal: not a git repository` | Vous n'êtes pas dans le dossier du dépôt | `cd` jusqu'au dossier contenant le sous-dossier caché `.git` |
| `Filename too long` | Chemin trop long sous Windows | `git config --global core.longpaths true` et clonez dans un dossier court |
| `warning: LF will be replaced by CRLF` | Conversion des fins de ligne | Normal avec `core.autocrlf true` : sans gravité |
| Un fichier ignoré est quand même suivi | Il avait été commité avant d'entrer dans `.gitignore` | `git rm --cached NOM_DU_FICHIER` puis commit |
| Keil indique « fichier modifié par un programme externe » | Un `git pull` a changé un fichier ouvert | Choisissez *Yes* (recharger), puis recompilez |
| Un commit contient un mot de passe | Secret versionné par erreur, **visible de tous si le dépôt est public** | **Changez immédiatement le mot de passe**, prévenez l'enseignant, puis demandez de l'aide pour nettoyer l'historique : supprimer le fichier dans un commit ultérieur ne suffit pas |

---

## 9. Récapitulatif à garder sous la main

```bash
# 1. Début de séance : récupérer le travail du binôme
git pull

# 2. Modifier les fichiers normalement, dans Keil ou Qt
#    (aucune commande Git ici : on édite, on compile, on teste)

# 3. Dès qu'une étape fonctionne (régulièrement) : enregistrer
git status
git add FICHIERS
git commit -m "Message clair"
#    puis retour à l'étape 2 pour la suite du travail

# 4. Fin de séance (au minimum) : envoyer
git pull
git push
```

**Les cinq réflexes :**

1. `git pull` avant de commencer.
2. Compiler avant de commiter.
3. Des commits petits, avec des messages clairs.
4. `git push` à la fin de chaque séance.
5. Jamais de mot de passe ni de fichiers générés dans le dépôt.

---

## Pour aller plus loin

- Documentation officielle (en français) : [Pro Git (fr)](https://git-scm.com/book/fr/v2)
- Aide-mémoire GitHub : [Aide-mémoire Git (PDF)](https://education.github.com/git-cheat-sheet-education.pdf)
- Interfaces graphiques, si vous préférez à la ligne de commande : *GitHub Desktop*, ou l'onglet « Source Control » de *Visual Studio Code*. Les principes décrits ici restent identiques.
