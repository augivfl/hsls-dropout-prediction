```python
import pandas as pd

df = pd.read_csv('hsls_17_student_pets_sr_v1_0.csv')

print(df.shape)
print(df.head())
```

    (23503, 9614)
       STU_ID  SCH_ID  X1NCESID  X2NCESID  STRAT_ID  PSU  X2UNIV1  X2UNIV2A  \
    0   10001      -5        -5        -5        -5   -5       11         1   
    1   10002      -5        -5        -5        -5   -5       11         1   
    2   10003      -5        -5        -5        -5   -5       11         1   
    3   10004      -5        -5        -5        -5   -5       10         1   
    4   10005      -5        -5        -5        -5   -5       11         1   
    
       X2UNIV2B  X3UNIV1  ...  W5W1W2W3W4PSRECORDS191  W5W1W2W3W4PSRECORDS192  \
    0         1     1111  ...                     0.0             2098.087446   
    1         1     1111  ...                     0.0                0.000000   
    2         1     1111  ...                     0.0                0.000000   
    3         7     1001  ...                     0.0                0.000000   
    4         1     1111  ...                     0.0                0.000000   
    
       W5W1W2W3W4PSRECORDS193  W5W1W2W3W4PSRECORDS194  W5W1W2W3W4PSRECORDS195  \
    0             1824.641398                     0.0             2431.665487   
    1                0.000000                     0.0                0.000000   
    2                0.000000                     0.0                0.000000   
    3                0.000000                     0.0                0.000000   
    4                0.000000                     0.0                0.000000   
    
       W5W1W2W3W4PSRECORDS196  W5W1W2W3W4PSRECORDS197  W5W1W2W3W4PSRECORDS198  \
    0                     0.0                     0.0             2457.423209   
    1                     0.0                     0.0                0.000000   
    2                     0.0                     0.0                0.000000   
    3                     0.0                     0.0                0.000000   
    4                     0.0                     0.0                0.000000   
    
       W5W1W2W3W4PSRECORDS199  W5W1W2W3W4PSRECORDS200  
    0                     0.0              2053.40787  
    1                     0.0                 0.00000  
    2                     0.0                 0.00000  
    3                     0.0                 0.00000  
    4                     0.0                 0.00000  
    
    [5 rows x 9614 columns]



```python
key_vars = ['X4EVERDROP', 'X1SES', 'X1TXMSCR', 'X1SEX', 'X1RACE', 
            'X1SCHOOLBEL', 'X1SCHOOLENG', 'X1STUEDEXPCT', 'X1POVERTY']

for var in key_vars:
    present = var in df.columns
    print(f"{var}: {'FOUND' if present else 'MISSING'}")
```

    X4EVERDROP: FOUND
    X1SES: FOUND
    X1TXMSCR: FOUND
    X1SEX: FOUND
    X1RACE: FOUND
    X1SCHOOLBEL: FOUND
    X1SCHOOLENG: FOUND
    X1STUEDEXPCT: FOUND
    X1POVERTY: FOUND



```python
x1_cols = [col for col in df.columns if col.startswith('X1')]
non_x1_cols = [col for col in df.columns if not col.startswith('X1')]

print(f"X1 columns: {len(x1_cols)}")
print(f"Non-X1 columns: {len(non_x1_cols)}")
```

    X1 columns: 191
    Non-X1 columns: 9423



```python
non_x1_df = df[non_x1_cols]
non_empty = [col for col in non_x1_cols if df[col].notna().any()]

print(f"Non-X1 columns with at least one non-null value: {len(non_empty)}")
print(non_empty[:20])
```

    Non-X1 columns with at least one non-null value: 9423
    ['STU_ID', 'SCH_ID', 'X2NCESID', 'STRAT_ID', 'PSU', 'X2UNIV1', 'X2UNIV2A', 'X2UNIV2B', 'X3UNIV1', 'X4UNIV1', 'W1STUDENT', 'W1PARENT', 'W1MATHTCH', 'W1SCITCH', 'W2STUDENT', 'W2W1STU', 'W2PARENT', 'W2W1PAR', 'W3STUDENT', 'W3W1STU']



```python
non_empty_real = [col for col in non_x1_cols if (df[col] != -5).any()]

print(f"Non-X1 columns with real data beyond -5: {len(non_empty_real)}")
print(non_empty_real[:30])
```

    Non-X1 columns with real data beyond -5: 8522
    ['STU_ID', 'X2UNIV1', 'X2UNIV2A', 'X2UNIV2B', 'X3UNIV1', 'X4UNIV1', 'W1STUDENT', 'W1PARENT', 'W1MATHTCH', 'W1SCITCH', 'W2STUDENT', 'W2W1STU', 'W2PARENT', 'W2W1PAR', 'W3STUDENT', 'W3W1STU', 'W3W1W2STU', 'W3W2STU', 'W3HSTRANS', 'W3STUDENTTR', 'W3W1STUTR', 'W3W1W2STUTR', 'W3W2STUTR', 'W4STUDENT', 'W4W1STU', 'W4W1STUP1', 'W4W1STUP1P2', 'W4W1W2W3STU', 'W5PSTRANS', 'W5PSRECORDS']



```python
missing_codes = [-1, -2, -3, -4, -5, -6, -7, -8, -9]

non_empty_real = [col for col in non_x1_cols if (~df[col].isin(missing_codes)).any()]

print(f"Non-X1 columns with real data beyond missing codes: {len(non_empty_real)}")
print(non_empty_real[:30])
```

    Non-X1 columns with real data beyond missing codes: 8522
    ['STU_ID', 'X2UNIV1', 'X2UNIV2A', 'X2UNIV2B', 'X3UNIV1', 'X4UNIV1', 'W1STUDENT', 'W1PARENT', 'W1MATHTCH', 'W1SCITCH', 'W2STUDENT', 'W2W1STU', 'W2PARENT', 'W2W1PAR', 'W3STUDENT', 'W3W1STU', 'W3W1W2STU', 'W3W2STU', 'W3HSTRANS', 'W3STUDENTTR', 'W3W1STUTR', 'W3W1W2STUTR', 'W3W2STUTR', 'W4STUDENT', 'W4W1STU', 'W4W1STUP1', 'W4W1STUP1P2', 'W4W1W2W3STU', 'W5PSTRANS', 'W5PSRECORDS']



```python
print(df['X2UNIV1'].value_counts())
print(df['X2UNIV1'].dtype)
```

    X2UNIV1
    11    18623
    10     2821
    1      1971
    0        88
    Name: count, dtype: int64
    int64



