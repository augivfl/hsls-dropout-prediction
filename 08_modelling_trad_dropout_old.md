# HSLS:09 Dropout Prediction — Modelling (Traditional Dropouts Only)

This notebook replicates the full modelling pipeline from notebook 06 but with a
modified target variable. Alternative completers (students who obtained a GED or
equivalent credential by F2) have been recoded from 1 to 0, leaving only traditional
dropouts as the positive class. The same train/test split indices are used for direct
comparison with the original models.

Target variable: X4EVERDROP_TRAD
- 0: Never dropped out OR obtained GED/equivalent by F2 (n=14,118, 89.3%)
- 1: Traditional dropout with no credential (n=1,700, 10.7%)


```python
%run 05_functions_for_modelling.ipynb
```


```python
import joblib
```


```python
df = pd.read_csv('hsls_clean_trad.csv')

with open('train_test_split.json') as f:
    split = json.load(f)

with open('feature_tiers.json') as f:
    feature_tiers = json.load(f)

train_idx = split['train']
test_idx  = split['test']

# Check stratification on new target variable
y_train = df.loc[train_idx, 'X4EVERDROP_TRAD']
y_test  = df.loc[test_idx,  'X4EVERDROP_TRAD']

print(f"Train dropout rate: {y_train.mean():.4f}")
print(f"Test dropout rate:  {y_test.mean():.4f}")
print(f"Train n: {len(y_train)}, dropouts: {y_train.sum()}")
print(f"Test n:  {len(y_test)},  dropouts: {y_test.sum()}")
```

    Train dropout rate: 0.1089
    Test dropout rate:  0.1018
    Train n: 12654, dropouts: 1378
    Test n:  3164,  dropouts: 322


# Tier 1 — Coarse Search


```python
tier1_search_trad, X_train_tier1_trad, y_train_tier1_trad, X_test_tier1_trad, y_test_tier1_trad = fit_models(
    features=feature_tiers['tier1'], df=df, train_idx=train_idx, test_idx=test_idx, target='X4EVERDROP_TRAD')

print("Best params:", tier1_search_trad.best_params_)
print("Best CV F1:", round(tier1_search_trad.best_score_, 4))
```

    Fitting 5 folds for each of 100 candidates, totalling 500 fits
    Best params: {'subsample': 0.7, 'n_estimators': 300, 'max_depth': 7, 'learning_rate': 0.01, 'colsample_bytree': 0.9}
    Best CV F1: 0.4127


# Tier 1 — Fine Search


