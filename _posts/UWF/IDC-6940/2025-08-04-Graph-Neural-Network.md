---
title: "GNN Predictive Weather Modeling"
date: 2025-08-04
categories:
  - school
  - uwf
  - idc-6940
tags:
  - capstone
  - deep-learning
  - fall-2025
excerpt: A starter assignment for calculating the mean of a vector in R.
mathjax: true
---

## Introduction

Accurate weather prediction is a crucial task with widespread
implications across agriculture, transportation, disaster preparedness,
and energy management. Traditional forecasting methods often rely on
statistical models or physics-based simulations, however, with the
advancement of graphical neural networks (GNN) we believe there is
potential in a more modern deep learning approach [\[Lam et al. 2023\]](#Lam-et-al.-2023 "GraphCast: Learning Skillful Medium-Range Global Weather Forecasting.")
[\[Keisler 2022\]](#Keisler-2022 "Forecasting Global Weather with Graph Neural Networks.").

In this project, we explore the predictive power of a traditional linear
regression model and a GNN on real-world weather station data
[\[Herzmann 2023\]](#Herzmann-2023 "IEM:: Download ASOS/AWOS/Metar Data."). Our aim is to evaluate whether the GNN's ability
to incorporate spatial relationships between stations offers a
measurable advantage over more conventional techniques.

The dataset consists of multiple weather stations located within the
same geographic region. Each station collects meteorological variables
over time, and can be represented as a node within a broader spatial
network. For the linear model baseline, a single model will be trained
using all stations' data aggregated per feature for each time step.

For the GNN the model will be trained on the entire network of stations,
where each node corresponds to a station and edges represent spatial
relationships. The graph is encoded via a dense adjacency matrix,
excluding self-connections. The GNN aims to leverage the inherent
spatial structure of the data, potentially capturing regional weather
patterns and inter-station dependencies that are invisible to
traditional models [\[Li et al. 2018\]](#Li-et-al.-2018 "Diffusion Convolutional Recurrent Neural Network: Data-Driven Traffic Forecasting.").

Our evaluation focuses on forecasting performance over the last 6-months
of the dataset. We asses how well each modelling approach predicts key
weather variables and investigate the conditions under which one model
may outperform the other.

## Methodology

### 1. Cleaning Process

The raw weather station data underwent a multistage cleaning and
preprocessing procedure designed to ensure temporal consistency, handle
missing values, and prepare the data for both linear and GNN-based
models. The steps are as follows:

-   Original Dataset: The dataset is of shape (time, feature) where each
    node has a record for each time step resulting in each time having 8
    records, one for each node.
-   Temporal Alignment: Missing time steps were filled for all stations
    to ensure uniform time coverage across the selected date range.
-   Temporal Compression: Data was downsampled from 1-hour intervals to
    6-hour intervals to reduce noise and improve model efficiency.
-   Feature Pruning: Features missing more than 10% of their values in
    over 6 out of 8 stations were removed.
-   Station Filtering: Stations missing more than two years of valid
    data were excluded to ensure consistency across nodes (some stations
    were newer than others).
-   Correlation Analysis: Performed both between-station and
    within-station correlation analysis to better understand the spatial
    and temporal dependencies among features.
-   Feature Scaling: Remaining features were scaled using appropriate
    normalization techniques (e.g., min-max, robust scaling) to
    facilitate model training.
-   Data Reshaping: The final dataset was transformed into a 3D array
    off shape (time, station, feature), serving as the unified input
    format for both the linear and GNN models.

### 2. Correlation Analysis

The correlation analysis is done to minimize data leakage as many
weather features are directly calculated from temperature and as such
may allow the model to accurately predict the temperature without
actually learning anything from the training process. As such two kinds
of correlation analysis were done.

#### Inter-node correlation

For the inter-node correlation each node was broken into its own dataset
and then within the node all features were compared against each other
in a standard correlation analysis. This was done to identify any
offending feature for removal as data leakage is highly likely in this
kind of machine learning task.

#### Intra-node correlation

For the intra-node correlation each feature was broken into its own
dataset and then within the feature all nodes were compared against each
other in a standard correlation analysis. Intra-node correlation was
done to see the importance of spatial information as if each node has a
strong correlation that would mean that there is little to gain from
viewing the problem as a graph network.

### 3. Feature Imputation

Interpolation was incredibly important for this task as the data used in
this project comes from real world weather sensors and as such is
plagued with quality issues. There are significant chunks of missing
data within this dataset so a robust imputation technique was required.
As the goal of this project is to compare a GNN vs. an LM it doesn't
make sense to use a GNN for feature imputation. As such a two stage
technique was applied.

#### Spatial Imputation

For the spatial imputation step each time step was analysed and any with
missing features had the missing values calculated based on the
neighboring nodes and the graph edge weights. This allows the imputation
process to only consider spatial relationships for imputing which is
more powerful than focusing purely on temporal imputation.

#### Temporal Imputation

In situations where all nodes are missing a feature it is impossible to
perform the spatial imputation step and as such a more naive temporal
imputation was required. In this instance as there is little to go on
and we don't wish to use a model for imputation the only thing that
could be done was to interpolate between the missing time steps and go
from there.

### 4. Graph Neural Network (GNN)

The GNN is structured to model the spatiotemporal dynamics of the
weather station network. It is implemented through PyTorch, the
architecture is inspired by the Diffusion Convolutional Recurrent Neural
Network (DCRNN) [\[Li et al. 2018\]](#Li-et-al.-2018 "Diffusion Convolutional Recurrent Neural Network: Data-Driven Traffic Forecasting.").

#### Graph Structure and Input Format

-   Graph Representation: Each weather station is represented as a node,
    and spatial relationships are encoded in a dense adjacency matrix
    (excluding self-loops)
-   Data Format: The input is formatted using the
    StaticGraphTemporalSignal class from torch-geometric-temporal, which
    supports the temporal sequences on a static graph.
-   Train/Validation/Test Split:
    -   Testing: Final 6 months of data (730 time steps at 6-hour
        resolution)
    -   Training: First 80% of the remaining data
    -   Validation: Final 20% of the remaining data

#### Model Architecture

The model consists of three stacked DCRNN layers followed by a linear
projection layer:

DCRN(140, 64) -\> ReLU -\> DCRN(64, 32) -\> ReLU -\> DCRN(32, 32) -\>
ReLU -\> Linear(32, 1)

-   DCRNN Layers: Capture both temporal patterns as well as spatial
    diffusion through the graph structure.
-   ReLU Activations: Introduce nonlinearity after each recurrent layer.
-   Linear Output: Maps the final hidden state to the predicted
    temperature at the next time step

#### Training Configuration:

-   Optimizer: Adam
-   Learning Rate: Initial rate of 0.01, reduced by a factor of 0.1 upon
    plateau, with a minimum of 10\^-5
-   Epochs: Trained for up to 100 epochs with early stopping based on
    validation loss plateau

The GNN uses the preceding 28 time steps to forecast the next
temperature value. However, it is equipped to model both temporal trends
and spatial interdependencies, which the linear model is not explicitly
designed to capture.

### 5. Linear Model

The linear baseline model was designed as a univariate time-series
regression task. It uses the preceding 28 time steps (equivalent to 7
days) to predict the target variable tmpf at the next time step.

#### Model Input:

At each time step t, the input vector aggregates the five base features
across all stations:

$$
\text{tmpf}_{t+1}=\text{features}_t+\text{features}_{t-1}+\dots+\text{features}_{t-27}
$$

Where:

$$
\text{features}_t=\text{\{tmpf, }\text{relh, }\text{sknt, }\text{drct}_{sin}\text{, drct}_{cos}\text{\}}
$$

-   tmpf: Temperature
-   relh: Relative Humidity
-   sknt: Wind Speed
-   $$\text{drct}_{sin}\text{, drct}_{cos}\text{: }$$Wind Direction
    encoded as sine and cosine components

This flattened representation allows a standard linear regression model
to be trained on a fixed-length vector for each prediction target and
simplifies the graph structure of the weather stations. This process of
flattening the stations into a single point does remove any spatial
information from the model, however, due to the close proximity of these
nodes this makes little to no impact on the final result.

The linear model uses the same training/test split as the GNN excluding
the validation set.

## Analysis and Results

### Data Exploration and Visualization

#### 1. Data Source:

The dataset used in this project was sourced from the Iowa Environmental
Mesonet (IEM) hosted by the Iowa State University [\[Herzmann 2023\]](#Herzmann-2023 "IEM:: Download ASOS/AWOS/Metar Data.").
The data follows observational standards set by the FAA Automated
Surface Observing System (ASOS) [\[Administration 2021\]](#Administration-2021 "Automated Surface Observing System."). For this project, we
selected a subset covering a 10-year period from 2010 to 2020, focusing
on nine weather stations located in south eastern Kansas. These nodes
were chosen based on geographic proximity and consistency of data
availability.

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-2971954892.png"
alt="Graph Neural Network Capstone Graphic, shows the South Western Kansas weather station layout used in this project."
caption="Figure 1: South Western Kansas station location."
%}

#### 2. Data Structure

The original dataset contains:

-   33 features
-   8 stations
-   96,408 hourly time steps
-   With intermittent missing values across both time and stations

A subset of key features relevant to this project is summarized below:

<div class="text-center" style="font-size: 0.6em;">
While additional features are present in the dataset, many were excluded form analysis due to limited relevance or poor data quality.
</div>

| Feature   | Description                                    |
|-----------|------------------------------------------------|
| station   | Station identifier code (3-4 characters)       |
| valid     | Timestamp of the observation                   |
| lon       | Longitude                                      |
| lat       | Latitude                                       |
| elevation | Elevation in feet                              |
| tmpf      | Air temperature (F)                            |
| relh      | Relative humidity (%)                          |
| drct      | Wind direction (degrees)                       |
| sknt      | Wind speed (knots)                             |
| p01i      | Precipitations (inches) over the previous hour |
| vsby      | visibility (miles)                             |

As well as a slice of the unprocessed csv data:

<!--
```r
# reticulate::repl_python()
```

```python
# %pip install polars==1.22.0
# %pip install numpy
# %pip install pandas
# %pip install seaborn
# %pip install matplotlib
# %pip install pyarrow
# %pip install datetime
# %pip install contextily
# %pip install h3==3.7.7
# %pip install shapely
# %pip install geopandas
# %pip install scikit-learn
# %pip install tqdm
# %pip install hypothesis==6.135.26
# %pip install torch==2.7.0
# %pip install torch-scatter==2.1.2 -f https://data.pyg.org/whl/torch-2.7.0+cpu.html
# %pip install torch-sparse==0.6.18 -f https://data.pyg.org/whl/torch-2.7.0+cpu.html
# %pip install torch-geometric==2.6.1
# %pip install torch-geometric-temporal==0.56.2
```
-->

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
import os
import typing
import datetime
from pathlib import Path
import shutil

import numpy as np
import polars as pl
import seaborn as sns
import pandas as pd
from tqdm import tqdm
import geopy.distance
import scipy
import sklearn
import geopandas
import shapely
import matplotlib.pyplot as plt

import torch
import torch_geometric_temporal

import contextily as ctx
from h3 import h3

start_date = datetime.datetime(2010, 1, 1, 0, 0)
end_date = datetime.datetime(2020, 12, 31, 0, 0)

seed = 3435
split_index = 730

pl.enable_string_cache()

data_path = r'kansas_asos_2010_2020.csv'

metar_schema = {'station': pl.Categorical,
                'valid': pl.Datetime,
                'lon': pl.Float64,
                'lat': pl.Float64,
                'elevation': pl.Float64,
                'tmpf': pl.Float64,
                'dwpf': pl.Float64,
                'relh': pl.Float64,
                'drct': pl.Float64,
                'sknt': pl.Float64,
                'p01i': pl.Float64,
                'alti': pl.Float64,
                'mslp': pl.Float64,
                'vsby': pl.Float64,
                'gust': pl.Float64,
                'skyc1': pl.Categorical,
                'skyc2': pl.Categorical,
                'skyc3': pl.Categorical,
                'skyc4': pl.Categorical,
                'skyl1': pl.Float64,
                'skyl2': pl.Float64,
                'skyl3': pl.Float64,
                'skyl4': pl.Float64,
                'wxcodes': pl.String,
                'ice_accretion_1hr': pl.Float64,
                'ice_accretion_3hr': pl.Float64,
                'ice_accretion_6hr': pl.Float64,
                'peak_wind_gust': pl.Float64,
                'peak_wind_drct': pl.Float64,
                'peak_wind_time': pl.Datetime,
                'feel': pl.Float64,
                'metar': pl.String,
                'snowdepth': pl.Float64}

asos_ldf = pl.scan_csv(data_path, null_values=['T', 'M', '///'], schema=metar_schema)\
    .drop('metar')\
    .with_columns(pl.col('valid').dt.round('1h').alias('valid'))

full_date_series = np.arange(start_date, end_date, datetime.timedelta(hours=1))

asos_df = asos_ldf\
    .collect()\
    .select(pl.col('station', 'lat', 'lon', 'elevation'))\
    .unique()\
    .join(pl.DataFrame({'valid': full_date_series}), how='cross')\
    .join(asos_ldf.collect(), on=['station', 'valid'], how='left')\
    .with_columns(pl.col('valid').dt.round('6h').alias('valid'))\
    .drop('lat_right', 'lon_right', 'elevation_right')\
    .group_by(['station', 'valid'])\
    .mean()\
    .with_columns(pl.col(pl.Float64).cast(pl.Float32))
    
asos_df.head()
```
</details>

shape: (5, 32)

| station | valid               | lat       | lon         | elevation   | tmpf      | dwpf      | relh      | drct       | sknt      | p01i | alti      | mslp | vsby | gust | skyc1 | skyc2 | skyc3 | skyc4 | skyl1       | skyl2  | skyl3 | skyl4 | wxcodes | ice_accretion_1hr | ice_accretion_3hr | ice_accretion_6hr | peak_wind_gust | peak_wind_drct | peak_wind_time | feel      | snowdepth | 
|:--------|:--------------------|:----------|:------------|:------------|:----------|:----------|:----------|:-----------|:----------|:-----|:----------|:-----|:-----|:-----|:------|:------|:------|:------|:------------|:-------|:------|:------|:--------|:------------------|:------------------|:------------------|:---------------|:---------------|:---------------|:----------|:----------| 
| cat     | datetime[μs]        | f32       | f32         | f32         | f32       | f32       | f32       | f32        | f32       | f32  | f32       | f32  | f32  | f32  | cat   | cat   | cat   | cat   | f32         | f32    | f32   | f32   | str     | f32               | f32               | f32               | f32            | f32            | datetime[μs]   | f32       | f32       |
| "3K3"   | 2018-07-29 00:00:00 | 37.991699 | -101.7463   | 1005.700012 | 80.400002 | 68.5      | 67.64222  | 92.777779  | 8.722222  | 0.0  | 30.068333 | null | 10.0 | 16.0 | null  | null  | null  | null  | 6833.333496 | 8000.0 | null  | null  | null    | null              | null              | null              | null           | null           | null           | 82.831108 | null      |
| "19S"   | 2016-10-29 00:00:00 | 37.496899 | -100.832901 | 892.570007  | null      | null      | null      | null       | null      | null | null      | null | null | null | null  | null  | null  | null  | null        | null   | null  | null  | null    | null              | null              | null              | null           | null           | null           | null      | null      |
| "HQG"   | 2014-05-07 06:00:00 | 37.163101 | -101.370499 | 956.52002   | null      | null      | null      | null       | null      | null | null      | null | null | null | null  | null  | null  | null  | null        | null   | null  | null  | null    | null              | null              | null              | null           | null           | null           | null      | null      |
| "EHA"   | 2014-06-19 00:00:00 | 37.000801 | -101.879997 | 1099.0      | 88.944443 | 46.866665 | 24.601667 | 182.777771 | 16.277779 | 0.0  | null      | null | null | null | null  | null  | null  | null  | null        | null   | null  | null  | null    | null              | null              | null              | null           | null           | null           | 86.109444 | null      |
| "ULS"   | 2020-05-14 00:00:00 | 37.604    | -101.373596 | 930.429993  | 81.199997 | 51.599998 | 42.671112 | 121.111115 | 13.222222 | 0.0  | 29.608889 | null | 10.0 | 28.0 | null  | null  | null  | null  | null        | null   | null  | null  | null    | null              | null              | null              | null           | null           | null           | 80.003891 | null      |

#### 3. Exploratory Data Analysis (EDA)

Initial exploratory analysis focused on filtering out low-quality
features and stations as well as general reduction in the dimensionality
of the dataset:

-   Features with more than 10% missing values were removed.
-   Stations with excessive missing data during the 2010-2020 window
    were dropped. One station was excluded entirely as it was introduced
    after 2020, and thus had no data within the selected range.
-   The remaining dataset was evaluated to ensure sufficient temporal
    coverage and consistency across stations and features.

The visual below shows all features that meet these conditions for
varying time slices, as well as a 0/1 flag for if the station is valid
within the time slice.

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
potential_features = asos_ldf.drop('valid', 'station', 'lat', 'lon', 'elevation').collect_schema().names()
feature_list = []

for feature in potential_features:
    if not asos_df.select(pl.col(feature).is_null().all()).item():
        feature_list.append(feature)

stations_list = asos_df\
    .select(pl.col('station'))\
    .unique()\
    .to_series()\
    .to_list()

def safe_index(item, lst):
    return item in lst
  
year_series = np.arange(start_date.year, end_date.year + 1, 1)
reduced_feature_df = pl.DataFrame(schema={**{'start_year': pl.Int64, 'end_year': pl.Int64, 'year_range': pl.Int64, 'year_label': pl.String, 'feature': pl.String},
                                          **{station: pl.Boolean for station in stations_list}
                                          })

for index, a in enumerate(year_series):
    for b in year_series[index:]:
        shifted_date_series = np.arange(datetime.date(a, 1, 1), datetime.date(b, 12, 31), datetime.timedelta(hours=6))
        year_filter_df = asos_df.filter(pl.col('valid').dt.year().is_between(a, b))
        for feature in feature_list:
            if (year_filter_df.select(pl.col(feature).null_count()) != year_filter_df.height).item():
                asos_pivot_df = year_filter_df\
                    .pivot(on='station', index='valid', values=feature, aggregate_function='mean')\
                    .drop('valid')
                valid_stations = [s.name for s in asos_pivot_df if not (s.null_count() > len(shifted_date_series)*0.1)]
                if len(valid_stations) >= 6:
                    if asos_pivot_df.var().select(pl.mean_horizontal(pl.all()).alias('mean')).item() > 5:
                        valid_stations.sort()
                        new_row = pl.DataFrame({**{'start_year': a,
                                                   'end_year': b,
                                                   'year_range': (b+1) - a,
                                                   'year_label': f'{a}-{b}',
                                                   'feature': feature},
                                                **{station: safe_index(station, valid_stations) for station in stations_list}
                                                })
                        reduced_feature_df = reduced_feature_df.vstack(new_row)

features = reduced_feature_df.select(pl.col('feature')).unique().to_series().to_list()

n_cols = 3
n_rows = -(-len(features) // n_cols)

fig, axes = plt.subplots(n_rows, n_cols, figsize=(n_cols * 4, n_rows * 3), constrained_layout=True,
                         sharex=True, sharey=True)
axes = axes.flatten()

for idx, feature in enumerate(features):
    plot_df = (
        reduced_feature_df
        .filter(pl.col('feature') == feature)
        .drop('feature', 'start_year', 'end_year', 'year_range')
        .to_pandas()
    )

    plot_df.index = plot_df['year_label'].astype(str)
    plot_df = plot_df.drop('year_label', axis=1)
    plot_df = plot_df.sort_index()

    ax = axes[idx]
    sns.heatmap(plot_df, cmap='magma', annot=True, cbar=False, ax=ax)

    ax.set_title(feature)

fig.suptitle("Station Participation by Feature and Year", fontsize=16)
fig.supxlabel("Stations")
fig.supylabel("Year")

plt.show()
```

</details>

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-2390309202.png"
alt="Graph Neural Network Capstone Graphic, shows which stations are valid through a binary flag for six key variables."
caption="Figure 2"
%}

With this visual a date range was selected from 2018 to 2020 as this
range had the most valid features and stations while also being quite
recent. As this range was selected the ULS station was dropped resulting
in seven valid stations and six valid features.

Below is a slice of the dataset at this stage of the EDA process:

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
df_t = reduced_feature_df\
    .filter(pl.col('year_label').eq('2018-2020'))\
    .drop('year_range', 'year_label', 'start_year', 'end_year')\
    .transpose(include_header=True)


new_headers = df_t.row(0)

df_t_no_header = df_t.slice(1, df_t.height - 1)

df_t_renamed = df_t_no_header.rename({old: str(new) for old, new in zip(df_t_no_header.columns, new_headers)})

valid_features = [col for col in df_t_renamed.columns if col != "feature"]
valid_stations = df_t_renamed\
    .with_columns(pl.col(valid_features)\
                  .map_elements(lambda x: x == 'true', return_dtype=pl.Boolean))\
    .filter( pl.all_horizontal([pl.col(col) for col in valid_features]))\
    .select(pl.col('feature'))\
    .to_series()\
    .to_list()

reduced_asos_df = asos_df\
    .filter(pl.col('valid').is_between(datetime.datetime(2018, 1, 1, 0, 0), datetime.datetime(2021, 1, 1, 0, 0)))\
    .filter(pl.col('station').is_in(valid_stations))\
    .select(pl.col(['station', 'valid', 'lat', 'lon', 'elevation'] + valid_features))\
    .sort(['valid', 'station'])\
    .with_columns([(pl.col('drct').map_elements(lambda x: np.sin(np.radians(x)), return_dtype=pl.Float64)).alias('drct_sin'),
                   (pl.col('drct').map_elements(lambda x: np.cos(np.radians(x)), return_dtype=pl.Float64)).alias('drct_cos')])\
    .drop('drct')

row_count, feature_count = reduced_asos_df.drop('station', 'valid', 'lat', 'lon', 'elevation').shape
valid_station = reduced_asos_df.select(pl.col('station')).head(7).to_series().to_list()
station_count = len(valid_station)
valid_features = reduced_asos_df.drop('station', 'valid', 'lat', 'lon', 'elevation').columns

station_matrix = reduced_asos_df.drop('station', 'valid', 'lat', 'lon', 'elevation').to_numpy().reshape(int(row_count/station_count), station_count, feature_count)

reduced_asos_df
```

</details>

shape: (30_667, 12)

| station | valid               | lat       | lon         | elevation   | tmpf      | dwpf      | relh      | sknt      | feel      | drct_sin  | drct_cos  |
|:--------|:--------------------|:----------|:------------|:------------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|
| cat     | datetime[μs]        | f32       | f32         | f32         | f32       | f32       | f32       | f32       | f32       | f64       | f64       |
| "GCK"   | 2018-01-01 00:00:00 | 37.927502 | -100.724403 | 881.0       | 9.283334  | -6.5      | 48.148335 | 8.666667  | -4.161667 | 0.851117  | 0.524977  |
| "LBL"   | 2018-01-01 00:00:00 | 37.044201 | -100.9599   | 879.0       | 12.316667 | -1.983333 | 52.014999 | 10.166667 | -1.721667 | 0.664796  | 0.747025  |
| "EHA"   | 2018-01-01 00:00:00 | 37.000801 | -101.879997 | 1099.0      | 15.555555 | 5.255556  | 63.382778 | 7.388889  | 4.482222  | 0.970763  | 0.24004   |
| "HQG"   | 2018-01-01 00:00:00 | 37.163101 | -101.370499 | 956.52002   | 14.311111 | -1.605556 | 48.468334 | 7.777778  | 2.681667  | 0.936332  | 0.351115  |
| "3K3"   | 2018-01-01 00:00:00 | 37.991699 | -101.7463   | 1005.700012 | 13.1      | -0.9      | 53.127777 | 6.777778  | 2.151111  | 0.981255  | 0.192712  |
| …       | …                   | …         | …           | …           | …         | …         | …         | …         | …         | …         | …         |
| "EHA"   | 2020-12-31 00:00:00 | 37.000801 | -101.879997 | 1099.0      | 42.355556 | 16.588888 | 34.955555 | 3.0       | 40.254444 | -0.533205 | -0.845986 |
| "HQG"   | 2020-12-31 00:00:00 | 37.163101 | -101.370499 | 956.52002   | 40.722221 | 13.255555 | 32.23111  | 1.666667  | 39.450001 | 0.977334  | -0.211704 |
| "3K3"   | 2020-12-31 00:00:00 | 37.991699 | -101.7463   | 1005.700012 | 40.200001 | 12.2      | 31.377777 | 4.555555  | 36.683334 | -0.700217 | -0.71393  |
| "JHN"   | 2020-12-31 00:00:00 | 37.578201 | -101.7304   | 1012.710022 | 40.711113 | 17.4      | 38.554443 | 4.777778  | 37.06889  | -0.824675 | -0.565607 |
| "19S"   | 2020-12-31 00:00:00 | 37.496899 | -100.832901 | 892.570007  | 39.922222 | 15.388889 | 36.552223 | 6.111111  | 35.113335 | -0.824675 | 0.565607  |


<!--
```python
# as this is true it means there are no time slices where all values are nan
# not np.any(np.all(np.isnan(station_matrix), axis=(1, 2)))
```
-->

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
def compute_node_distance(node1, node2, inverse=False):
    coords_1 = (node1[1], node1[0])
    coords_2 = (node2[1], node2[0])
    horizontal_distance = geopy.distance.geodesic(coords_1, coords_2).km
    if inverse:
        try:
            horizontal_distance = 1/horizontal_distance
        except ZeroDivisionError:
            horizontal_distance = 0
    return horizontal_distance

station_df = reduced_asos_df.select(pl.col(['station', 'lon', 'lat', 'elevation'])).unique().to_pandas()
grid_list = station_df.loc[:, ['lon', 'lat']].reset_index()[['lon', 'lat', 'index']].to_numpy().tolist()
grid_list = [sublist[:-1] + [int(sublist[-1])] for sublist in grid_list]


result_dict = {'index': [],
               'station': []}
result_dict = {**result_dict, **{str(i): [] for _, _, i in grid_list}}

for row_index, station in station_df.iterrows():
    for col_index, station2 in station_df.iterrows():
        result_dict[str(col_index)].append(compute_node_distance([station['lon'], station['lat']], [station2['lon'], station2['lat']], inverse=True))
    result_dict['station'].append(station['station'])
    result_dict['index'].append(row_index)

grid_map_df = pl.DataFrame(result_dict, schema={**{'station': pl.Categorical, 'index': pl.UInt64}, **{str(i): pl.Float64 for _, _, i in grid_list}})

scaled_idistance = sklearn.preprocessing.minmax_scale(grid_map_df.drop('station', 'index'))
modified_adjacency_df = grid_map_df.select(pl.col(['station', 'index']))\
    .join(pl.DataFrame(scaled_idistance, schema=[str(i) for i, _ in enumerate(scaled_idistance)]).with_row_index(),
          on='index')
```
</details>

#### 4. Graph Creation

To prepare the dataset for graph-based modeling, a spatial graph was
constructed:

-   Each station was treated as a node.
-   A dense adjacency matrix (excluding self-connections) was created by
    computing geodesic distances between stations.
-   Edge weights were defined as the inverse of the geodesic distance,
    scaled to a \[0, 1\] range using MinMax scaler. The closer two
    stations are, the stronger their connection in the graph.

Below is a visual of the full graph structure:

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
station_node_list = [shapely.geometry.Point(lon, lat) for lat, lon in station_df[['lat', 'lon']].to_numpy()]
stations_gdf = geopandas.GeoDataFrame(station_df.copy(), geometry=station_node_list, crs="EPSG:4326")

edge_list = []
n = len(station_node_list)
for i in range(n):
    for j in range(i + 1, n):
        edge_list.append(shapely.geometry.LineString([station_node_list[i], station_node_list[j]]))

edges_gdf = geopandas.GeoDataFrame(geometry=edge_list, crs="EPSG:4326")

fig, ax = plt.subplots(figsize=(18, 10))

edges_gdf.plot(ax=ax, color="xkcd:blue", linewidth=1, alpha=1)

stations_gdf.plot(ax=ax, color="xkcd:bright orange", markersize=10)

for i, row in stations_gdf.iterrows():
    ax.text(row.geometry.x, row.geometry.y, row['station'], fontsize=9)

ctx.add_basemap(ax, crs=stations_gdf.crs.to_string())
```

</details>

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-1377963172.png"
alt="Graph Neural Network Capstone Graphic, shows all the stations connected in a dense adjacency matrix."
caption="Figure 3: Full station network graph."
%}


#### 5. Spatiotemporal Imputation

Missing values were imputed through a two-stage process leveraging both
spatial and temporal structure:

1.  Spatial Imputation: Each missing value was estimated based on the
    value of neighboring nodes within the same time step, weighted by
    graph connectivity.
2.  Temporal Imputation: Remaining gaps were filled by interpolating
    along the time axis for each node individually.

While not a perfect method, this approach produced plausible and
continuous data, as visually confirmed during quality checks as shown
below.

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
adj = modified_adjacency_df.drop('station', 'index').to_numpy()
adj = adj / adj.sum(axis=1, keepdims=True)

def spatial_impute(data, adj):
    imputed = data.copy()
    T, N, F = data.shape

    for t in range(T):
        for i in range(N):
            for f in range(F):
                if np.isnan(imputed[t, i, f]):
                    neighbor_vals = imputed[t, :, f]
                    weights = adj[i]
                    mask = ~np.isnan(neighbor_vals)

                    if mask.sum() > 0:
                        imputed[t, i, f] = np.dot(weights[mask], neighbor_vals[mask]) / weights[mask].sum()

    return imputed

def spatiotemporal_impute(data, adj):
    data = spatial_impute(data, adj)

    T, N, F = data.shape
    for i in range(N):
        for f in range(F):
            series = data[:, i, f]
            mask = ~np.isnan(series)
            if mask.sum() == 0:
                continue
            indices = np.arange(T)
            data[:, i, f] = np.interp(indices, indices[mask], series[mask])

    return data

data_imputed = spatiotemporal_impute(station_matrix, adj)
```

</details>


Below is an example of the data requiring both spatial and temporal
imputation:

```python
_ = pd.DataFrame(station_matrix[:200, :, 2]).plot(legend=False, subplots=True, figsize=(20, 8))
```

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-3369587814.png"
alt="Graph Neural Network Capstone Graphic, shows station data plotted over time highlighting missing values."
caption="Figure 4: Time-series plot with missing data."
%}

Below is the same data post imputation:

```python
_ = pd.DataFrame(data_imputed[:200, :, 2]).plot(legend=False, subplots=True, figsize=(20, 8))
```

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-2264760746.png"
alt="Graph Neural Network Capstone Graphic, shows station data plotted over time with imputed values."
caption="Figure 5: Time-series plot with imputed data."
%}

#### 6. Correlation Analysis

To avoid feature redundancy and data leakage, a correlation analysis was
conducted:

-   Inter-node and Intra-node correlations were computed.
-   Inter-node correlation was done to find any relationships between
    features within a node.
-   Intra-node correlation was done to explore the importance of spatial
    information across features.
-   Two features, dwpf (dew point in f) and feel (feels-like temperature
    in f), showed high correlation with the target variable tmpf
    (temperature in f). Since these variables are partially derived from
    the target variable tmpf, they were removed to maintain model
    integrity and avoid leakage.

Below shows the inter-node correlation:

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
def node_correlation(data, node_number):
    T, N, F = data.shape

    corr_within_node = np.empty((N, F, F))

    for node in range(N):
        node_data = data[:, node, :]
        corr_within_node[node] = np.corrcoef(node_data, rowvar=False)

    corr_list = []
    for f_feature in range(F):
        inner_list = []
        for s_feature in range(F):
            if f_feature < s_feature:
                inner_list.append(np.nan)
            else:
                inner_list.append(float(corr_within_node[node_number, f_feature, s_feature]))
        corr_list.append(inner_list)
    return corr_list
  
def between_node_correlation(data, feature_number):
    feature_data = data[:, :, feature_number]

    corr_between_nodes = np.corrcoef(feature_data.T)
    corr_between_nodes[np.triu_indices(corr_between_nodes.shape[0], 1)] = np.nan

    return corr_between_nodes

fig, axes = plt.subplots(2, 4, figsize=(20, 10))
axes = axes.flatten()

for i in range(data_imputed.shape[1]):
    corr_matrix = pd.DataFrame(
        node_correlation(data_imputed, i),
        columns=valid_features,
        index=valid_features
    )
    _ = sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', vmin=-1, vmax=1, ax=axes[i])
    _ = axes[i].set_title(f'{valid_station[i]} Correlation')

_ = plt.tight_layout()
_ = plt.show()
```

</details>

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-499462486.png"
alt="Graph Neural Network Capstone Graphic, shows correlation matrix of features within a node ie one held node feature-to-feature."
caption="Figure 6"
%}

Below shows the intra-node correlation:

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
fig, axes = plt.subplots(2, 4, figsize=(20, 10))
axes = axes.flatten()

for i in range(data_imputed.shape[2]):
    corr_matrix = pd.DataFrame(
        between_node_correlation(data_imputed, i),
        columns=valid_station,
        index=valid_station
    )
    _ = sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', vmin=-1, vmax=1, ax=axes[i])
    _ = axes[i].set_title(f'Between-Node Correlation: {valid_features[i]}')

_ = plt.tight_layout()
_ = plt.show()
```

</details>

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-343898519.png"
alt="Graph Neural Network Capstone Graphic, shows correlation matrix of features between nodes ie one held feature node-to-node."
caption="Figure 7"
%}

#### 7. Final Preparation

After all preprocessing steps, the final dataset was reduced and
standardized:

-   5 features were retained: tmpf, relh, sknt, drct_sin, drct_cos
-   7 stations remained after filtering
-   4,381 time steps at 6-hour intervals (equivalent to 2 years of data)
    remained

The remaining unscaled features were scaled using the RobusScaler from
scikit-learn to mitigate the influence of outliers while preserving
overall data distribution. Which resulted in the below array.

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
T, N, F = data_imputed.shape
imputed_df = pl.from_numpy(data_imputed.reshape(T * N, F), schema=valid_features)\
    .drop('dwpf', 'feel')\
    .with_columns(pl.DataFrame([valid_station[i % len(valid_station)] for i in range(T*N)], schema={'station': pl.Categorical})\
          .join(pl.from_pandas(station_df),
                on='station',
                how='left'))\
    .select(pl.col(['station', 'lon', 'lat', 'elevation', 'tmpf', 'relh', 'sknt', 'drct_sin', 'drct_cos']))\
    .drop('lon', 'lat', 'elevation')\
    .with_columns(pl.col('relh')/100)

imputed_df = imputed_df.drop('tmpf', 'sknt')\
    .hstack(pl.DataFrame(sklearn.preprocessing.robust_scale(imputed_df.select(pl.col('tmpf'))), schema=['tmpf']))\
    .hstack(pl.DataFrame(sklearn.preprocessing.robust_scale(imputed_df.select(pl.col('sknt'))), schema=['sknt']))\
    .select(pl.col(['station', 'tmpf', 'relh', 'sknt', 'drct_sin', 'drct_cos']))

imputed_df

imputed_pd_df = imputed_df.to_pandas()
imputed_pl_df = imputed_df.drop('station')
```

</details>

shape: (30_667, 6)

| station | tmpf      | relh     | sknt      | drct_sin  | drct_cos  |
|:---     |:---       |:---      |:---       |:---       |:---       |
| cat     | f64       | f64      | f64       | f64       | f64       |
| GCK     | -1.355721 | 0.481483 | -0.080357 | 0.851117  | 0.524977  |
| LBL     | -1.265174 | 0.52015  | 0.160714  | 0.664796  | 0.747025  |
| EHA     | -1.168491 | 0.633828 | -0.285714 | 0.970763  | 0.24004   |
| HQG     | -1.205638 | 0.484683 | -0.223214 | 0.936332  | 0.351115  |
| 3K3     | -1.241791 | 0.531278 | -0.383929 | 0.981255  | 0.192712  |
| …       | …         | …        | …         | …         | …         |
| EHA     | -0.368491 | 0.349556 | -0.991072 | -0.533205 | -0.845986 |
| HQG     | -0.417247 | 0.322311 | -1.205357 | 0.977334  | -0.211704 |
| 3K3     | -0.432836 | 0.313778 | -0.741072 | -0.700217 | -0.71393  |
| JHN     | -0.417579 | 0.385544 | -0.705357 | -0.824675 | -0.565607 |
| 19S     | -0.441128 | 0.365522 | -0.491072 | -0.824675 | 0.565607  |

### Modeling and Results

#### Model 1: Graph Neural Network (DCRNN)

The Graph Neural Network was trained using the previous 28 time steps
(equivalent to 7 days) and leveraged a dense spatial graph connecting
all stations. This structure enabled the model to learn both temporal
sequences and spatial diffusion patters across the weather station
network.

-   Graph Structure: Dense graph with edge weights based on inverse
    geodesic distance
-   Architecture: Three stacked DCRNN layers + ReLU activations + Linear
    projection
-   Target: Next-step temperature prediction for each node

Results:

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
row_count, feature_count = imputed_pl_df.shape
station_count = imputed_df.select(pl.col('station').unique()).count().item()
np_imp_array = imputed_pl_df.to_numpy().reshape(int(row_count/station_count), station_count, feature_count)

def concat_past_timesteps(data, window=4):
    T, _, _ = data.shape

    slices = [data[i:T - window + i + 1] for i in range(window)]

    return np.concatenate(slices[::-1], axis=2)

lagged_array = concat_past_timesteps(np_imp_array, 28)

T, N, F = lagged_array.shape

edge_index = [[i, j] for i in range(N) for j in range(N) if i != j]
edge_weight = np.array([float(adj[edge[0], edge[1]]) for edge in edge_index])
edge_index = np.array(edge_index).T
features = [lagged_array[t, :, :] for t in range(T-1)]
targets = [lagged_array[t+1, :, 0] for t in range(T-1)]

class RecurrentGCN(torch.nn.Module):
    def __init__(self, node_features):
        super(RecurrentGCN, self).__init__()
        self.recurrent1 = torch_geometric_temporal.nn.recurrent.DCRNN(node_features, 64, 1)
        self.recurrent2 = torch_geometric_temporal.nn.recurrent.DCRNN(64, 32, 1)
        self.recurrent3 = torch_geometric_temporal.nn.recurrent.DCRNN(32, 32, 1)

        self.linear = torch.nn.Linear(32, 1)

    def forward(self, x, edge_index, edge_weight):
        h = self.recurrent1(x, edge_index, edge_weight)
        h = torch.nn.functional.relu(h)

        h = self.recurrent2(h, edge_index, edge_weight)
        h = torch.nn.functional.relu(h)

        h = self.recurrent3(h, edge_index, edge_weight)
        h = torch.nn.functional.relu(h)

        h = self.linear(h)
        return h

dataset = torch_geometric_temporal.signal.StaticGraphTemporalSignal(
    edge_index=edge_index,
    edge_weight=edge_weight,
    features=features,
    targets=targets
)

train_dataset, test_dataset = torch_geometric_temporal.temporal_signal_split(dataset, train_ratio=(T-split_index)/T)
train_dataset, validation_dataset = torch_geometric_temporal.temporal_signal_split(train_dataset, train_ratio=0.8)

gnn_model = RecurrentGCN(node_features = F)

optimizer = torch.optim.Adam(gnn_model.parameters(), lr=0.01)
scheduler = torch.optim.lr_scheduler.ReduceLROnPlateau(
    optimizer,
    mode='min',
    factor=0.1,
    patience=5,
    min_lr=1e-5,
)

if not os.path.exists('best_model.pt'):
    best_loss = float('inf')
    patience = 10
    trigger_times = 0
    target_epochs = 100

    gnn_model.train()
    with tqdm(total=target_epochs) as pbar:
        for epoch in range(target_epochs):
            gnn_model.train()
            torch.enable_grad()
            train_cost = 0

            for time, snapshot in enumerate(train_dataset):
                y_hat = gnn_model(snapshot.x, snapshot.edge_index, snapshot.edge_attr)
                train_cost += torch.mean((y_hat - snapshot.y)**2)

            train_cost /= (time + 1)
            train_cost.backward()
            optimizer.step()
            optimizer.zero_grad()

            scheduler.step(train_cost.item())

            gnn_model.eval()
            val_loss = 0
            with torch.no_grad():
                for time, snapshot in enumerate(validation_dataset):
                    y_hat = gnn_model(snapshot.x, snapshot.edge_index, snapshot.edge_attr)
                    val_loss += torch.mean((y_hat - snapshot.y)**2)
                val_loss /= (time + 1)

            pbar.set_postfix({
                'train_loss': train_cost.item(),
                'val_loss': val_loss.item(),
                'lr': optimizer.param_groups[0]['lr']
            })
            pbar.update(1)

            if best_loss < 0.05 and epoch > 1:
                print(f"Early stopping triggered at epoch {epoch + 1}")
                break

            if val_loss.item() < best_loss:
                best_loss = val_loss.item()
                trigger_times = 0
                torch.save(gnn_model.state_dict(), 'best_model.pt')
            else:
                trigger_times += 1
                if trigger_times >= patience:
                    print(f"Early stopping triggered at epoch {epoch + 1}")
                    break
else:
    gnn_model.load_state_dict(torch.load('best_model.pt'));

gnn_model.eval();
cost = 0
for time, snapshot in enumerate(test_dataset):
    y_hat = gnn_model(snapshot.x, snapshot.edge_index, snapshot.edge_attr)
    cost = cost + torch.mean((y_hat - snapshot.y)**2)
cost = cost / (time+1)
cost = cost.item()
print("MSE: {:.4f}".format(cost))
```

</details>

MSE: 0.0562

It is visually apparent that the model is able to follow the temporal
trends of the weather data with some latency in predictive results
appearing to be quite accurate.

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
all_preds = []
all_targets = []

gnn_model.eval();
with torch.no_grad():
    for snapshot in test_dataset:
        y_pred = gnn_model(snapshot.x, snapshot.edge_index, snapshot.edge_attr).squeeze()
        y_true = snapshot.y.squeeze()
        all_preds.append(y_pred.cpu().numpy())
        all_targets.append(y_true.cpu().numpy())

all_preds = np.stack(all_preds)
all_targets = np.stack(all_targets)

num_nodes = all_preds.shape[1]
num_time_steps = all_preds.shape[0]

cols = 1
rows = num_nodes

fig, axes = plt.subplots(rows, cols, figsize=(15, 20), sharex=True, sharey=True)

for node in range(num_nodes):
    ax = axes[node]
    _ = ax.plot(all_targets[:, node], label='Actual', color='blue', linewidth=1)
    _ = ax.plot(all_preds[:, node], label='Predicted', color='orange', linewidth=1)
    _ = ax.set_title(f'Station {stations_list[node]}')
    _ = ax.grid(True)

_ = axes[0].legend(loc='upper right')

_ = fig.suptitle("GNN Actual vs Predicted Over Time for Each Node", fontsize=16)
_ = fig.tight_layout(rect=[0, 0.03, 1, 0.95])
_ = plt.show()
```

</details>

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-1987425428.png"
alt="Graph Neural Network Capstone Graphic, shows the predicted value vs actual value over time by node for the GNN model."
caption="Figure 8"
%}


However, it is also apparent that there is a weird latency in the
results of the prediction as an absolute error of 1 is quite extreme
given the total range of the test features is -1.5 to 1.5.

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
all_preds = []
all_targets = []

gnn_model.eval();
with torch.no_grad():
    for snapshot in test_dataset:
        y_pred = gnn_model(snapshot.x, snapshot.edge_index, snapshot.edge_attr).squeeze()
        y_true = snapshot.y.squeeze()
        all_preds.append(y_pred.cpu().numpy())
        all_targets.append(y_true.cpu().numpy())

all_preds = np.stack(all_preds)
all_targets = np.stack(all_targets)

gnn_absolute_error = np.abs(np.subtract(np.array(all_preds, dtype=float), np.array(all_targets, dtype=float)))

fig, axes = plt.subplots(rows, cols, figsize=(15, 20), sharex=True, sharey=True)
for node in range(num_nodes):
    ax = axes[node]
    _ = ax.plot(gnn_absolute_error[:, node], label='Error', color='blue', linewidth=1)
    _ = ax.set_title(f'Station {stations_list[node]}')
    _ = ax.grid(True)

_ = axes[0].legend(loc='upper right')

_ = fig.suptitle("GNN Absolute Error for Each Station", fontsize=16)
_ = fig.tight_layout(rect=[0, 0.03, 1, 0.95])
_ = plt.show()
```

</details>

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-762293348.png"
alt="Graph Neural Network Capstone Graphic, shows the absolute error over time by node for the GNN model."
caption="Figure 9"
%}

#### Model 2: Linear Regression

The linear baseline model was trained using the same 28-time-step
history with aggregated weather station data to predict the next time
step's temperature. All features were flattened into a single vector,
treating the problem as a high-dimensional regression task with no
spatial awareness.

-   Strengths: Simplicity, interpretability, low training time
-   Weakness: Cannot leverage inter-station relationships or dynamic
    spatial trends.

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
flattened_array = np.mean(lagged_array, axis=1)

T, N, F = lagged_array.shape

X = flattened_array[:-1]
X_train, X_test = np.split(X, [T-split_index])
y_true = flattened_array[1:, 0]
y_train, y_test = np.split(y_true, [T-split_index])

for node_index in range(N):
  node_slice = lagged_array[:, node_index, :]
  _, node_test = np.split(node_slice, [T-split_index])

lr_model = sklearn.linear_model.LinearRegression()
lr_model.fit(X_train, y_train);
y_pred = lr_model.predict(X_test)

mse = sklearn.metrics.mean_squared_error(y_test, y_pred)
print(f"MSE: {mse:.4f}")
```

</details>

MSE: 0.0147

It is visually apparent that the model performs quite a bit better than
the GNN as there is little latency in the predicted results.

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
num_nodes = lagged_array[T-split_index:, :, :].shape[1]
num_time_steps = lagged_array[T-split_index:, :, :].shape[0]

cols = 1
rows = num_nodes

fig, axes = plt.subplots(rows, cols, figsize=(15, 20), sharex=True, sharey=True)

for node in range(num_nodes):
    all_targets = lagged_array[T-split_index+1:, node, 0]
    all_preds = lr_model.predict(lagged_array[T-split_index:-1, node, :])
    ax = axes[node]
    _ = ax.plot(all_targets, label='Actual', color='blue', linewidth=1)
    _ = ax.plot(all_preds, label='Predicted', color='orange', linewidth=1)
    _ = ax.set_title(f'Station {stations_list[node]}')
    _ = ax.grid(True)

_ = axes[0].legend(loc='upper right')

_ = fig.suptitle("LR Actual vs Predicted Over Time for Each Node", fontsize=16)
_ = fig.tight_layout(rect=[0, 0.03, 1, 0.95])
_ = plt.show()
```

</details>

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-528191597.png"
alt="Graph Neural Network Capstone Graphic, shows the predicted value vs actual value over time by node for the LR model."
caption="Figure 10"
%}