```python
weight_cols = ['W1STUDENT', 'W1PARENT', 'W1MATHTCH', 'W1SCITCH']

for col in weight_cols:
    print(f"\n--- {col} ---")
    print(f"Min: {df[col].min():.2f}")
    print(f"Max: {df[col].max():.2f}")
    print(f"Mean: {df[col].mean():.2f}")
    print(f"Zero or negative values: {(df[col] <= 0).sum()}")
```

    
    --- W1STUDENT ---
    Min: 0.00
    Max: 5500.48
    Mean: 175.08
    Zero or negative values: 2059
    
    --- W1PARENT ---
    Min: 0.00
    Max: 7603.29
    Mean: 175.08
    Zero or negative values: 7074
    
    --- W1MATHTCH ---
    Min: 0.00
    Max: 4198.71
    Mean: 170.46
    Zero or negative values: 7468
    
    --- W1SCITCH ---
    Min: 0.00
    Max: 6878.97
    Mean: 158.21
    Zero or negative values: 8874



```python
prefixes = ['S1', 'P1', 'M1', 'A1', 'C1']

for prefix in prefixes:
    cols = [col for col in df.columns if col.startswith(prefix)]
    print(f"{prefix} columns: {len(cols)}")
```

    S1 columns: 286
    P1 columns: 199
    M1 columns: 150
    A1 columns: 266
    C1 columns: 197



```python
missing_codes = [-1, -2, -3, -4, -5, -6, -7, -8, -9]
base_year_cols = [col for col in df.columns if col.startswith(('X1', 'S1', 'P1', 'M1', 'A1', 'C1'))]

empty = []
ruf_only = []
has_data = []

for col in base_year_cols:
    values = df[col].dropna().unique()
    real_values = [v for v in values if v not in missing_codes]
    
    if len(real_values) == 0:
        if all(v == -5 for v in values):
            ruf_only.append(col)
        else:
            empty.append(col)
    else:
        has_data.append(col)

print(f"Has real data: {len(has_data)}")
print(f"RUF only (-5): {len(ruf_only)}")
print(f"Effectively empty: {len(empty)}")
```

    Has real data: 910
    RUF only (-5): 379
    Effectively empty: 0



```python
summary = []

for col in has_data:
    total = len(df[col])
    missing = df[col].isin(missing_codes).sum()
    real_count = total - missing
    n_unique = df[col][~df[col].isin(missing_codes)].nunique()
    
    summary.append({
        'variable': col,
        'real_values': real_count,
        'missing_coded': missing,
        'pct_real': round(real_count / total * 100, 1),
        'n_unique': n_unique
    })

summary_df = pd.DataFrame(summary).sort_values('pct_real', ascending=False)
print(summary_df.head(20))
print(f"\nColumns with more than 50% real data: {(summary_df['pct_real'] > 50).sum()}")
```

             variable  real_values  missing_coded  pct_real  n_unique
    0           X1SEX        23497              6     100.0         2
    102      X1REGION        23503              0     100.0         4
    886  X1P1RELAT_IM        23503              0     100.0         2
    885   X1HISPAN_IM        23503              0     100.0         2
    884     X1RACE_IM        23503              0     100.0         2
    883      X1SEX_IM        23503              0     100.0         2
    882   X1TXMATH_IM        23503              0     100.0         2
    110      X1CQSTAT        23503              0     100.0         3
    109  X1AQDESIGNEE        23503              0     100.0         3
    107      X1AQSTAT        23503              0     100.0         5
    101      X1LOCALE        23503              0     100.0         4
    888  X1PAR1EDU_IM        23503              0     100.0         2
    100     X1CONTROL        23503              0     100.0         2
    92    X1TSCRSLINK        23503              0     100.0         4
    91       X1TSLINK        23503              0     100.0         5
    89      X1TSQSTAT        23503              0     100.0         3
    81    X1TMCRSLINK        23503              0     100.0         4
    80       X1TMLINK        23503              0     100.0         5
    78      X1TMQSTAT        23503              0     100.0         3
    75       X1PQSTAT        23503              0     100.0         2
    
    Columns with more than 50% real data: 825



```python
print(summary_df.tail(20))
print(f"\nColumns with less than 10% real data: {(summary_df['pct_real'] < 10).sum()}")
```

             variable  real_values  missing_coded  pct_real  n_unique
    400     P1USGRADE         1268          22235       5.4        11
    405      P1ELLNOW         1277          22226       5.4         3
    399       P1USYR9         1262          22241       5.4        16
    700  A1HRTEACHING         1216          22287       5.2        13
    389      P1HHOTHR         1181          22322       5.0         4
    522      P1QHELP2          912          22591       3.9         2
    521      P1QHELP1          912          22591       3.9         2
    523      P1QHELP4          912          22591       3.9         2
    511      P1FEEPRV          873          22630       3.7         2
    384    P1HHPARENT          585          22918       2.5         3
    376      S1FEEPRV          473          23030       2.0         2
    375       S1FEEIN          424          23079       1.8         2
    374      S1COSTIN          434          23069       1.8         8
    377      S1FEEOUT          305          23198       1.3         2
    512      P1FEEOUT          255          23248       1.1         2
    456      P1SKIPGK          188          23315       0.8         2
    457      P1SKIPG1          188          23315       0.8         2
    458      P1SKIPG2          188          23315       0.8         2
    459      P1SKIPG3          188          23315       0.8         2
    460      P1SKIPG8          188          23315       0.8         2
    
    Columns with less than 10% real data: 32



```python
bins = [0, 10, 25, 50, 75, 90, 100]
labels = ['0-10%', '10-25%', '25-50%', '50-75%', '75-90%', '90-100%']
summary_df['pct_bucket'] = pd.cut(summary_df['pct_real'], bins=bins, labels=labels, include_lowest=True)
print(summary_df['pct_bucket'].value_counts().sort_index())
```

    pct_bucket
    0-10%       32
    10-25%      15
    25-50%      38
    50-75%     358
    75-90%     340
    90-100%    127
    Name: count, dtype: int64