```python
fine_grid_tier1_trad = define_fine_grid(tier1_search_trad.cv_results_)
tier1_fine_search_trad = fit_fine_search(fine_grid_tier1_trad, X_train_tier1_trad, y_train_tier1_trad)
```

    Score range: 0.3417 — 0.4127
    Threshold: 0.3985 (34 combinations above threshold)
    
        mean_test_score  std_test_score  param_n_estimators  param_max_depth  param_learning_rate  param_subsample  param_colsample_bytree
    88         0.412689        0.016697                 300                7                0.010              0.7                     0.9
    62         0.412491        0.012935                 300               10                0.010              0.8                     0.6
    4          0.410960        0.016187                 750                7                0.005              0.6                     0.7
    75         0.410315        0.015306                1000                6                0.005              0.6                     1.0
    84         0.409089        0.014584                1000                6                0.005              0.9                     1.0
    8          0.408161        0.016170                1000                7                0.005              1.0                     0.8
    1          0.407924        0.015636                 300                7                0.005              0.7                     0.7
    89         0.407431        0.006056                 100                8                0.050              0.8                     0.7
    20         0.407178        0.015749                 500                6                0.010              0.9                     0.7
    57         0.407057        0.018373                 750                6                0.005              0.6                     0.7
    25         0.406709        0.014345                 300                7                0.005              1.0                     0.7
    37         0.406394        0.018872                 300                7                0.005              0.8                     0.5
    86         0.405148        0.012943                1000                5                0.005              0.9                     0.7
    3          0.404503        0.010751                1000                5                0.005              0.8                     0.6
    45         0.404309        0.010447                 500                5                0.010              0.9                     0.9
    23         0.404025        0.012629                 300                7                0.005              0.9                     0.7
    85         0.403688        0.015203                 750                5                0.005              0.6                     0.8
    95         0.403272        0.014853                 500                3                0.100              0.7                     0.5
    38         0.402949        0.008644                1000                4                0.010              0.8                     0.7
    6          0.402872        0.006464                 100                7                0.005              0.9                     1.0
    47         0.402238        0.015370                 300                3                0.050              0.6                     0.7
    71         0.402074        0.011033                 100                3                0.050              0.9                     0.6
    87         0.401344        0.018498                 500                6                0.005              1.0                     0.7
    21         0.401008        0.011559                 200                3                0.050              0.8                     0.8
    27         0.400692        0.013527                 500                5                0.005              0.7                     1.0
    70         0.400342        0.014260                 300                6                0.005              0.8                     0.6
    55         0.400120        0.013214                 100                4                0.050              0.7                     0.5
    68         0.399914        0.014030                 500                4                0.010              0.9                     0.6
    44         0.399531        0.014510                 200                3                0.100              0.9                     0.7
    56         0.399384        0.014706                 200               10                0.005              0.9                     0.9
    40         0.399348        0.010808                1000                4                0.005              0.9                     0.8
    24         0.399319        0.014450                 100                6                0.010              0.8                     1.0
    19         0.398937        0.009007                 500                3                0.050              1.0                     0.7
    69         0.398715        0.010869                 300                4                0.010              0.6                     0.9
    
    param_n_estimators
    300     9
    1000    7
    500     7
    100     5
    750     3
    200     3
    Name: count, dtype: int64
    
    param_max_depth
    7     8
    6     7
    3     6
    5     5
    4     5
    10    2
    8     1
    Name: count, dtype: int64
    
    param_learning_rate
    0.005    18
    0.010     8
    0.050     6
    0.100     2
    Name: count, dtype: int64
    
    param_subsample
    0.9    11
    0.8     8
    0.6     6
    0.7     5
    1.0     4
    Name: count, dtype: int64
    
    param_colsample_bytree
    0.7    13
    0.6     5
    1.0     5
    0.9     4
    0.8     4
    0.5     3
    Name: count, dtype: int64
    
    n_estimators: [300, 500, np.int64(750), 1000]
    max_depth: [3, np.int64(4), np.int64(5), 6, 7]
    learning_rate: [0.005, 0.01, 0.05]
    subsample: [0.6, np.float64(0.7), 0.8, 0.9]
    colsample_bytree: [0.6, 0.7, np.float64(0.8), np.float64(0.9), 1.0]
    
    Total combinations: 1200
    Total fits (x5 folds): 6000
    Fitting 5 folds for each of 1200 candidates, totalling 6000 fits
          mean_test_score  std_test_score  param_colsample_bytree  param_learning_rate  param_max_depth  param_n_estimators  param_subsample
    148          0.418051        0.012235                     0.6                0.010                7                 500              0.6
    153          0.416468        0.015932                     0.6                0.010                7                 750              0.7
    556          0.416035        0.012983                     0.8                0.005                7                1000              0.6
    632          0.415379        0.016064                     0.8                0.010                7                 750              0.6
    72           0.415285        0.012251                     0.6                0.005                7                 750              0.6
    317          0.414829        0.015109                     0.7                0.005                7                1000              0.7
    1115         0.414594        0.018994                     1.0                0.010                7                 750              0.9
    553          0.414519        0.017145                     0.8                0.005                7                 750              0.7
    552          0.414518        0.010435                     0.8                0.005                7                 750              0.6
    158          0.414448        0.014924                     0.6                0.010                7                1000              0.8


# Tier 2 — Coarse Search


```python
tier2_search_trad, X_train_tier2_trad, y_train_tier2_trad, X_test_tier2_trad, y_test_tier2_trad = fit_models(
    features=feature_tiers['tier2'], df=df, train_idx=train_idx, test_idx=test_idx, target='X4EVERDROP_TRAD')

print("Best params:", tier2_search_trad.best_params_)
print("Best CV F1:", round(tier2_search_trad.best_score_, 4))
```

    Fitting 5 folds for each of 100 candidates, totalling 500 fits
    Best params: {'subsample': 0.6, 'n_estimators': 500, 'max_depth': 10, 'learning_rate': 0.01, 'colsample_bytree': 0.5}
    Best CV F1: 0.4243


# Tier 2 — Fine Search


