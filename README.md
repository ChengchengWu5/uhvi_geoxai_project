# Newcastle Urban Heat Vulnerability Index (UHVI) GeoXAI Project

This project develops a lightweight GeoXAI workflow for assessing Urban Heat Vulnerability Index (UHVI) patterns across Lower Layer Super Output Areas (LSOAs) in Newcastle upon Tyne. It combines Earth observation, census, deprivation, environmental, and urban morphology indicators to produce a spatial UHVI dataset, explain the drivers of vulnerability using machine learning and SHAP, test climate adaptation scenarios and intervention priority, and present results through an audience-adaptive dashboard.

The workflow is implemented across three Jupyter notebooks:

1. `newcastle_uhvi.ipynb` – builds the UHVI dataset
2. `newcastle_geoxai_analysis.ipynb` – performs machine learning, SHAP GeoXAI analysis, adaptation scenario testing, and intervention prioritisation
3. `newcastle_geoxai_dashboard.ipynb` – creates an interactive audience-adaptive GeoXAI dashboard

---

## Project aims

The project aims to:

- Generate a Land Surface Temperature (LST) raster for Newcastle upon Tyne using Landsat 8/9 imagery
- Aggregate heat exposure and explanatory variables to LSOA level
- Construct a transparent Urban Heat Vulnerability Index using exposure, sensitivity, and adaptive capacity indicators
- Use XGBoost regression to model UHVI and identify key drivers
- Apply SHAP values to explain both global and local spatial drivers of UHVI
- Simulate adaptation scenarios such as increasing tree canopy, increasing greenspace, and reducing impervious surfaces
- Rank LSOAs by intervention priority
- Communicate findings through an audience-adaptive dashboard for residents, urban planners, public health practitioners, and researchers

---

## Study area

The study area is Newcastle upon Tyne, England, and the spatial unit of analysis is the 2021 LSOA.

The project uses British National Grid (`EPSG:27700`) for spatial analysis and converts data to WGS84 (`EPSG:4326`) for the interactive dashboard.

---

## Data sources

The notebooks use or derive data from the following sources:

| Theme | Data source / method | Main variables |
|---|---|---|
| Heat exposure | Landsat 8/9 Collection 2 Level-2 Surface Temperature via Google Earth Engine | Mean LST, 95th percentile LST |
| Spatial boundaries | ONS 2021 LSOA boundaries | LSOA geometry and identifiers |
| Hotspot analysis | Getis-Ord Gi* using Queen contiguity spatial weights | Hotspot / coldspot classification, hotspot binary variable |
| Vegetation and built-up surface | Sentinel-2 summer composite via Google Earth Engine | NDVI, NDBI, impervious surface proxy, greenspace proxy, tree canopy proxy |
| Urban morphology | OSMnx road network and Sentinel-2 built-up proxy | Road density, built-up density proxy |
| Social vulnerability | Census 2021 | Population density, age 65+, age 75+, disability percentage |
| Deprivation | Index of Deprivation 2025 | IMD score, income score, health deprivation score |


---

## Notebook 1: UHVI development

`newcastle_uhvi.ipynb` builds the main UHVI dataset.

### Main steps

1. **Create a Land Surface Temperature raster**
   - Uses Landsat 8/9 Collection 2 Level-2 Surface Temperature data
   - Applies cloud masking
   - Converts surface temperature to Celsius
   - Creates a summer median LST composite
   - Exports the raster to Google Drive

2. **Aggregate LST to LSOA level**
   - Downloads official ONS 2021 LSOA boundaries
   - Filters Newcastle upon Tyne LSOAs
   - Reprojects boundaries to `EPSG:27700`
   - Calculates zonal statistics, including mean LST and 95th percentile LST

3. **Run hotspot analysis**
   - Builds Queen contiguity spatial weights
   - Runs Getis-Ord Gi* analysis
   - Classifies LSOAs into hotspot, coldspot, or non-significant categories