It is also clear that the maximum absolute error is 0.8 which is quite a
bit less than the 1 for the GNN model.

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
num_nodes = lagged_array[T-split_index:, :, :].shape[1]
num_time_steps = lagged_array[T-split_index:, :, :].shape[0]

cols = 1
rows = num_nodes


fig, axes = plt.subplots(rows, cols, figsize=(15, 20), sharex=True, sharey=True)
for node in range(num_nodes):
    all_targets = lagged_array[T-split_index+1:, node, 0]
    all_preds = lr_model.predict(lagged_array[T-split_index:-1, node, :])
    lr_absolute_error = np.abs(np.subtract(all_targets, all_preds))

    ax = axes[node]
    _ = ax.plot(lr_absolute_error, label='Error', color='blue', linewidth=1)
    _ = ax.set_title(f'Station {stations_list[node]}')
    _ = ax.grid(True)

_ = axes[0].legend(loc='upper right')

_ = fig.suptitle("LR Absolute Error for Each Station", fontsize=16)
_ = fig.tight_layout(rect=[0, 0.03, 1, 0.95])
_ = plt.show()
```

</details>

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-4038232185.png"
alt="Graph Neural Network Capstone Graphic, shows the absolute error over time by node for the LR model."
caption="Figure 11"
%}

#### Comparing Errors

When both models are compared against each other per station it becomes
apparent how poorly the GNN is performing on these predictive tasks.
Overall the LM shows an average low error rate as well as more
consistent results. This shows that despite the fact that the linear
model is unable to consider spatial information, in this case spatial
information may not actually be important for the applied dataset.

<details markdown="1">
<summary class="theme-accent-color" style="cursor: pointer;">Click to expand code</summary>
{: .notice--primary}

```python
num_nodes = lagged_array[T-split_index:, :, :].shape[1]
num_time_steps = lagged_array[T-split_index:, :, :].shape[0]

