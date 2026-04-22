```python
%run 05_functions_for_modelling.ipynb
```


```python
df = pd.read_csv('hsls_clean.csv')

with open('train_test_split.json') as f:
    split = json.load(f)

with open('feature_tiers.json') as f:
    feature_tiers = json.load(f)

train_idx = split['train']
test_idx  = split['test']
```

# Tier 1 — Coarse Search


```python
tier1_search, X_train_tier1, y_train_tier1, X_test_tier1, y_test_tier1 = fit_models(
    features=feature_tiers['tier1'], df=df, train_idx=train_idx, test_idx=test_idx)

print("Best params:", tier1_search.best_params_)
print("Best CV F1:", round(tier1_search.best_score_, 4))
```

    Fitting 5 folds for each of 100 candidates, totalling 500 fits
    Best params: {'subsample': 0.6, 'n_estimators': 750, 'max_depth': 7, 'learning_rate': 0.005, 'colsample_bytree': 0.7}
    Best CV F1: 0.5137


# Tier 1 — Fine Search


```python
fine_grid_tier1 = define_fine_grid(tier1_search.cv_results_)
tier1_fine_search = fit_fine_search(fine_grid_tier1, X_train_tier1, y_train_tier1)
```

    Score range: 0.4477 — 0.5137
    Threshold: 0.5005 (16 combinations above threshold)
    
        mean_test_score  std_test_score  param_n_estimators  param_max_depth  param_learning_rate  param_subsample  param_colsample_bytree
    4          0.513746        0.008695                 750                7                0.005              0.6                     0.7
    75         0.513169        0.010957                1000                6                0.005              0.6                     1.0
    89         0.512398        0.010279                 100                8                0.050              0.8                     0.7
    88         0.509692        0.007781                 300                7                0.010              0.7                     0.9
    8          0.508961        0.010500                1000                7                0.005              1.0                     0.8
    56         0.508059        0.020297                 200               10                0.005              0.9                     0.9
    57         0.507465        0.007745                 750                6                0.005              0.6                     0.7
    84         0.507102        0.010877                1000                6                0.005              0.9                     1.0
    1          0.506886        0.008118                 300                7                0.005              0.7                     0.7
    20         0.506072        0.008188                 500                6                0.010              0.9                     0.7
    60         0.505088        0.015716                 500               10                0.010              0.6                     0.5
    62         0.504702        0.021136                 300               10                0.010              0.8                     0.6
    17         0.502987        0.010508                 200                4                0.200              0.8                     0.5
    19         0.502228        0.010934                 500                3                0.050              1.0                     0.7
    3          0.500839        0.006848                1000                5                0.005              0.8                     0.6
    23         0.500712        0.003092                 300                7                0.005              0.9                     0.7
    
    param_n_estimators
    1000    4
    300     4
    500     3
    750     2
    200     2
    100     1
    Name: count, dtype: int64
    
    param_max_depth
    7     5
    6     4
    10    3
    8     1
    4     1
    3     1
    5     1
    Name: count, dtype: int64
    
    param_learning_rate
    0.005    9
    0.010    4
    0.050    2
    0.200    1
    Name: count, dtype: int64
    
    param_subsample
    0.6    4
    0.8    4
    0.9    4
    0.7    2
    1.0    2
    Name: count, dtype: int64
    
    param_colsample_bytree
    0.7    7
    1.0    2
    0.9    2
    0.5    2
    0.6    2
    0.8    1
    Name: count, dtype: int64
    
    n_estimators: [300, 500, np.int64(750), 1000]
    max_depth: [6, 7, np.int64(8), 10]
    learning_rate: [0.005, 0.01, 0.05]
    subsample: [0.6, np.float64(0.7), 0.8, 0.9]
    colsample_bytree: [0.5, 0.6, 0.7, np.float64(0.8), 0.9, 1.0]
    
    Total combinations: 1152
    Total fits (x5 folds): 5760
    Fitting 5 folds for each of 1152 candidates, totalling 5760 fits
          mean_test_score  std_test_score  param_colsample_bytree  param_learning_rate  param_max_depth  param_n_estimators  param_subsample
    616          0.520715        0.009955                     0.8                0.005                8                 750              0.6
    433          0.520209        0.013277                     0.7                0.005               10                 300              0.7
    1005         0.519969        0.012566                     1.0                0.005                8                1000              0.7
    485          0.519922        0.015996                     0.7                0.010                8                 500              0.7
    612          0.519889        0.010203                     0.8                0.005                8                 500              0.6
    472          0.519292        0.011215                     0.7                0.010                7                 750              0.6
    604          0.519014        0.008844                     0.8                0.005                7                1000              0.6
    295          0.518366        0.015698                     0.6                0.010                8                 500              0.9
    620          0.518253        0.014229                     0.8                0.005                8                1000              0.6
    805          0.518145        0.011417                     0.9                0.005                8                 500              0.7


# Tier 2 — Coarse Search


```python
tier2_search, X_train_tier2, y_train_tier2, X_test_tier2, y_test_tier2 = fit_models(
    features=feature_tiers['tier2'], df=df, train_idx=train_idx, test_idx=test_idx)

print("Best params:", tier2_search.best_params_)
print("Best CV F1:", round(tier2_search.best_score_, 4))
```

    Fitting 5 folds for each of 100 candidates, totalling 500 fits
    Best params: {'subsample': 0.6, 'n_estimators': 750, 'max_depth': 7, 'learning_rate': 0.005, 'colsample_bytree': 0.7}
    Best CV F1: 0.5143


# Tier 2 — Fine Search