```python
im_cols = [col for col in df.columns if col.endswith('_IM')]
print(f"Imputation flag columns: {len(im_cols)}")
print(im_cols)
```

    Imputation flag columns: 130
    ['X1TXMATH_IM', 'X1SEX_IM', 'X1RACE_IM', 'X1HISPAN_IM', 'X1NATIVEL_IM', 'X1P1RELAT_IM', 'X1P2RELAT_IM', 'X1PAR1EDU_IM', 'X1PAR2EDU_IM', 'X1PAREDU_IM', 'X1PARPATT_IM', 'X1PAR1EMP_IM', 'X1PAR2EMP_IM', 'X1PAR1OCC_IM', 'X1PAR2OCC_IM', 'X1MOMREL_IM', 'X1MOMEDU_IM', 'X1MOMEMP_IM', 'X1MOMOCC_IM', 'X1DADREL_IM', 'X1DADEDU_IM', 'X1DADEMP_IM', 'X1DADOCC_IM', 'X1HHNUMB_IM', 'X1FAMINC_IM', 'X1POVERTY_IM', 'X1SES_IM', 'X1STUEDEX_IM', 'X1PAREDEX_IM', 'X2TXMATH_IM', 'X2SEX_IM', 'X2RACE_IM', 'X2HISPAN_IM', 'X2NATIVEL_IM', 'X2P1RELAT_IM', 'X2P2RELAT_IM', 'X2PAR1EDU_IM', 'X2PAR2EDU_IM', 'X2PAREDU_IM', 'X2PARPATT_IM', 'X2PAR1EMP_IM', 'X2PAR2EMP_IM', 'X2PAR1OCC_IM', 'X2PAR2OCC_IM', 'X2MOMREL_IM', 'X2MOMEDU_IM', 'X2MOMEMP_IM', 'X2MOMOCC_IM', 'X2DADREL_IM', 'X2DADEDU_IM', 'X2DADEMP_IM', 'X2DADOCC_IM', 'X2HHNUMB_IM', 'X2FAMINC_IM', 'X2POVERTY_IM', 'X2SES_IM', 'X2STUEDEX_IM', 'X2PAREDEX_IM', 'X3CLASSES_IM', 'X3WORK_IM', 'X3HSCRED_IM', 'X3HSCREDTY_IM', 'X3LASTHSD_IM', 'X3DROPSTAT_IM', 'X3EVERDROP_IM', 'X3CLGANDW_IM', 'X4X2SES_IM', 'X4HSCOMPDATE_IM', 'X4HSCOMPSTAT_IM', 'X4FB16ENRSTAT_IM', 'X4HS2PSMOS_IM', 'X4PSLFSTFB16_IM', 'X4ATNDCLG16FB_IM', 'X4EVRATNDCLG_IM', 'X4CHILDREN_IM', 'X4EMPHRSFB16_IM', 'X4INCOMECAT_IM', 'X4ANYJOB_IM', 'X4UNEMP16FB_IM', 'X4WORKING16FB_IM', 'X5PFYTFEDWRK_IM', 'X5PFYSTATEAMT_IM', 'X5PFYSTATNEED_IM', 'X5PFYSTATNOND_IM', 'X5PFYSTGTAMT_IM', 'X5PFYSTMILAMT_IM', 'X5PFYSTNDMRT_IM', 'X5PFYSTNDONLY_IM', 'X5PFYSTNOND1_IM', 'X5PFYSTVETAMT_IM', 'X5PFYSTWKAMT_IM', 'X5PFYSTMERIT_IM', 'X5PFYSTLNAMT_IM', 'X5PFYVOCHELP_IM', 'X5PFYINSTAMT_IM', 'X5PFYINGRTAMT_IM', 'X5PFYINSTNEED_IM', 'X5PFYINSMERIT_IM', 'X5PFYINATHAMT_IM', 'X5PFYINSTNOND1_IM', 'X5PFYEMPLWAIV_IM', 'X5PFYINSMILAMT_IM', 'X5PFYINLNAMT_IM', 'X5PFYINSWAIV_IM', 'X5PFYINSTWRK_IM', 'X5PFYINSTVETAMT_IM', 'X5PFYMERITAID_IM', 'X5PFYSEOGAMT_IM', 'X5PFYTITIVAMT_IM', 'X5PFYNEEDAID_IM', 'X5PFYTITIVAIDREC_IM', 'X5PFYSTAIDREC_IM', 'X5PFYINSTAIDREC_IM', 'X5PFYANYAIDREC_IM', 'X5PFYCAMPAMT_IM', 'X5PFYFEDNEED_IM', 'X5PFYFEDPACK_IM', 'X5PFYT4GRTAMT_IM', 'X5PFYNETPRICEALL_IM', 'X5PFYTOTAID2_IM', 'X5PFYNETPRICEGRT_IM', 'X5PFYPELLPACK_IM', 'X5PFYTOTLOAN_IM', 'X5PFYTOTLOAN2_IM', 'X5PFYTOTLOAN3_IM', 'X5EVRFEDAPP_IM', 'X5FEDAPP14_IM', 'X5FEDAPP15_IM', 'X5FEDAPP16_IM', 'X5PFYTUITION_IM']



```python
base_year_im = [col for col in im_cols if col.startswith('X1')]

im_summary = []

for col in base_year_im:
    counts = df[col].value_counts()
    real = counts.get(0, 0)
    imputed = counts.get(1, 0)
    total = real + imputed
    pct_imputed = round(imputed / total * 100, 1) if total > 0 else 0
    
    im_summary.append({
        'variable': col.replace('_IM', ''),
        'real': real,
        'imputed': imputed,
        'pct_imputed': pct_imputed
    })

im_df = pd.DataFrame(im_summary).sort_values('pct_imputed', ascending=False)

bins = [0, 5, 10, 25, 50, 100]
labels = ['0-5%', '5-10%', '10-25%', '25-50%', '50-100%']
im_df['imputation_bucket'] = pd.cut(im_df['pct_imputed'], bins=bins, labels=labels, include_lowest=True)

print(im_df.to_string())
print(f"\nImputation bucket distribution:")
print(im_df['imputation_bucket'].value_counts().sort_index())
```

         variable   real  imputed  pct_imputed imputation_bucket
    26      X1SES  16694     5175         23.7            10-25%
    25  X1POVERTY  21226     2277          9.7             5-10%
    23   X1HHNUMB  22118     1385          5.9             5-10%
    28  X1PAREDEX  22424     1079          4.6              0-5%
    11  X1PAR1EMP  22476     1027          4.4              0-5%
    10  X1PARPATT  22488     1015          4.3              0-5%
    24   X1FAMINC  22537      966          4.1              0-5%
    17   X1MOMEMP  22585      918          3.9              0-5%
    21   X1DADEMP  22602      901          3.8              0-5%
    12  X1PAR2EMP  22631      872          3.7              0-5%
    0    X1TXMATH  22840      663          2.8              0-5%
    9    X1PAREDU  23117      386          1.6              0-5%
    18   X1MOMOCC  23123      380          1.6              0-5%
    14  X1PAR2OCC  23119      384          1.6              0-5%
    13  X1PAR1OCC  23139      364          1.5              0-5%
    22   X1DADOCC  23180      323          1.4              0-5%
    27  X1STUEDEX  23195      308          1.3              0-5%
    8   X1PAR2EDU  23244      259          1.1              0-5%
    16   X1MOMEDU  23280      223          0.9              0-5%
    20   X1DADEDU  23283      220          0.9              0-5%
    7   X1PAR1EDU  23316      187          0.8              0-5%
    2      X1RACE  23415       88          0.4              0-5%
    3    X1HISPAN  23421       82          0.3              0-5%
    15   X1MOMREL  23486       17          0.1              0-5%
    19   X1DADREL  23478       25          0.1              0-5%
    6   X1P2RELAT  23477       26          0.1              0-5%
    5   X1P1RELAT  23491       12          0.1              0-5%
    1       X1SEX  23502        1          0.0              0-5%
    4   X1NATIVEL      0        0          0.0              0-5%
    
    Imputation bucket distribution:
    imputation_bucket
    0-5%       26
    5-10%       2
    10-25%      1
    25-50%      0
    50-100%     0
    Name: count, dtype: int64