cols = 1
rows = num_nodes

all_preds = []
all_targets = []

gnn_model.eval();
with torch.no_grad():
    for snapshot in test_dataset:
        y_pred = gnn_model(snapshot.x, snapshot.edge_index, snapshot.edge_attr).squeeze()
        y_true = snapshot.y.squeeze()
        all_preds.append(y_pred.cpu().numpy())
        all_targets.append(y_true.cpu().numpy())

all_preds = np.stack(all_preds)
all_targets = np.stack(all_targets)

gnn_absolute_error = np.abs(np.subtract(np.array(all_preds, dtype=float), np.array(all_targets, dtype=float)))

fig, axes = plt.subplots(rows, 1, figsize=(15, 20), sharex=True, sharey=True)
for node in range(num_nodes):
    all_targets = lagged_array[T-split_index+1:, node, 0]
    all_preds = lr_model.predict(lagged_array[T-split_index:-1, node, :])
    lr_absolute_error = np.abs(np.subtract(all_targets, all_preds))
    relative_error = np.abs(gnn_absolute_error[1:, node] - lr_absolute_error)

    ax = axes[node]
    _ = ax.plot(gnn_absolute_error[:, node], label='GNN - Error', color='blue', linewidth=1)
    _ = ax.plot(lr_absolute_error, label='LR - Error', color='orange', linewidth=1)
    _ = ax.set_title(f'Station {stations_list[node]}')
    _ = ax.set_xlim(left=0, right=500)