```python
fine_grid_tier2 = define_fine_grid(tier2_search.cv_results_)
tier2_fine_search = fit_fine_search(fine_grid_tier2, X_train_tier2, y_train_tier2)
```

    Score range: 0.4637 — 0.5143
    Threshold: 0.5042 (19 combinations above threshold)
    
        mean_test_score  std_test_score  param_n_estimators  param_max_depth  param_learning_rate  param_subsample  param_colsample_bytree
    4          0.514268        0.010234                 750                7                0.005              0.6                     0.7
    10         0.512903        0.014144                 200                8                0.050              1.0                     0.6
    62         0.512587        0.015379                 300               10                0.010              0.8                     0.6
    75         0.511920        0.009150                1000                6                0.005              0.6                     1.0
    60         0.511447        0.018528                 500               10                0.010              0.6                     0.5
    83         0.510475        0.011482                 100               10                0.050              1.0                     0.6
    88         0.510018        0.009028                 300                7                0.010              0.7                     0.9
    89         0.509915        0.012378                 100                8                0.050              0.8                     0.7
    57         0.509100        0.004822                 750                6                0.005              0.6                     0.7
    61         0.508561        0.019898                 750               15                0.005              0.6                     0.9
    80         0.507307        0.011555                 500                6                0.050              1.0                     0.8
    43         0.506632        0.011894                 100               15                0.010              0.6                     0.8
    63         0.506186        0.010644                 300                4                0.100              1.0                     0.5
    8          0.506161        0.009500                1000                7                0.005              1.0                     0.8
    26         0.506144        0.027332                 300                8                0.050              0.8                     0.5
    20         0.506081        0.007725                 500                6                0.010              0.9                     0.7
    96         0.505368        0.016717                 100               15                0.010              0.9                     0.8
    84         0.505209        0.007085                1000                6                0.005              0.9                     1.0
    30         0.505061        0.016407                 300                4                0.100              0.7                     0.9
    
    param_n_estimators
    300     5
    100     4
    750     3
    1000    3
    500     3
    200     1
    Name: count, dtype: int64
    
    param_max_depth
    6     5
    7     3
    8     3
    10    3
    15    3
    4     2
    Name: count, dtype: int64
    
    param_learning_rate
    0.005    6
    0.010    6
    0.050    5
    0.100    2
    Name: count, dtype: int64
    
    param_subsample
    0.6    6
    1.0    5
    0.8    3
    0.9    3
    0.7    2
    Name: count, dtype: int64
    
    param_colsample_bytree
    0.7    4
    0.8    4
    0.6    3
    0.5    3
    0.9    3
    1.0    2
    Name: count, dtype: int64
    
    n_estimators: [100, np.int64(200), 300, 500, 750, 1000]
    max_depth: [6, 7, 8, 10, 15]
    learning_rate: [0.005, 0.01, 0.05]
    subsample: [0.6, np.float64(0.7), 0.8, 0.9, 1.0]
    colsample_bytree: [0.5, 0.6, 0.7, 0.8, 0.9]
    
    Total combinations: 2250
    Total fits (x5 folds): 11250
    Fitting 5 folds for each of 2250 candidates, totalling 11250 fits
          mean_test_score  std_test_score  param_colsample_bytree  param_learning_rate  param_max_depth  param_n_estimators  param_subsample
    1555         0.522560        0.012692                     0.8                0.010                7                1000              0.6
    2001         0.522469        0.012738                     0.9                0.010                7                 750              0.7
    2006         0.521461        0.014119                     0.9                0.010                7                1000              0.7
    1550         0.521394        0.009529                     0.8                0.010                7                 750              0.6
    1107         0.521242        0.008493                     0.7                0.010                7                1000              0.8
    1686         0.521224        0.007630                     0.8                0.050                7                 200              0.7
    536          0.520849        0.009874                     0.6                0.005                8                1000              0.7
    2000         0.520662        0.011020                     0.9                0.010                7                 750              0.6
    2026         0.520554        0.010125                     0.9                0.010                8                 500              0.7
    206          0.520201        0.017469                     0.5                0.010                7                1000              0.7


# Tier 3 — Coarse Search


```python
tier3_search, X_train_tier3, y_train_tier3, X_test_tier3, y_test_tier3 = fit_models(
    features=feature_tiers['tier3'], df=df, train_idx=train_idx, test_idx=test_idx)

print("Best params:", tier3_search.best_params_)
print("Best CV F1:", round(tier3_search.best_score_, 4))
```

    Fitting 5 folds for each of 100 candidates, totalling 500 fits
    Best params: {'subsample': 0.6, 'n_estimators': 750, 'max_depth': 7, 'learning_rate': 0.005, 'colsample_bytree': 0.7}
    Best CV F1: 0.5354


# Tier 3 — Fine Search


```python
fine_grid_tier3 = define_fine_grid(tier3_search.cv_results_)
tier3_fine_search = fit_fine_search(fine_grid_tier3, X_train_tier3, y_train_tier3)
```

    Score range: 0.4664 — 0.5354
    Threshold: 0.5216 (25 combinations above threshold)
    
        mean_test_score  std_test_score  param_n_estimators  param_max_depth  param_learning_rate  param_subsample  param_colsample_bytree
    4          0.535370        0.016904                 750                7                0.005              0.6                     0.7
    8          0.535158        0.014687                1000                7                0.005              1.0                     0.8
    84         0.533916        0.013416                1000                6                0.005              0.9                     1.0
    75         0.533444        0.012413                1000                6                0.005              0.6                     1.0
    20         0.531084        0.016313                 500                6                0.010              0.9                     0.7
    88         0.529325        0.009452                 300                7                0.010              0.7                     0.9
    57         0.529109        0.008531                 750                6                0.005              0.6                     0.7
    19         0.528879        0.007645                 500                3                0.050              1.0                     0.7
    89         0.528746        0.015896                 100                8                0.050              0.8                     0.7
    38         0.526790        0.012185                1000                4                0.010              0.8                     0.7
    10         0.526642        0.010993                 200                8                0.050              1.0                     0.6
    3          0.526334        0.011054                1000                5                0.005              0.8                     0.6
    23         0.525616        0.010337                 300                7                0.005              0.9                     0.7
    63         0.525459        0.005006                 300                4                0.100              1.0                     0.5
    45         0.525225        0.008970                 500                5                0.010              0.9                     0.9
    47         0.525125        0.008237                 300                3                0.050              0.6                     0.7
    1          0.524867        0.010324                 300                7                0.005              0.7                     0.7
    30         0.524609        0.006447                 300                4                0.100              0.7                     0.9
    48         0.524270        0.011692                1000                4                0.010              1.0                     0.7
    95         0.524030        0.005819                 500                3                0.100              0.7                     0.5
    62         0.523427        0.007498                 300               10                0.010              0.8                     0.6
    86         0.523369        0.009479                1000                5                0.005              0.9                     0.7
    56         0.522524        0.014219                 200               10                0.005              0.9                     0.9
    37         0.521697        0.007131                 300                7                0.005              0.8                     0.5
    85         0.521695        0.006463                 750                5                0.005              0.6                     0.8
    
    param_n_estimators
    300     8
    1000    7
    500     4
    750     3
    200     2
    100     1
    Name: count, dtype: int64
    
    param_max_depth
    7     6
    6     4
    4     4
    5     4
    3     3
    8     2
    10    2
    Name: count, dtype: int64
    
    param_learning_rate
    0.005    12
    0.010     6
    0.050     4
    0.100     3
    Name: count, dtype: int64
    
    param_subsample
    0.9    6
    0.6    5
    1.0    5
    0.8    5
    0.7    4
    Name: count, dtype: int64
    
    param_colsample_bytree
    0.7    11
    0.9     4
    0.6     3
    0.5     3
    0.8     2
    1.0     2
    Name: count, dtype: int64
    
    n_estimators: [300, 500, np.int64(750), 1000]
    max_depth: [4, 5, 6, 7]
    learning_rate: [0.005, 0.01, 0.05]
    subsample: [0.6, np.float64(0.7), 0.8, 0.9, 1.0]
    colsample_bytree: [0.5, 0.6, 0.7, np.float64(0.8), 0.9]
    
    Total combinations: 1200
    Total fits (x5 folds): 6000
    Fitting 5 folds for each of 1200 candidates, totalling 6000 fits
          mean_test_score  std_test_score  param_colsample_bytree  param_learning_rate  param_max_depth  param_n_estimators  param_subsample
    558          0.545758        0.017482                     0.7                0.005                7                1000              0.9
    391          0.543683        0.015217                     0.6                0.010                7                 750              0.7
    628          0.543159        0.014427                     0.7                0.010                7                 500              0.9
    373          0.542831        0.013774                     0.6                0.010                6                 750              0.9
    1098         0.542555        0.013077                     0.9                0.010                6                1000              0.9
    868          0.542275        0.013542                     0.8                0.010                7                 500              0.9
    1108         0.541844        0.013165                     0.9                0.010                7                 500              0.9
    377          0.541771        0.011422                     0.6                0.010                6                1000              0.8
    153          0.541738        0.010294                     0.5                0.010                7                 750              0.9
    315          0.541735        0.017409                     0.6                0.005                7                1000              0.6


