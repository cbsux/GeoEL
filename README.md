# GeoEL
## I. Overview
**GeoEL (Geographically Weighted Ensemble Learning)** is a Python library dedicated to integrating the concept of geographical 
weighting with ensemble learning models, with the goal of efficiently performing regression analysis on data that exhibits 
spatial correlation. It combines geographical information with mainstream machine learning libraries such as `lightgbm`, 
`xgboost`, `catboost`, and `sklearn`, creating a comprehensive set of geographical analysis tools for users. From model training 
to prediction, and from feature importance evaluation to partial dependency analysis, GeoEL offers a full range of geographical
analysis functions, assisting users in deeply exploring the spatial characteristics and patterns within the data.

## II. Dependency Libraries
① If you want to directly use the Python library related to GeoEL for regression, you can install it directly using the 
following command:
```bash
pip install GeoEL
```

② If you want to modify, innovate and learn using relevant code, you can clone the code to your local machine:
```bash
 git clone https://github.com/cbsux/GeoEL.git
```
Then install it in the relevant directory:
```bash
pip install -r requirements.txt
```

## III. Usage 
The GeoEL library contains the following several modules:
 - `model`: The implementation of GeoEL models, including GWXGBoost, GWLightGBM, and GWCatboost. It encompasses functionalities for model training, prediction, feature importance evaluation, and local dependency analysis.
 - `select_bandwidth`: Defines the function of optimal bandwidth selection, which is selected by obtaining the optimal cross-validation value.
 - `kernels`: Provides kernel function calculations for geographically weighted models, supports multiple kernel function types, and calculates weight values based on bandwidth and distance, etc.
 - `search`: Provides two bandwidth search methods, the golden section method and the equal-spacing search method, for selecting the optimal bandwidth value.
 - `utils`: Provides some utility functions, such as the calculation of spherical distance, the instantiation of KDTree, the customization of weighted objective functions, etc.<br>
GeoEL provides a series of functions for spatial data regression analysis. The following part will introduce how to use the GeoEL library for model training, prediction, feature importance and local dependency analysis, etc.
### i. Model Initialization

```python
from GeoEL.model import GWXGBoost

gwModel = GWXGBoost(coords, feature, target, n_estimators=10, max_depth=3, bandwidth=10.0, kernel='bisquare',
                    criterion='mse', fixed=False, spherical=False, n_jobs=-1, random_state=None, feature_names=None)
```
- `coords`: A two-dimensional array representing the longitude and latitude coordinates of the samples.
- `feature`: A two-dimensional array representing the feature data of the samples.
- `target`: A one-dimensional array representing the target data of the samples.
- `n_estimators`: An integer representing the number of trees in XGBoost, defaulting to 10.
- `max_depth`: An integer representing the maximum depth of each tree, defaulting to 3.
- `bandwidth`: A floating-point number representing the bandwidth in geographical weighting, defaulting to 10.0.
- `kernel`: A string representing the kernel function in geographical weighting, with optional values of gaussian, exponential, bisquare, and tricube, defaulting to bisquare.
- `criterion`: A string representing the evaluation criterion in the random forest, with optional values of mse and mae, defaulting to mse.
- `fixed`: A boolean value indicating whether to use a fixed bandwidth, defaulting to False.
- `spherical`: A boolean value indicating whether to use spherical distance, defaulting to False.
- `n_jobs`: An integer representing the number of parallel computations, defaulting to -1.
- `random_state`: An integer representing the random seed, defaulting to None.
- `feature_names`: A list representing the names of the features, defaulting to None.
- `return`: The GeoEL model.<br>
Initialize a GeoEL model using the GeoEL class. The input coords are the longitude and latitude coordinates of the samples, feature is the feature data of the samples, and target is the target data of the samples. n_estimators, max_depth, bandwidth, kernel, criterion, fixed, spherical, n_jobs, and random_state represent the parameters of the XGBoost model, respectively. feature_names are the names of the features.