_ = axes[0].legend(loc='upper right')

_ = fig.suptitle("GNN vs. LR Absolute Error for Each Station", fontsize=16)
_ = fig.tight_layout(rect=[0, 0.03, 1, 0.95])
_ = plt.show()
```

</details>

{%
include figure
image_path="/assets/images/UWF/IDC-6940/clipboard-1735287296.png"
alt="Graph Neural Network Capstone Graphic, shows the absolute error over time by model for each node."
caption="Figure 12"
%}

#### Key Findings and Interpretation

1.  Small Graph structures do not matter: The GNN severely
    underperformed the linear model in both MSE and absolute error
    across all stations, highlighting the power of a standard linear
    model on a small scale weather system.
2.  Temporal Context is Crucial: Both models benefitted from the use of
    28 historical time steps, suggesting that short-term temporal trends
    are strong predictors of near-future temperature.
3.  Feature Engineering Adds Value: Replacing wind direction with
    sin/cosine components improved learning stability and reduced
    directional ambiguity.
4.  Graph Structure is Important: The small scale of the graph most
    likely played a significant role in the underperformance of the GNN
    model.
5.  Static Graphs are Restrictive: Not applying and encoding and
    decoding layer to the GNN most likely limited information transfer
    between nodes, however, this may not have been important for such a
    small system.

## Conclusion

### Summary of Key Results

-   A Graph Neural Network trained on a spatiotemporal weather data may
    not outperform a standard linear regression baseline for short-term
    temperature prediction.

-   Incorporating spatial structure through graph edges enabled the
    model to learn regional weather interactions that linear models
    could not.

-   Careful data preprocessing, including imputation, scaling, and
    circular feature handling was essential to achieving strong
    performance from both models.

-   GNNs are incredibly sensitive to parameter tuning and may outperform
    if provided a much larger model structure or more careful tuning.

### Implications and Future Work

These findings demonstrate the potential of traditional models compared
to graph-based deep learning approaches, however, it is also apparent
that the reliance on aggregating stations as is done with the linear
model most likely only worked due to the close proximity of the
stations. If the dataset was instead made from all of the stations
across the US I doubt it would be possible to aggregate in a way that
still preserves spatial information. As this is the case we still
believe there is potential in the application of a graph-based deep
learning approach when it comes to large scale weather forecasting.

Future improvements could include:

-   Proper multivariate analysis
-   Large geographic area
-   Application of an encoding-decoding step
-   More advanced GNN variants
-   Exploring other imputation techniques

## References
<span id="Administration-2021"></span>
Administration, Federal Aviation. 2021. “Automated Surface Observing System.” 
<br><https://www.weather.gov/media/asos/aum-toc.pdf>.

<span id="Herzmann-2023"></span>
Herzmann, Daryl. 2023. “IEM:: Download ASOS/AWOS/Metar Data.” Iowa Environmental Mesonet.
<br><https://www.mesonet.agron.iastate.edu/request/download.phtml?network=KS_ASOS>.

<span id="Keisler-2022"></span>
Keisler, Ryan. 2022. “Forecasting Global Weather with Graph Neural Networks.”
<br><https://arxiv.org/abs/2202.07575>.

<span id="Lam-et-al.-2023"></span>
Lam, Remi, Alvaro Sanchez-Gonzalez, Matthew Willson, Peter Wirnsberger, Meire Fortunato, Ferran Alet, Suman Ravuri, et al. 2023. “GraphCast: Learning Skillful Medium-Range Global Weather Forecasting.”
<br><https://arxiv.org/abs/2212.12794>.

<span id="Li-et-al.-2018"></span>
Li, Yaguang, Rose Yu, Cyrus Shahabi, and Yan Liu. 2018. “Diffusion Convolutional Recurrent Neural Network: Data-Driven Traffic Forecasting.”
<br><https://arxiv.org/abs/1707.01926>.