```python
# Check if X4EVERDROP has an imputation flag
print("X4EVERDROP_IM in columns:", 'X4EVERDROP_IM' in df.columns)

# Check X4EVERDROP value counts including missing codes
print("\nX4EVERDROP value counts:")
print(df['X4EVERDROP'].value_counts(dropna=False).sort_index())
```

    X4EVERDROP_IM in columns: False
    
    X4EVERDROP value counts:
    X4EVERDROP
    -9        3
    -8     6168
     0    14618
     1     2714
    Name: count, dtype: int64



```python
valid = df[df['X4EVERDROP'].isin([0, 1])]
print(f"Total usable students: {len(valid)}")
print(f"\nDropout (1): {(valid['X4EVERDROP'] == 1).sum()}")
print(f"Non-dropout (0): {(valid['X4EVERDROP'] == 0).sum()}")
print(f"\nDropout rate: {round((valid['X4EVERDROP'] == 1).mean() * 100, 1)}%")
```

    Total usable students: 17332
    
    Dropout (1): 2714
    Non-dropout (0): 14618
    
    Dropout rate: 15.7%



```python
base_year_prefixes = ('X1', 'S1', 'P1', 'M1', 'A1', 'C1')

base_year_usable = [
    col for col in has_data 
    if col.startswith(base_year_prefixes)
    and not col.endswith('_IM')
]

print(f"Usable base year feature columns: {len(base_year_usable)}")
print(f"Plus STU_ID and X4EVERDROP")
print(f"Total working columns: {len(base_year_usable) + 2}")
```

    Usable base year feature columns: 882
    Plus STU_ID and X4EVERDROP
    Total working columns: 884


# HSLS:09 Dropout Prediction — Data Exploration

## Key Decisions Log

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Feature wave | Base year only (X1, S1, P1, M1, A1, C1) | Model must be usable without follow-up surveys |
| Target variable | X4EVERDROP | Captures full high school dropout history |
| Survey weights | Not applied | Predictive modelling goal, not population inference |
| Missing codes | -1 to -9 recoded to NaN | All negative codes represent missing in NCES data |
| Dropped rows | X4EVERDROP == -8 or -9 | Unit non-response and missing outcome unusable |
| RUF variables | Excluded | Only -5 values on PUF, no usable information |
| Imputation flags (_IM) | Excluded as features | Metadata only, not predictive variables |
| Usable sample | 17,332 students | 14,618 non-dropouts, 2,714 dropouts (15.7%) |
| Base year universe | 882 usable feature columns | After excluding RUF-only and _IM columns |


```python
# Base year respondents
print("=== BASE YEAR ===")
print(df['X1SQSTAT'].value_counts().sort_index())

print(21001+29+414+2059)
```

    === BASE YEAR ===
    X1SQSTAT
    1    21001
    2       29
    3      414
    8     2059
    Name: count, dtype: int64
    23503



```python
# F1 wave - X2DROPSTAT and X2EVERDROP
print("=== F1 (2012) ===")
print("\nX2DROPSTAT:")
print(df['X2DROPSTAT'].value_counts().sort_index())
print("\nX2EVERDROP:")
print(df['X2EVERDROP'].value_counts().sort_index())
```

    === F1 (2012) ===
    
    X2DROPSTAT:
    X2DROPSTAT
    0    20807
    1      595
    2      142
    3     1237
    4      483
    9      239
    Name: count, dtype: int64
    
    X2EVERDROP:
    X2EVERDROP
    0    21529
    1     1974
    Name: count, dtype: int64



```python
# U13 wave - X3DROPSTAT and X3EVERDROP
print("=== U13 (2013) ===")
print("\nX3DROPSTAT:")
print(df['X3DROPSTAT'].value_counts().sort_index())
print("\nX3EVERDROP:")
print(df['X3EVERDROP'].value_counts().sort_index())
print("\nX3EVERDROP_IM:")
print(df['X3EVERDROP_IM'].value_counts().sort_index())
```

    === U13 (2013) ===
    
    X3DROPSTAT:
    X3DROPSTAT
    -8     4945
     0    15535
     1      721
     2      556
     3      939
     4      807
    Name: count, dtype: int64
    
    X3EVERDROP:
    X3EVERDROP
    0    20927
    1     2576
    Name: count, dtype: int64
    
    X3EVERDROP_IM:
    X3EVERDROP_IM
    0    23407
    2       96
    Name: count, dtype: int64



```python
# F2 wave - X4EVERDROP
print("=== F2 (2016) ===")
print("\nX4EVERDROP:")
print(df['X4EVERDROP'].value_counts().sort_index())
```

    === F2 (2016) ===
    
    X4EVERDROP:
    X4EVERDROP
    -9        3
    -8     6168
     0    14618
     1     2714
    Name: count, dtype: int64



```python
# Survey attrition across waves
print("=== SURVEY ATTRITION ===")
print(f"Base year total: {len(df)}")
print(f"F1 non-response (-8 in X2EVERDROP): not applicable - all coded")
print(f"F2 unit non-response (X4EVERDROP == -8): {(df['X4EVERDROP'] == -8).sum()}")
print(f"F2 missing (X4EVERDROP == -9): {(df['X4EVERDROP'] == -9).sum()}")
print(f"F2 usable (X4EVERDROP in 0,1): {df['X4EVERDROP'].isin([0,1]).sum()}")
```

    === SURVEY ATTRITION ===
    Base year total: 23503
    F1 non-response (-8 in X2EVERDROP): not applicable - all coded
    F2 unit non-response (X4EVERDROP == -8): 6168
    F2 missing (X4EVERDROP == -9): 3
    F2 usable (X4EVERDROP in 0,1): 17332