4. **Create explanatory variables**
   - Derives Sentinel-2 NDVI, NDBI, impervious surface, greenspace, and tree canopy proxies
   - Calculates road density using OSMnx
   - Adds Census 2021 demographic and vulnerability variables
   - Adds Index of Deprivation 2025 variables

5. **Construct UHVI**
   - Exposure variables:
     - `lst_mean`
     - `lst_p95`
     - `hotspot_binary`
   - Sensitivity variables:
     - `age_65_pct`
     - `age_75_pct`
     - `disability_pct`
     - `health_deprivation_score`
     - `population_density`
   - Adaptive capacity variables:
     - `tree_canopy_pct`
     - `green_pct`
     - `income_score`

The final UHVI is calculated as:

```text
UHVI raw = exposure_index + sensitivity_index - adaptive_capacity_index
```

The raw score is then normalised to a 0–1 scale and classified into five vulnerability classes:

- Very Low
- Low
- Moderate
- High
- Very High

### Main outputs

- `newcastle_lsoa_lst_gistar.csv`
- `newcastle_lsoa_lst_gistar.gpkg`
- `newcastle_uhvi_geoxai_inputs.csv`
- `newcastle_uhvi_geoxai_inputs.gpkg`
- `uhvi_map_continuous.png`

---

## Notebook 2: GeoXAI analysis

`newcastle_geoxai_analysis.ipynb` models UHVI and explains spatial drivers using XGBoost and SHAP.

### Main steps

1. **UHVI exploration**
   - Loads the final UHVI GeoPackage
   - Checks UHVI distribution and class counts
   - Identifies the top 20 most vulnerable LSOAs
   - Maps UHVI classes and continuous UHVI values

2. **Correlation analysis**
   - Calculates Pearson correlations between UHVI and explanatory variables
   - Saves a correlation table and bar chart

3. **XGBoost regression modelling**
   - Uses UHVI as the target variable
   - Uses environmental, morphology, demographic, and deprivation indicators as predictors
   - Splits data into training and test sets
   - Evaluates model performance using:
     - MAE
     - RMSE
     - R²
     - 5-fold cross-validation

The model uses the following configuration:

```python
XGBRegressor(
    n_estimators=300,
    max_depth=3,
    learning_rate=0.05,
    subsample=0.9,
    colsample_bytree=0.9,
    objective="reg:squarederror",
    random_state=42
)
```

4. **Feature importance**
   - Extracts XGBoost feature importance
   - Saves `xgboost_feature_importance.csv`

5. **SHAP GeoXAI analysis**
   - Uses SHAP TreeExplainer to explain the XGBoost model
   - Produces a SHAP summary plot
   - Creates spatial SHAP maps for the top local drivers
   - Identifies the strongest local driver for each LSOA
   - Produces a local SHAP waterfall plot for the highest-UHVI LSOA

6. **Adaptation scenarios**
   - Scenario 1: increase tree canopy by 20%
   - Scenario 2: increase greenspace by 20%
   - Scenario 3: reduce impervious surface and built-up density proxy by 10%
   - Scenario 4: combined intervention scenario

Each scenario predicts a revised UHVI and calculates the estimated UHVI reduction.

7. **Intervention priority**
   - Combines normalised current UHVI and normalised combined scenario benefit
   - Creates an intervention priority score
   - Classifies LSOAs into five priority levels:
     - Very Low
     - Low
     - Moderate
     - High
     - Very High

### Main outputs

- `correlation_with_uhvi.csv`
- `correlation_with_uhvi.png`
- `xgboost_feature_importance.csv`
- `newcastle_uhvi_shap_values.gpkg`
- `newcastle_geoxai_adaptation_scenarios.csv`
- `newcastle_geoxai_adaptation_scenarios.gpkg`
- `newcastle_geoxai.geojson`

---

## Notebook 3: Audience-adaptive GeoXAI dashboard

`newcastle_geoxai_dashboard.ipynb` creates a Colab-native interactive dashboard using Plotly and ipywidgets

The dashboard loads the most complete available dataset in the following order:

