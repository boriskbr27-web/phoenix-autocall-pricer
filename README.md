# Phoenix Autocall Pricer

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=flat&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=flat&logo=jupyter&logoColor=white)
![Monte Carlo](https://img.shields.io/badge/Pricing-Monte%20Carlo-534AB7?style=flat)
![Greeks](https://img.shields.io/badge/Greeks-Δ%20Γ%20ν%20Θ%20ρ-1D9E75?style=flat)
![License](https://img.shields.io/badge/License-MIT-green?style=flat)

> Pricer complet d'un Phoenix Autocall multi-sous-jacent par simulation Monte Carlo (10 000 trajectoires), avec calcul des Greeks par différences finies centrées et surface de volatilité implicite 3D.

---

## Contenu du notebook

| Section | Description |
|---|---|
| 1 | Rappel théorique — GBM, mesure risque-neutre, formules des Greeks |
| 2 | Paramétrage du produit (barrières, coupon, effet mémoire, panier) |
| 3 | Moteur Monte Carlo vectorisé — 10 000 simulations |
| 4 | Calcul des Greeks : Δ, Γ, ν, Θ, ρ (différences finies centrées) |
| 5 | Surface de volatilité implicite 3D + smiles par maturité |
| 6 | Distribution des payoffs + distribution temporelle des rappels |
| 7 | Analyse de sensibilité prix vs moneyness / vol / taux |
| 8 | Récapitulatif exécutif |

---

## Modèle mathématique

### Dynamique du sous-jacent — GBM sous $\mathbb{Q}$

$$dS_i = r \, S_i \, dt + \sigma_i \, S_i \, dW_i^\mathbb{Q}$$

**Discrétisation d'Euler-Maruyama :**

$$S_{t+\Delta t} = S_t \cdot \exp\!\left[\left(r - \frac{\sigma^2}{2}\right)\Delta t + \sigma \sqrt{\Delta t}\, Z\right], \quad Z \sim \mathcal{N}(0,1)$$

### Structure Phoenix Autocall

| Condition à $t_k$ | Événement |
|---|---|
| $\text{Basket}_{t_k} \geq B_{\text{rappel}}$ | Rappel anticipé → $N \times (1 + k \cdot c)$ |
| $B_{\text{coupon}} \leq \text{Basket}_{t_k} < B_{\text{rappel}}$ | Coupon versé (ou mémorisé) |
| $\text{Basket}_{t_k} < B_{\text{coupon}}$ | Aucun coupon (mémorisé si effet mémoire) |

**Payoff à maturité (non rappelé) :**

$$\text{Payoff}_T = N \cdot \begin{cases} 1 + C_{\text{total}} & \text{si } \text{Basket}_T \geq B_{\text{capital}} \\ \text{Basket}_T + C_{\text{total}} & \text{si } \text{Basket}_T < B_{\text{capital}} \end{cases}$$

**Prix :** $V_0 = e^{-rT} \cdot \mathbb{E}^\mathbb{Q}[\text{Payoff}_T]$

---

## Greeks — Différences finies centrées

| Greek | Formule | Interprétation |
|---|---|---|
| **Delta** $\Delta$ | $\frac{V(S+h) - V(S-h)}{2h}$ | Sensibilité au prix du sous-jacent |
| **Gamma** $\Gamma$ | $\frac{V(S+h) - 2V(S) + V(S-h)}{h^2}$ | Convexité par rapport au sous-jacent |
| **Vega** $\nu$ | $\frac{V(\sigma+h) - V(\sigma-h)}{2h}$ | Sensibilité à la volatilité (+1%) |
| **Theta** $\Theta$ | $\frac{V(T-h) - V(T)}{h}$ | Décroissance temporelle (par jour) |
| **Rho** $\rho$ | $\frac{V(r+h) - V(r-h)}{2h}$ | Sensibilité au taux sans risque (+1%) |

---

## Paramètres par défaut

```python
UNDERLYINGS   = {"EuroStoxx 50": {"S0": 4800, "vol": 0.148},
                 "CAC 40":       {"S0": 8100, "vol": 0.152}}
BASKET_METHOD = "worst"      # worst-of
T             = 5.0          # maturité 5 ans
FREQ          = 4            # observations trimestrielles
r             = 0.035        # taux sans risque 3.5%
B_RECALL      = 1.00         # barrière de rappel 100%
B_COUPON      = 0.70         # barrière de coupon 70%
B_CAPITAL     = 0.60         # barrière de protection 60%
COUPON_RATE   = 0.08         # coupon annuel 8%
MEMORY_EFFECT = True         # effet mémoire activé
N_SIMS        = 10_000       # simulations Monte Carlo
```

---

## Outputs produits

- **Prix (VAN espérée)** et rendement vs nominal
- **Greeks complets** : Δ, Γ, ν, Θ, ρ par sous-jacent
- **VaR 95%** et **CVaR 95%** (expected shortfall)
- **Probabilité de rappel anticipé** et durée moyenne
- **Distribution des payoffs** (histogramme coloré)
- **Surface de volatilité implicite** (3D + smiles par maturité)
- **Courbes de sensibilité** prix vs moneyness / σ / r

---

## Utilisation

### Google Colab (recommandé — aucune installation)

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

1. Ouvrir [colab.research.google.com](https://colab.research.google.com)
2. `File → Upload notebook` → sélectionner `phoenix_autocall_pricer.ipynb`
3. `Runtime → Run all`

### En local

```bash
git clone https://github.com/boriskbr27-web/phoenix-autocall-pricer.git
cd phoenix-autocall-pricer
pip install numpy pandas matplotlib scipy
jupyter notebook phoenix_autocall_pricer.ipynb
```

---

## Dépendances

| Librairie | Usage |
|---|---|
| `numpy` | Vectorisation GBM, génération aléatoire |
| `pandas` | Tableaux récapitulatifs |
| `matplotlib` | Graphiques, surface 3D |
| `scipy` | Interpolation spline (surface de vol) |

---

## Projets liés

| Repo | Description |
|---|---|
| [`structured-products-simulator`](https://github.com/boriskbr27-web/structured-products-simulator) | Simulateur interactif React — Phoenix, Capital Garanti, Reverse Convertible, Booster |
| `brvm-portfolio-analysis` | *(à venir)* Analyse quantitative portefeuille BRVM |
| `mt5-trading-ea` | *(à venir)* Expert Advisor MetaTrader 5 — mean reversion |

---

## Auteur

**Jean-Marie Boris KABORÉ**  
MBA Trading & Finance de Marché — ESLSCA Business School Paris (Promo 2026)  
Middle/Back Office Investment Officer — Gresham Banque Privée (Groupe APICIL)  
Chargé de cours Dérivés Financiers — ESLSCA

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Profil-0077B5?style=flat&logo=linkedin)](https://www.linkedin.com/in/boris-kabore-60809219b)
[![GitHub](https://img.shields.io/badge/GitHub-boriskbr27--web-333?style=flat&logo=github)](https://github.com/boriskbr27-web)

---

*Projet académique et pédagogique. Les simulations ne constituent pas un conseil en investissement.*