```python
# Cross-tabulate response patterns across all three follow-up waves
df['BY_resp'] = df['X1SQSTAT'].isin([1,2,3]).astype(int)
df['F1_resp'] = df['X2SQSTAT'].isin([1,2,3,4]).astype(int)
df['U13_resp'] = df['X3SQSTAT'].isin([1,2,3,4,5]).astype(int)
df['F2_resp'] = df['X4SQSTAT'].isin([1,2,3,4]).astype(int)

patterns = df.groupby(['BY_resp', 'F1_resp', 'U13_resp', 'F2_resp']).size().reset_index(name='count')
patterns['pattern'] = (patterns['BY_resp'].astype(str) + '-' + 
                       patterns['F1_resp'].astype(str) + '-' + 
                       patterns['U13_resp'].astype(str) + '-' + 
                       patterns['F2_resp'].astype(str))
print(patterns[['pattern', 'count']].sort_values('count', ascending=False).to_string())
```

        pattern  count
    15  1-1-1-1  13283
    14  1-1-1-0   2574
    13  1-1-0-1   1430
    12  1-1-0-0   1336
    8   1-0-0-0   1162
    7   0-1-1-1   1152
    11  1-0-1-1    797
    10  1-0-1-0    463
    9   1-0-0-1    399
    4   0-1-0-0    305
    6   0-1-1-0    273
    5   0-1-0-1    241
    0   0-0-0-0     49
    1   0-0-0-1     23
    3   0-0-1-1     10
    2   0-0-1-0      6


    /var/folders/zq/n2ytqrdj68x900tvyf0nfcyw0000gn/T/ipykernel_53853/2201967787.py:2: PerformanceWarning: DataFrame is highly fragmented.  This is usually the result of calling `frame.insert` many times, which has poor performance.  Consider joining all columns at once using pd.concat(axis=1) instead. To get a de-fragmented frame, use `newframe = frame.copy()`
      df['BY_resp'] = df['X1SQSTAT'].isin([1,2,3]).astype(int)
    /var/folders/zq/n2ytqrdj68x900tvyf0nfcyw0000gn/T/ipykernel_53853/2201967787.py:3: PerformanceWarning: DataFrame is highly fragmented.  This is usually the result of calling `frame.insert` many times, which has poor performance.  Consider joining all columns at once using pd.concat(axis=1) instead. To get a de-fragmented frame, use `newframe = frame.copy()`
      df['F1_resp'] = df['X2SQSTAT'].isin([1,2,3,4]).astype(int)
    /var/folders/zq/n2ytqrdj68x900tvyf0nfcyw0000gn/T/ipykernel_53853/2201967787.py:4: PerformanceWarning: DataFrame is highly fragmented.  This is usually the result of calling `frame.insert` many times, which has poor performance.  Consider joining all columns at once using pd.concat(axis=1) instead. To get a de-fragmented frame, use `newframe = frame.copy()`
      df['U13_resp'] = df['X3SQSTAT'].isin([1,2,3,4,5]).astype(int)
    /var/folders/zq/n2ytqrdj68x900tvyf0nfcyw0000gn/T/ipykernel_53853/2201967787.py:5: PerformanceWarning: DataFrame is highly fragmented.  This is usually the result of calling `frame.insert` many times, which has poor performance.  Consider joining all columns at once using pd.concat(axis=1) instead. To get a de-fragmented frame, use `newframe = frame.copy()`
      df['F2_resp'] = df['X4SQSTAT'].isin([1,2,3,4]).astype(int)



```python
usable = df[
    (df['BY_resp'] == 1) & 
    (df['X4EVERDROP'].isin([0, 1]))
]
print(f"Base year respondents with valid X4EVERDROP: {len(usable)}")
print(f"\nDropout rate: {round(usable['X4EVERDROP'].mean() * 100, 1)}%")
```

    Base year respondents with valid X4EVERDROP: 15906
    
    Dropout rate: 15.4%



```python
usable_im = usable[usable['X3EVERDROP_IM'] == 2]
print(f"Imputed students in usable sample: {len(usable_im)}")
print(f"\nX4EVERDROP distribution for imputed students:")
print(usable_im['X4EVERDROP'].value_counts().sort_index())
```

    Imputed students in usable sample: 61
    
    X4EVERDROP distribution for imputed students:
    X4EVERDROP
    1    61
    Name: count, dtype: int64



```python
# Response mapping from base year respondents only
by_respondents = df[df['BY_resp'] == 1]
print(f"Base year respondents: {len(by_respondents)}")
print(f"\nOf those, F1 response:")
print(by_respondents['F1_resp'].value_counts().sort_index())
print(f"\nOf those, U13 response:")
print(by_respondents['U13_resp'].value_counts().sort_index())
print(f"\nOf those, F2 response:")
print(by_respondents['F2_resp'].value_counts().sort_index())
```

    Base year respondents: 21444
    
    Of those, F1 response:
    F1_resp
    0     2821
    1    18623
    Name: count, dtype: int64
    
    Of those, U13 response:
    U13_resp
    0     4327
    1    17117
    Name: count, dtype: int64
    
    Of those, F2 response:
    F2_resp
    0     5535
    1    15909
    Name: count, dtype: int64



```python
# Dropout mapping - X2DROPSTAT for base year respondents only
print("X2DROPSTAT for base year respondents:")
print(by_respondents['X2DROPSTAT'].value_counts().sort_index())
print("\nX2EVERDROP for base year respondents:")
print(by_respondents['X2EVERDROP'].value_counts().sort_index())
```

    X2DROPSTAT for base year respondents:
    X2DROPSTAT
    0    19056
    1      515
    2      122
    3     1117
    4      483
    9      151
    Name: count, dtype: int64
    
    X2EVERDROP for base year respondents:
    X2EVERDROP
    0    19690
    1     1754
    Name: count, dtype: int64



