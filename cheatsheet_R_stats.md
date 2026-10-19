# 📐 Cheat Sheet R — Bases du langage & Statistiques

> Convention : R indexe à partir de **1** (pas 0 comme python/numpy). L'affectation "idiomatique" utilise `<-` plutôt que `=`.

## Sommaire
1. [Variables, types et affectation](#1-variables-types-et-affectation)
2. [Vecteurs et opérations vectorisées](#2-vecteurs-et-opérations-vectorisées)
3. [Structures de données](#3-structures-de-données)
4. [Import / export de données](#4-import--export-de-données)
5. [Indexation et manipulation de data.frame](#5-indexation-et-manipulation-de-dataframe)
6. [La famille `apply`](#6-la-famille-apply)
7. [Fonctions logiques et conditions](#7-fonctions-logiques-et-conditions)
8. [Statistiques descriptives](#8-statistiques-descriptives)
9. [Graphiques de base](#9-graphiques-de-base)
10. [Tests statistiques](#10-tests-statistiques)
11. [ANOVA](#11-anova)
12. [Régression linéaire (`lm`)](#12-régression-linéaire-lm)
13. [Régression logistique et modèles linéaires généralisés (`glm`)](#13-régression-logistique-et-modèles-linéaires-généralisés-glm)
14. [Diagnostic et comparaison de modèles](#14-diagnostic-et-comparaison-de-modèles)
15. [Récapitulatif des pièges classiques](#-récapitulatif-des-pièges-classiques)

---

## 1. Variables, types et affectation

| Fonction / syntaxe | Explication |
|---|---|
| `x <- 5` | Affectation, forme **idiomatique** en R (utilisée dans presque tout le code R existant). `x = 5` fonctionne aussi, mais est déconseillé en dehors du passage d'arguments à une fonction. |
| `class(x)` | Type "utilisateur" de l'objet (`numeric`, `character`, `logical`, `factor`, `data.frame`...). |
| `typeof(x)` | Type de stockage interne (`double`, `integer`, `character`...) — plus bas niveau que `class`. |
| `str(x)` | Structure détaillée d'un objet (très utilisé pour explorer un `data.frame` ou une `list` inconnue). |
| `is.na(x)` | Teste si une valeur est manquante (`NA`). |
| `NULL` vs `NA` | `NA` = valeur manquante **au sein** d'un vecteur (compte comme un élément) ; `NULL` = absence totale de valeur (souvent utilisé pour supprimer un élément d'une liste). |

```r
x <- 5
message <- "bonjour"
print(class(x))       # "numeric"
str(data.frame(a=1:3, b=c("x","y","z")))
```

---

## 2. Vecteurs et opérations vectorisées

Le vecteur est la structure de base de R : même un scalaire est en réalité un vecteur de taille 1. La plupart des opérations sont **vectorisées** (appliquées élément par élément), sans boucle explicite.

| Fonction | Paramètres essentiels | Explication |
|---|---|---|
| `c(a, b, c, ...)` | valeurs à concaténer | Construit un vecteur ("combine"). Fonction la plus utilisée de R. |
| `seq(from, to, by=)` / `1:10` | `by` = pas | Suite arithmétique. `1:10` est un raccourci pour `seq(1, 10, by=1)`. |
| `rep(x, times=, each=)` | `times` = répétitions de la séquence entière, `each` = répétitions élément par élément | Répétition de valeurs. |
| `length(x)` | — | Nombre d'éléments. |
| `x[i]` | `i` = indice (commence à **1**), ou vecteur d'indices, ou vecteur logique | Indexation. `x[-i]` exclut l'élément `i`. `x[x > 5]` = indexation booléenne. |
| `sort(x)` / `order(x)` | — | `sort` trie les valeurs ; `order` renvoie les **indices** qui trieraient le vecteur (utile pour trier un data.frame selon une colonne). |
| `rev(x)` | — | Inverse l'ordre des éléments. |
| `which(condition)` | — | Renvoie les **indices** où la condition est vraie (équivalent de `np.where` sur un vecteur 1D). |

```r
v <- c(3, 1, 4, 1, 5)
v2 <- v * 2                  # opération vectorisée, pas de boucle
v[v > 2]                     # indexation booléenne : 3 4 5
which(v > 2)                 # indices correspondants : 1 3 5
v[order(v)]                  # équivalent de sort(v)
```

⚠️ **Recyclage (recycling)** : si deux vecteurs de tailles différentes sont combinés, R **répète silencieusement** le plus court pour compléter, sans erreur (ni toujours d'avertissement) — source fréquente de bugs discrets.
```r
c(1,2,3,4) + c(1,2)   # -> 2 4 4 6  (le vecteur court est recyclé : 1,2,1,2)
```

---

## 3. Structures de données

| Structure | Création | Explication |
|---|---|---|
| **Vecteur** | `c(1,2,3)` | Une seule dimension, **un seul type** de données. |
| **Matrice** | `matrix(data, nrow=, ncol=)` | Tableau 2D, un seul type. `dim(m)`, `t(m)` (transposée), `m %*% m2` (produit matriciel — ⚠️ `*` fait un produit terme à terme, comme en numpy). |
| **`data.frame`** | `data.frame(col1=..., col2=...)` | Équivalent d'un DataFrame pandas : tableau 2D où **chaque colonne peut avoir un type différent**. Structure centrale pour les statistiques en R. |
| **`list`** | `list(a=1, b="texte", c=TRUE)` | Collection hétérogène et de longueurs variables, accessible par `$` ou `[[ ]]` (équivalent d'un dictionnaire python, en plus permissif). |
| **`factor`** | `factor(x, levels=, labels=)` | Variable **catégorielle** — essentiel pour les statistiques (une variable catégorielle stockée en `character` n'est pas traitée pareil par `lm`/`glm`/`aov`). `levels()` donne les catégories, `nlevels()` leur nombre. |

```r
df <- data.frame(age = c(23, 45, 31), sexe = factor(c("F", "M", "F")))
str(df)          # vérifie bien que 'sexe' est un factor et pas un character !
levels(df$sexe)  # "F" "M"
```

---

## 4. Import / export de données

| Fonction | Paramètres essentiels | Explication |
|---|---|---|
| `read.csv(fichier, header=, sep=, dec=)` | `header=TRUE` (1ère ligne = noms de colonnes), `sep=","`, `dec="."` (`dec=","` pour un CSV à la française) | Charge un CSV dans un `data.frame`. |
| `read.table(fichier, header=, sep=)` | idem, plus générique | Pour des formats moins standards (espaces, tabulations...). |
| `write.csv(df, fichier, row.names=)` | `row.names=FALSE` (évite d'ajouter une colonne d'index inutile) | Sauvegarde un `data.frame`. |
| `head(df, n)` / `tail(df, n)` | `n` = nb de lignes (déf. 6) | Aperçu du début/fin d'un data.frame. |
| `View(df)` | — | Ouvre le data.frame dans un visualiseur façon tableur (RStudio). |
| `summary(df)` | — | Résumé statistique automatique de chaque colonne. |

```r
df <- read.csv("data.csv", header = TRUE, sep = ";")
summary(df)
```

---

## 5. Indexation et manipulation de data.frame

| Syntaxe | Explication |
|---|---|
| `df$colonne` | Accès à une colonne par son nom (renvoie un vecteur). |
| `df[, "colonne"]` ou `df[["colonne"]]` | Équivalents de `df$colonne`. |
| `df[i, j]` | Ligne `i`, colonne `j` (indices **ou** noms). `df[i, ]` = ligne entière, `df[, j]` = colonne entière. |
| `df[df$age > 30, ]` | Filtrage par condition (équivalent de l'indexation booléenne pandas). |
| `subset(df, condition, select=colonnes)` | Alternative plus lisible au filtrage par crochets. |
| `merge(df1, df2, by="colonne")` | Fusion de deux data.frames sur une colonne commune (équivalent de `pd.merge`). |
| `aggregate(y ~ groupe, data=df, FUN=mean)` | Agrégation par groupe (équivalent de `groupby().agg()` en pandas), avec la **syntaxe formule** `~` très utilisée en statistiques R (voir §11-13). |

```r
df[df$age > 30, "sexe"]                     # filtre + sélection d'une colonne
subset(df, age > 30, select = c(age, sexe)) # équivalent plus lisible
aggregate(age ~ sexe, data = df, FUN = mean) # âge moyen par sexe
```

---

## 6. La famille `apply`

Équivalent R des compréhensions/vectorisations numpy : appliquer une fonction à chaque élément d'une structure, sans boucle `for` explicite.

| Fonction | Entrée | Sortie | Explication |
|---|---|---|---|
| `sapply(X, FUN)` | vecteur/liste | vecteur (ou matrice si possible) | La plus utilisée au quotidien : simplifie automatiquement le résultat. |
| `lapply(X, FUN)` | vecteur/liste | **liste** | Comme `sapply` mais renvoie toujours une liste (pas de simplification) — plus prévisible dans du code robuste. |
| `apply(X, MARGIN, FUN)` | matrice/data.frame | vecteur/matrice | `MARGIN=1` = par ligne, `MARGIN=2` = par colonne (équivalent de `axis=1`/`axis=0` en numpy, mais **inversé** !). |
| `tapply(X, INDEX, FUN)` | vecteur | vecteur | Applique `FUN` à `X`, **groupé** par les catégories de `INDEX` (proche d'un `groupby` sur un seul vecteur). |
| `mapply(FUN, X, Y)` | plusieurs vecteurs | vecteur | Version "parallèle" de `sapply`, sur plusieurs arguments en même temps. |

```r
sapply(df, class)                 # type de chaque colonne
apply(m, 1, sum)                  # somme de chaque ligne d'une matrice
tapply(df$age, df$sexe, mean)     # âge moyen par sexe (alternative à aggregate)
```

⚠️ **`apply(X, MARGIN, ...)`** : `MARGIN=1` = lignes, `MARGIN=2` = colonnes — c'est l'**inverse** de la convention `axis` en numpy (`axis=0`=colonnes, `axis=1`=lignes) — source classique de confusion pour qui vient de python.

---

## 7. Fonctions logiques et conditions

| Syntaxe | Explication |
|---|---|
| `if (cond) { ... } else { ... }` | Comme en python, mais accolades `{}` au lieu de l'indentation. |
| `ifelse(cond, val_si_vrai, val_si_faux)` | Version **vectorisée** de `if`, équivalent de `np.where` — à utiliser sur des vecteurs entiers plutôt qu'un `if` classique (qui ne teste qu'une seule condition scalaire). |
| `&` / `|` (vectorisés) vs `&&` / `||` (scalaires) | `&`/`|` s'appliquent élément par élément sur des vecteurs entiers ; `&&`/`||` ne comparent que le **premier élément** et s'utilisent dans un `if()`. Ne pas les confondre. |
| `%in%` | Test d'appartenance : `x %in% c(1,2,3)` (équivalent de `in` en python, mais vectorisé). |

```r
df$categorie <- ifelse(df$age > 30, "senior", "junior")   # vectorisé, pas de boucle
if (nrow(df) > 0 && all(df$age > 0)) { print("ok") }        # && pour un test scalaire dans un if
```

---

## 8. Statistiques descriptives

| Fonction | Explication |
|---|---|
| `mean(x)`, `median(x)`, `sd(x)`, `var(x)` | Moyenne, médiane, écart-type, variance. ⚠️ Toutes renvoient `NA` si `x` contient des `NA`, sauf si on ajoute `na.rm=TRUE`. |
| `summary(x)` | Min, quartiles, médiane, moyenne, max en une seule commande — le réflexe numéro 1 pour explorer une variable. |
| `quantile(x, probs=)` | Quantiles personnalisés (`probs=c(0.1, 0.9)` pour les 10e/90e percentiles). |
| `table(x)` | Table de contingence / comptage par catégorie (équivalent de `value_counts`). `table(x, y)` = tableau croisé à deux variables. |
| `cor(x, y, method=)` | `method` = `"pearson"` (déf.), `"spearman"` (rangs), `"kendall"` | Corrélation entre deux variables. |

```r
mean(df$age, na.rm = TRUE)     # ne jamais oublier na.rm en présence de valeurs manquantes
table(df$sexe)
cor(df$age, df$revenu, method = "pearson")
```

---

## 9. Graphiques de base

| Fonction | Paramètres essentiels | Explication |
|---|---|---|
| `plot(x, y, type=, col=, main=, xlab=, ylab=)` | `type` = `"p"` (points, déf.), `"l"` (ligne), `"b"` (les deux) | Fonction générique : `plot(model_lm)` produit même des graphiques de diagnostic (voir §14). |
| `hist(x, breaks=)` | `breaks` = nombre d'intervalles | Histogramme. |
| `boxplot(y ~ groupe, data=df)` | syntaxe formule `~` | Boîtes à moustaches par groupe. |
| `pairs(df)` | — | Matrice de nuages de points croisant toutes les colonnes numériques (équivalent de `sns.pairplot`). |
| `abline(model)` / `abline(a=, b=)` | — | Ajoute une droite à un graphique existant — très utilisé pour superposer la droite de régression d'un `lm` sur un nuage de points. |

```r
plot(df$age, df$revenu)
abline(lm(revenu ~ age, data = df), col = "red")   # ajoute la droite de régression
boxplot(revenu ~ sexe, data = df)
```

---

## 10. Tests statistiques

| Fonction | Import | Paramètres essentiels | Explication |
|---|---|---|---|
| `t.test(x, y, paired=)` | base R | `paired=TRUE` pour des échantillons appariés, `FALSE` (déf.) sinon | Test de comparaison de deux moyennes. `t.test(x)` seul teste si la moyenne diffère de 0. |
| `chisq.test(table)` | base R | `table` = table de contingence (`table(x,y)`) | Test du χ² d'indépendance entre deux variables catégorielles. |
| `shapiro.test(x)` | base R | — | Test de normalité (H0 = les données suivent une loi normale) — souvent utilisé pour valider les hypothèses avant un test paramétrique (`t.test`, ANOVA). |
| `cor.test(x, y)` | base R | `method=` idem `cor()` | Teste si la corrélation observée est significativement différente de 0 (donne aussi un intervalle de confiance). |
| `wilcox.test(x, y)` | base R | — | Équivalent **non-paramétrique** du `t.test` (ne suppose pas la normalité), à utiliser si `shapiro.test` rejette la normalité. |

```r
t.test(df$revenu[df$sexe == "F"], df$revenu[df$sexe == "M"])
chisq.test(table(df$sexe, df$categorie))
shapiro.test(df$revenu)   # si p < 0.05, la normalité est rejetée -> envisager wilcox.test
```

💡 **Lecture systématique** : tous ces tests renvoient un objet avec une **p-value** (`$p.value`) — par convention, `p < 0.05` est considéré comme significatif, mais ce seuil doit être choisi *a priori* et interprété avec prudence (voir piège multi-tests en fin de fiche).

---

## 11. ANOVA

L'ANOVA (analyse de variance) généralise le `t.test` à **plus de deux groupes** : elle teste si la moyenne d'une variable diffère entre plusieurs catégories.

| Fonction | Paramètres essentiels | Explication |
|---|---|---|
| `aov(y ~ groupe, data=df)` | `y ~ groupe` = syntaxe formule ("y expliqué par groupe") | Ajuste le modèle d'ANOVA à un facteur. |
| `summary(model_aov)` | — | Affiche la table d'ANOVA (somme des carrés, F, p-value). |
| `TukeyHSD(model_aov)` | — | **Comparaisons post-hoc** : une ANOVA significative dit juste "au moins un groupe diffère", `TukeyHSD` indique **lesquels** diffèrent deux à deux, avec correction pour les comparaisons multiples. |
| `aov(y ~ groupe1 * groupe2, data=df)` | `*` = effets principaux + interaction | ANOVA à deux facteurs, avec test de l'**interaction** entre les deux facteurs. |

```r
model <- aov(revenu ~ categorie, data = df)
summary(model)          # p-value globale : la catégorie a-t-elle un effet sur le revenu ?
TukeyHSD(model)          # quelles paires de catégories diffèrent précisément ?

# ANOVA à deux facteurs avec interaction
model2 <- aov(revenu ~ sexe * categorie, data = df)
summary(model2)
```

⚠️ **L'ANOVA suppose la normalité des résidus et l'homogénéité des variances entre groupes** (`bartlett.test` ou `car::leveneTest` pour vérifier cette dernière) — à valider avant de conclure, sinon préférer une alternative non-paramétrique (`kruskal.test`).

---

## 12. Régression linéaire (`lm`)

| Fonction | Paramètres essentiels | Explication |
|---|---|---|
| `lm(y ~ x1 + x2, data=df)` | `y ~ x1 + x2` = `y` expliqué par `x1` et `x2` (formule) | Ajuste un modèle linéaire par moindres carrés. `y ~ .` = utiliser **toutes** les autres colonnes du data.frame comme variables explicatives. |
| `summary(model)` | — | Coefficients estimés, écarts-types, p-values, `R²`, `R² ajusté`. Le tableau de référence pour interpréter un modèle. |
| `coef(model)` | — | Récupère uniquement les coefficients estimés. |
| `predict(model, newdata=)` | `newdata` = nouveau data.frame avec les mêmes colonnes explicatives | Prédictions sur de nouvelles données. |
| `residuals(model)` | — | Résidus du modèle (observé - prédit), indispensables pour le diagnostic (§14). |
| `confint(model)` | — | Intervalles de confiance des coefficients. |

```r
model <- lm(revenu ~ age + anciennete, data = df)
summary(model)
# interprétation : Estimate = effet de la variable, Pr(>|t|) = p-value du coefficient

predictions <- predict(model, newdata = data.frame(age = 40, anciennete = 5))
```

👉 **Interactions et termes polynomiaux** dans la formule : `y ~ x1 * x2` (effets principaux + interaction), `y ~ poly(x, 2)` (terme quadratique), `y ~ x - 1` (sans intercept/biais).

---

## 13. Régression logistique et modèles linéaires généralisés (`glm`)

Généralisation de `lm` à des variables réponse **non gaussiennes** : binaire (régression logistique), comptage (Poisson), etc. — via une fonction de lien qui relie la moyenne de `y` à une combinaison linéaire des variables explicatives.

| Fonction | Paramètres essentiels | Explication |
|---|---|---|
| `glm(y ~ x1 + x2, data=df, family=binomial)` | `family=binomial` (logistique, `y` doit être 0/1 ou un `factor` à 2 niveaux), `family=binomial(link="logit")` (équivalent explicite), `family=poisson` (comptages) | Ajuste un modèle linéaire généralisé. La **régression logistique** est le cas `family=binomial`. |
| `summary(model)` | — | Coefficients en **échelle logit** (log-odds), pas directement en probabilité — voir transformation ci-dessous. |
| `predict(model, newdata=, type=)` | `type="response"` (indispensable pour obtenir une **probabilité**, sinon renvoie le score en logit par défaut) | Prédictions. |
| `exp(coef(model))` | — | Transforme les coefficients logit en **odds ratios**, bien plus interprétables ("chaque unité de `x` multiplie la cote par exp(coef)"). |

```r
df$y_bin <- factor(df$y_bin)   # s'assurer que la variable réponse est bien binaire/factor
model <- glm(y_bin ~ age + revenu, data = df, family = binomial)
summary(model)

exp(coef(model))                          # odds ratios, plus interprétables que les coefficients bruts
proba <- predict(model, newdata = df, type = "response")   # probabilités entre 0 et 1
classe_predite <- ifelse(proba > 0.5, 1, 0)                 # seuillage pour obtenir une classe
```

⚠️ **`predict.glm` sans `type="response"`** renvoie le score en échelle **logit** (pouvant être négatif ou > 1), pas une probabilité — piège très fréquent qui fausse silencieusement tout seuillage ultérieur.

---

## 14. Diagnostic et comparaison de modèles

| Fonction | Explication |
|---|---|
| `plot(model)` | Sur un objet `lm`/`glm` : produit 4 graphiques de diagnostic (résidus vs valeurs ajustées, QQ-plot de normalité des résidus, homogénéité de variance, points influents) — réflexe systématique après tout `lm`/`glm`. |
| `AIC(model)` / `BIC(model)` | Critères d'information : plus **bas** = meilleur compromis ajustement/complexité. Permettent de comparer des modèles **non emboîtés**, contrairement à `anova()`. |
| `anova(model1, model2)` | Compare deux modèles **emboîtés** (l'un est un cas particulier de l'autre) via un test statistique — répond à "la variable supplémentaire améliore-t-elle significativement l'ajustement ?". |
| `step(model, direction=)` | `direction` = `"both"`/`"forward"`/`"backward"` | Sélection automatique de variables par optimisation de l'AIC, itérativement — pratique mais à utiliser avec discernement (ne remplace pas une réflexion métier sur les variables pertinentes). |

```r
model1 <- lm(revenu ~ age, data = df)
model2 <- lm(revenu ~ age + anciennete, data = df)

anova(model1, model2)     # anciennete apporte-t-elle un gain significatif ?
AIC(model1); AIC(model2)  # comparaison alternative, utilisable même sur des modèles non emboîtés

model_final <- step(model2, direction = "backward")
plot(model_final)         # vérification graphique des hypothèses du modèle retenu
```

---

## 📌 Récapitulatif des pièges classiques

1. **R indexe à partir de 1**, pas de 0 — habitude à prendre en venant de python/numpy.
2. **Recyclage silencieux des vecteurs** de tailles différentes dans une opération — pas d'erreur, juste une répétition automatique parfois non désirée.
3. **`&`/`|` (vectorisés) vs `&&`/`||` (scalaires, pour un `if`)** — utiliser le mauvais opérateur dans le mauvais contexte produit une erreur ou un résultat tronqué (un seul élément comparé).
4. **`apply(X, MARGIN, ...)` : `MARGIN=1`=lignes, `MARGIN=2`=colonnes** — inversé par rapport à la convention `axis` de numpy.
5. **Une variable catégorielle stockée en `character` n'est pas automatiquement traitée comme telle** par `lm`/`glm`/`aov` tant qu'elle n'est pas convertie en `factor` — toujours vérifier avec `str(df)`.
6. **`mean()`/`sd()`/... renvoient `NA` en présence de valeurs manquantes**, sauf `na.rm=TRUE` explicitement précisé.
7. **`predict.glm()` sans `type="response"`** renvoie un score en échelle logit, pas une probabilité — toujours préciser `type="response"` en régression logistique.
8. **Une ANOVA significative ne dit pas quels groupes diffèrent** — il faut un test post-hoc (`TukeyHSD`) pour identifier les paires concernées.
9. **`anova()` ne compare que des modèles emboîtés** — pour comparer des modèles non emboîtés, utiliser `AIC`/`BIC`.
10. **Multiplier les tests statistiques sur les mêmes données augmente le risque de faux positifs** (problème des comparaisons multiples) — une correction (Bonferroni, Tukey, `p.adjust()`) est nécessaire dès qu'on effectue plusieurs tests sur le même jeu de données.
