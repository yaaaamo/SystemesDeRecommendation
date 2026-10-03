# Systèmes de recommandation : de la popularité au filtrage collaboratif

Recommandation de dépôts GitHub à partir des étoiles données par les utilisateurs. Ce projet compare une **baseline par popularité** à un **filtrage collaboratif basé sur les objets** (item-based collaborative filtering), évalués avec Precision@K, Recall@K et F1@K.

> TP du module **IA pour le Génie Logiciel** — Master 2 Informatique, parcours Génie Logiciel, Université de Montpellier.
> Enseignante : Imen Ben Sassi.

**Auteures :** Yagmur AYDEMIR, Liza BOUROUINA

---

## Résultat principal

| Modèle | Precision@10 | Recall@10 | F1@10 |
| --- | --- | --- | --- |
| Popularité (baseline fournie) | 0,018 | 0,035 | 0,023 |
| Item-based CF (N = 20) | 0,173 | 0,376 | **0,231** |

Le filtrage item-based multiplie le F1 par **7 à 15** selon K, et recommande **428 dépôts distincts** (74,2 % du catalogue) contre **15** pour la popularité.

---

## Contenu du dépôt

```
.
├── TP_IA_GL_Popularity_based_RS.ipynb   # Notebook complet (code commenté + résultats + analyse)
├── slides/                              # Support de présentation
└── README.md
```

## Données

Les données sont une extraction GitHub fournie avec l'énoncé du TP :

| Fichier | Contenu |
| --- | --- |
| `github-dataset.csv` | Métadonnées des dépôts (`stars_count`, `forks_count`, `language`, …) |
| `users_stars.csv` | Informations sur les utilisateurs (`location`, `company`, `languages_used`, …) |
| `user_repo_stars_filtered_min_10.csv` | Interactions utilisateur → dépôt (au moins 10 par utilisateur) |

Après nettoyage : **13 407 interactions**, **932 dépôts**, **901 utilisateurs**.

> Les fichiers CSV ne sont pas inclus dans ce dépôt. Placez-les dans votre Google Drive (`MyDrive/`) ou adaptez les chemins dans la cellule de chargement.

## Exécution

**Sur Google Colab (recommandé)**

1. Ouvrir le notebook dans Colab.
2. Déposer les trois fichiers CSV à la racine de `MyDrive`.
3. *Exécution → Tout exécuter*.

**En local**

```bash
pip install pandas numpy scikit-learn jupyter
jupyter notebook TP_IA_GL_Popularity_based_RS.ipynb
```

Supprimer la cellule `drive.mount(...)` et remplacer les chemins `/content/drive/MyDrive/...` par ceux de vos fichiers.

---

## Méthode

**Protocole d'évaluation.** Les étoiles de chaque utilisateur sont découpées en 70 % train / 30 % test (`random_state = 42`). Une recommandation est pertinente si le dépôt figure dans le test de l'utilisateur. Métriques : Precision@K, Recall@K, F1@K pour K ∈ {5, 10, 15, 20}.

**Étape 1 — Baseline par popularité (fournie).** Les dépôts sont triés par `stars_count` ; chaque utilisateur reçoit les K premiers qu'il n'a pas encore étoilés.

**Étape 2 — Filtrage collaboratif basé sur les objets.**

1. Matrice binaire utilisateurs × dépôts (901 × 577) : 1 si l'utilisateur a étoilé le dépôt.
2. Similarité cosinus entre chaque paire de dépôts (577 × 577) : deux dépôts sont proches s'ils sont étoilés par les mêmes utilisateurs.
3. Pour chaque dépôt, conservation des N voisins les plus similaires, N ∈ {3, 5, 10, 20}.
4. Score d'un dépôt candidat = somme de ses similarités avec les dépôts déjà étoilés ; recommandation des K meilleurs scores.

**Étape 3 — Évaluation et comparaison**, avec une analyse par profil d'utilisateur (activité, nombre de langages), une mesure de la couverture du catalogue et une étude du démarrage à froid.

## Résultats détaillés

**Effet du nombre de voisins N (F1@10)**

| N | 3 | 5 | 10 | 20 |
| --- | --- | --- | --- | --- |
| F1@10 | 0,212 | 0,226 | 0,228 | 0,231 |

Gain régulier mais faible, avec un plateau dès N = 5.

**F1@10 selon le profil d'utilisateur**

| Groupe | Utilisateurs | Popularité | Item-based |
| --- | --- | --- | --- |
| 7–8 étoiles en train | 452 | 0,015 | 0,204 |
| 12 étoiles ou plus | 178 | 0,044 | 0,271 |
| 0–2 langages | 131 | 0,029 | 0,293 |
| Plus de 10 langages | 437 | 0,014 | 0,211 |

L'item-based profite surtout aux utilisateurs actifs et spécialisés. Son pire groupe dépasse le meilleur groupe de la popularité.

## Analyse critique

| | Popularité | Item-based |
| --- | --- | --- |
| **Avantages** | Simple ; fonctionne pour un nouvel utilisateur | ~10× plus performant ; recommandations diversifiées ; aucune métadonnée nécessaire |
| **Limites** | Aucune personnalisation ; biais de popularité | Démarrage à froid (utilisateur et dépôt) ; coût en O(nombre de dépôts²) |

**Limites de l'évaluation :**

- `stars_count` est parfois incohérent (certains dépôts étoilés dans les données valent 0), ce qui pénalise la baseline.
- Environ 4,8 dépôts de test par utilisateur : Precision@20 ne peut guère dépasser 0,24.
- Un seul découpage aléatoire, sans intervalle de confiance.
- 1,9 % des interactions de test portent sur des dépôts absents du train, donc introuvables par l'item-based.

**Perspective :** un système hybride, item-based quand l'historique suffit, popularité ou filtrage par contenu en cas de démarrage à froid.

---

## Usage de l'IA

Des outils d'IA générative ont été utilisés à des fins de clarification de concepts, de relecture des formulations et d'aide ponctuelle au débogage. L'implémentation, les choix méthodologiques, l'exécution des expériences, l'analyse des résultats et la validation finale restent notre travail.
