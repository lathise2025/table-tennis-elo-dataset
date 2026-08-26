# Publier — le repo est déjà créé

https://github.com/lathise2025/table-tennis-elo-dataset

Il est vide, public, avec la description déjà remplie. Il ne manque que les
fichiers. Depuis **ce dossier** :

```bash
git init
git add -A
git commit -m "Table tennis pre-match Elo ratings — free 2,000-row sample"
git branch -M main
git remote add origin https://github.com/lathise2025/table-tennis-elo-dataset.git
git push -u origin main
```

---

## Après le push — 2 minutes qui comptent vraiment

Sur la page du repo, **roue dentée à droite de « About »** :

Website :
```
https://lathiselam.gumroad.com/l/tt-ratings
```

Topics (c'est ce qui te fait trouver dans la recherche GitHub — ne les saute pas) :
```
dataset  elo-rating  sports-analytics  table-tennis  sports-betting
backtesting  calibration  pandas
```

## Vérifie que le rendu est bon

- `calibration_dark.png` doit s'afficher en haut du README
- `notebooks/01_calibration_check.ipynb` doit montrer ses graphiques et ses
  sorties directement dans GitHub, sans rien relancer

Ce notebook lisible sans rien installer est ton meilleur argument de vente. Si
les sorties n'apparaissent pas, dis-le-moi.
