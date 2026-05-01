# Between Morality and Law: Democracy and Sex Work Policy: Analysis Notebook

This notebook contains the full data pipeline and analysis for the project **"Between Morality and Law: Democracy and Sex Work Policy"**. It combines three cross-national datasets to examine whether sex work laws align with public attitudes, and how democratic institutions shape that relationship.

## Data Sources

| Dataset | Provider | Variables used |
|---|---|---|
| **V-Dem** | Varieties of Democracy Project | `v2x_libdem` (Liberal Democracy Index), `v2x_polyarchy` (Electoral Democracy Index) |
| **EVS/WVS** | European/World Values Survey (joint dataset) | `F119` — justifiability of prostitution (1–10 scale), `gwght` — survey weights |
| **UNAIDS** | UNAIDS legal environment indicators | Criminalization of selling, buying, organizing sex work; overall punitive regime flag |

Datasets are too big to be stored in GitHub, that is why they must be downloaded separately and placed locally before running the notebook. File paths in the code point to a local Windows directory (`C:\Users\Ксения\Downloads\`) — update these to match your own environment.

The link for the initial datasets: https://ceuedu-my.sharepoint.com/:f:/g/personal/kolpakova_kseniia_student_ceu_edu/IgCzi4L_tcsCQbZ90b1FjRSwAXpe7SymU21veI2qqdaRtKs?e=nEoVM6


## Notebook Structure

### 1. V-Dem + EVS/WVS Dataset Preparation
- Loads and filters the V-Dem dataset to retain only the democracy indicators and years relevant to the project (2017 onward).
- Loads the EVS/WVS dataset and restricts to valid responses on the `F119` item (values 1–10).
- Computes **weighted country-year means** of attitudes using survey weights (`gwght`).
- When the same country-year appears in multiple survey waves, retains the wave with more observations.
- Converts EVS/WVS alpha-2 country codes to ISO 3166-1 alpha-3 for merging.

### 2. UNAIDS Dataset Preparation
- Reads the UNAIDS CSV (semicolon-delimited, Windows encoding).
- Reshapes the wide multi-header format into a long tidy structure.
- Constructs three binary policy outcome variables:
  - `overall_punitive_regime` — 1 if any punitive indicator is present, 0 if explicitly non-criminalized
  - `seller_side_punitive_strict` — 1 if selling or ancillary selling is criminalized
  - `buyer_side_punitive_strict` — 1 if buying or ancillary buying is criminalized
- Standardizes country names to ISO3 codes, with a manual lookup table for non-standard names.

### 3. Merging
- Uses **nearest-year matching** (`pd.merge_asof`) to link each survey country-year to the closest available UNAIDS policy observation for the same country.
- Records the year gap between the survey and the matched policy observation.
- Merges in V-Dem democracy scores by exact country-year.
- Produces three final analysis datasets: `policy_df`, `seller_df`, `buyer_df`.

### 4. Analysis

#### Descriptive plots
- Bar chart of policy regime distribution (punitive vs. non-punitive).
- Strip plots with group means comparing public attitudes across policy regime types — separately for overall, seller-side, and buyer-side policy.
- Strip plots comparing democracy scores across policy regime types.
- Scatter plot of mean attitudes vs. standard deviation of attitudes (internal dispersion).
- Scatter plot of mean attitudes vs. Liberal Democracy Index.

#### Regression models
- **Logistic regression** with overall punitive regime as the outcome (Models 1–2), using `F119_mean` and either `v2x_libdem` or `v2x_polyarchy` as predictors.
- **Logistic regression** with seller-side criminalization as the outcome (Models 3–4), same predictors.
- **Logistic regression** with buyer-side criminalization as the outcome.
- Regression tables exported to Excel (`table1_main_policy_models.xlsx`, `table2_seller_policy_models.xlsx`).

#### Congruence analysis
- Constructs a policy-opinion congruence score: `congruence = 1 − |policy − normalized_attitude|`, where attitudes are rescaled to [0,1] to match the binary policy variable.
- **OLS regression** of congruence on Liberal Democracy Index — separately for overall, seller-side, and buyer-side policy.
- Scatter plots of congruence vs. democracy with labeled outliers.


## Outputs

All figures and tables are saved to the local Downloads folder. Update the `savefig` and `to_excel` paths before running:

| File | Content |
|---|---|
| `policy_distribution.png` | Bar chart of policy regime counts |
| `values_by_policy.png` | Attitudes by overall policy regime |
| `values_by_seller_policy.png` | Attitudes by seller-side policy |
| `values_by_buyer_policy.png` | Attitudes by buyer-side policy |
| `democracy_by_policy_regime.png` | Democracy scores by overall policy regime |
| `congruence_vs_democracy.png` | Overall congruence vs. Liberal Democracy Index |
| `seller_congruence_vs_democracy.png` | Seller-side congruence vs. democracy |
| `buyer_congruence_vs_democracy.png` | Buyer-side congruence vs. democracy |
| `mean_vs_sd_values.png` | Average attitudes and internal dispersion |
| `PwD9.png` | Attitudes vs. Liberal Democracy Index scatter |
| `table1_main_policy_models.xlsx` | Regression table: overall punitive regime models |
| `table2_seller_policy_models.xlsx` | Regression table: seller-side models |


