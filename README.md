# Gestion des présences

Application desktop Tkinter de gestion des employés et de leurs pointages. Le
noyau métier et la base SQLite restent indépendants de l'interface graphique.

## Installation Python (Windows PowerShell)

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
```

Activer l'environnement dans un terminal :

```powershell
.\.venv\Scripts\Activate.ps1
```

Tkinter est fourni avec Python. `requirements.txt` contient OpenCV pour la
caméra QR et qrcode/Pillow pour les cartes QR ; `requirements-dev.txt` ajoute
pytest.

## Démarrage et configuration

```powershell
.\.venv\Scripts\python.exe -m gestion_presence
```

Pour un test non graphique de l'infrastructure (connexion SQLite + création des
tables), utiliser :

```powershell
.\.venv\Scripts\python.exe -m gestion_presence --check-db
```

Au premier lancement, une interface de connexion s'ouvre. Sur une base vide,
elle permet de créer le premier administrateur. L'application crée le dossier
et le fichier SQLite sous `~/.gestion_presence/gestion_presence.db`, crée les
tables, puis vérifie la connexion. Variables facultatives :

- `GESTION_PRESENCE_DATABASE_PATH` : chemin du fichier SQLite ;
- `GESTION_PRESENCE_DATABASE_URL` : URL SQLite alternative ;
- `GESTION_PRESENCE_TIMEZONE` : fuseau IANA, par défaut `Africa/Kinshasa` ;
- `GESTION_PRESENCE_LOG_LEVEL` : niveau standard Python, par défaut `INFO`.

## Démo locale VS Code

Le dossier `.demo/` est une base distincte, ignorée par Git, afin de ne pas
ajouter de données de test à la base utilisateur. Dans VS Code :

1. Ouvrir **Terminal > Exécuter la tâche... > Installer les dépendances**.
   Pour créer le venv auparavant, exécuter la tâche **Créer l'environnement Python**.
2. Appuyer sur `F5`, puis choisir **Gestion des présences — Démo**. La tâche
   avant lancement peuple la base locale et ouvre la fenêtre de connexion.
3. Choisir un compte de démonstration ci-dessous. Pour tester au clavier,
   ouvrir **Employés > Carte QR** et scanner l'image PNG avec l'écran de pointage.
   Pour tester la caméra, brancher une webcam et appuyer sur « Démarrer la caméra ».
4. `Ctrl+Shift+P` > **Tasks: Run Task** permet de lancer **Exécuter tous les tests**
   ou **Vérifier SQLite**.

Comptes **réservés à la base locale de démonstration** (ne pas réutiliser en
production) :

| Compte | Rôle | Mot de passe |
|---|---|---|
| `admin.demo` | Administration complète | `DemoPresence2026!` |
| `rh.demo` | RH | `DemoPresence2026!` |
| `superviseur.demo` | Consultation | `DemoPresence2026!` |

L'amorçage est relançable et complète les données manquantes sans dupliquer
employés, horaires et pointages. Le rapport comporte des exemples du dernier
jour ouvrable et la page d'accueil reçoit un pointage ouvert à la date courante.
Les cartes QR générées sont dans `.demo/qr_codes/`. La lecture d'empreinte reste
un écran indicatif seulement ; aucun lecteur ou SDK fictif n'est simulé.

Les tâches, la configuration Python, les recommandations d'extensions et les
configurations F5 se trouvent dans [`.vscode/`](./.vscode/).

Créer le premier administrateur, de manière interactive et sans mot de passe
par défaut :

```powershell
.\.venv\Scripts\python.exe -m gestion_presence --create-admin
```

Cette commande ne fonctionne que tant qu'aucun utilisateur n'existe. Les
services ne valident ni ne ferment la transaction du demandeur : utiliser une
session SQLAlchemy et une transaction, par exemple `with session.begin():`.

## Services métier

- `services.employees.EmployeeService` : création avec identifiant attribué,
  recherche, consultation, modification et désactivation ;
- `services.attendance.AttendanceService` : pointages QR, contrôle des appareils,
  entrée/sortie, anomalies courantes et rapport journalier ;
- `services.authentication.AuthenticationService` et `services.rbac.RbacService` :
  bootstrap sécurisé, hachage PBKDF2, authentification et vérification des droits ;
- services de départements, horaires, appareils, paramètres et audit disponibles
  dans `gestion_presence/services`.

## Interfaces

Après authentification, le menu affiche uniquement les sections autorisées par
les rôles associés au compte. Les écrans inclus :

- tableau de bord et pointage QR par caméra ou saisie d'un identifiant ;
- employés (création, modification, activation) et export individuel du QR en PNG ;
- services, horaires, utilisateurs, rôles/permissions et pointeuses ;
- rapports filtrables par période, employé et service, exportables en CSV ;
- journal d'audit et configuration de l'organisation.

Le pointage nécessite une pointeuse QR active, à créer dans « Pointeuses ».
La détection par empreinte est visible comme fonctionnalité préparée mais reste
désactivée tant que le lecteur et son SDK ne sont pas définis. L'accès caméra
n'est ouvert qu'après action sur « Démarrer la caméra ».

La page « Configuration » permet de choisir la langue de l'interface entre
français et anglais. Le changement s'applique immédiatement; le bouton
« Enregistrer » mémorise également cette préférence pour les prochains
démarrages.

Au démarrage, une fenêtre de chargement affiche une jauge pendant la vérification
des mises à jour. Cette vérification n'empêche pas l'application de démarrer en
cas d'indisponibilité de GitHub. Si une version plus récente est trouvée, elle
est proposée à l'administrateur dans « Configuration ».

## Mises à jour GitHub (dépôt public)

Seul un compte ayant le rôle système **ADMINISTRATEUR** voit le bouton et peut
lancer la vérification dans **Configuration**. L'application recherche les
versions stables du dépôt public
[`BenjiMat92/gestion-de-presence`](https://github.com/BenjiMat92/gestion-de-presence)
en cliquant sur **Vérifier les mises à jour**. Si une version plus récente
existe, l'application télécharge automatiquement `GestionPresence-Setup.exe`,
vérifie le SHA-256 publié puis lance l'installation silencieuse et ferme
l'application pour permettre le remplacement des fichiers. Aucune donnée
d'identification GitHub n'est requise. En cas d'erreur, un message explicite
est affiché.

Le workflow [release.yml](./.github/workflows/release.yml) construit
l'application Windows et publie l'installateur ainsi que son checksum à chaque
tag `vMAJOR.MINOR.PATCH` poussé. Le dépôt doit contenir ces fichiers avant
qu'une mise à jour puisse être proposée.

### Construire un installateur local

Après installation des dépendances de développement et d'Inno Setup 6, lancer
la tâche VS Code **Construire l'installateur Windows** ou exécuter :

```powershell
.\build_installer.ps1
```

Le script produit `dist\GestionPresence-Setup.exe` et son fichier
`dist\GestionPresence-Setup.exe.sha256`. La version de l'installateur est lue
depuis `gestion_presence\version.py`; pour publier une mise à jour, utiliser
une version/tag supérieure à celle déjà installée.

Les dates-heures enregistrées sont des dates-heures UTC sans indicateur de fuseau
(convention SQLite). Les appels de pointage doivent recevoir une valeur consciente
de fuseau ; la date métier est déterminée dans le fuseau configuré.

## Tests

```powershell
.\.venv\Scripts\python.exe -m pytest -q
```
