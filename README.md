# Logiciel SNSM — Gestion des secouristes et de la logistique

Application de bureau Windows destinée à la station SNSM de Pleubian. Elle regroupe la gestion des secouristes, des plongeurs, des DPS, du matériel, des exercices et des suivis individuels.

## Fonctionnalités

- Registre des équipiers secouristes et suivi individuel des formations, échéances et niveaux d’acquisition.
- Gestion des plongeurs, de leurs photos et documents, et accès à leur fiche depuis le tableau de bord.
- Planification et suivi des dispositifs prévisionnels de secours (DPS).
- Inventaire du matériel SPB et du matériel de secourisme, avec génération de QR codes.
- Création et suivi d’exercices de secourisme, avec affectation de secouristes.
- Modèles Word de publipostage, sauvegardes et connexion Google Drive.
- Configuration des images de la station et du centre Plongeur Pro.
- Service de communication Android sur le port TCP **8765**.
- Vérification et téléchargement de l’installateur des GitHub Releases.
- Affichage et historique des versions du logiciel.

## Prérequis

- Windows 10 ou 11.
- Python **3.13** recommandé pour lancer l’application depuis les sources.
- Une connexion Internet pour OAuth Google Drive et la mise à jour du logiciel.
- Microsoft Word est nécessaire aux fonctions d’émargement qui effectuent une conversion via Word.

## Installation depuis les sources

Depuis la racine du projet, dans PowerShell :

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python .\src\main.py
```

Pour générer des exécutables Windows, installer également PyInstaller :

```powershell
python -m pip install pyinstaller
```

## Données et bases SQLite

Au premier lancement, l’application crée le dossier `data` et initialise automatiquement `data\base_snsm.db`, ses tables et les répertoires nécessaires. L’historique des versions est stocké dans cette base.

Deux bases complémentaires, utilisées par des fonctions spécifiques, sont actuellement attendues à la racine du logiciel :

- `plongeurs.db` : fiches des plongeurs et matériel SPB associé ;
- `pse_referentiel.db` : référentiel des fiches PSE.

Elles ne sont pas créées par l’application. Pour disposer de ces fonctions, placez ces fichiers à côté de `src` lors d’un lancement depuis les sources, ou à côté de `SNSM.exe` après déploiement. La base `plongeurs.db` doit notamment contenir la table `plongeurs`, et le référentiel doit contenir les tables `chapitres` et `sous_chapitres`.

Exemple d’organisation après déploiement :

```text
SNSM/
├── SNSM.exe
├── plongeurs.db
├── pse_referentiel.db
├── _internal/                 # fichiers de l’application créés par PyInstaller
└── data/
    ├── base_snsm.db           # créée automatiquement
    ├── snsm.log               # journal de l’application
    ├── DPS/
    ├── sauvegardes/
    └── modeles_publipostage/
```

Les bases et documents peuvent contenir des données personnelles. Sauvegardez-les régulièrement et ne distribuez pas des copies contenant des données réelles sans autorisation.

## Création de l’exécutable Windows

Depuis la racine du projet, avec les dépendances installées :

```powershell
python -m PyInstaller --noconfirm --clean --windowed --onedir --name SNSM --collect-all customtkinter --collect-data certifi src\main.py
```

Le résultat est dans `dist\SNSM`. Copiez ensuite les bases complémentaires requises à côté de l’exécutable :

```powershell
Copy-Item .\plongeurs.db .\dist\SNSM\plongeurs.db
Copy-Item .\pse_referentiel.db .\dist\SNSM\pse_referentiel.db
```

Distribuez le dossier `dist\SNSM` **en entier**. `SNSM.exe` dépend du dossier `_internal` et ne doit pas en être séparé. Le paquet PyInstaller est un dossier exécutable, pas un installateur Windows.

## Version et historique

Le numéro affiché dans la fenêtre principale est défini par `VERSION_LOGICIEL` dans `src\config.py`. À chaque lancement, cette version est inscrite une fois dans la table `historique_versions_logiciel` de `data\base_snsm.db`. Pour publier une nouvelle version, mettez à jour cette constante, reconstruisez le paquet et publiez l’installateur Windows dans une GitHub Release.

Le service de mise à jour recherche un unique fichier `.exe` dans la dernière release publique du dépôt configuré dans `src\services\mise_a_jour.py`. Il vérifie l’empreinte SHA-256 fournie par GitHub avant de proposer le lancement. Une release sans installateur `.exe` ou sans empreinte SHA-256 n’est pas installée.

## Dépannage

- **Base des plongeurs introuvable** : vérifiez que `plongeurs.db` est à côté de `SNSM.exe`.
- **Référentiel PSE introuvable** : vérifiez la présence de `pse_referentiel.db` à côté de `SNSM.exe`.
- **Le port 8765 est déjà utilisé** : fermez l’autre application qui utilise ce port, puis relancez SNSM.
- **Erreur au démarrage** : consultez `data\snsm.log`.