```python
# X3DROPSTAT and X3EVERDROP for base year respondents
print("X3DROPSTAT for base year respondents:")
print(by_respondents['X3DROPSTAT'].value_counts().sort_index())
print("\nX3EVERDROP for base year respondents:")
print(by_respondents['X3EVERDROP'].value_counts().sort_index())
```

    X3DROPSTAT for base year respondents:
    X3DROPSTAT
    -8     4327
     0    14381
     1      643
     2      508
     3      854
     4      731
    Name: count, dtype: int64
    
    X3EVERDROP for base year respondents:
    X3EVERDROP
    0    19136
    1     2308
    Name: count, dtype: int64



```python
# Where are the 96 imputed students in F1?
imputed_all = df[df['X3EVERDROP_IM'] == 2]
print(f"Total imputed in X3EVERDROP: {len(imputed_all)}")
print(f"\nTheir F1 response pattern:")
print(imputed_all['F1_resp'].value_counts().sort_index())
print(f"\nTheir X2EVERDROP:")
print(imputed_all['X2EVERDROP'].value_counts().sort_index())
print(f"\nTheir X2DROPSTAT:")
print(imputed_all['X2DROPSTAT'].value_counts().sort_index())
```

    Total imputed in X3EVERDROP: 96
    
    Their F1 response pattern:
    F1_resp
    0    38
    1    58
    Name: count, dtype: int64
    
    Their X2EVERDROP:
    X2EVERDROP
    0    96
    Name: count, dtype: int64
    
    Their X2DROPSTAT:
    X2DROPSTAT
    0    86
    4     3
    9     7
    Name: count, dtype: int64



```python
# X4EVERDROP composition for final usable sample
# Breaking down the 1s in X4EVERDROP by where they came from
usable_no_imputed = usable[usable['X3EVERDROP_IM'] != 2]
print(f"Final working sample (excl. imputed): {len(usable_no_imputed)}")
print(f"\nX4EVERDROP distribution:")
print(usable_no_imputed['X4EVERDROP'].value_counts().sort_index())
print(f"\nDropout rate: {round(usable_no_imputed['X4EVERDROP'].mean() * 100, 1)}%")

# Of the dropouts, how many were already X2EVERDROP=1?
dropouts = usable_no_imputed[usable_no_imputed['X4EVERDROP'] == 1]
print(f"\nOf {len(dropouts)} dropouts in final sample:")
print(f"Already dropout at X2: {(dropouts['X2EVERDROP'] == 1).sum()}")
print(f"Not dropout at X2: {(dropouts['X2EVERDROP'] == 0).sum()}")
```

    Final working sample (excl. imputed): 15845
    
    X4EVERDROP distribution:
    X4EVERDROP
    0    13462
    1     2383
    Name: count, dtype: int64
    
    Dropout rate: 15.0%
    
    Of 2383 dropouts in final sample:
    Already dropout at X2: 1248
    Not dropout at X2: 1135



```python
print(f"BY=1 and F2=1: {len(usable)}")
print(f"Of those, X3EVERDROP_IM == 2: {(usable['X3EVERDROP_IM'] == 2).sum()}")
print(f"Of those, X3EVERDROP_IM == 0: {(usable['X3EVERDROP_IM'] == 0).sum()}")
print(f"Of those, X3EVERDROP_IM NaN or other: {usable['X3EVERDROP_IM'].isna().sum()}")
print(f"\nX3EVERDROP_IM value counts in usable sample:")
print(usable['X3EVERDROP_IM'].value_counts(dropna=False).sort_index())
```

    BY=1 and F2=1: 15906
    Of those, X3EVERDROP_IM == 2: 61
    Of those, X3EVERDROP_IM == 0: 15845
    Of those, X3EVERDROP_IM NaN or other: 0
    
    X3EVERDROP_IM value counts in usable sample:
    X3EVERDROP_IM
    0    15845
    2       61
    Name: count, dtype: int64



```python
by_respondents['X3DROP_source'] = 'non-dropout'

by_respondents.loc[by_respondents['X2EVERDROP'] == 1, 'X3DROP_source'] = 'carried from X2'
by_respondents.loc[
    (by_respondents['X2EVERDROP'] == 0) & 
    (by_respondents['X3DROPSTAT'] == 1), 'X3DROP_source'] = 'new X3 dropout'
by_respondents.loc[
    (by_respondents['X2EVERDROP'] == 0) & 
    (by_respondents['X3DROPSTAT'] == 2), 'X3DROP_source'] = 'new X3 alt completer'

print(by_respondents['X3DROP_source'].value_counts())
```

    X3DROP_source
    non-dropout             19136
    carried from X2          1754
    new X3 dropout            294
    new X3 alt completer      260
    Name: count, dtype: int64


    /var/folders/zq/n2ytqrdj68x900tvyf0nfcyw0000gn/T/ipykernel_53853/2555112472.py:1: PerformanceWarning: DataFrame is highly fragmented.  This is usually the result of calling `frame.insert` many times, which has poor performance.  Consider joining all columns at once using pd.concat(axis=1) instead. To get a de-fragmented frame, use `newframe = frame.copy()`
      by_respondents['X3DROP_source'] = 'non-dropout'



```python
print(1754+294+260)
```

    2308



```python
# Start with final working sample (excl. imputed)
ws = usable_no_imputed.copy()

# Classify each dropout in the final sample by when they first dropped out
ws['dropout_source'] = 'never dropped out'

# Carried from X2 (confirmed at F1)
ws.loc[
    (ws['X4EVERDROP'] == 1) & 
    (ws['X2EVERDROP'] == 1), 
    'dropout_source'] = '1. Confirmed at F1 (X2EVERDROP=1)'

# New at U13 - spring 2013 dropout
ws.loc[
    (ws['X4EVERDROP'] == 1) & 
    (ws['X2EVERDROP'] == 0) & 
    (ws['X3DROPSTAT'] == 1), 
    'dropout_source'] = '2. New at U13 - spring 2013 dropout'

# New at U13 - alternative completer
ws.loc[
    (ws['X4EVERDROP'] == 1) & 
    (ws['X2EVERDROP'] == 0) & 
    (ws['X3DROPSTAT'] == 2), 
    'dropout_source'] = '3. New at U13 - alternative completer (GED)'

# New at F2 - reported dropout in F2 interview
ws.loc[
    (ws['X4EVERDROP'] == 1) & 
    (ws['X2EVERDROP'] == 0) & 
    (~ws['X3DROPSTAT'].isin([1, 2])), 
    'dropout_source'] = '4. New at F2 - reported in F2 interview'

print(ws['dropout_source'].value_counts().sort_index())
print(f"\nTotal: {len(ws)}")
```

    dropout_source
    1. Confirmed at F1 (X2EVERDROP=1)               1248
    2. New at U13 - spring 2013 dropout              159
    3. New at U13 - alternative completer (GED)      181
    4. New at F2 - reported in F2 interview          795
    never dropped out                              13462
    Name: count, dtype: int64
    
    Total: 15845



