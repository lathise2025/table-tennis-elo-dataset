# Data Dictionary — Table Tennis Ratings Dataset

Tous les identifiants (`player_id`, `match_id`) sont des hashs opaques stables : le même joueur porte le même `player_id` dans tous les fichiers, ce qui permet les jointures, sans exposer d'identifiant source.

## players.csv
| Colonne | Type | Description |
|---|---|---|
| player_id | string | Identifiant opaque stable du joueur |
| elo | float | Rating Elo courant (set-points-Elo, voir METHODOLOGY) |
| solidite | float [0-1] | Part des victoires nettes (≥2 sets d'écart) sur les victoires |
| matchs_joues | int | Nombre de matchs joués |
| nom_reel | string | Nom du joueur |

## matches.csv (fichier principal)
| Colonne | Type | Description |
|---|---|---|
| match_id | string | Identifiant opaque du match |
| date | date | Jour du match (AAAA-MM-JJ) |
| league | string | Ligue |
| player_a, player_b | string | player_id des deux joueurs |
| elo_a_pre, elo_b_pre | float | Rating Elo **avant** le match (aucune fuite du futur) |
| sol_a_pre, sol_b_pre | float | Solidité **avant** le match |
| p_model | float [0-1] | Probabilité de victoire de A estimée par le modèle, avant le match |
| winner | A/B | Vainqueur réel |
| sets_a, sets_b | int | Score en sets |

Note : les tout premiers matchs de l'historique portent des ratings à 1500 (valeur initiale, avant que le joueur ait un historique). Le pouvoir prédictif apparaît une fois les ratings établis.

## ratings_history.csv
Trajectoire des ratings, un point par joueur et par semaine (semaine ISO).

## calibration.csv
Table de calibration : pour chaque bande de confiance annoncée, la fréquence de victoire réellement observée. Permet de convertir un score du modèle en probabilité empirique.
