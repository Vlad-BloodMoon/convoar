# Convoar Windows pour Web_3d — guide complet

[English](WINDOWS_WEB3D.en.md) | **Français**

> Ce dépôt est un fork de [Misterblue/convoar](https://github.com/Misterblue/convoar). Cette distribution propose un **ZIP Windows prêt à l'emploi**, comprenant `convoar.exe` et les bibliothèques nécessaires. Le paquet est préparé et testé automatiquement par GitHub Actions. Python s'installe séparément pour utiliser Web_3d.

## 1. Télécharger les bons fichiers

1. Dans les [Releases de ce fork](https://github.com/Vlad-BloodMoon/convoar/releases), télécharger **Convoar-Windows-Web3D.zip** depuis **Assets**, et non les archives automatiques « Source code ».
2. Extraire entièrement le ZIP, par exemple dans `C:\Web3D\Convoar\`. On doit y trouver `Convoar\convoar.exe` et **ses fichiers DLL**, à conserver ensemble.
3. Télécharger [Web_3d](https://github.com/Klaatou/Web_3d) et extraire les sources. L'outil `oar2glb.py` se trouve dans `dist\`. Vérifier également le manuel de la version téléchargée : les fichiers `dist/` et l'outil de conversion sont fournis avec ce projet.
4. Disposer d'un **OAR OpenSimulator** sur le PC. Garder une copie de sauvegarde de l'OAR original.
5. Convoar nécessite **.NET Framework 4.8** sous Windows ; si absent, utiliser l'installateur Microsoft officiel. Il ne suffit pas de disposer de .NET 8 ou .NET 9.

Pour un premier essai, **ne déplacer aucun fichier DLL hors de Convoar**. Le ZIP est prévu pour Windows 64 bits ; d'autres environnements ne sont pas validés par ce test.

## 2. Vérifier Convoar avec PowerShell

Ouvrir le dossier `Convoar` extrait dans l'Explorateur, cliquer la barre d'adresse, saisir `powershell`, puis Entrée. Dans **PowerShell**, lancer :

```powershell
.\convoar.exe --help
```

Il faut impérativement **`.\`** : taper seulement `convoar.exe` donne souvent « commande introuvable » même si le fichier est présent. La liste des options confirme que le programme peut démarrer ; **cela ne garantit pas encore la conversion de tous les OAR**.

## 3. Installer Python sur Windows (pour Web_3d)

**Python est requis pour `oar2glb.py`, pas pour lancer directement `convoar.exe`.**

1. Télécharger Python pour Windows 64 bits depuis [python.org](https://www.python.org/downloads/windows/). Choisir une version encore maintenue compatible avec votre version de Web_3d (Python 3.9 ou plus selon sa documentation). Notre essai avec **Python 3.14.10** a fonctionné.
2. Dans l'installateur classique, cocher **Add python.exe to PATH**, puis **Install Now**. Noter le chemin d'installation affiché ; il varie suivant la version et le compte Windows.
3. Fermer **toutes** les anciennes fenêtres PowerShell puis en ouvrir une nouvelle.
4. Vérifier :

```powershell
python --version
py -3 --version
```

Un seul des deux peut fonctionner : **le lanceur `py` n'est pas obligatoire**.

### Quand Windows répond « Python was not found »

C'est souvent un alias Microsoft Store ou un PATH non actualisé. Tester d'abord dans un **nouveau** PowerShell :

```powershell
Get-Command python -ErrorAction SilentlyContinue
where.exe python
```

Si Python est installé mais non trouvé, utiliser son chemin **absolu**. Exemple réel avec Python 3.14 installé pour un utilisateur (remplacer `UTILISATEUR` et la version) :

```powershell
& "C:\Users\UTILISATEUR\AppData\Local\Programs\Python\Python314\python.exe" --version
```

- Le **`&`** est nécessaire dans PowerShell pour exécuter un chemin entre guillemets.
- Le nom du dossier est `Python314` pour Python 3.14 ; pour Python 3.13, utiliser `Python313`, etc.
- Le chemin ci-dessus est **un exemple** : retrouver l'emplacement exact dans l'installateur ou avec l'Explorateur.
- Si un raccourci Microsoft Store intercepte `python`, vérifier **Paramètres Windows → Applications → Paramètres avancés des applications → Alias d'exécution des applications** (intitulés variables), puis désactiver si nécessaire les alias Python associés au Store.
- On peut aussi ajouter le vrai dossier d'installation Python au PATH utilisateur puis rouvrir PowerShell. **Inutile de modifier le PATH système**.
- Ne pas lancer aveuglément une installation depuis le Microsoft Store si Python est déjà installé.

## 4. Conversion OpenSimulator OAR → GLB pour Web_3d

Ouvrir PowerShell dans le dossier `dist` de Web_3d, celui qui contient `oar2glb.py`. Exemple **avec des chemins personnalisables** :

```powershell
$python = "C:\Users\UTILISATEUR\AppData\Local\Programs\Python\Python314\python.exe"
$oar = "C:\Web3D\MaRegion.oar"
$convoar = "C:\Web3D\Convoar\Convoar\convoar.exe"

& $python .\oar2glb.py $oar --convoar $convoar
```

Si `python --version` fonctionne, remplacer `& $python` par `python`.

**Important :** l'outil `oar2glb.py` est fourni par **Web_3d**, et non par Convoar. Il pilote Convoar, corrige le résultat et produit le fichier `.glb`. Le GLB généré apparaît normalement **à côté de l'OAR**. Convoar seul produit un résultat glTF suivant ses options.

### Textures JPEG2000 et problèmes fréquents

Sur certains OAR, Convoar peut afficher `Decoded image with unhandled number of components: 0`. Une ligne d'erreur n'implique pas nécessairement que tout le traitement est interrompu. Vérifier d'abord si la commande se termine et si le GLB est créé.

En cas d'échec dû aux textures, installer Pillow dans **le même Python** :

```powershell
& $python -m pip install Pillow
```

Puis relancer :

```powershell
& $python .\oar2glb.py $oar --convoar $convoar --reencode
```

Cette option retraite les textures de l'OAR dans un fichier temporaire avant conversion. Certaines textures illisibles peuvent être remplacées par une couleur neutre : **contrôler visuellement le résultat**. La conversion d'un OAR lourd peut consommer beaucoup de CPU et de RAM ; surveiller le processus et attendre le retour à l'invite. Conserver le journal `oar2glb.log` pour diagnostiquer les problèmes.

## 5. Voir le modèle dans le navigateur et le publier

1. Ouvrir `dist/glb-viewer.html` de Web_3d dans Edge/Chrome.
2. Choisir **Ouvrir .glb** et sélectionner le résultat.
3. Inspecter les objets et textures ; régler le point d'arrivée et l'orientation dans le viewer.
4. Pour un site web, choisir le **profil public**, puis **Exporter la structure modulaire (.zip)**.
5. Extraire le ZIP dans un répertoire desservi en HTTPS. Il contient en principe `index.html`, `NOM.glb`, `NOM.config.json`, `NOM.zones.json` et `lib/`.
6. Pour afficher une région à l'URL racine, créer un `default.json` contenant `{"region":"NOM"}` où `NOM` est le nom du modèle **sans `.glb`**. Ne pas confondre la région par défaut et le point d'arrivée `default` dans `NOM.zones.json`.

La page modulaire doit être servie en **HTTP(S)** ; ne pas simplement double-cliquer son `index.html` localement. Chat et avatars multiples exigent **en plus** le serveur WebSocket facultatif de Web_3d.

## 6. Dépannage résumé

| Symptôme | Vérification |
| --- | --- |
| `convoar.exe` « non reconnu » | PowerShell : `.\convoar.exe --help` depuis son dossier |
| DLL manquante | Extraire tout le ZIP ; conserver tous les fichiers dans `Convoar` |
| `py` « non reconnu » | Tester `python --version` ; `py` est facultatif |
| `python` ouvre le Store | Vérifier PATH / alias ; exécuter le vrai `python.exe` avec `& "chemin"` |
| `No module named PIL` | `& $python -m pip install Pillow` |
| Texture JPEG2000 incorrecte | `--reencode` après installation de Pillow |
| Modèle converti mais site sans région | Vérifier `default.json`, les noms des fichiers et les erreurs réseau |
| Page lourde ou lente | Vérifier la taille du GLB ; consulter les options de tuiles de Web_3d |

## 7. Portée, licence et crédits

Cette archive est distribuée pour faciliter le test du pipeline **OpenSimulator OAR → Convoar → Web_3d GLB**. Le workflow vérifie le lancement sous Windows et la présence des dépendances, **mais pas une conversion OAR de bout en bout à chaque Release**.

Convoar : © ses auteurs, sous licence Apache-2.0 (voir `LICENSE` et `THIRDPARTYLICENSES`). Web_3d : projet indépendant de [Klaatou](https://github.com/Klaatou/Web_3d), licence propre à son dépôt. Pour les autres DLL, conserver leurs licences et avis d'origine. Respecter les droits sur les objets et textures contenus dans les OAR publiés.