```python
# Percentage breakdown
counts = ws['dropout_source'].value_counts().sort_index()
total_dropouts = (ws['X4EVERDROP'] == 1).sum()
total = len(ws)

print("=== CONTRIBUTION TO FINAL DROPOUT COUNT ===")
for source, count in counts.items():
    if source != 'never dropped out':
        pct_of_dropouts = round(count / total_dropouts * 100, 1)
        pct_of_sample = round(count / total * 100, 1)
        print(f"{source}")
        print(f"  Count: {count} | % of all dropouts: {pct_of_dropouts}% | % of total sample: {pct_of_sample}%")

print(f"\nTotal dropouts: {total_dropouts} ({round(total_dropouts/total*100,1)}% of sample)")
print(f"Never dropped out: {counts['never dropped out']} ({round(counts['never dropped out']/total*100,1)}% of sample)")
```

    === CONTRIBUTION TO FINAL DROPOUT COUNT ===
    1. Confirmed at F1 (X2EVERDROP=1)
      Count: 1248 | % of all dropouts: 52.4% | % of total sample: 7.9%
    2. New at U13 - spring 2013 dropout
      Count: 159 | % of all dropouts: 6.7% | % of total sample: 1.0%
    3. New at U13 - alternative completer (GED)
      Count: 181 | % of all dropouts: 7.6% | % of total sample: 1.1%
    4. New at F2 - reported in F2 interview
      Count: 795 | % of all dropouts: 33.4% | % of total sample: 5.0%
    
    Total dropouts: 2383 (15.0% of sample)
    Never dropped out: 13462 (85.0% of sample)



```python
# Cumulative dropout count building up wave by wave
print("=== CUMULATIVE DROPOUT BUILD-UP ===")
after_f1 = (ws['X2EVERDROP'] == 1).sum()
after_u13 = after_f1 + (ws['dropout_source'] == '2. New at U13 - spring 2013 dropout').sum() + \
            (ws['dropout_source'] == '3. New at U13 - alternative completer (GED)').sum()
after_f2 = (ws['X4EVERDROP'] == 1).sum()

print(f"After F1  (spring 2012): {after_f1} dropouts ({round(after_f1/total*100,1)}%)")
print(f"After U13 (2013):        {after_u13} dropouts ({round(after_u13/total*100,1)}%)")
print(f"After F2  (2016):        {after_f2} dropouts ({round(after_f2/total*100,1)}%)")
print(f"\nAdded at U13: {after_u13 - after_f1}")
print(f"Added at F2:  {after_f2 - after_u13}")
```

    === CUMULATIVE DROPOUT BUILD-UP ===
    After F1  (spring 2012): 1248 dropouts (7.9%)
    After U13 (2013):        1588 dropouts (10.0%)
    After F2  (2016):        2383 dropouts (15.0%)
    
    Added at U13: 340
    Added at F2:  795



```python
# Full classification separating dropouts and GED at each wave
ws['detailed_source'] = 'never dropped out'

# F1 confirmed - split by type
ws.loc[
    (ws['X4EVERDROP'] == 1) & 
    (ws['X2EVERDROP'] == 1) & 
    (ws['X2DROPSTAT'] == 2), 
    'detailed_source'] = '1a. F1 - alternative completer (GED)'

ws.loc[
    (ws['X4EVERDROP'] == 1) & 
    (ws['X2EVERDROP'] == 1) & 
    (ws['X2DROPSTAT'] != 2), 
    'detailed_source'] = '1b. F1 - traditional dropout'

# U13 new additions
ws.loc[
    (ws['X4EVERDROP'] == 1) & 
    (ws['X2EVERDROP'] == 0) & 
    (ws['X3DROPSTAT'] == 2), 
    'detailed_source'] = '2a. U13 - alternative completer (GED)'

ws.loc[
    (ws['X4EVERDROP'] == 1) & 
    (ws['X2EVERDROP'] == 0) & 
    (ws['X3DROPSTAT'] == 1), 
    'detailed_source'] = '2b. U13 - traditional dropout'

# F2 new additions - split by S4DROPOUTHS and check for GED
ws.loc[
    (ws['X4EVERDROP'] == 1) & 
    (ws['X2EVERDROP'] == 0) & 
    (~ws['X3DROPSTAT'].isin([1, 2])), 
    'detailed_source'] = '3. F2 - reported in F2 interview'

print(ws['detailed_source'].value_counts().sort_index())
print(f"\nTotal: {len(ws)}")
```

    detailed_source
    1a. F1 - alternative completer (GED)        99
    1b. F1 - traditional dropout              1149
    2a. U13 - alternative completer (GED)      181
    2b. U13 - traditional dropout              159
    3. F2 - reported in F2 interview           795
    never dropped out                        13462
    Name: count, dtype: int64
    
    Total: 15845



```python
# GED vs traditional dropout totals
ged = ws['detailed_source'].isin([
    '1a. F1 - alternative completer (GED)',
    '2a. U13 - alternative completer (GED)'
]).sum()

traditional = ws['detailed_source'].isin([
    '1b. F1 - traditional dropout',
    '2b. U13 - traditional dropout',
    '3. F2 - reported in F2 interview'
]).sum()

print(f"Total GED/alternative completers: {ged} ({round(ged/2383*100,1)}% of dropouts)")
print(f"Total traditional dropouts: {traditional} ({round(traditional/2383*100,1)}% of dropouts)")
print(f"Total dropouts: {ged + traditional}")
```

    Total GED/alternative completers: 280 (11.7% of dropouts)
    Total traditional dropouts: 2103 (88.3% of dropouts)
    Total dropouts: 2383



```python
# Can we split F2 additions into GED vs traditional?
print("X4EVERGED for F2-only dropouts:")
f2_only = ws[ws['detailed_source'] == '3. F2 - reported in F2 interview']
print(f2_only['X4EVERGED'].value_counts(dropna=False).sort_index())
```

    X4EVERGED for F2-only dropouts:
    X4EVERGED
    0    657
    1    138
    Name: count, dtype: int64