```python
fine_grid_tier2_trad = define_fine_grid(tier2_search_trad.cv_results_)
tier2_fine_search_trad = fit_fine_search(fine_grid_tier2_trad, X_train_tier2_trad, y_train_tier2_trad)
```

    Score range: 0.3548 — 0.4243
    Threshold: 0.4104 (16 combinations above threshold)
    
        mean_test_score  std_test_score  param_n_estimators  param_max_depth  param_learning_rate  param_subsample  param_colsample_bytree
    60         0.424280        0.025344                 500               10                0.010              0.6                     0.5
    88         0.421980        0.016430                 300                7                0.010              0.7                     0.9
    56         0.421456        0.008768                 200               10                0.005              0.9                     0.9
    62         0.421189        0.029336                 300               10                0.010              0.8                     0.6
    4          0.419549        0.015173                 750                7                0.005              0.6                     0.7
    75         0.419218        0.007476                1000                6                0.005              0.6                     1.0
    8          0.417555        0.014096                1000                7                0.005              1.0                     0.8
    57         0.417045        0.007430                 750                6                0.005              0.6                     0.7
    84         0.416673        0.007886                1000                6                0.005              0.9                     1.0
    43         0.415997        0.026178                 100               15                0.010              0.6                     0.8
    38         0.415334        0.007127                1000                4                0.010              0.8                     0.7
    89         0.415014        0.023564                 100                8                0.050              0.8                     0.7
    20         0.413400        0.012701                 500                6                0.010              0.9                     0.7
    45         0.412344        0.006363                 500                5                0.010              0.9                     0.9
    86         0.411674        0.005670                1000                5                0.005              0.9                     0.7
    22         0.410788        0.013622                 200                3                0.100              0.6                     0.7
    
    param_n_estimators
    1000    5
    500     3
    300     2
    200     2
    750     2
    100     2
    Name: count, dtype: int64
    
    param_max_depth
    6     4
    10    3
    7     3
    5     2
    15    1
    4     1
    8     1
    3     1
    Name: count, dtype: int64
    
    param_learning_rate
    0.010    7
    0.005    7
    0.050    1
    0.100    1
    Name: count, dtype: int64
    
    param_subsample
    0.6    6
    0.9    5
    0.8    3
    0.7    1
    1.0    1
    Name: count, dtype: int64
    
    param_colsample_bytree
    0.7    7
    0.9    3
    1.0    2
    0.8    2
    0.5    1
    0.6    1
    Name: count, dtype: int64
    
    n_estimators: [100, 200, 300, 500, 750, 1000]
    max_depth: [6, 7, np.int64(8), 10]
    learning_rate: [0.005, 0.01, 0.05, 0.1]
    subsample: [0.6, np.float64(0.7), 0.8, 0.9]
    colsample_bytree: [0.7, 0.8, 0.9, 1.0]
    
    Total combinations: 1536
    Total fits (x5 folds): 7680
    Fitting 5 folds for each of 1536 candidates, totalling 7680 fits
          mean_test_score  std_test_score  param_colsample_bytree  param_learning_rate  param_max_depth  param_n_estimators  param_subsample
    563          0.435295        0.022931                     0.8                0.010               10                 300              0.9
    940          0.432613        0.019684                     0.9                0.010               10                 200              0.6
    81           0.432347        0.018187                     0.7                0.005               10                 300              0.7
    1228         0.431880        0.011211                     1.0                0.005               10                 200              0.6
    944          0.431822        0.024216                     0.9                0.010               10                 300              0.6
    465          0.431773        0.017403                     0.8                0.005               10                 300              0.7
    84           0.431300        0.023713                     0.7                0.005               10                 500              0.6
    553          0.431014        0.018229                     0.8                0.010               10                 100              0.7
    559          0.430956        0.012850                     0.8                0.010               10                 200              0.9
    1292         0.430707        0.023261                     1.0                0.010                7                1000              0.6


# Tier 3 — Coarse Search


```python
tier3_search_trad, X_train_tier3_trad, y_train_tier3_trad, X_test_tier3_trad, y_test_tier3_trad = fit_models(
    features=feature_tiers['tier3'], df=df, train_idx=train_idx, test_idx=test_idx, target='X4EVERDROP_TRAD')

print("Best params:", tier3_search_trad.best_params_)
print("Best CV F1:", round(tier3_search_trad.best_score_, 4))
```

    Fitting 5 folds for each of 100 candidates, totalling 500 fits
    Best params: {'subsample': 0.6, 'n_estimators': 750, 'max_depth': 7, 'learning_rate': 0.005, 'colsample_bytree': 0.7}
    Best CV F1: 0.4452


# Tier 3 — Fine Search


```python
fine_grid_tier3_trad = define_fine_grid(tier3_search_trad.cv_results_)
tier3_fine_search_trad = fit_fine_search(fine_grid_tier3_trad, X_train_tier3_trad, y_train_tier3_trad)
```

    Score range: 0.3435 — 0.4452
    Threshold: 0.4248 (19 combinations above threshold)
    
        mean_test_score  std_test_score  param_n_estimators  param_max_depth  param_learning_rate  param_subsample  param_colsample_bytree
    4          0.445159        0.007797                 750                7                0.005              0.6                     0.7
    84         0.440325        0.006670                1000                6                0.005              0.9                     1.0
    88         0.440009        0.012060                 300                7                0.010              0.7                     0.9
    20         0.439342        0.006404                 500                6                0.010              0.9                     0.7
    1          0.437293        0.003132                 300                7                0.005              0.7                     0.7
    75         0.436619        0.003137                1000                6                0.005              0.6                     1.0
    57         0.435410        0.008180                 750                6                0.005              0.6                     0.7
    45         0.435028        0.008330                 500                5                0.010              0.9                     0.9
    8          0.434203        0.007846                1000                7                0.005              1.0                     0.8
    38         0.432942        0.008031                1000                4                0.010              0.8                     0.7
    86         0.432497        0.005983                1000                5                0.005              0.9                     0.7
    23         0.432443        0.005818                 300                7                0.005              0.9                     0.7
    85         0.431451        0.004750                 750                5                0.005              0.6                     0.8
    3          0.429856        0.005724                1000                5                0.005              0.8                     0.6
    87         0.428447        0.011036                 500                6                0.005              1.0                     0.7
    48         0.427707        0.007912                1000                4                0.010              1.0                     0.7
    37         0.427282        0.005407                 300                7                0.005              0.8                     0.5
    25         0.427034        0.012261                 300                7                0.005              1.0                     0.7
    56         0.424861        0.016618                 200               10                0.005              0.9                     0.9
    
    param_n_estimators
    1000    7
    300     5
    750     3
    500     3
    200     1
    Name: count, dtype: int64
    
    param_max_depth
    7     7
    6     5
    5     4
    4     2
    10    1
    Name: count, dtype: int64
    
    param_learning_rate
    0.005    14
    0.010     5
    Name: count, dtype: int64
    
    param_subsample
    0.9    6
    0.6    4
    1.0    4
    0.8    3
    0.7    2
    Name: count, dtype: int64
    
    param_colsample_bytree
    0.7    10
    0.9     3
    1.0     2
    0.8     2
    0.6     1
    0.5     1
    Name: count, dtype: int64
    
    n_estimators: [300, 500, 750, 1000]
    max_depth: [5, 6, 7]
    learning_rate: [0.005, 0.01]
    subsample: [0.6, np.float64(0.7), np.float64(0.8), 0.9, 1.0]
    colsample_bytree: [0.7, 0.8, 0.9, 1.0]
    
    Total combinations: 480
    Total fits (x5 folds): 2400
    Fitting 5 folds for each of 480 candidates, totalling 2400 fits
         mean_test_score  std_test_score  param_colsample_bytree  param_learning_rate  param_max_depth  param_n_estimators  param_subsample
    465         0.450181        0.010754                     1.0                0.010                7                 500              0.6
    436         0.446105        0.007778                     1.0                0.010                5                1000              0.7
    292         0.445168        0.006887                     0.9                0.005                7                 750              0.8
    50          0.445159        0.007797                     0.7                0.005                7                 750              0.6
    90          0.444622        0.010936                     0.7                0.010                6                 750              0.6
    396         0.444507        0.002383                     1.0                0.005                6                1000              0.7
    340         0.444480        0.004242                     0.9                0.010                7                 300              0.6
    316         0.443797        0.012936                     0.9                0.010                5                1000              0.7
    415         0.443271        0.014465                     1.0                0.005                7                1000              0.6
    100         0.443087        0.005000                     0.7                0.010                7                 300              0.6


