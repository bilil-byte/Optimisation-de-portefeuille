
# Optimisation de Portefeuille : Markowitz vs Black-Litterman

Comparaison de deux modèles d'allocation d'actifs sur 10 titres du S&P 500, avec simulation Monte Carlo et analyse bootstrap de la robustesse des allocations.

---

## Objectif

Ce projet construit un pipeline complet d'optimisation de portefeuille comparant :
- **Markowitz (Mean-Variance Optimization)** : approche classique par frontière efficiente
- **Black-Litterman** : modèle bayésien combinant les rendements d'équilibre du marché avec des views investisseur

L'objectif est d'évaluer les deux modèles sur des métriques de performance historique, des mesures de risque simulées, et la stabilité des allocations face à l'incertitude d'estimation.

---

## Données

| Ticker | Société | Secteur |
|--------|---------|---------|
| AAPL | Apple | Technologie |
| JPM | JPMorgan Chase | Finance |
| JNJ | Johnson & Johnson | Santé |
| XOM | ExxonMobil | Énergie |
| AMZN | Amazon | Consommation discrétionnaire |
| PG | Procter & Gamble | Consommation de base |
| NEE | NextEra Energy | Utilities |
| CAT | Caterpillar | Industrie |
| AMT | American Tower | Immobilier (REIT) |
| GLD | SPDR Gold Shares | Matières premières |

- **Période d'entraînement** : 2017–2021
- **Période de test** : 2022
- **Source** : `yfinance`

---

## Pipeline

**1. Collecte & Préparation des données:** Les prix de clôture ajustés sont téléchargés via `yfinance` sur la période 2017–2022, puis découpés en ensemble d'entraînement (2017–2021) et de test (2022). Les rendements logarithmiques journaliers sont calculés et les valeurs manquantes supprimées.
 
**2. Estimation des paramètres :** Le vecteur de rendements espérés annualisés **μ** et la matrice de covariance annualisée **Σ** sont estimés sur les données d'entraînement. La covariance est calculée via l'estimateur de **Ledoit-Wolf** afin de réduire le bruit d'estimation inhérent aux échantillons finis.
 
**3. Portefeuille de Markowitz (MVO) :** Le portefeuille de variance minimale et le portefeuille maximisant le ratio de Sharpe sont calculés par optimisation quadratique (SLSQP). La frontière efficiente est ensuite.
 
**4. Portefeuille de Black-Litterman:** Les rendements d'équilibre implicites **Π** sont obtenus par reverse optimization à partir des poids de capitalisation boursière et d'un coefficient d'aversion au risque δ calibré sur les données. Quatre views investisseur sont formulées (deux absolues, deux relatives), encodées dans la matrice P et le vecteur Q, avec une incertitude Ω proportionnelle à τ·PΣPᵀ. La mise à jour bayésienne produit le vecteur de rendements ajustés **μ_BL**, sur lequel une optimisation max-Sharpe est relancée.
 
**5. Simulation Monte Carlo (GBM) :** 10 000 trajectoires de la valeur de chaque portefeuille sont simulées sur l'horizon de test via un Mouvement Brownien Géométrique multivarié. La décomposition de Cholesky de Σ est utilisée pour générer des chocs aléatoires corrélés entre actifs. Les sorties sont des fan charts (quantiles P5/P50/P95) et des métriques de risque de queue : VaR 95% et CVaR 95%.
 
**6. Analyse Bootstrap de la robustesse :** 500 versions bruitées de μ (Markowitz) et Π (Black-Litterman) sont générées par perturbation gaussienne (σ = 1,5%). Les poids optimaux sont recalculés à chaque tirage et l'écart-type des allocations résultantes est comparé entre les deux modèles.
 
---

## Résultats

### Métriques risque / rendement

| | Markowitz | Black-Litterman |
|---|---|---|
| Rendement espéré | 26,9% | 22,0% |
| Volatilité | 18,2% | 25,1% |
| Ratio de Sharpe | **1,25** | 0,71 |

### Métriques Monte Carlo (simulation sur 1 an)

| | Markowitz | Black-Litterman |
|---|---|---|
| Valeur finale médiane | 1,264 | 1,192 |
| P5 (pires scénarios) | 0,937 | 0,791 |
| VaR 95% | 6,3% | 20,9% |
| CVaR 95% | **13,3%** | 28,8% |

### Principaux enseignements

- Markowitz domine empiriquement sur l'ensemble des métriques historiques et simulées sur cette période
- Le risque de queue de Black-Litterman est significativement plus élevé (CVaR 28,8% vs 13,3%), directement lié à sa volatilité plus haute et son rendement espéré plus faible
- Contrairement aux attentes théoriques habituelles, les allocations BL se révèlent moins stables face aux perturbations des inputs — la compression des rendements espérés vers l'équilibre crée une quasi-indifférence entre actifs concurrents, rendant l'optimiseur très sensible aux petites variations
- Le paramètre τ n'a pratiquement aucun effet sur μ_BL, Ω étant définie proportionnellement à τ, ce qui fait que les deux termes s'annulent dans la formule de mise à jour bayésienne

---

## Structure du projet

```
portfolio-optimization/
│
├── Allocation_d_actifs___Modèle_Black-Litterman.ipynb   # Notebook principal
└── README.md
```

---

## Auteure

**Lilian Nkwemfo** — MSc Finance & Big Data, Neoma Business School  