```python
# Cumulative build-up split by type
print("=== CUMULATIVE BUILD-UP BY TYPE ===")
print(f"\nAfter F1 (spring 2012):")
print(f"  Traditional dropout: {(ws['detailed_source'] == '1b. F1 - traditional dropout').sum()}")
print(f"  GED completer:       {(ws['detailed_source'] == '1a. F1 - alternative completer (GED)').sum()}")
print(f"  Total:               {(ws['X2EVERDROP'] == 1).sum()}")

print(f"\nAdded at U13 (2013):")
print(f"  Traditional dropout: {(ws['detailed_source'] == '2b. U13 - traditional dropout').sum()}")
print(f"  GED completer:       {(ws['detailed_source'] == '2a. U13 - alternative completer (GED)').sum()}")
print(f"  Total added:         340")

print(f"\nAdded at F2 (2016):")
print(f"  F2 reported:         {(ws['detailed_source'] == '3. F2 - reported in F2 interview').sum()}")
```

    === CUMULATIVE BUILD-UP BY TYPE ===
    
    After F1 (spring 2012):
      Traditional dropout: 1149
      GED completer:       99
      Total:               1248
    
    Added at U13 (2013):
      Traditional dropout: 159
      GED completer:       181
      Total added:         340
    
    Added at F2 (2016):
      F2 reported:         795



```python
# Complete classification including F2 GED split
ws['final_source'] = ws['detailed_source']

ws.loc[
    (ws['detailed_source'] == '3. F2 - reported in F2 interview') &
    (ws['X4EVERGED'] == 1),
    'final_source'] = '3a. F2 - alternative completer (GED)'

ws.loc[
    (ws['detailed_source'] == '3. F2 - reported in F2 interview') &
    (ws['X4EVERGED'] != 1),
    'final_source'] = '3b. F2 - traditional dropout'

print(ws['final_source'].value_counts().sort_index())
print(f"\nTotal: {len(ws)}")

# Full GED vs traditional split
total_ged = ws['final_source'].isin([
    '1a. F1 - alternative completer (GED)',
    '2a. U13 - alternative completer (GED)',
    '3a. F2 - alternative completer (GED)'
]).sum()

total_trad = ws['final_source'].isin([
    '1b. F1 - traditional dropout',
    '2b. U13 - traditional dropout',
    '3b. F2 - traditional dropout'
]).sum()

print(f"\nFinal GED total across all waves: {total_ged} ({round(total_ged/2383*100,1)}% of dropouts)")
print(f"Final traditional dropout total:  {total_trad} ({round(total_trad/2383*100,1)}% of dropouts)")
print(f"Sum: {total_ged + total_trad}")

# Cumulative build-up full picture
print("\n=== FULL CUMULATIVE BUILD-UP ===")
trad_f1 = 1149
ged_f1 = 99
trad_u13 = 159
ged_u13 = 181
trad_f2 = (ws['final_source'] == '3b. F2 - traditional dropout').sum()
ged_f2 = (ws['final_source'] == '3a. F2 - alternative completer (GED)').sum()

print(f"\nAfter F1  (spring 2012): {trad_f1 + ged_f1} total | {trad_f1} traditional | {ged_f1} GED")
print(f"Added U13 (2013):         {trad_u13 + ged_u13} total | {trad_u13} traditional | {ged_u13} GED")
print(f"After U13 (2013):         {trad_f1+ged_f1+trad_u13+ged_u13} total | {trad_f1+trad_u13} traditional | {ged_f1+ged_u13} GED")
print(f"Added F2  (2016):         {trad_f2 + ged_f2} total | {trad_f2} traditional | {ged_f2} GED")
print(f"After F2  (2016):         {trad_f1+ged_f1+trad_u13+ged_u13+trad_f2+ged_f2} total | {trad_f1+trad_u13+trad_f2} traditional | {ged_f1+ged_u13+ged_f2} GED")
```

    final_source
    1a. F1 - alternative completer (GED)        99
    1b. F1 - traditional dropout              1149
    2a. U13 - alternative completer (GED)      181
    2b. U13 - traditional dropout              159
    3a. F2 - alternative completer (GED)       138
    3b. F2 - traditional dropout               657
    never dropped out                        13462
    Name: count, dtype: int64
    
    Total: 15845
    
    Final GED total across all waves: 418 (17.5% of dropouts)
    Final traditional dropout total:  1965 (82.5% of dropouts)
    Sum: 2383
    
    === FULL CUMULATIVE BUILD-UP ===
    
    After F1  (spring 2012): 1248 total | 1149 traditional | 99 GED
    Added U13 (2013):         340 total | 159 traditional | 181 GED
    After U13 (2013):         1588 total | 1308 traditional | 280 GED
    Added F2  (2016):         795 total | 657 traditional | 138 GED
    After F2  (2016):         2383 total | 1965 traditional | 418 GED



```python
# Get the 15,906 students satisfying conditions 1 and 2
pre_imputation = df[
    (df['X1SQSTAT'].isin([1, 2, 3])) &
    (df['X4EVERDROP'].isin([0, 1]))
].copy()

print(f"Pre-imputation sample: {len(pre_imputation)}")

print("\nX2DROPSTAT:")
print(pre_imputation['X2DROPSTAT'].value_counts().sort_index())

print("\nX2EVERDROP:")
print(pre_imputation['X2EVERDROP'].value_counts().sort_index())

print("\nX3DROPSTAT:")
print(pre_imputation['X3DROPSTAT'].value_counts().sort_index())

print("\nX3EVERDROP:")
print(pre_imputation['X3EVERDROP'].value_counts().sort_index())
```

    Pre-imputation sample: 15906
    
    X2DROPSTAT:
    X2DROPSTAT
    0    14406
    1      321
    2       99
    3      828
    4      202
    9       50
    Name: count, dtype: int64
    
    X2EVERDROP:
    X2EVERDROP
    0    14658
    1     1248
    Name: count, dtype: int64
    
    X3DROPSTAT:
    X3DROPSTAT
    -8     1829
     0    11951
     1      460
     2      399
     3      681
     4      586
    Name: count, dtype: int64
    
    X3EVERDROP:
    X3EVERDROP
    0    14257
    1     1649
    Name: count, dtype: int64



```python
print(f"X3EVERDROP = 1 in pre-imputation sample: {(pre_imputation['X3EVERDROP'] == 1).sum()}")
print(f"X4EVERDROP = 1 in pre-imputation sample: {(pre_imputation['X4EVERDROP'] == 1).sum()}")
print(f"X4EVERDROP = 0 in pre-imputation sample: {(pre_imputation['X4EVERDROP'] == 0).sum()}")
```

    X3EVERDROP = 1 in pre-imputation sample: 1649
    X4EVERDROP = 1 in pre-imputation sample: 2444
    X4EVERDROP = 0 in pre-imputation sample: 13462



```python

```