# Tier 4 — Coarse Search


```python
tier4_search, X_train_tier4, y_train_tier4, X_test_tier4, y_test_tier4 = fit_models(
    features=feature_tiers['tier4'], df=df, train_idx=train_idx, test_idx=test_idx)

print("Best params:", tier4_search.best_params_)
print("Best CV F1:", round(tier4_search.best_score_, 4))
```

    Fitting 5 folds for each of 100 candidates, totalling 500 fits


    /opt/anaconda3/envs/hsls_env/lib/python3.11/site-packages/joblib/externals/loky/process_executor.py:782: UserWarning: A worker stopped while some jobs were given to the executor. This can be caused by a too short worker timeout or by a memory leak.
      warnings.warn(


    Best params: {'subsample': 1.0, 'n_estimators': 1000, 'max_depth': 7, 'learning_rate': 0.005, 'colsample_bytree': 0.8}
    Best CV F1: 0.5451


# Tier 4 — Fine Search


```python
fine_grid_tier4 = define_fine_grid(tier4_search.cv_results_)
tier4_fine_search = fit_fine_search(fine_grid_tier4, X_train_tier4, y_train_tier4)
```

    Score range: 0.4661 — 0.5451
    Threshold: 0.5293 (26 combinations above threshold)
    
        mean_test_score  std_test_score  param_n_estimators  param_max_depth  param_learning_rate  param_subsample  param_colsample_bytree
    8          0.545095        0.020159                1000                7                0.005              1.0                     0.8
    89         0.544589        0.013746                 100                8                0.050              0.8                     0.7
    63         0.543773        0.004907                 300                4                0.100              1.0                     0.5
    75         0.543768        0.011398                1000                6                0.005              0.6                     1.0
    4          0.543401        0.010958                 750                7                0.005              0.6                     0.7
    88         0.541052        0.010152                 300                7                0.010              0.7                     0.9
    62         0.540580        0.014545                 300               10                0.010              0.8                     0.6
    44         0.539001        0.013693                 200                3                0.100              0.9                     0.7
    84         0.538332        0.013023                1000                6                0.005              0.9                     1.0
    38         0.537774        0.010173                1000                4                0.010              0.8                     0.7
    20         0.537474        0.009760                 500                6                0.010              0.9                     0.7
    57         0.536979        0.012813                 750                6                0.005              0.6                     0.7
    56         0.536934        0.017798                 200               10                0.005              0.9                     0.9
    19         0.536700        0.011059                 500                3                0.050              1.0                     0.7
    48         0.535160        0.014177                1000                4                0.010              1.0                     0.7
    13         0.535011        0.009471                1000                4                0.050              1.0                     0.7
    3          0.533873        0.009743                1000                5                0.005              0.8                     0.6
    82         0.532832        0.015689                 300                5                0.100              1.0                     0.9
    95         0.532777        0.011132                 500                3                0.100              0.7                     0.5
    22         0.532461        0.009048                 200                3                0.100              0.6                     0.7
    30         0.532379        0.011587                 300                4                0.100              0.7                     0.9
    86         0.532263        0.010146                1000                5                0.005              0.9                     0.7
    1          0.532067        0.008260                 300                7                0.005              0.7                     0.7
    45         0.531831        0.007883                 500                5                0.010              0.9                     0.9
    85         0.531416        0.010731                 750                5                0.005              0.6                     0.8
    21         0.529905        0.006026                 200                3                0.050              0.8                     0.8
    
    param_n_estimators
    1000    8
    300     6
    200     4
    500     4
    750     3
    100     1
    Name: count, dtype: int64
    
    param_max_depth
    4     5
    3     5
    5     5
    7     4
    6     4
    10    2
    8     1
    Name: count, dtype: int64
    
    param_learning_rate
    0.005    10
    0.100     6
    0.010     6
    0.050     4
    Name: count, dtype: int64
    
    param_subsample
    1.0    6
    0.9    6
    0.8    5
    0.6    5
    0.7    4
    Name: count, dtype: int64
    
    param_colsample_bytree
    0.7    12
    0.9     5
    0.8     3
    0.5     2
    1.0     2
    0.6     2
    Name: count, dtype: int64
    
    n_estimators: [200, 300, 500, np.int64(750), 1000]
    max_depth: [3, 4, 5]
    learning_rate: [0.005, 0.01, np.float64(0.05), 0.1]
    subsample: [0.6, np.float64(0.7), 0.8, 0.9, 1.0]
    colsample_bytree: [0.7, 0.8, 0.9]
    
    Total combinations: 900
    Total fits (x5 folds): 4500
    Fitting 5 folds for each of 900 candidates, totalling 4500 fits
         mean_test_score  std_test_score  param_colsample_bytree  param_learning_rate  param_max_depth  param_n_estimators  param_subsample
    539         0.549883        0.012338                     0.8                 0.10                3                 500              1.0
    201         0.549848        0.012019                     0.7                 0.05                5                 200              0.7
    506         0.549126        0.009260                     0.8                 0.05                5                 300              0.7
    809         0.549090        0.015015                     0.9                 0.05                5                 300              1.0
    244         0.549045        0.016503                     0.7                 0.10                3                 750              1.0
    505         0.549019        0.019988                     0.8                 0.05                5                 300              0.6
    806         0.548375        0.016228                     0.9                 0.05                5                 300              0.7
    206         0.548331        0.013453                     0.7                 0.05                5                 300              0.7
    147         0.548218        0.013623                     0.7                 0.01                5                1000              0.8
    746         0.548206        0.012831                     0.9                 0.01                5                1000              0.7


# Evaluation of all models


```python
evaluate_model(tier1_fine_search, X_train_tier1, y_train_tier1, X_test_tier1, y_test_tier1, tier=1)
evaluate_model(tier2_fine_search, X_train_tier2, y_train_tier2, X_test_tier2, y_test_tier2, tier=2)
evaluate_model(tier3_fine_search, X_train_tier3, y_train_tier3, X_test_tier3, y_test_tier3, tier=3)
evaluate_model(tier4_fine_search, X_train_tier4, y_train_tier4, X_test_tier4, y_test_tier4, tier=4)
```

    Tier 1 — Training Set Evaluation
    Accuracy:  0.8974
    ROC-AUC:   0.9653
    
                  precision    recall  f1-score   support
    
               0       0.98      0.90      0.94     10766
               1       0.61      0.89      0.72      1888
    
        accuracy                           0.90     12654
       macro avg       0.79      0.90      0.83     12654
    weighted avg       0.92      0.90      0.90     12654
    
    Tier 1 — Test Set Evaluation
    Accuracy:  0.8344
    ROC-AUC:   0.8494
    
                  precision    recall  f1-score   support
    
               0       0.93      0.87      0.90      2692
               1       0.46      0.64      0.54       472
    
        accuracy                           0.83      3164
       macro avg       0.70      0.76      0.72      3164
    weighted avg       0.86      0.83      0.85      3164
    
    Tier 2 — Training Set Evaluation
    Accuracy:  0.9373
    ROC-AUC:   0.9900
    
                  precision    recall  f1-score   support
    
               0       1.00      0.93      0.96     10766
               1       0.71      0.97      0.82      1888
    
        accuracy                           0.94     12654
       macro avg       0.85      0.95      0.89     12654
    weighted avg       0.95      0.94      0.94     12654
    
    Tier 2 — Test Set Evaluation
    Accuracy:  0.8439
    ROC-AUC:   0.8532
    
                  precision    recall  f1-score   support
    
               0       0.93      0.88      0.91      2692
               1       0.48      0.61      0.54       472
    
        accuracy                           0.84      3164
       macro avg       0.71      0.75      0.72      3164
    weighted avg       0.86      0.84      0.85      3164
    
    Tier 3 — Training Set Evaluation
    Accuracy:  0.9248
    ROC-AUC:   0.9856
    
                  precision    recall  f1-score   support
    
               0       0.99      0.92      0.95     10766
               1       0.68      0.95      0.79      1888
    
        accuracy                           0.92     12654
       macro avg       0.83      0.93      0.87     12654
    weighted avg       0.94      0.92      0.93     12654
    
    Tier 3 — Test Set Evaluation
    Accuracy:  0.8426
    ROC-AUC:   0.8637
    
                  precision    recall  f1-score   support
    
               0       0.94      0.88      0.90      2692
               1       0.48      0.66      0.55       472
    
        accuracy                           0.84      3164
       macro avg       0.71      0.77      0.73      3164
    weighted avg       0.87      0.84      0.85      3164
    
    Tier 4 — Training Set Evaluation
    Accuracy:  0.9222
    ROC-AUC:   0.9840
    
                  precision    recall  f1-score   support
    
               0       0.99      0.92      0.95     10766
               1       0.67      0.96      0.79      1888
    
        accuracy                           0.92     12654
       macro avg       0.83      0.94      0.87     12654
    weighted avg       0.94      0.92      0.93     12654
    
    Tier 4 — Test Set Evaluation
    Accuracy:  0.8404
    ROC-AUC:   0.8658
    
                  precision    recall  f1-score   support
    
               0       0.94      0.87      0.90      2692
               1       0.48      0.68      0.56       472
    
        accuracy                           0.84      3164
       macro avg       0.71      0.77      0.73      3164
    weighted avg       0.87      0.84      0.85      3164
    



```python
from sklearn.metrics import f1_score, recall_score, precision_score, roc_auc_score, accuracy_score

summary = []

for tier, fine_search, X_train, y_train, X_test, y_test in [
    (1, tier1_fine_search, X_train_tier1, y_train_tier1, X_test_tier1, y_test_tier1),
    (2, tier2_fine_search, X_train_tier2, y_train_tier2, X_test_tier2, y_test_tier2),
    (3, tier3_fine_search, X_train_tier3, y_train_tier3, X_test_tier3, y_test_tier3),
    (4, tier4_fine_search, X_train_tier4, y_train_tier4, X_test_tier4, y_test_tier4),
]:
    model = fine_search.best_estimator_
    y_pred = model.predict(X_test)
    y_prob = model.predict_proba(X_test)[:, 1]
    
    summary.append({
        'Tier': tier,
        'Features': X_test.shape[1],
        'CV F1': round(fine_search.best_score_, 4),
        'Test Accuracy': round(accuracy_score(y_test, y_pred), 4),
        'Test Precision': round(precision_score(y_test, y_pred), 4),
        'Test Recall': round(recall_score(y_test, y_pred), 4),
        'Test F1': round(f1_score(y_test, y_pred), 4),
        'Test ROC-AUC': round(roc_auc_score(y_test, y_prob), 4),
    })

summary_df = pd.DataFrame(summary)
print(summary_df.to_string(index=False))
```

     Tier  Features  CV F1  Test Accuracy  Test Precision  Test Recall  Test F1  Test ROC-AUC
        1       130 0.5207         0.8344          0.4606       0.6441   0.5371        0.8494
        2       192 0.5226         0.8439          0.4817       0.6144   0.5400        0.8532
        3       436 0.5458         0.8426          0.4799       0.6568   0.5546        0.8637
        4       833 0.5499         0.8404          0.4754       0.6758   0.5582        0.8658



```python
show_feature_importance(tier1_fine_search, X_train_tier1, tier=1)
show_feature_importance(tier2_fine_search, X_train_tier2, tier=2)
show_feature_importance(tier3_fine_search, X_train_tier3, tier=3)
show_feature_importance(tier4_fine_search, X_train_tier4, tier=4)
```


    
![png](06_modelling_files/06_modelling_21_0.png)
    


    Tier 1 — Bottom 20 features:
    S1EARTHS09       0.005247
    P1REPEATG2       0.005213
    M1CERT912        0.005172
    S1ADVM09         0.005166
    S1ADVPHYSIC09    0.005165
    P1REPEATG3       0.005153
    X1BLACK          0.005134
    X1RACE           0.005123
    S1OTHPHYS09      0.005099
    X1P1RELATION     0.005075
    P1USBORN9        0.004913
    S1TRIGM09        0.004790
    X1HISPANIC       0.004758
    S1REVM09         0.004717
    P1REPEATG6       0.004715
    X1SEX            0.004696
    P1DD             0.004552
    S1MFALL09        0.004421
    S1SFALL09        0.004143
    P1REPEATG9       0.004021
    



    
![png](06_modelling_files/06_modelling_21_2.png)
    


    Tier 2 — Bottom 20 features:
    X1BLACK        0.003604
    X1WHITE        0.003593
    S1PREALGM09    0.003590
    X1PAR1EDU      0.003588
    S1EARTHS09     0.003546
    X1RACE         0.003540
    A1G9TCHREF     0.003535
    S1TRIGM09      0.003512
    P1USBORN9      0.003495
    S1SFALL09      0.003359
    X1SEX          0.003357
    S1REVM09       0.003353
    S1INTGM209     0.003349
    P1HHPARENT     0.003301
    S1MFALL09      0.003233
    X1HISPANIC     0.003144
    P1JOBEVER2     0.003004
    S1ANGEOM09     0.002742
    P1JOBEVER1     0.002735
    S1ADVM09       0.002229
    



    
![png](06_modelling_files/06_modelling_21_4.png)
    


    Tier 3 — Bottom 20 features:
    C1PRNTREFER      0.001345
    X1BLACK          0.001333
    X1PAR1EDU        0.001284
    S1NOTALKCLG      0.001281
    S1MFALL09        0.001236
    P1INTELLECT      0.001231
    P1REPEATG6       0.001231
    X1MOMREL         0.001222
    S1OTHPHYS09      0.001222
    S1INTGM209       0.001215
    P1SKIPGK         0.001211
    S1GENS09         0.001160
    S1OTHBIOS09      0.001116
    X1HISPANIC       0.001017
    X1DUALLANG       0.000923
    S1REVM09         0.000880
    S1ADVM09         0.000791
    S1TRIGM09        0.000790
    P1JOBEVER1       0.000425
    S1ADVPHYSIC09    0.000000
    



    
![png](06_modelling_files/06_modelling_21_6.png)
    


    Tier 4 — Bottom 20 features:
    P1CAMPMS        0.0
    P1QHELP         0.0
    P1PREPPAY       0.0
    P1HELPPAY       0.0
    P1FEEOUT        0.0
    P1ADMITREQ      0.0
    P1ABLEBA        0.0
    P1STEMDISC      0.0
    P1NOOUTSCH      0.0
    P1RELIGGRP      0.0
    P1ADHD          0.0
    P1ARTS          0.0
    P1COUNSELOR     0.0
    P1SCHCHOICE     0.0
    P1SKIPG3        0.0
    P1SKIPG1        0.0
    P1SKIPGK        0.0
    P1FRIEND        0.0
    P1LEARN         0.0
    C1BAMAJ_STEM    0.0
    



```python
compare_feature_importance(
    [tier1_fine_search, tier2_fine_search, tier3_fine_search, tier4_fine_search],
    [X_train_tier1, X_train_tier2, X_train_tier3, X_train_tier4])
```

    Feature overlap across tiers (top 20 each):
    Feature                     T1   T2   T3   T4  Tiers
    --------------------------------------------------
    P1DROPOUT                    1    1    1    6      4
    X1STDOB                      9   11    8   20      4
    X1CONTROL                    6    7   10    8      4
    P1SUSPEND                    2    2    3   14      4
    S1S8GRADE                    3    4    2    4      4
    S1M8GRADE                    4    5    5    9      4
    P1REPEATGRD                  8    6    7    -      3
    X1FAMINCOME                  -    3    4   15      3
    P1REPEATG7                   5   12   16    -      3
    X1PARPATTERN                 -   10   13    7      3
    P1HONORS                     7    9   14    -      3
    P1BAMAJ1_STEM                -   18   17    -      2
    P1SPSREL                     -   16    -   10      2
    S1LATE                      14   17    -    -      2
    X1SESQ5_U                    -    8   12    -      2
    X1POVERTY185                 -   15    6    -      2
    P1SKIPG8                    11   19    -    -      2
    P1OWNHOME                    -   20   20    -      2
    S1SUREHSGRAD                 -    -    9   18      2
    P1REPEATG8                  13    -   11    -      2
    P1SKIPG2                    12   13    -    -      2
    
    Average rank across tiers (only where in top 20):
    Feature                     Avg Rank  Tiers
    ---------------------------------------------
    P1DROPOUT                       2.25      4
    S1S8GRADE                       3.25      4
    P1SUSPEND                       5.25      4
    S1M8GRADE                       5.75      4
    X1CONTROL                       7.75      4
    X1STDOB                        12.00      4
    P1REPEATGRD                     7.00      3
    X1FAMINCOME                     7.33      3
    X1PARPATTERN                   10.00      3
    P1HONORS                       10.00      3
    P1REPEATG7                     11.00      3
    X1SESQ5_U                      10.00      2
    X1POVERTY185                   10.50      2
    P1REPEATG8                     12.00      2
    P1SKIPG2                       12.50      2
    P1SPSREL                       13.00      2
    S1SUREHSGRAD                   13.50      2
    P1SKIPG8                       15.00      2
    S1LATE                         15.50      2
    P1BAMAJ1_STEM                  17.50      2
    P1OWNHOME                      20.00      2



```python
import joblib

joblib.dump(tier1_fine_search, 'tier1_fine_search.pkl')
joblib.dump(tier2_fine_search, 'tier2_fine_search.pkl')
joblib.dump(tier3_fine_search, 'tier3_fine_search.pkl')
joblib.dump(tier4_fine_search, 'tier4_fine_search.pkl')

print("All models saved.")
```

    All models saved.



```python
joblib.dump((X_train_tier1, y_train_tier1, X_test_tier1, y_test_tier1), 'data_tier1.pkl')
joblib.dump((X_train_tier2, y_train_tier2, X_test_tier2, y_test_tier2), 'data_tier2.pkl')
joblib.dump((X_train_tier3, y_train_tier3, X_test_tier3, y_test_tier3), 'data_tier3.pkl')
joblib.dump((X_train_tier4, y_train_tier4, X_test_tier4, y_test_tier4), 'data_tier4.pkl')

print("All data saved.")
```

    All data saved.



```python
plot_shap(tier1_fine_search, X_train_tier1, tier=1)
plot_shap(tier2_fine_search, X_train_tier2, tier=2)
plot_shap(tier3_fine_search, X_train_tier3, tier=3)
plot_shap(tier4_fine_search, X_train_tier4, tier=4)
```

    Tier 1 — SHAP Summary Plot



    
![png](06_modelling_files/06_modelling_25_1.png)
    


    Tier 2 — SHAP Summary Plot



    
![png](06_modelling_files/06_modelling_25_3.png)
    


    Tier 3 — SHAP Summary Plot



    
![png](06_modelling_files/06_modelling_25_5.png)
    


    Tier 4 — SHAP Summary Plot



    
![png](06_modelling_files/06_modelling_25_7.png)
    



```python
probs_df = ensemble_predictions(
    [tier1_fine_search, tier2_fine_search, tier3_fine_search, tier4_fine_search],
    [X_test_tier1, X_test_tier2, X_test_tier3, X_test_tier4],
    y_test_tier1
)
```

    === Probability Averaging Ensemble ===
    
    Threshold 0.3:
                  precision    recall  f1-score   support
    
               0       0.96      0.72      0.82      2692
               1       0.35      0.85      0.49       472
    
        accuracy                           0.74      3164
       macro avg       0.66      0.78      0.66      3164
    weighted avg       0.87      0.74      0.77      3164
    
    
    Threshold 0.4:
                  precision    recall  f1-score   support
    
               0       0.95      0.81      0.88      2692
               1       0.42      0.76      0.54       472
    
        accuracy                           0.81      3164
       macro avg       0.68      0.79      0.71      3164
    weighted avg       0.87      0.81      0.83      3164
    
    
    Threshold 0.5:
                  precision    recall  f1-score   support
    
               0       0.94      0.88      0.91      2692
               1       0.50      0.65      0.57       472
    
        accuracy                           0.85      3164
       macro avg       0.72      0.77      0.74      3164
    weighted avg       0.87      0.85      0.86      3164
    
    
    Threshold 0.6:
                  precision    recall  f1-score   support
    
               0       0.92      0.93      0.93      2692
               1       0.58      0.54      0.56       472
    
        accuracy                           0.87      3164
       macro avg       0.75      0.74      0.74      3164
    weighted avg       0.87      0.87      0.87      3164
    
    
    === Vote-Based Ensemble ===
    Min votes=1: Precision=0.409, Recall=0.784, F1=0.537
    Min votes=2: Precision=0.458, Recall=0.686, F1=0.550
    Min votes=3: Precision=0.518, Recall=0.606, F1=0.559
    Min votes=4: Precision=0.586, Recall=0.515, F1=0.548



```python
total = len(probs_df)
unanimous_dropout = (probs_df['votes'] == 4).sum()
unanimous_safe = (probs_df['votes'] == 0).sum()
split = total - unanimous_dropout - unanimous_safe
    
print(f"Total test students: {total}")
print(f"All 4 models agree: DROPOUT — {unanimous_dropout} ({unanimous_dropout/total*100:.1f}%)")
print(f"All 4 models agree: SAFE    — {unanimous_safe} ({unanimous_safe/total*100:.1f}%)")
print(f"Models disagree            — {split} ({split/total*100:.1f}%)")
print()
print("Vote distribution:")
print(probs_df['votes'].value_counts().sort_index())
```

    Total test students: 3164
    All 4 models agree: DROPOUT — 415 (13.1%)
    All 4 models agree: SAFE    — 2259 (71.4%)
    Models disagree            — 490 (15.5%)
    
    Vote distribution:
    votes
    0    2259
    1     198
    2     155
    3     137
    4     415
    Name: count, dtype: int64



```python
print("Accuracy within each vote group:")
for v in range(5):
    group = probs_df[probs_df['votes'] == v]
    actual_dropouts = group['true_label'].sum()
    total = len(group)
    dropout_rate = actual_dropouts / total
    print(f"Votes={v}: {total} students, {actual_dropouts} actual dropouts ({dropout_rate*100:.1f}%)")
```

    Accuracy within each vote group:
    Votes=0: 2259 students, 102 actual dropouts (4.5%)
    Votes=1: 198 students, 46 actual dropouts (23.2%)
    Votes=2: 155 students, 38 actual dropouts (24.5%)
    Votes=3: 137 students, 43 actual dropouts (31.4%)
    Votes=4: 415 students, 243 actual dropouts (58.6%)



```python
import joblib

# Load models
tier1_fine_search = joblib.load('tier1_fine_search.pkl')
tier2_fine_search = joblib.load('tier2_fine_search.pkl')
tier3_fine_search = joblib.load('tier3_fine_search.pkl')
tier4_fine_search = joblib.load('tier4_fine_search.pkl')

# Load data
X_train_tier1, y_train_tier1, X_test_tier1, y_test_tier1 = joblib.load('data_tier1.pkl')
X_train_tier2, y_train_tier2, X_test_tier2, y_test_tier2 = joblib.load('data_tier2.pkl')
X_train_tier3, y_train_tier3, X_test_tier3, y_test_tier3 = joblib.load('data_tier3.pkl')
X_train_tier4, y_train_tier4, X_test_tier4, y_test_tier4 = joblib.load('data_tier4.pkl')

# Load dataframe and split
df = pd.read_csv('hsls_clean.csv')

with open('train_test_split.json') as f:
    split = json.load(f)
train_idx = split['train']
test_idx  = split['test']

with open('feature_tiers.json') as f:
    feature_tiers = json.load(f)

print("All loaded successfully.")
```

    All loaded successfully.



```python
results = pd.DataFrame(tier4_fine_search.cv_results_)
print(results.columns.tolist())
```

    ['mean_fit_time', 'std_fit_time', 'mean_score_time', 'std_score_time', 'param_colsample_bytree', 'param_learning_rate', 'param_max_depth', 'param_n_estimators', 'param_subsample', 'params', 'split0_test_score', 'split1_test_score', 'split2_test_score', 'split3_test_score', 'split4_test_score', 'mean_test_score', 'std_test_score', 'rank_test_score']



```python
from sklearn.metrics import precision_recall_curve
import matplotlib.pyplot as plt

model = tier4_fine_search.best_estimator_
y_prob = model.predict_proba(X_test_tier4)[:, 1]

precisions, recalls, thresholds = precision_recall_curve(y_test_tier4, y_prob)

plt.figure(figsize=(10, 6))
plt.plot(recalls, precisions)
plt.xlabel('Recall')
plt.ylabel('Precision')
plt.title('Tier 4 — Precision-Recall Curve')
plt.grid(True)
plt.show()

# Find threshold giving ~80% recall
for thresh, prec, rec in zip(thresholds, precisions[:-1], recalls[:-1]):
    if 0.78 <= rec <= 0.82:
        print(f"Threshold: {thresh:.3f}, Precision: {prec:.3f}, Recall: {rec:.3f}")
```


    
![png](06_modelling_files/06_modelling_31_0.png)
    


    Threshold: 0.297, Precision: 0.353, Recall: 0.820
    Threshold: 0.298, Precision: 0.354, Recall: 0.820
    Threshold: 0.298, Precision: 0.354, Recall: 0.820
    Threshold: 0.298, Precision: 0.354, Recall: 0.820
    Threshold: 0.298, Precision: 0.355, Recall: 0.820
    Threshold: 0.298, Precision: 0.355, Recall: 0.820
    Threshold: 0.299, Precision: 0.355, Recall: 0.820
    Threshold: 0.300, Precision: 0.356, Recall: 0.820
    Threshold: 0.300, Precision: 0.356, Recall: 0.820
    Threshold: 0.301, Precision: 0.356, Recall: 0.820
    Threshold: 0.301, Precision: 0.357, Recall: 0.820
    Threshold: 0.301, Precision: 0.357, Recall: 0.820
    Threshold: 0.301, Precision: 0.357, Recall: 0.820
    Threshold: 0.302, Precision: 0.358, Recall: 0.820
    Threshold: 0.302, Precision: 0.358, Recall: 0.820
    Threshold: 0.302, Precision: 0.358, Recall: 0.820
    Threshold: 0.303, Precision: 0.359, Recall: 0.820
    Threshold: 0.303, Precision: 0.359, Recall: 0.820
    Threshold: 0.303, Precision: 0.359, Recall: 0.820
    Threshold: 0.303, Precision: 0.360, Recall: 0.820
    Threshold: 0.304, Precision: 0.359, Recall: 0.818
    Threshold: 0.304, Precision: 0.359, Recall: 0.818
    Threshold: 0.304, Precision: 0.360, Recall: 0.818
    Threshold: 0.305, Precision: 0.360, Recall: 0.818
    Threshold: 0.305, Precision: 0.360, Recall: 0.818
    Threshold: 0.305, Precision: 0.361, Recall: 0.818
    Threshold: 0.305, Precision: 0.360, Recall: 0.816
    Threshold: 0.305, Precision: 0.360, Recall: 0.816
    Threshold: 0.306, Precision: 0.361, Recall: 0.816
    Threshold: 0.306, Precision: 0.361, Recall: 0.816
    Threshold: 0.306, Precision: 0.362, Recall: 0.816
    Threshold: 0.307, Precision: 0.362, Recall: 0.816
    Threshold: 0.308, Precision: 0.362, Recall: 0.816
    Threshold: 0.308, Precision: 0.363, Recall: 0.816
    Threshold: 0.309, Precision: 0.363, Recall: 0.816
    Threshold: 0.310, Precision: 0.363, Recall: 0.816
    Threshold: 0.310, Precision: 0.364, Recall: 0.816
    Threshold: 0.310, Precision: 0.363, Recall: 0.814
    Threshold: 0.311, Precision: 0.363, Recall: 0.814
    Threshold: 0.312, Precision: 0.364, Recall: 0.814
    Threshold: 0.312, Precision: 0.364, Recall: 0.814
    Threshold: 0.312, Precision: 0.364, Recall: 0.814
    Threshold: 0.312, Precision: 0.365, Recall: 0.814
    Threshold: 0.313, Precision: 0.365, Recall: 0.814
    Threshold: 0.314, Precision: 0.365, Recall: 0.814
    Threshold: 0.314, Precision: 0.366, Recall: 0.814
    Threshold: 0.314, Precision: 0.366, Recall: 0.814
    Threshold: 0.314, Precision: 0.366, Recall: 0.814
    Threshold: 0.315, Precision: 0.367, Recall: 0.814
    Threshold: 0.315, Precision: 0.367, Recall: 0.814
    Threshold: 0.316, Precision: 0.367, Recall: 0.814
    Threshold: 0.316, Precision: 0.368, Recall: 0.814
    Threshold: 0.317, Precision: 0.368, Recall: 0.814
    Threshold: 0.317, Precision: 0.369, Recall: 0.814
    Threshold: 0.318, Precision: 0.369, Recall: 0.814
    Threshold: 0.318, Precision: 0.369, Recall: 0.814
    Threshold: 0.319, Precision: 0.370, Recall: 0.814
    Threshold: 0.319, Precision: 0.370, Recall: 0.814
    Threshold: 0.320, Precision: 0.370, Recall: 0.814
    Threshold: 0.320, Precision: 0.371, Recall: 0.814
    Threshold: 0.320, Precision: 0.370, Recall: 0.811
    Threshold: 0.320, Precision: 0.369, Recall: 0.809
    Threshold: 0.320, Precision: 0.370, Recall: 0.809
    Threshold: 0.321, Precision: 0.369, Recall: 0.807
    Threshold: 0.321, Precision: 0.370, Recall: 0.807
    Threshold: 0.321, Precision: 0.369, Recall: 0.805
    Threshold: 0.321, Precision: 0.369, Recall: 0.805
    Threshold: 0.322, Precision: 0.370, Recall: 0.805
    Threshold: 0.323, Precision: 0.369, Recall: 0.803
    Threshold: 0.323, Precision: 0.369, Recall: 0.803
    Threshold: 0.324, Precision: 0.370, Recall: 0.803
    Threshold: 0.325, Precision: 0.370, Recall: 0.803
    Threshold: 0.325, Precision: 0.370, Recall: 0.803
    Threshold: 0.326, Precision: 0.371, Recall: 0.803
    Threshold: 0.328, Precision: 0.371, Recall: 0.803
    Threshold: 0.329, Precision: 0.372, Recall: 0.803
    Threshold: 0.329, Precision: 0.372, Recall: 0.803
    Threshold: 0.329, Precision: 0.371, Recall: 0.801
    Threshold: 0.330, Precision: 0.372, Recall: 0.801
    Threshold: 0.330, Precision: 0.372, Recall: 0.801
    Threshold: 0.330, Precision: 0.372, Recall: 0.801
    Threshold: 0.331, Precision: 0.373, Recall: 0.801
    Threshold: 0.331, Precision: 0.373, Recall: 0.801
    Threshold: 0.331, Precision: 0.374, Recall: 0.801
    Threshold: 0.331, Precision: 0.373, Recall: 0.799
    Threshold: 0.332, Precision: 0.373, Recall: 0.799
    Threshold: 0.332, Precision: 0.374, Recall: 0.799
    Threshold: 0.333, Precision: 0.374, Recall: 0.799
    Threshold: 0.333, Precision: 0.374, Recall: 0.799
    Threshold: 0.333, Precision: 0.375, Recall: 0.799
    Threshold: 0.334, Precision: 0.375, Recall: 0.799
    Threshold: 0.334, Precision: 0.375, Recall: 0.799
    Threshold: 0.335, Precision: 0.376, Recall: 0.799
    Threshold: 0.335, Precision: 0.375, Recall: 0.797
    Threshold: 0.335, Precision: 0.376, Recall: 0.797
    Threshold: 0.335, Precision: 0.376, Recall: 0.797
    Threshold: 0.336, Precision: 0.376, Recall: 0.797
    Threshold: 0.337, Precision: 0.377, Recall: 0.797
    Threshold: 0.338, Precision: 0.377, Recall: 0.797
    Threshold: 0.338, Precision: 0.378, Recall: 0.797
    Threshold: 0.338, Precision: 0.377, Recall: 0.794
    Threshold: 0.339, Precision: 0.377, Recall: 0.794
    Threshold: 0.339, Precision: 0.377, Recall: 0.792
    Threshold: 0.339, Precision: 0.377, Recall: 0.792
    Threshold: 0.339, Precision: 0.376, Recall: 0.790
    Threshold: 0.339, Precision: 0.377, Recall: 0.790
    Threshold: 0.339, Precision: 0.377, Recall: 0.790
    Threshold: 0.339, Precision: 0.377, Recall: 0.788
    Threshold: 0.340, Precision: 0.377, Recall: 0.788
    Threshold: 0.340, Precision: 0.377, Recall: 0.788
    Threshold: 0.340, Precision: 0.378, Recall: 0.788
    Threshold: 0.340, Precision: 0.378, Recall: 0.788
    Threshold: 0.341, Precision: 0.378, Recall: 0.788
    Threshold: 0.342, Precision: 0.379, Recall: 0.788
    Threshold: 0.342, Precision: 0.379, Recall: 0.788
    Threshold: 0.342, Precision: 0.380, Recall: 0.788
    Threshold: 0.342, Precision: 0.380, Recall: 0.788
    Threshold: 0.342, Precision: 0.380, Recall: 0.788
    Threshold: 0.342, Precision: 0.381, Recall: 0.788
    Threshold: 0.342, Precision: 0.381, Recall: 0.788
    Threshold: 0.344, Precision: 0.382, Recall: 0.788
    Threshold: 0.344, Precision: 0.382, Recall: 0.788
    Threshold: 0.344, Precision: 0.382, Recall: 0.788
    Threshold: 0.344, Precision: 0.383, Recall: 0.788
    Threshold: 0.345, Precision: 0.382, Recall: 0.786
    Threshold: 0.345, Precision: 0.382, Recall: 0.786
    Threshold: 0.346, Precision: 0.383, Recall: 0.786
    Threshold: 0.346, Precision: 0.383, Recall: 0.786
    Threshold: 0.346, Precision: 0.384, Recall: 0.786
    Threshold: 0.347, Precision: 0.383, Recall: 0.784
    Threshold: 0.347, Precision: 0.383, Recall: 0.784
    Threshold: 0.347, Precision: 0.384, Recall: 0.784
    Threshold: 0.348, Precision: 0.384, Recall: 0.784
    Threshold: 0.350, Precision: 0.385, Recall: 0.784
    Threshold: 0.351, Precision: 0.385, Recall: 0.784
    Threshold: 0.352, Precision: 0.385, Recall: 0.784
    Threshold: 0.352, Precision: 0.385, Recall: 0.782
    Threshold: 0.352, Precision: 0.385, Recall: 0.782
    Threshold: 0.353, Precision: 0.386, Recall: 0.782
    Threshold: 0.353, Precision: 0.386, Recall: 0.782
    Threshold: 0.354, Precision: 0.386, Recall: 0.782
    Threshold: 0.354, Precision: 0.387, Recall: 0.782
    Threshold: 0.354, Precision: 0.387, Recall: 0.782
    Threshold: 0.355, Precision: 0.388, Recall: 0.782
    Threshold: 0.355, Precision: 0.388, Recall: 0.782
    Threshold: 0.355, Precision: 0.388, Recall: 0.782
    Threshold: 0.356, Precision: 0.389, Recall: 0.782
    Threshold: 0.357, Precision: 0.389, Recall: 0.782



```python
import shap

model = tier4_fine_search.best_estimator_
explainer = shap.TreeExplainer(model)

# Get SHAP values for test set
shap_values = explainer.shap_values(X_test_tier4)

# Find S1EDUEXPECT index
feature_names = X_test_tier4.columns.tolist()
if 'S1EDUEXPECT' in feature_names:
    idx = feature_names.index('S1EDUEXPECT')
    
    # Plot dependence plot for S1EDUEXPECT
    shap.dependence_plot('S1EDUEXPECT', shap_values, X_test_tier4, show=True)
else:
    print("S1EDUEXPECT not in Tier 4 features")
```


    
![png](06_modelling_files/06_modelling_32_0.png)
    



```python
from xgboost import plot_tree
import matplotlib.pyplot as plt

model = tier1_fine_search.best_estimator_

fig, ax = plt.subplots(figsize=(40, 20))
plot_tree(model, num_trees=0, ax=ax)
plt.tight_layout()
plt.show()
```

    /opt/anaconda3/envs/hsls_env/lib/python3.11/site-packages/xgboost/plotting.py:268: FutureWarning: The `num_trees` parameter is deprecated, use `tree_idx` insetad. 
      warnings.warn(



    
![png](06_modelling_files/06_modelling_33_1.png)
    



```python

```