# Tier 4 — Coarse Search


```python
tier4_search_trad, X_train_tier4_trad, y_train_tier4_trad, X_test_tier4_trad, y_test_tier4_trad = fit_models(
    features=feature_tiers['tier4'], df=df, train_idx=train_idx, test_idx=test_idx, target='X4EVERDROP_TRAD')

print("Best params:", tier4_search_trad.best_params_)
print("Best CV F1:", round(tier4_search_trad.best_score_, 4))
```

    Fitting 5 folds for each of 100 candidates, totalling 500 fits


    /opt/anaconda3/envs/hsls_env/lib/python3.11/site-packages/joblib/externals/loky/process_executor.py:782: UserWarning: A worker stopped while some jobs were given to the executor. This can be caused by a too short worker timeout or by a memory leak.
      warnings.warn(


    Best params: {'subsample': 0.6, 'n_estimators': 750, 'max_depth': 7, 'learning_rate': 0.005, 'colsample_bytree': 0.7}
    Best CV F1: 0.4549


# Tier 4 — Fine Search


```python
fine_grid_tier4_trad = define_fine_grid(tier4_search_trad.cv_results_)
tier4_fine_search_trad = fit_fine_search(fine_grid_tier4_trad, X_train_tier4_trad, y_train_tier4_trad)
```

    Score range: 0.3530 — 0.4549
    Threshold: 0.4345 (23 combinations above threshold)
    
        mean_test_score  std_test_score  param_n_estimators  param_max_depth  param_learning_rate  param_subsample  param_colsample_bytree
    4          0.454872        0.006276                 750                7                0.005              0.6                     0.7
    75         0.448806        0.011344                1000                6                0.005              0.6                     1.0
    84         0.448572        0.008977                1000                6                0.005              0.9                     1.0
    20         0.447912        0.005244                 500                6                0.010              0.9                     0.7
    57         0.447076        0.009508                 750                6                0.005              0.6                     0.7
    89         0.446383        0.013630                 100                8                0.050              0.8                     0.7
    62         0.442947        0.027435                 300               10                0.010              0.8                     0.6
    88         0.442363        0.004424                 300                7                0.010              0.7                     0.9
    45         0.442121        0.007793                 500                5                0.010              0.9                     0.9
    8          0.441723        0.008037                1000                7                0.005              1.0                     0.8
    19         0.441186        0.007582                 500                3                0.050              1.0                     0.7
    44         0.440774        0.012843                 200                3                0.100              0.9                     0.7
    63         0.439203        0.012884                 300                4                0.100              1.0                     0.5
    23         0.439168        0.007764                 300                7                0.005              0.9                     0.7
    38         0.438715        0.006727                1000                4                0.010              0.8                     0.7
    48         0.438591        0.008649                1000                4                0.010              1.0                     0.7
    56         0.438391        0.014536                 200               10                0.005              0.9                     0.9
    95         0.438214        0.015497                 500                3                0.100              0.7                     0.5
    86         0.438188        0.006600                1000                5                0.005              0.9                     0.7
    1          0.437637        0.007567                 300                7                0.005              0.7                     0.7
    47         0.436609        0.016087                 300                3                0.050              0.6                     0.7
    3          0.435830        0.005575                1000                5                0.005              0.8                     0.6
    21         0.434584        0.008495                 200                3                0.050              0.8                     0.8
    
    param_n_estimators
    1000    7
    300     6
    500     4
    200     3
    750     2
    100     1
    Name: count, dtype: int64
    
    param_max_depth
    7     5
    3     5
    6     4
    5     3
    4     3
    10    2
    8     1
    Name: count, dtype: int64
    
    param_learning_rate
    0.005    10
    0.010     6
    0.050     4
    0.100     3
    Name: count, dtype: int64
    
    param_subsample
    0.9    7
    0.8    5
    0.6    4
    1.0    4
    0.7    3
    Name: count, dtype: int64
    
    param_colsample_bytree
    0.7    12
    0.9     3
    1.0     2
    0.6     2
    0.8     2
    0.5     2
    Name: count, dtype: int64
    
    n_estimators: [300, 500, np.int64(750), 1000]
    max_depth: [3, np.int64(4), np.int64(5), 6, 7]
    learning_rate: [0.005, 0.01, 0.05]
    subsample: [0.6, np.float64(0.7), 0.8, 0.9, 1.0]
    colsample_bytree: [0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
    
    Total combinations: 1800
    Total fits (x5 folds): 9000
    Fitting 5 folds for each of 1800 candidates, totalling 9000 fits


    /opt/anaconda3/envs/hsls_env/lib/python3.11/site-packages/joblib/externals/loky/process_executor.py:782: UserWarning: A worker stopped while some jobs were given to the executor. This can be caused by a too short worker timeout or by a memory leak.
      warnings.warn(


          mean_test_score  std_test_score  param_colsample_bytree  param_learning_rate  param_max_depth  param_n_estimators  param_subsample
    1070         0.465375        0.017424                     0.8                0.010                6                 750              0.6
    771          0.463224        0.006105                     0.7                0.010                6                 750              0.7
    696          0.460782        0.013717                     0.7                0.005                7                1000              0.7
    471          0.459894        0.011612                     0.6                0.010                6                 750              0.7
    1596         0.459537        0.008674                     1.0                0.005                7                1000              0.7
    172          0.459491        0.009273                     0.5                0.010                6                 750              0.8
    485          0.459455        0.009323                     0.6                0.010                7                 500              0.6
    1387         0.459011        0.006111                     0.9                0.010                7                 500              0.8
    1086         0.458834        0.008831                     0.8                0.010                7                 500              0.7
    170          0.458745        0.011787                     0.5                0.010                6                 750              0.6



```python
evaluate_model(tier1_fine_search_trad, X_train_tier1_trad, y_train_tier1_trad, X_test_tier1_trad, y_test_tier1_trad, tier=1)
evaluate_model(tier2_fine_search_trad, X_train_tier2_trad, y_train_tier2_trad, X_test_tier2_trad, y_test_tier2_trad, tier=2)
evaluate_model(tier3_fine_search_trad, X_train_tier3_trad, y_train_tier3_trad, X_test_tier3_trad, y_test_tier3_trad, tier=3)
evaluate_model(tier4_fine_search_trad, X_train_tier4_trad, y_train_tier4_trad, X_test_tier4_trad, y_test_tier4_trad, tier=4)
```

    Tier 1 — Training Set Evaluation
    Accuracy:  0.8863
    ROC-AUC:   0.9624
    
                  precision    recall  f1-score   support
    
               0       0.99      0.88      0.93     11276
               1       0.49      0.90      0.63      1378
    
        accuracy                           0.89     12654
       macro avg       0.74      0.89      0.78     12654
    weighted avg       0.93      0.89      0.90     12654
    
    Tier 1 — Test Set Evaluation
    Accuracy:  0.8259
    ROC-AUC:   0.8260
    
                  precision    recall  f1-score   support
    
               0       0.95      0.85      0.90      2842
               1       0.32      0.61      0.42       322
    
        accuracy                           0.83      3164
       macro avg       0.63      0.73      0.66      3164
    weighted avg       0.89      0.83      0.85      3164
    
    Tier 2 — Training Set Evaluation
    Accuracy:  0.9580
    ROC-AUC:   0.9979
    
                  precision    recall  f1-score   support
    
               0       1.00      0.95      0.98     11276
               1       0.72      0.99      0.84      1378
    
        accuracy                           0.96     12654
       macro avg       0.86      0.97      0.91     12654
    weighted avg       0.97      0.96      0.96     12654
    
    Tier 2 — Test Set Evaluation
    Accuracy:  0.8692
    ROC-AUC:   0.8238
    
                  precision    recall  f1-score   support
    
               0       0.94      0.91      0.93      2842
               1       0.39      0.51      0.44       322
    
        accuracy                           0.87      3164
       macro avg       0.67      0.71      0.68      3164
    weighted avg       0.89      0.87      0.88      3164
    
    Tier 3 — Training Set Evaluation
    Accuracy:  0.9399
    ROC-AUC:   0.9909
    
                  precision    recall  f1-score   support
    
               0       1.00      0.94      0.97     11276
               1       0.65      0.96      0.78      1378
    
        accuracy                           0.94     12654
       macro avg       0.82      0.95      0.87     12654
    weighted avg       0.96      0.94      0.94     12654
    
    Tier 3 — Test Set Evaluation
    Accuracy:  0.8594
    ROC-AUC:   0.8346
    
                  precision    recall  f1-score   support
    
               0       0.95      0.89      0.92      2842
               1       0.37      0.56      0.45       322
    
        accuracy                           0.86      3164
       macro avg       0.66      0.72      0.68      3164
    weighted avg       0.89      0.86      0.87      3164
    
    Tier 4 — Training Set Evaluation
    Accuracy:  0.9397
    ROC-AUC:   0.9933
    
                  precision    recall  f1-score   support
    
               0       1.00      0.93      0.97     11276
               1       0.65      0.98      0.78      1378
    
        accuracy                           0.94     12654
       macro avg       0.82      0.96      0.87     12654
    weighted avg       0.96      0.94      0.94     12654
    
    Tier 4 — Test Set Evaluation
    Accuracy:  0.8638
    ROC-AUC:   0.8422
    
                  precision    recall  f1-score   support
    
               0       0.95      0.89      0.92      2842
               1       0.39      0.60      0.47       322
    
        accuracy                           0.86      3164
       macro avg       0.67      0.75      0.70      3164
    weighted avg       0.89      0.86      0.88      3164
    



```python

```


```python
from sklearn.metrics import f1_score, recall_score, precision_score, roc_auc_score, accuracy_score

summary_trad = []

for tier, fine_search, X_train, y_train, X_test, y_test in [
    (1, tier1_fine_search_trad, X_train_tier1_trad, y_train_tier1_trad, X_test_tier1_trad, y_test_tier1_trad),
    (2, tier2_fine_search_trad, X_train_tier2_trad, y_train_tier2_trad, X_test_tier2_trad, y_test_tier2_trad),
    (3, tier3_fine_search_trad, X_train_tier3_trad, y_train_tier3_trad, X_test_tier3_trad, y_test_tier3_trad),
    (4, tier4_fine_search_trad, X_train_tier4_trad, y_train_tier4_trad, X_test_tier4_trad, y_test_tier4_trad),
]:
    model = fine_search.best_estimator_
    y_pred = model.predict(X_test)
    y_prob = model.predict_proba(X_test)[:, 1]
    
    summary_trad.append({
        'Tier': tier,
        'Features': X_test.shape[1],
        'CV F1': round(fine_search.best_score_, 4),
        'Test Accuracy': round(accuracy_score(y_test, y_pred), 4),
        'Test Precision': round(precision_score(y_test, y_pred), 4),
        'Test Recall': round(recall_score(y_test, y_pred), 4),
        'Test F1': round(f1_score(y_test, y_pred), 4),
        'Test ROC-AUC': round(roc_auc_score(y_test, y_prob), 4),
    })

summary_trad_df = pd.DataFrame(summary_trad)
print(summary_trad_df.to_string(index=False))
```

     Tier  Features  CV F1  Test Accuracy  Test Precision  Test Recall  Test F1  Test ROC-AUC
        1       130 0.4181         0.8259          0.3168       0.6149   0.4182        0.8260
        2       192 0.4353         0.8692          0.3905       0.5093   0.4420        0.8238
        3       436 0.4502         0.8594          0.3721       0.5559   0.4458        0.8346
        4       833 0.4654         0.8638          0.3895       0.5963   0.4712        0.8422



```python
show_feature_importance(tier1_fine_search_trad, X_train_tier1_trad, tier=1)
show_feature_importance(tier2_fine_search_trad, X_train_tier2_trad, tier=2)
show_feature_importance(tier3_fine_search_trad, X_train_tier3_trad, tier=3)
show_feature_importance(tier4_fine_search_trad, X_train_tier4_trad, tier=4)
```


    
![png](08_modelling_trad_dropout_files/08_modelling_trad_dropout_23_0.png)
    


    Tier 1 — Bottom 20 features:
    S1SFALL09      0.005299
    X1SEX          0.005287
    P1USGRADE      0.005264
    X1RACE         0.005116
    P1DD           0.005093
    P1USBORN9      0.005074
    P1REPEATG5     0.004989
    X1WHITE        0.004962
    P1REPEATG1     0.004814
    A1G9OTHER      0.004577
    S1ANGEOM09     0.004500
    P1REPEATGK     0.004301
    P1REPEATG3     0.004301
    S1MFALL09      0.004268
    A1G9TCHREF     0.004059
    S1REVM09       0.003595
    S1OTHBIOS09    0.003540
    P1AUTISM       0.003538
    P1REPEATG9     0.002994
    P1REPEATG4     0.002250
    



    
![png](08_modelling_trad_dropout_files/08_modelling_trad_dropout_23_2.png)
    


    Tier 2 — Bottom 20 features:
    P1REPEATGK         0.002955
    X1WHITE            0.002938
    X1SEX              0.002937
    X1PAR1OCC_STEM1    0.002884
    P1USGRADE          0.002775
    X1RACE             0.002700
    X1PAR2OCC_STEM1    0.002692
    X1PAR1EDU          0.002689
    A1G9TCHREF         0.002681
    S1STATSM09         0.002594
    S1REVM09           0.002456
    S1ANGEOM09         0.002198
    P1JOBEVER1         0.002144
    S1OTHENVS09        0.001879
    P1SKIPG8           0.001851
    P1REPEATG4         0.001754
    P1REPEATG9         0.001647
    P1AUTISM           0.001631
    S1ADVM09           0.000000
    S1ADVPHYSIC09      0.000000
    



    
![png](08_modelling_trad_dropout_files/08_modelling_trad_dropout_23_4.png)
    


    Tier 3 — Bottom 20 features:
    S1SFALL09        0.001055
    C1POORGRADES     0.001047
    S1MTUTOR         0.001026
    X1SEX            0.001017
    X1PAR1EDU        0.000978
    P1REPEATG5       0.000968
    A1G9TCHREF       0.000901
    S1MCLUB          0.000874
    X1HISPANIC       0.000867
    P1JOBEVER1       0.000839
    S1ANGEOM09       0.000803
    S1OTHENVS09      0.000639
    S1REVM09         0.000609
    P1REPEATG9       0.000569
    S1FEEPRV         0.000562
    S1GENS09         0.000562
    P1REPEATG4       0.000425
    S1ADVBIOS09      0.000372
    P1JOBEVER2       0.000000
    S1ADVPHYSIC09    0.000000
    



    
![png](08_modelling_trad_dropout_files/08_modelling_trad_dropout_23_6.png)
    


    Tier 4 — Bottom 20 features:
    A1G9TCHREF       0.000407
    S1REVM09         0.000398
    S1GENS09         0.000391
    A1G9GRADES       0.000386
    X1PAR1EDU        0.000378
    P1ADHDMED        0.000376
    X1DUALLANG       0.000352
    A1NOTIFY         0.000332
    S1MFALL09        0.000306
    P1REPEATG1       0.000299
    P1SKIPG8         0.000290
    P1REPEATG9       0.000229
    P1REPEATG4       0.000147
    C1HSTOWORKNO     0.000098
    C1POSTSECREQ     0.000077
    X1TXMQUINT       0.000000
    S1ADVPHYSIC09    0.000000
    S1ANGEOM09       0.000000
    S1OTHPHYS09      0.000000
    S1OTHENVS09      0.000000
    



```python
compare_feature_importance(
    [tier1_fine_search_trad, tier2_fine_search_trad, tier3_fine_search_trad, tier4_fine_search_trad],
    [X_train_tier1_trad, X_train_tier2_trad, X_train_tier3_trad, X_train_tier4_trad])
```

    Feature overlap across tiers (top 20 each):
    Feature                     T1   T2   T3   T4  Tiers
    --------------------------------------------------
    S1S8GRADE                    3    4    5   11      4
    S1M8GRADE                    5    5    6   16      4
    X1CONTROL                    6    8   13   15      4
    P1SUSPEND                    2    2    3   13      4
    P1DROPOUT                    1    1    2    8      4
    X1FAMINCOME                  -    3    4   12      3
    P1HONORS                     7   13   14    -      3
    X1POVERTY185                 -    6    9   10      3
    P1REPEATGRD                  4    9   20    -      3
    P1SKIPG2                    13   14    -    -      2
    X1SES_U                      -    7   16    -      2
    X1STDOB                     10   12    -    -      2
    P1REPEATG7                   8    -   17    -      2
    X1SESQ5_U                    -   18    8    -      2
    P1ADHD                       -   20    -   18      2
    P1SKIPGRD                    9    -    1    -      2
    S1SUREHSGRAD                 -    -   10   20      2
    X1PAREDU                     -   15   12    -      2
    
    Average rank across tiers (only where in top 20):
    Feature                     Avg Rank  Tiers
    ---------------------------------------------
    P1DROPOUT                       3.00      4
    P1SUSPEND                       5.00      4
    S1S8GRADE                       5.75      4
    S1M8GRADE                       8.00      4
    X1CONTROL                      10.50      4
    X1FAMINCOME                     6.33      3
    X1POVERTY185                    8.33      3
    P1REPEATGRD                    11.00      3
    P1HONORS                       11.33      3
    P1SKIPGRD                       5.00      2
    X1STDOB                        11.00      2
    X1SES_U                        11.50      2
    P1REPEATG7                     12.50      2
    X1SESQ5_U                      13.00      2
    P1SKIPG2                       13.50      2
    X1PAREDU                       13.50      2
    S1SUREHSGRAD                   15.00      2
    P1ADHD                         19.00      2



```python
# Load original tier 4 data and model
X_train_t4, y_train_t4, X_test_t4, y_test_t4 = joblib.load('data_tier4.pkl')
tier4_orig = joblib.load('tier4_fine_search.pkl')

# Get predicted probabilities from original model on test set
model_orig = tier4_orig.best_estimator_
y_prob_orig = model_orig.predict_proba(X_test_t4)[:, 1]

# Get GED status for test set students
ged_test = df.loc[test_idx, 'X4EVERGED'].values
everdrop_test = df.loc[test_idx, 'X4EVERDROP'].values

results = pd.DataFrame({
    'true_dropout': everdrop_test,
    'ged': ged_test,
    'prob': y_prob_orig
})

print("Mean predicted dropout probability by group:")
print(results.groupby(['true_dropout', 'ged'])['prob'].describe())
```

    Mean predicted dropout probability by group:
                       count      mean       std       min       25%       50%  \
    true_dropout ged                                                             
    0            0    2692.0  0.208651  0.221299  0.000941  0.041710  0.114164   
    1            0     322.0  0.630549  0.303392  0.003889  0.368592  0.669620   
                 1     150.0  0.625850  0.268010  0.008428  0.434570  0.668643   
    
                           75%       max  
    true_dropout ged                      
    0            0    0.309971  0.983841  
    1            0    0.903373  0.999902  
                 1    0.864764  0.999868  



```python
# Compare S1EDUEXPECT distribution across the three groups
df_analysis = df.copy()
df_analysis['group'] = 'non-dropout'
df_analysis.loc[(df_analysis['X4EVERDROP'] == 1) & (df_analysis['X4EVERGED'] == 0), 'group'] = 'traditional_dropout'
df_analysis.loc[(df_analysis['X4EVERDROP'] == 1) & (df_analysis['X4EVERGED'] == 1), 'group'] = 'ged_completer'

print(pd.crosstab(df_analysis['group'], df_analysis['S1EDUEXPECT'], normalize='index').round(3))
```

    S1EDUEXPECT           1.0    2.0    3.0    4.0    5.0    6.0    7.0    8.0   \
    group                                                                         
    ged_completer        0.012  0.257  0.006  0.077  0.006  0.127  0.012  0.126   
    non-dropout          0.002  0.083  0.005  0.049  0.005  0.175  0.012  0.222   
    traditional_dropout  0.014  0.241  0.014  0.072  0.006  0.118  0.011  0.132   
    
    S1EDUEXPECT           9.0    10.0   11.0  
    group                                     
    ged_completer        0.006  0.131  0.240  
    non-dropout          0.009  0.236  0.202  
    traditional_dropout  0.004  0.140  0.247  



```python
import shap

model = tier4_fine_search_trad.best_estimator_
explainer = shap.TreeExplainer(model)

# Get SHAP values for test set
shap_values = explainer.shap_values(X_test_tier4_trad)

# Find S1EDUEXPECT index
feature_names = X_test_tier4_trad.columns.tolist()
if 'S1EDUEXPECT' in feature_names:
    idx = feature_names.index('S1EDUEXPECT')
    
    # Plot dependence plot for S1EDUEXPECT
    shap.dependence_plot('S1EDUEXPECT', shap_values, X_test_tier4_trad, show=True)
else:
    print("S1EDUEXPECT not in Tier 4 features")
```


    
![png](08_modelling_trad_dropout_files/08_modelling_trad_dropout_27_0.png)
    



```python
print(pd.crosstab(
    df_analysis.loc[df_analysis['S1SUREHSGRAD'] > 0, 'group'],
    df_analysis.loc[df_analysis['S1SUREHSGRAD'] > 0, 'S1SUREHSGRAD'],
    normalize='index'
).round(3))
```

    S1SUREHSGRAD           1.0    2.0    3.0    4.0
    group                                          
    ged_completer        0.668  0.284  0.042  0.006
    non-dropout          0.894  0.100  0.005  0.002
    traditional_dropout  0.677  0.273  0.035  0.015



```python
print(pd.crosstab(
    df_analysis['group'],
    df_analysis['P1DROPOUT'],
    normalize='index'
).round(3))
```

    P1DROPOUT              0.0    1.0
    group                            
    ged_completer        0.889  0.111
    non-dropout          1.000  0.000
    traditional_dropout  0.752  0.248



```python
print(pd.crosstab(
    df_analysis.loc[df_analysis['P1DROPOUT'].isin([0, 1]), 'group'],
    df_analysis.loc[df_analysis['P1DROPOUT'].isin([0, 1]), 'P1DROPOUT'],
    normalize='index'
).round(3))
```

    P1DROPOUT              0.0    1.0
    group                            
    ged_completer        0.889  0.111
    non-dropout          1.000  0.000
    traditional_dropout  0.752  0.248



```python
from xgboost import plot_tree
import matplotlib.pyplot as plt

model = tier1_fine_search_trad.best_estimator_

fig, ax = plt.subplots(figsize=(40, 20))
plot_tree(model, num_trees=0, ax=ax)
plt.tight_layout()
plt.show()
```

    /opt/anaconda3/envs/hsls_env/lib/python3.11/site-packages/xgboost/plotting.py:268: FutureWarning: The `num_trees` parameter is deprecated, use `tree_idx` insetad. 
      warnings.warn(



    
![png](08_modelling_trad_dropout_files/08_modelling_trad_dropout_31_1.png)
    



```python

```