1. `newcastle_geoxai_adaptation_scenarios.gpkg`
2. `newcastle_uhvi_shap_values.gpkg`
3. `newcastle_full_uhvi.gpkg`

### Dashboard audiences

The dashboard provides different explanations for four audiences:

| Audience | Explanation focus |
|---|---|
| Resident | Plain-language explanation of local heat vulnerability and practical cooling advice |
| Urban planner | Scenario benefit, intervention priority, and planning interpretation |
| Public health | Exposure, sensitivity, adaptive capacity, and population vulnerability indicators |
| Researcher | SHAP-based local driver explanation and model-oriented interpretation |

### Dashboard controls

The dashboard includes widgets for:

- Selecting audience type
- Selecting an adaptation scenario
- Selecting an LSOA

---

## Key Python packages

The notebooks use the following main packages:

```text
geopandas
pandas
numpy
matplotlib
rasterio
rasterstats
libpysal
esda
scikit-learn
xgboost
shap
osmnx
earthengine-api
geemap
plotly
ipywidgets
mapclassify
```

In Google Colab, selected packages are installed directly inside the notebooks using `pip`.

---

## How to run the project

### 1. Set up Google Drive

The notebooks assume the following base folder:

```text
/content/drive/MyDrive/LST_Project
```

Create subfolders such as:

```text
data/
outputs/
figures/
models/
```

### 2. Run the notebooks in order

Run the notebooks in this sequence:

```text
1. newcastle_uhvi.ipynb
2. newcastle_geoxai_analysis.ipynb
3. newcastle_geoxai_dashboard.ipynb
```

The first notebook creates the UHVI inputs. The second notebook adds GeoXAI explanations, adaptation scenarios, and intervention priority. The third notebook loads the final outputs and displays the dashboard.

### 3. Authenticate external services

The workflow requires Google Earth Engine authentication for Landsat and Sentinel-2 data access.

OSMnx is used to download road network data. If Overpass API access is unstable, road density calculation may need to be rerun or replaced with a more stable local road dataset.

---

## Example outputs

The project produces:

- LSOA-level LST maps
- Getis-Ord Gi* hotspot and coldspot maps
- Continuous UHVI map
- UHVI class map
- Correlation plot with UHVI
- XGBoost observed vs predicted plot
- XGBoost feature importance chart
- SHAP summary plot
- Spatial SHAP driver maps
- Local SHAP waterfall plot
- Scenario benefit maps
- Intervention priority map
- Audience-adaptive dashboard explanations

---

## Interpretation

This project does not simply map heat exposure. It combines spatial heat exposure, population sensitivity, adaptive capacity, and model explainability to identify where heat vulnerability is highest, why it is high, and which areas may benefit most from adaptation interventions.

The GeoXAI component is mainly provided through:

- Machine learning modelling of UHVI using XGBoost
- Global feature importance analysis
- SHAP explanations of model behaviour
- Spatial mapping of SHAP values
- Local LSOA-level driver explanations
- Scenario-based intervention prioritisation
- Audience-adaptive explanation design

---

## Limitations

- Some environmental variables are proxy measures derived from remote sensing rather than direct field observations
- Tree canopy and greenspace estimates depend on Sentinel-2 classification thresholds
- The adaptation scenarios are simplified what-if simulations rather than engineering design models
- The XGBoost model explains relationships within the constructed UHVI dataset; it should not be interpreted as proving causal effects
- Results are sensitive to indicator selection, normalisation method, spatial scale, and data year
- Further validation with local planning, public health, or environmental datasets would strengthen the analysis

---

## Possible future development

- Comparing UHVI across multiple summers or heatwave periods
- Adding air temperature, energy demand, housing quality, or health outcome data
- Testing alternative weighting schemes for UHVI construction
- Comparing XGBoost with random forest, linear models, or spatial regression
- Adding uncertainty analysis for remote sensing and index construction
- Developing a web-based Dash or Streamlit version of the dashboard
- Adding more audience adaptive interactive visualization to the dashboard
- Conducting user testing with residents, planners, public health stakeholders, and researchers