### ii. Model Training
```python
gwModel.fit()
```
Calling the `fit()` method will train the GeoEL model. This method uses Parallel for parallel computation. For each sample, it calculates weights using spatial information (longitude and latitude), and then fits a local XGBoost model. These local models are trained with weighted data, taking into account the spatial proximity of samples.
### iii. Model Prediction
```python
pred = gwModel.predict(pred_coords, pred_x)
```
- `pred_coords`: A two-dimensional array representing the longitude and latitude coordinates of the samples to be predicted.
- `pred_x`: A two-dimensional array representing the feature data of the samples to be predicted.
- `return`: A one-dimensional array representing the prediction results.<br>
Use the `predict()` method to make predictions on new data. The input `pred_coords` are the coordinates of the prediction data, and `pred_x` is the feature data of the prediction. For each prediction sample, a weighted average prediction is made based on its weights (calculated using spatial information) and the trained local models.
### iv. Feature Importance
```python
# Get and plot local feature importance
local_importance = gwModel.get_local_feature_importance(model_index, importance_type='weight')
gwModel.plot_local_feature_importance(model_index, importance_type='weight')

# Get and plot global feature importance
global_importance = gwModel.get_global_feature_importance(importance_type='weight')
gwModel.plot_global_feature_importance(importance_type='weight')
```
- `model_index`: An integer representing the index of the local model.
- `importance_type`: A string representing the type of feature importance, with optional values of weight, gain, and cover, defaulting to weight.
- `return`: Feature importance.<br>
 the get_local_feature_importance and plot_local_feature_importance methods to get and plot local feature importance, and use the get_global_feature_importance and plot_global_feature_importance methods to get and plot global feature importance.

### v. Partial Dependence Analysis
```python
# Get and plot local partial dependence
local_partial = gwModel.get_local_partial_dependence(model_index, feature_index)
gwModel.plot_local_partial_dependence(model_index, feature_index)

# Get and plot global partial dependence
global_partial = gwModel.get_global_partial_dependence(feature_index)
gwModel.plot_global_partial_dependence(feature_index)
```
- `model_index`: An integer representing the index of the local model.
- `feature_index`: An integer representing the index of the feature.
- `return`: Partial dependence.<br>
Use the get_local_partial_dependence and plot_local_partial_dependence methods to get and plot local partial dependence, and use the get_global_partial_dependence and plot_global_partial_dependence methods to get and plot global partial dependence.

### vi. Bandwidth Selection

```python
from GeoEL.select_bandwidth import SelectBandwidth

sele = SelectBandwidth(coords=coords, feature=X, target=y, model_type='XGBoost', n_estimators=10, max_depth=4, kernel='gaussian',
                       criterion='mse', fixed=False, spherical=False, n_jobs=4, random_state=1).search(verbose=True)
print(bw)
```
- `coords`: A two-dimensional array representing the longitude and latitude coordinates of the samples.
- `feature`: A two-dimensional array representing the feature data of the samples.
- `target`: A one-dimensional array representing the target data of the samples.
- `model_type`: A string representing the type of model, with optional values of XGBoost, LightGBM, and Catboost, defaulting to XGBoost.
- `n_estimators`: An integer representing the number of trees in XGBoost, defaulting to 10.
- `max_depth`: An integer representing the maximum depth of each tree, defaulting to 4.
- `kernel`: A string representing the kernel function in geographical weighting, with optional values of gaussian, exponential, bisquare, and tricube, defaulting to gaussian.
- `criterion`: A string representing the evaluation criterion in the random forest, with optional values of mse and mae, defaulting to mse.
- `fixed`: A boolean value indicating whether to use a fixed bandwidth, defaulting to False.
- `spherical`: A boolean value indicating whether to use spherical distance, defaulting to False.
- `n_jobs`: An integer representing the number of parallel computations, defaulting to 4.
- `random_state`: An integer representing the random seed, defaulting to 1.
- `return`: A floating-point number representing the optimal bandwidth.<br>
Initialize a bandwidth selector using the `SelectBandwidth` class. The input `coords` are the longitude and latitude coordinates of the samples, `feature` is the feature data of the samples, and `target` is the target data of the samples. `n_estimators`, `max_depth`, `kernel`, `criterion`, `fixed`, `spherical`, `n_jobs`, and `random_state` represent the parameters of the XGBoost model, respectively. Call the `search` method for bandwidth selection, which returns the optimal bandwidth through cross-validation.
## IV. Example of Code Usage
Here are some examples of using the GeoEL library:<br>
①[GWXGBoost + georgia dataset](https://github.com/cbsux/GeoEL/blob/master/notebook/georgia_example(GWXGBoost).ipynb)
