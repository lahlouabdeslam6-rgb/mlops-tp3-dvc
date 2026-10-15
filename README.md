# TP3 DVC - Versionner les donnees et rendre le pipeline reproductible

**Ismail Lahlou - AI - G6**

Projet de detection de fraude bancaire reprenant les cinq scripts du TP2 et ajoute un pipeline DVC, un stockage distant local et deux runs MLflow lies aux empreintes exactes des donnees.

## Resultats verifies

| Version | Transactions | Fraudes | Train / test | F1 | Precision | Rappel |
|---|---:|---:|---|---:|---:|---:|
| v1 | 5000 | 568 | 3750 / 1250 | 0.8636 | 0.8769 | 0.8507 |
| v2 | 7000 | 936 | 5250 / 1750 | 0.8423 | 0.8698 | 0.8166 |

- v1 : `cf9f6f86c141e7df32c61715c493483f`, commit `2b73bc7`.
- v2 : `ed3a72472f975230ea8e99161e7cd285`, commit `0ff8ecf`.
- Configuration finale : 300 arbres, profondeur 12, random_state 42.
- Bonus sur v2 : profondeur 8, F1 0.8493 ; profondeur 16, F1 0.8474.

## Executer sous PowerShell

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
dvc pull
dvc repro
dvc dag
```

Le remote `stockage` correspond au dossier voisin `../dvc-stockage`. Il reste local a la machine de l'auteur : un clone GitHub ne donne pas acces a ce stockage. Pour partager les objets DVC, il faut copier ce stockage ou configurer un remote partage et ses droits d'acces. En l'absence d'objets distants, `python donnees.py` regenere la v1, puis une seule execution de `python nouvelles_donnees.py` produit la v2 ; `dvc repro` reconstruit les sorties. Ne pas relancer ces generateurs sur un jeu que l'on veut conserver.

## Consulter les versions

```powershell
git checkout v1
dvc checkout
(Import-Csv data/transactions.csv).Count # 5000
git checkout main
dvc checkout
(Import-Csv data/transactions.csv).Count # 7000
```

Si le cache est vide, utiliser `dvc pull` avec acces au stockage distant. Git versionne les scripts, tickets, parametres, dvc.lock et petites metriques ; les CSV, modeles, cache et base MLflow ne sont pas dans Git.

## MLflow

Dans un premier terminal :

```powershell
mlflow server --host 127.0.0.1 --port 5000 --backend-store-uri sqlite:///mlflow.db --default-artifact-root ./mlartifacts
```

Dans un second terminal active :

```powershell
$env:PYTHONIOENCODING = 'utf-8'
python src/journaliser.py
```

L'experience `tp3-dvc` contient les deux runs historiques sur la machine d'origine. Chaque run conserve les tags `commit`, `donnees_md5`, `donnees_lignes`, les parametres et metriques, ainsi que `dvc.lock` en artefact. Relancer le script cree un run supplementaire. La base SQLite et les artefacts restent locaux.

## Environnement observe

Windows / PowerShell, Python 3.14.3, DVC 3.67.1, MLflow 3.17.1, scikit-learn 1.9.1, NumPy 2.5.4, pandas 3.0.6, matplotlib 3.11.2, joblib 1.6.0, PyYAML 6.0.3. Le fichier requirements.txt reprend les dependances du sujet ; ces versions documentent l'environnement de validation.
