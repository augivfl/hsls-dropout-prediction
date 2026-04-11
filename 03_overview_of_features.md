```python
import re
import pandas as pd

CODEBOOK = "Codebook_HSLS_17_PETS.txt"
```


```python
text = open(CODEBOOK, encoding="utf-8-sig").read()
entries = re.split(r"-{80,}", text)
print(f"Number of blocks: {len(entries)}")
```

    Number of blocks: 20603



```python
# Find a block containing a variable you know well
for i, entry in enumerate(entries):
    if "Name:       X1SES" in entry:
        print(f"Block index: {i}")
        print(entry[:1500])
        break
```

    Block index: 210
    
    
    
    File:       STUDENT
    Name:       X1SES
    Position:   587
    Length:     7
    Label:      X1 Socio-economic status composite
    
    Description:
    This composite variable is used to measure a construct for socioeconomic status. X1SES is calculated using parent/guardians' education (X1PAR1EDU and X1PAR2EDU), occupation (X1PAR1OCC2 and X1PAR2OCC2), and family income (X1FAMINCOME). For cases with nonresponding parent/guardians, 5 imputed values are generated (X1SES1-X1SES5), X1SES is computed as the average of the 5 imputed values, and the imputation flag is set as X1SES_IM=1 (values for parent/guardian education, occupation, and income are set to -8). When education, occupation, or family income are imputed using other information provided by the responding parent/guardian, X1SES is constructed from the combination of actual and imputed parent/guardian values. For these cases, the values of X1SES1-X1SES5 are equivalent to X1SES and X1SES_IM=2. Otherwise, the responding parent/guardian provided responses for all input variables so that the values of X1SES1-X1SES5 are again equivalent to X1SES and X1SES_IM=0. For more information on this variable, please refer to section 7.3.2.2 and appendix k of the HSLS:09 Base-Year Data File Documentation (NCES 2011-328).
    
    
                                                                               Mean       Std Deviation 
    Category                            Min                 Max          Unweighted          Unweighted 
    ------------------- ----------------



```python
for i, entry in enumerate(entries):
    if "Name:       X3DROPSTAT" in entry:
        print(entry[:2000])
        break
```

    
    
    
    File:       STUDENT
    Name:       X3DROPSTAT
    Position:   1352
    Length:     2
    Label:      X3 U13 dropout status
    
    Description:
    Sample member status of dropout from high school.
    
    
    SAS Logic:
    if X3SQSTAT=8 then X3DROPSTAT=-8;
    else if X3HSCRED=1 and X3HSCREDTYPE=1 then X3DROPSTAT=0;
    else if X3HSCRED=0 and ((201300<X3LASTHSDATE<201304) or 0<X3LASTHSDATE<201300) then X3DROPSTAT=1;
    else if X3HSCRED=1 and X3HSCREDTYPE in (2,3) then X3DROPSTAT=2;
    else X3DROPSTAT=4;
    if X3DROPSTAT in (0,4) and X2EVERDROP=1 then X3DROPSTAT=3;
    Sources: 2013 update questionnaire
    
                                                                            Frequency             Percent 
    Category            Label                                              Unweighted          Unweighted 
    ------------------- ----------------------------------------- ------------------- ------------------- 
    0                   Not dropout/alternative completer                      15,535               66.10 
    1                   Spring term 2013 dropout                                  721                3.07 
    2                   Alternative completer                                     556                2.37 
    3                   Student/parent/prior school report of dr                  939                4.00 
                        opout episode
    4                   Status unknown                                            807                3.43 
    -8                  Unit non-response                                       4,945               21.04 
    TOTAL                                                                      23,503              100.01 
    
    
    



```python
def parse_entry(entry):
    name_m    = re.search(r"Name:\s+(\S+)", entry)
    label_m   = re.search(r"Label:\s+(.+)", entry)
    desc_m    = re.search(r"Description:\s*\n(.*?)(?=SAS Logic:|Sources:|Category\s+Label|Mean\s+Std|\Z)", entry, re.DOTALL)
    sas_m     = re.search(r"SAS Logic:\s*\n(.*?)(?=Sources:|Category\s+Label|Mean\s+Std|\Z)", entry, re.DOTALL)
    src_m     = re.search(r"Sources?:\s*(.+?)(?:\n\n|\Z)", entry, re.DOTALL)

    if not name_m:
        return None

    return {
        "variable":    name_m.group(1).strip(),
        "label":       label_m.group(1).strip() if label_m else "",
        "description": desc_m.group(1).strip()  if desc_m  else "",
        "sas_logic":   sas_m.group(1).strip()   if sas_m   else "",
        "sources":     src_m.group(1).strip()    if src_m   else "",
    }

parsed = [parse_entry(e) for e in entries]
parsed = [p for p in parsed if p is not None]
print(f"Successfully parsed: {len(parsed)}")
```

    Successfully parsed: 10301



```python
# Check a description composite
x1ses = next(p for p in parsed if p["variable"] == "X1SES")
print("=== X1SES ===")
print(f"Label:       {x1ses['label']}")
print(f"Description: {x1ses['description'][:200]}")
print(f"SAS Logic:   '{x1ses['sas_logic']}'")
print(f"Sources:     {x1ses['sources']}")

print()

# Check a SAS composite
x3drop = next(p for p in parsed if p["variable"] == "X3DROPSTAT")
print("=== X3DROPSTAT ===")
print(f"Label:       {x3drop['label']}")
print(f"Description: {x3drop['description'][:200]}")
print(f"SAS Logic:   {x3drop['sas_logic'][:200]}")
print(f"Sources:     {x3drop['sources']}")
```

    === X1SES ===
    Label:       X1 Socio-economic status composite
    Description: This composite variable is used to measure a construct for socioeconomic status. X1SES is calculated using parent/guardians' education (X1PAR1EDU and X1PAR2EDU), occupation (X1PAR1OCC2 and X1PAR2OCC2)
    SAS Logic:   ''
    Sources:     
    
    === X3DROPSTAT ===
    Label:       X3 U13 dropout status
    Description: Sample member status of dropout from high school.
    SAS Logic:   if X3SQSTAT=8 then X3DROPSTAT=-8;
    else if X3HSCRED=1 and X3HSCREDTYPE=1 then X3DROPSTAT=0;
    else if X3HSCRED=0 and ((201300<X3LASTHSDATE<201304) or 0<X3LASTHSDATE<201300) then X3DROPSTAT=1;
    else if X3H
    Sources:     2013 update questionnaire



```python
col_names = open("hsls_column_names.txt").read().strip().split("\n")

parsed_names = {p["variable"] for p in parsed}
missing = [v for v in col_names if v not in parsed_names]

print(f"Your variables:     {len(col_names)}")
print(f"Found in codebook:  {len(col_names) - len(missing)}")
print(f"Missing:            {len(missing)}")
print(missing)
```

    Your variables:     913
    Found in codebook:  913
    Missing:            0
    []



```python
codebook_df = pd.DataFrame(parsed)
codebook_df = codebook_df[codebook_df["variable"].isin(col_names)].reset_index(drop=True)
print(codebook_df.shape)
codebook_df.head()
```

    (1125, 5)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>variable</th>
      <th>label</th>
      <th>description</th>
      <th>sas_logic</th>
      <th>sources</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>STU_ID</td>
      <td>Student ID</td>
      <td>Student identifier assigned for all base year ...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>1</th>
      <td>X1SEX</td>
      <td>X1 Student's sex</td>
      <td>Sex of the sample member, taken from the base ...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>2</th>
      <td>X1RACE</td>
      <td>X1 Student's race/ethnicity-composite</td>
      <td>X1RACE characterizes the sample member's race/...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>3</th>
      <td>X1HISPANIC</td>
      <td>X1 Student is Hispanic/Latino/Latina-composite</td>
      <td>The sample member's race/ethnicity is characte...</td>
      <td></td>
      <td></td>
    </tr>
    <tr>
      <th>4</th>
      <td>X1WHITE</td>
      <td>X1 Student is White-composite</td>
      <td>The sample member's race/ethnicity is characte...</td>
      <td></td>
      <td></td>
    </tr>
  </tbody>
</table>
</div>




```python
dups = codebook_df[codebook_df.duplicated(subset="variable", keep=False)]
print(f"Duplicate variable names: {dups['variable'].nunique()}")
print(dups["variable"].value_counts().head(20))
```

    Duplicate variable names: 212
    variable
    X1CONTROL       2
    X1LOCALE        2
    X1REGION        2
    X1SCHOOLCLI     2
    X1COUPERTEA     2
    X1COUPERCOU     2
    X1COUPERPRI     2
    X1AQSTAT        2
    X1AQDATE        2
    X1AQDESIGNEE    2
    X1CQSTAT        2
    X1CQDATE        2
    A1SCHCONTROL    2
    A1NOTIFY        2
    A1MTHSCIFAIR    2
    A1MSSUMMER      2
    A1MSAFTERSCH    2
    A1MSMENTOR      2
    A1MSSPEAKER     2
    A1MSFLDTRIP     2
    Name: count, dtype: int64



```python
codebook_df = codebook_df.drop_duplicates(subset="variable", keep="first").reset_index(drop=True)
print(codebook_df.shape)
```

    (913, 5)



```python
def get_instrument(v):
    if v.startswith("X1"):   return "NCES_composite"
    if v.startswith("S1"):   return "student_BY"
    if v.startswith("P1"):   return "parent_BY"
    if v.startswith("M1"):   return "mathteacher_BY"
    if v.startswith("A1"):   return "admin_BY"
    if v.startswith("C1"):   return "counselor_BY"
    if v == "STU_ID":        return "identifier"
    if v == "X4EVERDROP":    return "target"
    return "other"

codebook_df["instrument"] = codebook_df["variable"].apply(get_instrument)
codebook_df["instrument"].value_counts()
```




    instrument
    student_BY        271
    NCES_composite    161
    counselor_BY      150
    parent_BY         141
    mathteacher_BY    134
    admin_BY           54
    identifier          1
    target              1
    Name: count, dtype: int64




```python
def get_construct_type(row):
    if row["sas_logic"]:
        return "sas_composite"
    if any(kw in row["description"].lower() for kw in [
        "calculated using", "inputs to", "constructed from",
        "derived from", "based on", "computed from",
        "created from", "uses ", "using "
    ]):
        return "description_composite"
    return "raw"

codebook_df["construct_type"] = codebook_df.apply(get_construct_type, axis=1)
codebook_df["construct_type"].value_counts()
```




    construct_type
    raw                      704
    description_composite    208
    sas_composite              1
    Name: count, dtype: int64




```python
# Tokens to ignore when scanning for variable name references
IGNORE_TOKENS = {
    "IF", "THEN", "ELSE", "AND", "OR", "NOT", "IN", "DO", "END",
    "NULL", "MISSING", "MAX", "MIN", "SUM", "MEAN", "ABS", "INT",
    "SUBSTR", "UPCASE", "PUT", "INPUT", "CAT", "CATS", "CATX",
    "LENGTH", "LABEL", "FORMAT", "INFORMAT", "RETAIN", "ARRAY",
    "OUTPUT", "RETURN", "STOP", "RUN", "DATA", "SET", "MERGE",
    "BY", "WHERE", "KEEP", "DROP", "RENAME", "CLASS", "MODEL",
    "VAR", "WEIGHT", "FREQ", "TABLES", "TRUE", "FALSE",
    "NCESID", "CCD", "PSS", "NCES", "GED", "IEP", "AP", "IB",
    "ONET", "STEM", "SAT", "ACT", "PSAT", "GPA",
}

VARNAME_RE = re.compile(r'\b([A-Z][A-Z0-9_]{2,})\b')
all_known  = set(codebook_df["variable"])

def extract_parents(row):
    text = row["sas_logic"] if row["sas_logic"] else row["description"]
    candidates = VARNAME_RE.findall(text)
    parents = set()
    for c in candidates:
        if c == row["variable"]:      continue
        if c in IGNORE_TOKENS:        continue
        if c not in all_known:        continue
        if not any(ch.isdigit() for ch in c): continue
        parents.add(c)
    return sorted(parents)

codebook_df["parents"] = codebook_df.apply(extract_parents, axis=1)
codebook_df["n_parents"] = codebook_df["parents"].apply(len)

# Verify on known variables
for v in ["X1SES", "X1MTHID", "X1SCHOOLBEL", "X3DROPSTAT"]:
    if v in all_known:
        row = codebook_df[codebook_df["variable"] == v].iloc[0]
        print(f"{v}: {row['parents']}")
```

    X1SES: ['X1FAMINCOME', 'X1PAR1EDU', 'X1PAR1OCC2', 'X1PAR2EDU', 'X1PAR2OCC2', 'X1SES1', 'X1SES5', 'X1SES_IM']
    X1MTHID: ['S1MPERSON1', 'S1MPERSON2']
    X1SCHOOLBEL: ['S1GOODGRADES', 'S1PROUD', 'S1SAFE', 'S1SCHWASTE', 'S1TALKPROB']



```python
def compute_depth_and_leaves(varname, memo, visiting):
    if varname in memo:
        return memo[varname]
    if varname in visiting:
        # cycle detected - break it
        return (0, {varname})
    
    visiting.add(varname)
    
    row = codebook_df[codebook_df["variable"] == varname]
    if row.empty:
        result = (0, {varname})
    else:
        parents = row.iloc[0]["parents"]
        if not parents:
            result = (0, {varname})
        else:
            child_results = [compute_depth_and_leaves(p, memo, visiting) for p in parents]
            max_depth = max(r[0] for r in child_results)
            all_leaves = set().union(*[r[1] for r in child_results])
            result = (max_depth + 1, all_leaves)
    
    visiting.remove(varname)
    memo[varname] = result
    return result

memo = {}
for v in codebook_df["variable"]:
    compute_depth_and_leaves(v, memo, set())

codebook_df["depth"]     = codebook_df["variable"].apply(lambda v: memo[v][0])
codebook_df["n_leaves"]  = codebook_df["variable"].apply(lambda v: len(memo[v][1]))

# Verify
for v in ["X1SES", "X1MTHID", "X1SCHOOLBEL", "S1GOODGRADES"]:
    if v in all_known:
        row = codebook_df[codebook_df["variable"] == v].iloc[0]
        print(f"{v}: depth={row['depth']}, n_leaves={row['n_leaves']}")
```

    X1SES: depth=5, n_leaves=16
    X1MTHID: depth=1, n_leaves=2
    X1SCHOOLBEL: depth=1, n_leaves=5
    S1GOODGRADES: depth=0, n_leaves=1



```python
codebook_df.to_csv("codebook_parsed.csv", index=False)
print(codebook_df.shape)
codebook_df.head()
```

    (913, 11)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>variable</th>
      <th>label</th>
      <th>description</th>
      <th>sas_logic</th>
      <th>sources</th>
      <th>instrument</th>
      <th>construct_type</th>
      <th>parents</th>
      <th>n_parents</th>
      <th>depth</th>
      <th>n_leaves</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>STU_ID</td>
      <td>Student ID</td>
      <td>Student identifier assigned for all base year ...</td>
      <td></td>
      <td></td>
      <td>identifier</td>
      <td>raw</td>
      <td>[]</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>X1SEX</td>
      <td>X1 Student's sex</td>
      <td>Sex of the sample member, taken from the base ...</td>
      <td></td>
      <td></td>
      <td>NCES_composite</td>
      <td>description_composite</td>
      <td>[]</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>X1RACE</td>
      <td>X1 Student's race/ethnicity-composite</td>
      <td>X1RACE characterizes the sample member's race/...</td>
      <td></td>
      <td></td>
      <td>NCES_composite</td>
      <td>description_composite</td>
      <td>[X1BLACK, X1HISPANIC, X1WHITE]</td>
      <td>3</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>X1HISPANIC</td>
      <td>X1 Student is Hispanic/Latino/Latina-composite</td>
      <td>The sample member's race/ethnicity is characte...</td>
      <td></td>
      <td></td>
      <td>NCES_composite</td>
      <td>description_composite</td>
      <td>[X1BLACK, X1RACE, X1WHITE]</td>
      <td>3</td>
      <td>2</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>X1WHITE</td>
      <td>X1 Student is White-composite</td>
      <td>The sample member's race/ethnicity is characte...</td>
      <td></td>
      <td></td>
      <td>NCES_composite</td>
      <td>description_composite</td>
      <td>[X1BLACK, X1HISPANIC, X1RACE]</td>
      <td>3</td>
      <td>1</td>
      <td>3</td>
    </tr>
  </tbody>
</table>
</div>




```python
import json
import os

SAVE_FILE = "tier1_assignments.json"

# Load existing progress if it exists
if os.path.exists(SAVE_FILE):
    assignments = json.load(open(SAVE_FILE))
else:
    assignments = {}

for _, row in codebook_df.iterrows():
    v = row["variable"]
    
    # Skip identifier and target
    if row["instrument"] in ("identifier", "target"):
        continue
    
    # Skip already assigned
    if v in assignments:
        continue
    
    print(f"\n{'='*60}")
    print(f"Variable:    {v}")
    print(f"Label:       {row['label']}")
    print(f"Instrument:  {row['instrument']}")
    print(f"Description: {row['description'][:400]}")
    
    tier1 = input("\nTier 1? (y/n/q to quit): ").strip().lower()
    if tier1 == "q":
        break
    
    certain = input("Certain? (y/n): ").strip().lower()
    
    assignments[v] = {
        "tier_1":   tier1 == "y",
        "certain":  certain == "y",
    }
    
    json.dump(assignments, open(SAVE_FILE, "w"), indent=2)

print(f"\nProgress: {len(assignments)}/911 variables assigned")

```

    
    Progress: 911/911 variables assigned



```python
assignments = json.load(open("tier1_assignments.json"))

for variable, decision in assignments.items():
    if decision["certain"]:
        continue
    
    print(f"\n{'='*60}")
    print(f"Variable: {variable}")
    print(f"Current:  tier_1={decision['tier_1']}")
    
    # Show label from codebook_df
    row = codebook_df[codebook_df["variable"] == variable]
    if not row.empty:
        print(f"Label:    {row.iloc[0]['label']}")
        print(f"Description: {row.iloc[0]['description'][:300]}")
    
    tier1 = input("\nTier 1? (y/n/q to quit): ").strip().lower()
    if tier1 == "q":
        break
    
    assignments[variable]["tier_1"] = (tier1 == "y")
    assignments[variable]["certain"] = True
    
    json.dump(assignments, open("tier1_assignments.json", "w"), indent=2)

print("Done")
```

    Done



```python
tier1 = json.load(open("tier1_assignments.json"))
tier1_vars = {k for k, v in tier1.items() if v["tier_1"]}

SAVE_FILE = "tier2_assignments.json"

if os.path.exists(SAVE_FILE):
    assignments = json.load(open(SAVE_FILE))
else:
    assignments = {}

last_variable = None

for _, row in codebook_df.iterrows():
    v = row["variable"]

    if row["instrument"] in ("identifier", "target"):
        continue
    if v in tier1_vars:
        continue
    if v in assignments:
        continue

    print(f"\n{'='*60}")
    print(f"Variable:    {v}")
    print(f"Label:       {row['label']}")
    print(f"Instrument:  {row['instrument']}")
    print(f"Description: {row['description'][:400]}")

    tier2 = input("\nTier 2? (y/n/r to redo last/q to quit): ").strip().lower()
    if tier2 == "q":
        break
    elif tier2 == "r":
        if last_variable:
            assignments[last_variable]["certain"] = False
            json.dump(assignments, open(SAVE_FILE, "w"), indent=2)
            print(f"Marked {last_variable} as uncertain — will reappear next run")
        else:
            print("Nothing to redo")
        continue

    certain = input("Certain? (y/n): ").strip().lower()

    assignments[v] = {
        "tier_2": tier2 == "y",
        "certain": certain == "y",
    }
    last_variable = v

    json.dump(assignments, open(SAVE_FILE, "w"), indent=2)

print(f"\nProgress: {len(assignments)}/{911 - len(tier1_vars)} variables assigned")
```

    
    ============================================================
    Variable:    C1CLGPREP
    Label:       C1 B27A School has counselor designated for college readiness/selection/apply
    Instrument:  counselor_BY
    Description: Does your school have one or more counselors whose primary responsibility is...
    assisting students with college readiness, selection, and applications?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1WORKFORCE
    Label:       C1 B27B School has counselor designated for workforce preparation/placement
    Instrument:  counselor_BY
    Description: Does your school have one or more counselors whose primary responsibility is...
    assisting students with preparation for and placement into the workforce?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1CLGFAIR
    Label:       C1 B28A School holds or participates in college fairs
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1POSTSECREQ
    Label:       C1 B28B School consults with postsecondary reps about requirement/qualifications
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1VISITCLG
    Label:       C1 B28C School organizes student visits to colleges
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1INFOSESSN
    Label:       C1 B28E School holds info session on transition to college for students/parents
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1FINANCEAID
    Label:       C1 B28F School assists students with finding financial aid for college
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1ASSISTOTH
    Label:       C1 B28I School takes other steps to assist with HS to college transition
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1CLUSTER
    Label:       C1 B30 Career Clusters/Pathways/Programs of Study (POS) offered
    Instrument:  counselor_BY
    Description: Are Career Clusters, Pathways, or Programs of Study (POS) offered to students in [your school]?
      Yes
      No
    
    Note: Question wording was customized in the survey instrument such that the respondent's school name appeared in place of 'your school'.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1INDVCRS
    Label:       C1 B31 Student not enrolled in Career Clusters etc. may take course in program
    Instrument:  counselor_BY
    Description: Can high school students who are not enrolled in Career Clusters, Pathways, or Programs of Study (POS) take individual courses in these programs?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1INTERN
    Label:       C1 B32A School offers internships with local employers
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBFAIR
    Label:       C1 B32B School offers job fairs
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBGUIDE
    Label:       C1 B32C School offers career guides or skills assessments
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1EMPLOYER
    Label:       C1 B32D School offers school/classroom presentations by local employers
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1AWARENESS
    Label:       C1 B32E School offers career awareness activities
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1CAREERUNIT
    Label:       C1 B32G School offers career information units in subject-matter courses
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1WORKSTUDY
    Label:       C1 B32H School offers exploratory work experience programs/co-op/workstudy/EBCE
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1CAREERDAY
    Label:       C1 B32I School offers career days or nights
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1ASSEMBLIES
    Label:       C1 B32J School offers vocational oriented assemblies and speakers in classes
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBVISIT
    Label:       C1 B32L School offers job site visits/field trips
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBSHADOW
    Label:       C1 B32M School offers job shadowing
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBTEST
    Label:       C1 B32O School offers tests for career planning purposes
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBINFOCMP
    Label:       C1 B32Q School offers computerized career information resources
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBINFONON
    Label:       C1 B32R School offers non-computerized career information resources
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1HSTOWRKOTH
    Label:       C1 B32S School assists students with transition from HS to work in other ways
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1HSTOWORKNO
    Label:       C1 B32T School doesn't assist students with transition from high school to work
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1G9MMSCNSL
    Label:       C1 C02A Importance of MS counselor recommendation for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Middle school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MHSCNSL
    Label:       C1 C02B Importance of HS counselor recommendation for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    High school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MMSTCHR
    Label:       C1 C02C Importance of MS teacher recommendation for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Middle school teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MMSCOURS
    Label:       C1 C02D Importance of courses taken in MS for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Courses taken in middle school
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                                                        
    
    ============================================================
    Variable:    C1G9MMSACHV
    Label:       C1 C02E Importance of achievement in MS courses for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Achievement in middle school courses
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                                                  
    
    ============================================================
    Variable:    C1G9MENDTST
    Label:       C1 C02F Importance of end-of-year/course test for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Results of district or state end-of-year or end-of-course exams
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MPLACTST
    Label:       C1 C02G Importance of placement tests for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MSTNDTST
    Label:       C1 C02H Importance of standardized tests for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Results of standardized tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MPLAN
    Label:       C1 C02I Importance of career/education plan for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MSELECT
    Label:       C1 C02J Importance of student/parent choice for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMGRADES
    Label:       C1 C04A Importance of prior grades for 10th to 12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Prior grades including grades from a prerequisite class
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" and "A little important" recoded as "Not at all important, a little important, or somewhat important" on the public
    
    ============================================================
    Variable:    C1UPMPLACTST
    Label:       C1 C04B Importance of placement tests for 10th to 12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMTCHR
    Label:       C1 C04C Importance of teacher's recommendation for 10-12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Previous year's teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMSELECT
    Label:       C1 C04D Importance of student/parent choice for 10th-12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMPLAN
    Label:       C1 C04E Importance of career/education plan for 10th-12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMSCHED
    Label:       C1 C04F Importance of master schedule for 10th to 12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Master schedule considerations
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSCNSL
    Label:       C1 C06A Importance of MS counselor recommendation for grade 9 science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Middle school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SHSCNSL
    Label:       C1 C06B Importance of HS counselor recommendation for grade 9 science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    High school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSTCHR
    Label:       C1 C06C Importance of MS teacher recommendation for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Middle school teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSCOURS
    Label:       C1 C06D Importance of courses taken in MS for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Courses taken in middle school
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSACHV
    Label:       C1 C06E Importance of achievement in MS courses for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Achievement in middle school courses
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SENDTST
    Label:       C1 C06F Importance of end-of-year/course test for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Results of district or state end-of-year or end-of-course exams
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SPLACTST
    Label:       C1 C06G Importance of placement tests for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SSTNDTST
    Label:       C1 C06H Importance of standardized tests for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Results of standardized tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SPLAN
    Label:       C1 C06I Importance of career/education plan for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SSELECT
    Label:       C1 C06J Importance of student/parent choice for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSGRADES
    Label:       C1 C08A Importance of prior grades for 10th to 12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Prior grades including grades from a prerequisite class
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                          
    
    ============================================================
    Variable:    C1UPSPLACTST
    Label:       C1 C08B Importance of placement tests for 10th to 12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSTCHR
    Label:       C1 C08C Importance of teacher's recommendation for 10th-12th science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Previous year's teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSSELECT
    Label:       C1 C08D Importance of student/parent choice for 10-12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                                      
    
    ============================================================
    Variable:    C1UPSPLAN
    Label:       C1 C08E Importance of career/education plan for 10-12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSSCHED
    Label:       C1 C08F Importance of master schedule for 10th to 12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Master schedule considerations
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TTEACHING
    Label:       C1 D01A Teachers in this school set high standards for teaching
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    set high standards for teaching.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                            Frequency
    
    ============================================================
    Variable:    C1TLEARNING
    Label:       C1 D01B Teachers in this school set high standards for students' learning
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    set high standards for students' learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                           
    
    ============================================================
    Variable:    C1TBELIEVE
    Label:       C1 D01C Teachers in this school believe all students can do well
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    believe all students can do well.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TGIVEUP
    Label:       C1 D01D Teachers in this school have given up on some students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    have given up on some students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TCARE
    Label:       C1 D01E Teachers in this school care only about smart students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    care only about smart students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency          
    
    ============================================================
    Variable:    C1TEXPECT
    Label:       C1 D01F Teachers in this school expect very little from students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    expect very little from students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TWORKHARD
    Label:       C1 D01G Teachers in this school work hard to make sure all students learn
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    work hard to make sure all students are learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1CLEARNING
    Label:       C1 D02A Counselors in this school set high standards for students' learning
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    set high standards for students' learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                      
    
    ============================================================
    Variable:    C1CBELIEVE
    Label:       C1 D02B Counselors in this school believe all students can do well
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    believe all students can do well.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                            Fre
    
    ============================================================
    Variable:    C1CGIVEUP
    Label:       C1 D02C Counselors in this school have given up on some students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    have given up on some students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency     
    
    ============================================================
    Variable:    C1CCARE
    Label:       C1 D02D Counselors in this school care only about smart students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    care only about smart students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency     
    
    ============================================================
    Variable:    C1CEXPECT
    Label:       C1 D02E Counselors in this school expect very little from students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    expect very little from students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency   
    
    ============================================================
    Variable:    C1CWORKHARD
    Label:       C1 D02F Counselors in this school work hard to make sure all students learn
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    work hard to make sure all students are learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                               
    
    ============================================================
    Variable:    C1PLEARNING
    Label:       C1 D03A Principal in this school sets high standards for students' learning
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    sets high standards for students' learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                        
    
    ============================================================
    Variable:    C1PBELIEVE
    Label:       C1 D03B Principal in this school believes all students can do well
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    believes all students can do well.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                            Frequ
    
    ============================================================
    Variable:    C1PGIVEUP
    Label:       C1 D03C Principal in this school has given up on some students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    has given up on some students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1PCARE
    Label:       C1 D03D Principal in this school cares only about smart students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    cares only about smart students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency       
    
    ============================================================
    Variable:    C1PEXPECT
    Label:       C1 D03E Principal in this school expects very little from students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    expects very little from students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency     
    
    ============================================================
    Variable:    C1PWORKHARD
    Label:       C1 D03F Principal in this school works hard to make sure all students learn
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    works hard to make sure all students are learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                 
    
    ============================================================
    Variable:    C1HIMAJ_STEM
    Label:       C1 D06C Counselor's major for highest level of education STEM code
    Instrument:  counselor_BY
    Description: Major STEM indicators are constructed using the following logic:
    
    array majora P1HIMAJ61 P1BAMAJ61 P1HIMAJ62 P1BAMAJ62 P2HIMAJ61 P2HIMAJ62 
                 M1HIMAJ6 M1BAMAJ6 N1HIMAJ6 N1BAMAJ6 
                 A1HIMAJ6 A1BAMAJ6 A2HIMAJ6 A2BAMAJ6 
                 C1HIMAJ6 C1BAMAJ6 S3FIELD6;
    array majorn P1HIMAJ61n P1BAMAJ61n P1HIMAJ62n P1BAMAJ62n P2HIMAJ61n P2HIMAJ62n 
                 M1HIMAJ6n M1BAMAJ6n N1HIMA
    
    ============================================================
    Variable:    C1BAMAJ_STEM
    Label:       C1 D07C Counselor's major for Bachelor's degree STEM code
    Instrument:  counselor_BY
    Description: Major STEM indicators are constructed using the following logic:
    
    array majora P1HIMAJ61 P1BAMAJ61 P1HIMAJ62 P1BAMAJ62 P2HIMAJ61 P2HIMAJ62 
                 M1HIMAJ6 M1BAMAJ6 N1HIMAJ6 N1BAMAJ6 
                 A1HIMAJ6 A1BAMAJ6 A2HIMAJ6 A2BAMAJ6 
                 C1HIMAJ6 C1BAMAJ6 S3FIELD6;
    array majorn P1HIMAJ61n P1BAMAJ61n P1HIMAJ62n P1BAMAJ62n P2HIMAJ61n P2HIMAJ62n 
                 M1HIMAJ6n M1BAMAJ6n N1HIMA
    
    ============================================================
    Variable:    X1TXMTH1
    Label:       X1 Mathematics theta score - multiple imputation value 1 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (1 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMTH2
    Label:       X1 Mathematics theta score - multiple imputation value 2 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (2 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMTH3
    Label:       X1 Mathematics theta score - multiple imputation value 3 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (3 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMTH4
    Label:       X1 Mathematics theta score - multiple imputation value 4 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (4 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMTH5
    Label:       X1 Mathematics theta score - multiple imputation value 5 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (5 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMSEM1
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 1 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (1 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1TXMSEM2
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 2 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (2 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1TXMSEM3
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 3 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (3 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1TXMSEM4
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 4 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (4 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1TXMSEM5
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 5 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (5 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1SES1
    Label:       X1 Socio-economic status composite - multiple imputation value 1 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (1 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES2
    Label:       X1 Socio-economic status composite - multiple imputation value 2 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (2 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES3
    Label:       X1 Socio-economic status composite - multiple imputation value 3 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (3 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES4
    Label:       X1 Socio-economic status composite - multiple imputation value 4 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (4 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES5
    Label:       X1 Socio-economic status composite - multiple imputation value 5 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (5 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES1_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 1 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (1 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES2_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 2 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (2 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES3_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 3 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (3 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES4_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 4 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (4 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES5_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 5 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (5 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1TXMATH_IM
    Label:       X1 Imputation flag for X1TXM math scores
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1TXMTH was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1SEX_IM
    Label:       X1 Imputation flag for X1SEX
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1SEX was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1RACE_IM
    Label:       X1 Imputation flag for X1RACE
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1RACE was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1HISPAN_IM
    Label:       X1 Imputation flag for X1HISPANIC
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1HISPANIC was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1NATIVEL_IM
    Label:       X1 Imputation flag for X1NATIVELANG
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1NATIVELANG was statistically imputed or not imputed.
    
    
    Variable suppressed with -5 values on the public use file.
    
                                                                            Frequency
    
    ============================================================
    Variable:    X1P1RELAT_IM
    Label:       X1 Imputation flag for X1P1RELATION
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1P1RELATION was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1P2RELAT_IM
    Label:       X1 Imputation flag for X1P2RELATION
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1P2RELATION was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR1EDU_IM
    Label:       X1 Imputation flag for X1PAR1EDU
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR1EDU was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR2EDU_IM
    Label:       X1 Imputation flag for X1PAR2EDU
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR2EDU was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAREDU_IM
    Label:       X1 Imputation flag for X1PAREDU
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1PAREDU were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PARPATT_IM
    Label:       X1 Imputation flag for X1PARPATTERN
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1PARPATTERN were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR1EMP_IM
    Label:       X1 Imputation flag for X1PAR1EMP
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR1EMP was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR2EMP_IM
    Label:       X1 Imputation flag for X1PAR2EMP
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR2EMP was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR1OCC_IM
    Label:       X1 Imputation flag for X1PAR1OCC2
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR1OCC2 and  X1PAR1OCC6 was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR2OCC_IM
    Label:       X1 Imputation flag for X1PAR2OCC2
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR2OCC2 and X1PAR2OCC6 was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1MOMREL_IM
    Label:       X1 Imputation flag for X1MOMREL
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1MOMREL were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1MOMEDU_IM
    Label:       X1 Imputation flag for X1MOMEDU
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1MOMEDU were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1MOMEMP_IM
    Label:       X1 Imputation flag for X1MOMEMP
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1MOMEMP were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1MOMOCC_IM
    Label:       X1 Imputation flag for X1MOMOCC2
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1MOMOCC2 and X1MOMOCC6 were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1DADREL_IM
    Label:       X1 Imputation flag for X1DADREL
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1DADREL were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1DADEDU_IM
    Label:       X1 Imputation flag for X1DADEDU
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1DADEDU were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1DADEMP_IM
    Label:       X1 Imputation flag for X1DADEMP
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1DADEMP were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1DADOCC_IM
    Label:       X1 Imputation flag for X1DADOCC2
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1DADOCC2 and X1DADOCC6 were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1HHNUMB_IM
    Label:       X1 Imputation flag for X1HHNUMBER
    Instrument:  NCES_composite
    Description: Flag indicating whether one or both of the input variables P1HHLT18 and P1HHGE18 for the composite X1HHNUMBER were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1FAMINC_IM
    Label:       X1 Imputation flag for X1FAMINCOME
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1FAMINCOME was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1POVERTY_IM
    Label:       X1 Imputation flag for X1POVERTY/X1POVERTY130/X1POVERTY185
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1POVERTY/X1POVERTY130/X1POVERTY185 were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1SES_IM
    Label:       X1 Imputation flag for X1SES
    Instrument:  NCES_composite
    Description: Flag indicating whether SES or any inputs to SES were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1STUEDEX_IM
    Label:       X1 Imputation flag for X1STUEDEXPCT
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1STUEDEXPCT was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAREDEX_IM
    Label:       X1 Imputation flag for X1PAREDEXPCT
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAREDEXPCT was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    Progress: 781/781 variables assigned



```python
assignments = json.load(open("tier2_assignments.json"))

for variable, decision in assignments.items():
    if decision["certain"]:
        continue
    
    print(f"\n{'='*60}")
    print(f"Variable: {variable}")
    print(f"Current:  tier_2={decision['tier_2']}")
    
    # Show label from codebook_df
    row = codebook_df[codebook_df["variable"] == variable]
    if not row.empty:
        print(f"Label:    {row.iloc[0]['label']}")
        print(f"Description: {row.iloc[0]['description'][:300]}")
    
    tier2 = input("\nTier 2? (y/n/q to quit): ").strip().lower()
    if tier2 == "q":
        break
    
    assignments[variable]["tier_2"] = (tier2 == "y")
    assignments[variable]["certain"] = True
    
    json.dump(assignments, open("tier2_assignments.json", "w"), indent=2)

print("Done")
```

    
    ============================================================
    Variable: A1G9COMMUNTY
    Current:  tier_2=False
    Label:    A1 A26C Offers 9th grade learning communities separate from rest of school
    Description: Does your high school offer any of the following programs to assist 9th graders who are struggling academically?
    (check all that apply)
      Summer program prior to entry into high school that provides supplemental instruction in reading and math
      Small learning communities or Achievement Academies fo
    
    ============================================================
    Variable: A1FILLMTH
    Current:  tier_2=False
    Label:    A1 C05 Ease of filling high school mathematics teaching vacancies
    Description: How easy or difficult was it to fill the high school teaching vacancies in the mathematics department in your school?  Would you say...
      easy
      somewhat difficult
      very difficult or
      you could not fill the vacancies in the math department?
    
    
    "Could not fill math department" recoded with "Very dif
    
    ============================================================
    Variable: A1YRSADMIN
    Current:  tier_2=False
    Label:    A1 E10 Years served as principal of any school
    Description: Including this school year, how many years have you served as the principal of [your school] or any other school?
    
    Note: Question wording was customized in the survey instrument such that the respondent's school name appeared in place of 'your school'.
    
    
    Recoded 13 and 15 as 14, 16 and 18 as 17, and
    
    ============================================================
    Variable: C1PURSUE
    Current:  tier_2=False
    Label:    C1 B20A School has program to encourage underrepresented student in math/science
    Description: Does your school have any formal programs to...
    encourage underrepresented students to pursue mathematics or science?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: C1INFORM
    Current:  tier_2=False
    Label:    C1 B20B School has program to inform parent about math/science higher ed/careers
    Description: Does your school have any formal programs to...
    inform parents or guardians about mathematics or science higher education or career opportunities?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: C1ENCCLG
    Current:  tier_2=False
    Label:    C1 B20C School has program to encourage student not considering college to do so
    Description: Does your school have any formal programs to...
    encourage students who might not be considering college to do so?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: C1ONLINE
    Current:  tier_2=False
    Label:    C1 B21B Courses not offered by school available on-line
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
     
    
    ============================================================
    Variable: C1OTHERHS
    Current:  tier_2=False
    Label:    C1 B21C Courses not offered by school available at other district high school
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
     
    
    ============================================================
    Variable: C1COMCLG
    Current:  tier_2=False
    Label:    C1 B21D Courses not offered by school available at community college
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
     
    
    ============================================================
    Variable: C14YRCLG
    Current:  tier_2=False
    Label:    C1 B21E Courses not offered by school available at 4-year college
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
     
    
    ============================================================
    Variable: C1OTHERWAY
    Current:  tier_2=False
    Label:    C1 B21F Courses not offered by school available in some other way
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
     
    
    ============================================================
    Variable: C1NOWAY
    Current:  tier_2=False
    Label:    C1 B21G School doesn't have any options for taking courses not offered by school
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
     
    
    ============================================================
    Variable: C1MCOMPTST
    Current:  tier_2=False
    Label:    C1 B22 School requires a mathematics competency test
    Description: Does your school require students to take a mathematics competency test such as an end-of-course exam, end-of-year high school proficiency exam, or exit exam?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: C1DROPOUT
    Current:  tier_2=False
    Label:    C1 B24 School has a formal dropout prevention program for high school students
    Description: Does your school have a formal dropout prevention program for students in high school?
    This may be a whole-school restructuring program or a targeted program that operates on a smaller scale within the school or community organization(s) and enrolls students identified as at risk of dropping out.
      
    
    ============================================================
    Variable: C1CLGPREP
    Current:  tier_2=True
    Label:    C1 B27A School has counselor designated for college readiness/selection/apply
    Description: Does your school have one or more counselors whose primary responsibility is...
    assisting students with college readiness, selection, and applications?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: C1WORKFORCE
    Current:  tier_2=True
    Label:    C1 B27B School has counselor designated for workforce preparation/placement
    Description: Does your school have one or more counselors whose primary responsibility is...
    assisting students with preparation for and placement into the workforce?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: C1CLUSTER
    Current:  tier_2=False
    Label:    C1 B30 Career Clusters/Pathways/Programs of Study (POS) offered
    Description: Are Career Clusters, Pathways, or Programs of Study (POS) offered to students in [your school]?
      Yes
      No
    
    Note: Question wording was customized in the survey instrument such that the respondent's school name appeared in place of 'your school'.
    
    
                                                        
    
    ============================================================
    Variable: C1INDVCRS
    Current:  tier_2=False
    Label:    C1 B31 Student not enrolled in Career Clusters etc. may take course in program
    Description: Can high school students who are not enrolled in Career Clusters, Pathways, or Programs of Study (POS) take individual courses in these programs?
      Yes
      No
    
    
                                                                            Frequency             Percent
    Done



```python
tier1 = json.load(open("tier1_assignments.json"))
tier2 = json.load(open("tier2_assignments.json"))

already_assigned = set()
for k, v in tier1.items():
    if v["tier_1"]:
        already_assigned.add(k)
for k, v in tier2.items():
    if v["tier_2"]:
        already_assigned.add(k)

SAVE_FILE = "exclusions.json"

if os.path.exists(SAVE_FILE):
    exclusions = json.load(open(SAVE_FILE))
else:
    exclusions = {}

last_variable = None

for _, row in codebook_df.iterrows():
    v = row["variable"]

    if row["instrument"] in ("identifier", "target"):
        continue
    if v in already_assigned:
        continue
    if v in exclusions:
        continue

    print(f"\n{'='*60}")
    print(f"Variable:    {v}")
    print(f"Label:       {row['label']}")
    print(f"Instrument:  {row['instrument']}")
    print(f"Description: {row['description'][:400]}")

    exclude = input("\nExclude entirely? (y/n/r to redo last/q to quit): ").strip().lower()
    if exclude == "q":
        break
    elif exclude == "r":
        if last_variable:
            exclusions[last_variable]["certain"] = False
            json.dump(exclusions, open(SAVE_FILE, "w"), indent=2)
            print(f"Marked {last_variable} as uncertain — will reappear next run")
        else:
            print("Nothing to redo")
        continue

    certain = input("Certain? (y/n): ").strip().lower()

    exclusions[v] = {
        "exclude": exclude == "y",
        "certain": certain == "y",
    }
    last_variable = v

    json.dump(exclusions, open(SAVE_FILE, "w"), indent=2)

remaining = 911 - len(already_assigned) - len(exclusions)
print(f"\nProgress: {len(exclusions)} exclusion decisions made, ~{remaining} remaining")
```

    
    ============================================================
    Variable:    C1ENCCLG
    Label:       C1 B20C School has program to encourage student not considering college to do so
    Instrument:  counselor_BY
    Description: Does your school have any formal programs to...
    encourage students who might not be considering college to do so?
      Yes
      No
    
    
                                                                            Frequency             Percent


    
    ============================================================
    Variable:    C1ONLINE
    Label:       C1 B21B Courses not offered by school available on-line
    Instrument:  counselor_BY
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
      Courses at a local community college
      Courses at a nearby 4-year college or university
      Students 
    
    ============================================================
    Variable:    C1OTHERHS
    Label:       C1 B21C Courses not offered by school available at other district high school
    Instrument:  counselor_BY
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
      Courses at a local community college
      Courses at a nearby 4-year college or university
      Students 
    
    ============================================================
    Variable:    C1COMCLG
    Label:       C1 B21D Courses not offered by school available at community college
    Instrument:  counselor_BY
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
      Courses at a local community college
      Courses at a nearby 4-year college or university
      Students 
    
    ============================================================
    Variable:    C14YRCLG
    Label:       C1 B21E Courses not offered by school available at 4-year college
    Instrument:  counselor_BY
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
      Courses at a local community college
      Courses at a nearby 4-year college or university
      Students 
    
    ============================================================
    Variable:    C1OTHERWAY
    Label:       C1 B21F Courses not offered by school available in some other way
    Instrument:  counselor_BY
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
      Courses at a local community college
      Courses at a nearby 4-year college or university
      Students 
    
    ============================================================
    Variable:    C1NOWAY
    Label:       C1 B21G School doesn't have any options for taking courses not offered by school
    Instrument:  counselor_BY
    Description: In which of the following ways may a student take a course for credit if it is not offered by your school?
    (Check all that apply.)
      Independent study
      On-line or distance learning courses
      Courses at another traditional high school in the district
      Courses at a local career or technical school
      Courses at a local community college
      Courses at a nearby 4-year college or university
      Students 
    
    ============================================================
    Variable:    C1MCOMPTST
    Label:       C1 B22 School requires a mathematics competency test
    Instrument:  counselor_BY
    Description: Does your school require students to take a mathematics competency test such as an end-of-course exam, end-of-year high school proficiency exam, or exit exam?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1DROPOUT
    Label:       C1 B24 School has a formal dropout prevention program for high school students
    Instrument:  counselor_BY
    Description: Does your school have a formal dropout prevention program for students in high school?
    This may be a whole-school restructuring program or a targeted program that operates on a smaller scale within the school or community organization(s) and enrolls students identified as at risk of dropping out.
      Yes
      No
    
    
                                                                            Frequency        
    
    ============================================================
    Variable:    C1CLGPREP
    Label:       C1 B27A School has counselor designated for college readiness/selection/apply
    Instrument:  counselor_BY
    Description: Does your school have one or more counselors whose primary responsibility is...
    assisting students with college readiness, selection, and applications?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1WORKFORCE
    Label:       C1 B27B School has counselor designated for workforce preparation/placement
    Instrument:  counselor_BY
    Description: Does your school have one or more counselors whose primary responsibility is...
    assisting students with preparation for and placement into the workforce?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1CLGFAIR
    Label:       C1 B28A School holds or participates in college fairs
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1POSTSECREQ
    Label:       C1 B28B School consults with postsecondary reps about requirement/qualifications
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1VISITCLG
    Label:       C1 B28C School organizes student visits to colleges
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1INFOSESSN
    Label:       C1 B28E School holds info session on transition to college for students/parents
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1FINANCEAID
    Label:       C1 B28F School assists students with finding financial aid for college
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1ASSISTOTH
    Label:       C1 B28I School takes other steps to assist with HS to college transition
    Instrument:  counselor_BY
    Description: Which of the following steps does this school take to assist students with the transition from high school to college?
    (Check all that apply.)
      Holds or participates in college fairs
      Consults with postsecondary school representatives about requirements and qualifications sought
      Organizes student visits to colleges
      Enrolls students in special programs that help them plan or prepare for colle
    
    ============================================================
    Variable:    C1CLUSTER
    Label:       C1 B30 Career Clusters/Pathways/Programs of Study (POS) offered
    Instrument:  counselor_BY
    Description: Are Career Clusters, Pathways, or Programs of Study (POS) offered to students in [your school]?
      Yes
      No
    
    Note: Question wording was customized in the survey instrument such that the respondent's school name appeared in place of 'your school'.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1INDVCRS
    Label:       C1 B31 Student not enrolled in Career Clusters etc. may take course in program
    Instrument:  counselor_BY
    Description: Can high school students who are not enrolled in Career Clusters, Pathways, or Programs of Study (POS) take individual courses in these programs?
      Yes
      No
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1INTERN
    Label:       C1 B32A School offers internships with local employers
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBFAIR
    Label:       C1 B32B School offers job fairs
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBGUIDE
    Label:       C1 B32C School offers career guides or skills assessments
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1EMPLOYER
    Label:       C1 B32D School offers school/classroom presentations by local employers
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1AWARENESS
    Label:       C1 B32E School offers career awareness activities
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1CAREERUNIT
    Label:       C1 B32G School offers career information units in subject-matter courses
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1WORKSTUDY
    Label:       C1 B32H School offers exploratory work experience programs/co-op/workstudy/EBCE
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1CAREERDAY
    Label:       C1 B32I School offers career days or nights
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1ASSEMBLIES
    Label:       C1 B32J School offers vocational oriented assemblies and speakers in classes
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBVISIT
    Label:       C1 B32L School offers job site visits/field trips
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBSHADOW
    Label:       C1 B32M School offers job shadowing
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBTEST
    Label:       C1 B32O School offers tests for career planning purposes
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBINFOCMP
    Label:       C1 B32Q School offers computerized career information resources
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1JOBINFONON
    Label:       C1 B32R School offers non-computerized career information resources
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1HSTOWRKOTH
    Label:       C1 B32S School assists students with transition from HS to work in other ways
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1HSTOWORKNO
    Label:       C1 B32T School doesn't assist students with transition from high school to work
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1G9MMSCNSL
    Label:       C1 C02A Importance of MS counselor recommendation for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Middle school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MHSCNSL
    Label:       C1 C02B Importance of HS counselor recommendation for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    High school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MMSTCHR
    Label:       C1 C02C Importance of MS teacher recommendation for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Middle school teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MMSCOURS
    Label:       C1 C02D Importance of courses taken in MS for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Courses taken in middle school
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                                                        
    
    ============================================================
    Variable:    C1G9MMSACHV
    Label:       C1 C02E Importance of achievement in MS courses for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Achievement in middle school courses
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                                                  
    
    ============================================================
    Variable:    C1G9MENDTST
    Label:       C1 C02F Importance of end-of-year/course test for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Results of district or state end-of-year or end-of-course exams
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MPLACTST
    Label:       C1 C02G Importance of placement tests for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MSTNDTST
    Label:       C1 C02H Importance of standardized tests for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Results of standardized tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MPLAN
    Label:       C1 C02I Importance of career/education plan for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MSELECT
    Label:       C1 C02J Importance of student/parent choice for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMGRADES
    Label:       C1 C04A Importance of prior grades for 10th to 12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Prior grades including grades from a prerequisite class
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" and "A little important" recoded as "Not at all important, a little important, or somewhat important" on the public
    
    ============================================================
    Variable:    C1UPMPLACTST
    Label:       C1 C04B Importance of placement tests for 10th to 12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMTCHR
    Label:       C1 C04C Importance of teacher's recommendation for 10-12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Previous year's teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMSELECT
    Label:       C1 C04D Importance of student/parent choice for 10th-12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMPLAN
    Label:       C1 C04E Importance of career/education plan for 10th-12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMSCHED
    Label:       C1 C04F Importance of master schedule for 10th to 12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Master schedule considerations
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSCNSL
    Label:       C1 C06A Importance of MS counselor recommendation for grade 9 science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Middle school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SHSCNSL
    Label:       C1 C06B Importance of HS counselor recommendation for grade 9 science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    High school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSTCHR
    Label:       C1 C06C Importance of MS teacher recommendation for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Middle school teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSCOURS
    Label:       C1 C06D Importance of courses taken in MS for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Courses taken in middle school
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSACHV
    Label:       C1 C06E Importance of achievement in MS courses for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Achievement in middle school courses
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SENDTST
    Label:       C1 C06F Importance of end-of-year/course test for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Results of district or state end-of-year or end-of-course exams
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SPLACTST
    Label:       C1 C06G Importance of placement tests for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SSTNDTST
    Label:       C1 C06H Importance of standardized tests for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Results of standardized tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SPLAN
    Label:       C1 C06I Importance of career/education plan for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SSELECT
    Label:       C1 C06J Importance of student/parent choice for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSGRADES
    Label:       C1 C08A Importance of prior grades for 10th to 12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Prior grades including grades from a prerequisite class
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                          
    
    ============================================================
    Variable:    C1UPSPLACTST
    Label:       C1 C08B Importance of placement tests for 10th to 12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSTCHR
    Label:       C1 C08C Importance of teacher's recommendation for 10th-12th science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Previous year's teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSSELECT
    Label:       C1 C08D Importance of student/parent choice for 10-12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                                      
    
    ============================================================
    Variable:    C1UPSPLAN
    Label:       C1 C08E Importance of career/education plan for 10-12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSSCHED
    Label:       C1 C08F Importance of master schedule for 10th to 12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Master schedule considerations
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TTEACHING
    Label:       C1 D01A Teachers in this school set high standards for teaching
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    set high standards for teaching.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                            Frequency
    
    ============================================================
    Variable:    C1TLEARNING
    Label:       C1 D01B Teachers in this school set high standards for students' learning
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    set high standards for students' learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                           
    
    ============================================================
    Variable:    C1TBELIEVE
    Label:       C1 D01C Teachers in this school believe all students can do well
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    believe all students can do well.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TGIVEUP
    Label:       C1 D01D Teachers in this school have given up on some students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    have given up on some students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TCARE
    Label:       C1 D01E Teachers in this school care only about smart students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    care only about smart students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency          
    
    ============================================================
    Variable:    C1TEXPECT
    Label:       C1 D01F Teachers in this school expect very little from students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    expect very little from students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TWORKHARD
    Label:       C1 D01G Teachers in this school work hard to make sure all students learn
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    work hard to make sure all students are learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1CLEARNING
    Label:       C1 D02A Counselors in this school set high standards for students' learning
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    set high standards for students' learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                      
    
    ============================================================
    Variable:    C1CBELIEVE
    Label:       C1 D02B Counselors in this school believe all students can do well
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    believe all students can do well.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                            Fre
    
    ============================================================
    Variable:    C1CGIVEUP
    Label:       C1 D02C Counselors in this school have given up on some students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    have given up on some students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency     
    
    ============================================================
    Variable:    C1CCARE
    Label:       C1 D02D Counselors in this school care only about smart students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    care only about smart students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency     
    
    ============================================================
    Variable:    C1CEXPECT
    Label:       C1 D02E Counselors in this school expect very little from students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    expect very little from students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency   
    
    ============================================================
    Variable:    C1CWORKHARD
    Label:       C1 D02F Counselors in this school work hard to make sure all students learn
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    work hard to make sure all students are learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                               
    
    ============================================================
    Variable:    C1PLEARNING
    Label:       C1 D03A Principal in this school sets high standards for students' learning
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    sets high standards for students' learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                        
    
    ============================================================
    Variable:    C1PBELIEVE
    Label:       C1 D03B Principal in this school believes all students can do well
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    believes all students can do well.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                            Frequ
    
    ============================================================
    Variable:    C1PGIVEUP
    Label:       C1 D03C Principal in this school has given up on some students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    has given up on some students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1PCARE
    Label:       C1 D03D Principal in this school cares only about smart students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    cares only about smart students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency       
    
    ============================================================
    Variable:    C1PEXPECT
    Label:       C1 D03E Principal in this school expects very little from students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    expects very little from students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency     
    
    ============================================================
    Variable:    C1PWORKHARD
    Label:       C1 D03F Principal in this school works hard to make sure all students learn
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    works hard to make sure all students are learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                 
    
    ============================================================
    Variable:    C1HIMAJ_STEM
    Label:       C1 D06C Counselor's major for highest level of education STEM code
    Instrument:  counselor_BY
    Description: Major STEM indicators are constructed using the following logic:
    
    array majora P1HIMAJ61 P1BAMAJ61 P1HIMAJ62 P1BAMAJ62 P2HIMAJ61 P2HIMAJ62 
                 M1HIMAJ6 M1BAMAJ6 N1HIMAJ6 N1BAMAJ6 
                 A1HIMAJ6 A1BAMAJ6 A2HIMAJ6 A2BAMAJ6 
                 C1HIMAJ6 C1BAMAJ6 S3FIELD6;
    array majorn P1HIMAJ61n P1BAMAJ61n P1HIMAJ62n P1BAMAJ62n P2HIMAJ61n P2HIMAJ62n 
                 M1HIMAJ6n M1BAMAJ6n N1HIMA
    
    ============================================================
    Variable:    C1BAMAJ_STEM
    Label:       C1 D07C Counselor's major for Bachelor's degree STEM code
    Instrument:  counselor_BY
    Description: Major STEM indicators are constructed using the following logic:
    
    array majora P1HIMAJ61 P1BAMAJ61 P1HIMAJ62 P1BAMAJ62 P2HIMAJ61 P2HIMAJ62 
                 M1HIMAJ6 M1BAMAJ6 N1HIMAJ6 N1BAMAJ6 
                 A1HIMAJ6 A1BAMAJ6 A2HIMAJ6 A2BAMAJ6 
                 C1HIMAJ6 C1BAMAJ6 S3FIELD6;
    array majorn P1HIMAJ61n P1BAMAJ61n P1HIMAJ62n P1BAMAJ62n P2HIMAJ61n P2HIMAJ62n 
                 M1HIMAJ6n M1BAMAJ6n N1HIMA
    
    ============================================================
    Variable:    X1TXMTH1
    Label:       X1 Mathematics theta score - multiple imputation value 1 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (1 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMTH2
    Label:       X1 Mathematics theta score - multiple imputation value 2 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (2 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMTH3
    Label:       X1 Mathematics theta score - multiple imputation value 3 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (3 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMTH4
    Label:       X1 Mathematics theta score - multiple imputation value 4 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (4 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMTH5
    Label:       X1 Mathematics theta score - multiple imputation value 5 of 5
    Instrument:  NCES_composite
    Description: Mathematics theta score multiple imputation value (5 of 5). When the math test data were missing for student survey respondents, the math theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5.  The theta score provides a norm-referenced measurement of achievement, that is, an estimate of achievement relative to the population (fall 200
    
    ============================================================
    Variable:    X1TXMSEM1
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 1 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (1 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1TXMSEM2
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 2 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (2 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1TXMSEM3
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 3 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (3 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1TXMSEM4
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 4 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (4 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1TXMSEM5
    Label:       X1 Mathematics standard error of measurement - multiple imputation value 5 of 5
    Instrument:  NCES_composite
    Description: Mathematics standard error of measurement multiple imputation value (5 of 5). When the math test data were missing for student survey respondents, the math standard error of measurement (SEM) for the raw theta score was imputed with multiple imputation technique, with 5 imputed values. X1TXMTH is the mean of X1TXMTH1-5. The standard error of measurement for the raw theta score indicates the precis
    
    ============================================================
    Variable:    X1SES1
    Label:       X1 Socio-economic status composite - multiple imputation value 1 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (1 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES2
    Label:       X1 Socio-economic status composite - multiple imputation value 2 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (2 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES3
    Label:       X1 Socio-economic status composite - multiple imputation value 3 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (3 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES4
    Label:       X1 Socio-economic status composite - multiple imputation value 4 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (4 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES5
    Label:       X1 Socio-economic status composite - multiple imputation value 5 of 5
    Instrument:  NCES_composite
    Description: This variables contain the imputed value (5 of 5) for X1SES, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES is the mean of X1SES1-X1SES5 and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES1_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 1 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (1 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES2_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 2 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (2 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES3_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 3 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (3 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES4_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 4 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (4 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1SES5_U
    Label:       X1 SES derived with locale (urbanicity) - multiple imputation value 5 of 5
    Instrument:  NCES_composite
    Description: This variable contain the imputed values (5 of 5) for X1SES_U, generated through a multiple imputation model, for responding students without a responding parent/guardian.  X1SES_U is the mean of X1SES1_U-X1SES5_U and X1SES_IM=1.
    
    ============================================================
    Variable:    X1TXMATH_IM
    Label:       X1 Imputation flag for X1TXM math scores
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1TXMTH was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1SEX_IM
    Label:       X1 Imputation flag for X1SEX
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1SEX was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1RACE_IM
    Label:       X1 Imputation flag for X1RACE
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1RACE was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1HISPAN_IM
    Label:       X1 Imputation flag for X1HISPANIC
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1HISPANIC was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1NATIVEL_IM
    Label:       X1 Imputation flag for X1NATIVELANG
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1NATIVELANG was statistically imputed or not imputed.
    
    
    Variable suppressed with -5 values on the public use file.
    
                                                                            Frequency
    
    ============================================================
    Variable:    X1P1RELAT_IM
    Label:       X1 Imputation flag for X1P1RELATION
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1P1RELATION was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1P2RELAT_IM
    Label:       X1 Imputation flag for X1P2RELATION
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1P2RELATION was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR1EDU_IM
    Label:       X1 Imputation flag for X1PAR1EDU
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR1EDU was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR2EDU_IM
    Label:       X1 Imputation flag for X1PAR2EDU
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR2EDU was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAREDU_IM
    Label:       X1 Imputation flag for X1PAREDU
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1PAREDU were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PARPATT_IM
    Label:       X1 Imputation flag for X1PARPATTERN
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1PARPATTERN were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR1EMP_IM
    Label:       X1 Imputation flag for X1PAR1EMP
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR1EMP was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR2EMP_IM
    Label:       X1 Imputation flag for X1PAR2EMP
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR2EMP was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR1OCC_IM
    Label:       X1 Imputation flag for X1PAR1OCC2
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR1OCC2 and  X1PAR1OCC6 was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAR2OCC_IM
    Label:       X1 Imputation flag for X1PAR2OCC2
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAR2OCC2 and X1PAR2OCC6 was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1MOMREL_IM
    Label:       X1 Imputation flag for X1MOMREL
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1MOMREL were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1MOMEDU_IM
    Label:       X1 Imputation flag for X1MOMEDU
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1MOMEDU were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1MOMEMP_IM
    Label:       X1 Imputation flag for X1MOMEMP
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1MOMEMP were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1MOMOCC_IM
    Label:       X1 Imputation flag for X1MOMOCC2
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1MOMOCC2 and X1MOMOCC6 were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1DADREL_IM
    Label:       X1 Imputation flag for X1DADREL
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1DADREL were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1DADEDU_IM
    Label:       X1 Imputation flag for X1DADEDU
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1DADEDU were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1DADEMP_IM
    Label:       X1 Imputation flag for X1DADEMP
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1DADEMP were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1DADOCC_IM
    Label:       X1 Imputation flag for X1DADOCC2
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1DADOCC2 and X1DADOCC6 were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1HHNUMB_IM
    Label:       X1 Imputation flag for X1HHNUMBER
    Instrument:  NCES_composite
    Description: Flag indicating whether one or both of the input variables P1HHLT18 and P1HHGE18 for the composite X1HHNUMBER were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1FAMINC_IM
    Label:       X1 Imputation flag for X1FAMINCOME
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1FAMINCOME was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1POVERTY_IM
    Label:       X1 Imputation flag for X1POVERTY/X1POVERTY130/X1POVERTY185
    Instrument:  NCES_composite
    Description: Flag indicating whether any of the inputs to X1POVERTY/X1POVERTY130/X1POVERTY185 were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1SES_IM
    Label:       X1 Imputation flag for X1SES
    Instrument:  NCES_composite
    Description: Flag indicating whether SES or any inputs to SES were statistically imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1STUEDEX_IM
    Label:       X1 Imputation flag for X1STUEDEXPCT
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1STUEDEXPCT was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    X1PAREDEX_IM
    Label:       X1 Imputation flag for X1PAREDEXPCT
    Instrument:  NCES_composite
    Description: Flag indicating whether the variable X1PAREDEXPCT was statistically imputed or not imputed.
    
    
                                                                            Frequency             Percent
    
    Progress: 719 exclusion decisions made, ~0 remaining



```python
assignments = json.load(open("exclusions.json"))

for variable, decision in assignments.items():
    if decision["certain"]:
        continue
    
    print(f"\n{'='*60}")
    print(f"Variable: {variable}")
    print(f"Current:  exclude={decision['exclude']}")
    
    row = codebook_df[codebook_df["variable"] == variable]
    if not row.empty:
        print(f"Label:    {row.iloc[0]['label']}")
        print(f"Description: {row.iloc[0]['description'][:300]}")
    
    exclude = input("\nExclude? (y/n/q to quit): ").strip().lower()
    if exclude == "q":
        break
    
    assignments[variable]["exclude"] = (exclude == "y")
    assignments[variable]["certain"] = True
    
    json.dump(assignments, open("exclusions.json", "w"), indent=2)

print("Done")
```

    
    ============================================================
    Variable: C1GOAL3
    Current:  exclude=False
    Label:    C1 A08 School counseling program's third most emphasized goal
    Description: Of the two goals remaining, which one does your school's counseling program emphasize more? Would you say...
      helping students plan and prepare for their work roles after high school
      helping students with personal growth and development
      helping students plan and prepare for postsecondary school


    Done



```python
assignments = json.load(open("exclusions.json"))

excluded = {k: v for k, v in assignments.items() if v['exclude']}
print(f"Currently excluded: {len(excluded)}")
print()

for variable, decision in excluded.items():
    print(f"\n{'='*60}")
    print(f"Variable: {variable}")
    
    row = codebook_df[codebook_df["variable"] == variable]
    if not row.empty:
        print(f"Label:    {row.iloc[0]['label']}")
        print(f"Description: {row.iloc[0]['description'][:300]}")
    
    keep_excluded = input("\nKeep excluded? (y/n/q to quit): ").strip().lower()
    if keep_excluded == "q":
        break
    
    if keep_excluded == "n":
        assignments[variable]["exclude"] = False
    
    assignments[variable]["certain"] = True
    json.dump(assignments, open("exclusions.json", "w"), indent=2)

print("Done")
```

    Currently excluded: 52
    
    
    ============================================================
    Variable: X1PARRESP
    Label:    X1 Whether parent questionnaire respondent is Parent 1
    Description: Indicates whether or not the parent questionnaire respondent is "parent #1"; that is, the parent to whom all "parent #1" variables (e.g. X1P1RELATION, X1PAR1EMP, P1YRBORN1, P1USYR1, etc.) refer.  The parent questionnaire respondent is always "parent #1" except in cases where: (1) the respondent is a
    
    ============================================================
    Variable: X1MOMRESP
    Label:    X1 Whether parent questionnaire respondent is mother
    Description: Indicates whether or not the parent questionnaire respondent is a biological, adoptive, or step mother.  X1MOMRESP is derived from three composite variables (X1P1RELATION, X1P2RELATION, and X1PARRESP).
    
    
                                                                            Frequency             Pe
    
    ============================================================
    Variable: X1DADRESP
    Label:    X1 Whether parent questionnaire respondent is father
    Description: Indicates whether or not the parent questionnaire respondent is a biological, adoptive, or step father.  X1DADRESP is derived from three composite variables (X1P1RELATION, X1P2RELATION, and X1PARRESP).
    
    
                                                                            Frequency             Pe
    
    ============================================================
    Variable: X1TESTSTAT
    Label:    X1 Student mathematics assessment status
    Description: X1TESTSTAT indicates whether base-year HSLS mathematics assessment data are available on the data file for any given sample member.
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: X1TESTDATE
    Label:    X1 Student mathematics assessment date (YYYYMM)
    Description: Month and year the sample member completed the base-year HSLS mathematics assessment.
    
    
    Dates recoded on the public use file as follows: 200909 as 200910, 200912 as 200911, and after 2009 as 201001.
    
    ============================================================
    Variable: X1SQSTAT
    Label:    X1 Student questionnaire status
    Description: X1SQSTAT indicates whether a complete base year student interview is available on the data file; X1SQSTAT also indicates the mode of the base year student interview, and whether the student responded in-school or out-of-school. For an explanation of a responding case, please see chapter 2 of the HSL
    
    ============================================================
    Variable: X1SQDATE
    Label:    X1 Student questionnaire date (YYYYMM)
    Description: Month and year the sample member responded to the base year student interview.
    
    
    Dates recoded on the public use file as follows: 200909 as 200910, 200912 as 200911, and after 2009 as 201001.
    
    ============================================================
    Variable: X1PQSTAT
    Label:    X1 Parent questionnaire status
    Description: X1PQSTAT indicates whether a complete base year parent interview is available on the data file; it also indicates the mode of the base year parent interview, and whether the parent responded to a full-length or abbreviated interview.  For an explanation of a responding case, please see chapter 2 of 
    
    ============================================================
    Variable: X1PQDATE
    Label:    X1 Parent questionnaire date (YYYYMM)
    Description: Month and year the sample member's parent responded to the base year parent quesionnaire.
    
    
    Dates recoded on the public use file as follows: 200909 as 200910, 200912 as 200911, and after 2009 as 201001.
    
    ============================================================
    Variable: X1TMQSTAT
    Label:    X1 Math teacher questionnaire status
    Description: X1TMQSTAT indicates whether a complete base year math teacher interview is available on the data file; X1TMQSTAT also indicates the mode of the base year math teacher interview, and whether the math teacher responded to a full-length or abbreviated interview. For an explanation of a responding case,
    
    ============================================================
    Variable: X1TMQDATE
    Label:    X1 Math teacher questionnaire date (YYYYMM)
    Description: Month and year the math teacher responded to the base year teacher questionnaire. If the student indicated that he or she was not taking a fall math class, this variable is set to -7.
    
    
    Dates recoded on the public use file as follows: 200909 as 200910, 200912 as 200911, and after 2009 as 201001.
    
    ============================================================
    Variable: X1TMLINK
    Label:    X1 Student to math teacher link descriptor
    Description: X1TMLINK characterizes the linkage between the student and the base-year math teacher associated with that student on the HSLS data file. The values assigned are a product of comparison between student-provided teacher information and the teacher information provided by the school. Values of 1 throu
    
    ============================================================
    Variable: X1TMCRSLINK
    Label:    X1 Student to math teacher course-level link descriptor
    Description: X1TMCRSLINK characterizes the linkage between the student and the course-level data provided by the math teacher associated with that student on the HSLS data file. Values of 1 are assigned in cases where X1TMLINK=1 and the student confirmed enrollment in the associated course and could be linked us
    
    ============================================================
    Variable: X1TMRACE
    Label:    X1 Math teacher's race/ethnicity-composite
    Description: X1TMRACE characterizes the race/ethnicity of the sample member's math teacher by summarizing the following math teacher questionnaire variables:  M1HISP, M1WHITE, M1BLACK, M1ASIAN, M1PACISLE, and M1AMINDIAN. If the student indicated that he or she was not taking a fall math class, this variable is s
    
    ============================================================
    Variable: X1TSQSTAT
    Label:    X1 Science teacher questionnaire status
    Description: X1TSQSTAT indicates whether a complete base year science teacher interview is available on the data file; X1TSQSTAT also indicates the mode of the base year science teacher interview, and whether the science teacher responded to a full-length or abbreviated interview. For an explanation of a respond
    
    ============================================================
    Variable: X1TSQDATE
    Label:    X1 Science teacher questionnaire date (YYYYMM)
    Description: Month and year the science teacher responded to the base year teacher questionnaire. If the student indicated that he or she was not taking a fall science class, this variable is set to -7.
    
    
    Dates recoded on the public use file as follows: 200909 as 200910, 200912 as 200911, and after 2009 as 20100
    
    ============================================================
    Variable: X1TSLINK
    Label:    X1 Student to science teacher link descriptor
    Description: X1TSLINK characterizes the linkage between the student and the base-year science teacher associated with that student on the HSLS data file. The values assigned are a product of comparison between student-provided teacher information and the teacher information provided by the school. Values of 1 th
    
    ============================================================
    Variable: X1TSCRSLINK
    Label:    X1 Student to science teacher course-level link descriptor
    Description: X1TSCRSLINK characterizes the linkage between the student and the course-level data provided by the science teacher associated with that student on the HSLS data file. Values of 1 are assigned in cases where X1TSLINK=1 and the student confirmed enrollment in the associated course and could be linked
    
    ============================================================
    Variable: X1TSRACE
    Label:    X1 Science teacher race/ethnicity-composite
    Description: X1TSRACE characterizes the race/ethnicity of the sample member's science teacher by summarizing the following science teacher questionnaire variables:  N1HISP, N1WHITE, N1BLACK, N1ASIAN, N1PACISLE, and N1AMINDIAN. If the student indicated that he or she was not taking a fall science class, this vari
    
    ============================================================
    Variable: X1AQSTAT
    Label:    X1 administrator questionnaire status
    Description: X1AQSTAT indicates whether a complete  base year administrator interview is available on the data file; X1AQSTAT also indicates the mode of the base year administrator interview, and whether the administrator responded to a full-length or abbreviated interview. For an explanation of a responding cas
    
    ============================================================
    Variable: X1AQDATE
    Label:    X1 administrator questionnaire date (YYYYMM)
    Description: Month and year the school administrator responded to the base year administrator questionnaire.
    
    
    Dates recoded on the public use file as follows: 200909 as 200910, 200912 as 200911, and after 2009 as 201001.
    
    ============================================================
    Variable: X1AQDESIGNEE
    Label:    X1 administrator questionnaire designee respondent (designee resp v. no designee)
    Description: Indicates whether an adminstrator designee completed the applicable portion of the administrator questionnaire.  An administrator designee was allowed to complete all sections of the administrator questionnaire except for the "Goals and Background" section (i.e. administrator questionnaire variables
    
    ============================================================
    Variable: X1CQSTAT
    Label:    X1 counselor questionnaire status
    Description: X1CQSTAT indicates whether a complete base year counselor interview is available on the data file; X1CQSTAT also indicates the mode of the base year counselor interview. For an explanation of a responding case, please see chapter 2 of the HSLS:09 Base-Year Data File Documentation (NCES 2011-328).
    
    
    
    
    ============================================================
    Variable: X1CQDATE
    Label:    X1 counselor questionnaire date (YYYYMM)
    Description: Month and year the school counselor responded to the base year counselor questionnaire.
    
    
    Dates recoded on the public use file as follows: 200909 as 200910, 200912 as 200911, and after 2009 as 201001.
    
    ============================================================
    Variable: S1SEX
    Label:    S1 A01 9th grader's sex
    Description: What is your sex?
      Male
      Female
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: S1BIRTHMON
    Label:    S1 A06A 9th grader's month of birth
    Description: What is your birth date?
      January
      February
      March
      April
      May
      June
      July
      August
      September
      October
      November
      December
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: S1BIRTHYR
    Label:    S1 A06C 9th grader's year of birth
    Description: What is your year of birth?
    
    
    Years earlier than 1992 recoded to 1992 or earlier and years later than 1996 recoded to 1996 or later on the public use file.
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable: M1SEX
    Label:    M1 A01 Math teacher's sex
    Description: [We would like to confirm your sex.]
    Are you male or female?
      Male
      Female
    
    Note:  Bracketed text above was used for telephone interviews only; it was not displayed in web self-administered interviews.
    
    
                                                                            Frequency             
    
    ============================================================
    Variable: A1SCHCONTROL
    Label:    A1 A02 School control
    Description: Our records indicate that [your school] is a [public/private] school. Is this correct?
      Yes
      No
    
    Note: Question wording was customized in the survey instrument such that the respondent's school name appeared in place of 'your school', and such that 'public' or 'private' was conditionally displayed
    
    ============================================================
    Variable: A1MTHSCIFAIR
    Label:    A1 A25A Holds math or science fairs/workshops/competitions
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSSUMMER
    Label:    A1 A25B Partners w/ college/university that offers math/science summer program
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSAFTERSCH
    Label:    A1 A25C Sponsors a math or science after-school program
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSMENTOR
    Label:    A1 A25D Pairs students with mentors in math or science
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSSPEAKER
    Label:    A1 A25E Brings in guest speakers to talk about math or science
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSFLDTRIP
    Label:    A1 A25F Takes students on math- or science-relevant field trips
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSPRGMS
    Label:    A1 A25G Tells students about math/science contests/websites/blogs/other programs
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSPDLEARN
    Label:    A1 A25I Requires teacher prof development in how students learn math/science
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSPDINTRST
    Label:    A1 A25J Requires teacher prof development in increasing interest in math/science
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSOTHER
    Label:    A1 A25K Raises students math/science interest/achievement in another way
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1MSNONE
    Label:    A1 A25L Doesn't do any of these to raise math/science interest/achievement
    Description: Does your school do any of the following to raise high school students' interest and achievement in math or science?
    (check all that apply)
      Hold school-wide math or science fairs, workshops, or competitions
      Partner with community colleges or universities that offer math or science summer program
    
    ============================================================
    Variable: A1G9SUMMER
    Label:    A1 A26A Offers pre-HS summer reading/math instruction for struggling 9th graders
    Description: Does your high school offer any of the following programs to assist 9th graders who are struggling academically?
    (check all that apply)
      Summer program prior to entry into high school that provides supplemental instruction in reading and math
      Small learning communities or Achievement Academies fo
    
    ============================================================
    Variable: A1G9OVERAGE
    Label:    A1 A26B Offers learning communities for over-age student lacking HS prerequisite
    Description: Does your high school offer any of the following programs to assist 9th graders who are struggling academically?
    (check all that apply)
      Summer program prior to entry into high school that provides supplemental instruction in reading and math
      Small learning communities or Achievement Academies fo
    
    ============================================================
    Variable: A1G9COMMUNTY
    Label:    A1 A26C Offers 9th grade learning communities separate from rest of school
    Description: Does your high school offer any of the following programs to assist 9th graders who are struggling academically?
    (check all that apply)
      Summer program prior to entry into high school that provides supplemental instruction in reading and math
      Small learning communities or Achievement Academies fo
    
    ============================================================
    Variable: A1G9BLOCKSCH
    Label:    A1 A26D Offers block scheduling to assist struggling 9th graders
    Description: Does your high school offer any of the following programs to assist 9th graders who are struggling academically?
    (check all that apply)
      Summer program prior to entry into high school that provides supplemental instruction in reading and math
      Small learning communities or Achievement Academies fo
    
    ============================================================
    Variable: A1G9DOUBLE
    Label:    A1 A26E Offers catch-up courses/double-dosing to assist struggling 9th graders
    Description: Does your high school offer any of the following programs to assist 9th graders who are struggling academically?
    (check all that apply)
      Summer program prior to entry into high school that provides supplemental instruction in reading and math
      Small learning communities or Achievement Academies fo
    
    ============================================================
    Variable: A1G9STUDY
    Label:    A1 A26F Offers study skill seminar/class for struggling 9th graders
    Description: Does your high school offer any of the following programs to assist 9th graders who are struggling academically?
    (check all that apply)
      Summer program prior to entry into high school that provides supplemental instruction in reading and math
      Small learning communities or Achievement Academies fo
    
    ============================================================
    Variable: A1G9TEACHER
    Label:    A1 A26G Offers assistance for teachers working with struggling 9th graders
    Description: Does your high school offer any of the following programs to assist 9th graders who are struggling academically?
    (check all that apply)
      Summer program prior to entry into high school that provides supplemental instruction in reading and math
      Small learning communities or Achievement Academies fo
    
    ============================================================
    Variable: A1G9OTHRPROG
    Label:    A1 A26I Offers another program to assist struggling 9th graders
    Description: Does your high school offer any of the following programs to assist 9th graders who are struggling academically?
    (check all that apply)
      Summer program prior to entry into high school that provides supplemental instruction in reading and math
      Small learning communities or Achievement Academies fo
    
    ============================================================
    Variable: A1OFFERDOPRV
    Label:    A1 B02B Dropout prevention program offered on-site
    Description: Which of the following programs or courses does [your school] offer on-site?
    (check all that apply)
      Alternative program
      Dropout prevention program
      College Board Advanced Placement (AP) courses
      None of these
    
    Note: Question wording was customized in the survey instrument such that the respond
    
    ============================================================
    Variable: A1OFFERNONE
    Label:    A1 B02D None of these programs or courses are offered on-site
    Description: Which of the following programs or courses does [your school] offer on-site?
    (check all that apply)
      Alternative program
      Dropout prevention program
      College Board Advanced Placement (AP) courses
      None of these
    
    Note: Question wording was customized in the survey instrument such that the respond
    
    ============================================================
    Variable: A1FILLMTH
    Label:    A1 C05 Ease of filling high school mathematics teaching vacancies
    Description: How easy or difficult was it to fill the high school teaching vacancies in the mathematics department in your school?  Would you say...
      easy
      somewhat difficult
      very difficult or
      you could not fill the vacancies in the math department?
    
    
    "Could not fill math department" recoded with "Very dif
    
    ============================================================
    Variable: A1FILLSCI
    Label:    A1 C06 Ease of filling high school science teaching vacancies
    Description: How easy or difficult was it to fill the high school teaching vacancies in the science department in your school? Would you say...
      easy
      somewhat difficult
      very difficult or
      you could not fill the vacancies in the science department?
    
    
    "Could not fill science department" recoded with "Very di
    Done



```python
tier1 = json.load(open("tier1_assignments.json"))
tier2 = json.load(open("tier2_assignments.json"))
exclusions = json.load(open("exclusions.json"))

already_assigned = set()
for k, v in tier1.items():
    if v["tier_1"]:
        already_assigned.add(k)
for k, v in tier2.items():
    if v["tier_2"]:
        already_assigned.add(k)
for k, v in exclusions.items():
    if v["exclude"]:
        already_assigned.add(k)

SAVE_FILE = "tier3_assignments.json"

if os.path.exists(SAVE_FILE):
    assignments = json.load(open(SAVE_FILE))
else:
    assignments = {}

last_variable = None

for _, row in codebook_df.iterrows():
    v = row["variable"]

    if row["instrument"] in ("identifier", "target"):
        continue
    if v in already_assigned:
        continue
    if v in assignments:
        continue

    print(f"\n{'='*60}")
    print(f"Variable:    {v}")
    print(f"Label:       {row['label']}")
    print(f"Instrument:  {row['instrument']}")
    print(f"Description: {row['description'][:400]}")

    tier3 = input("\nTier 3? (y/n/r to redo last/q to quit): ").strip().lower()
    if tier3 == "q":
        break
    elif tier3 == "r":
        if last_variable:
            assignments[last_variable]["certain"] = False
            json.dump(assignments, open(SAVE_FILE, "w"), indent=2)
            print(f"Marked {last_variable} as uncertain — will reappear next run")
        else:
            print("Nothing to redo")
        continue

    certain = input("Certain? (y/n): ").strip().lower()

    assignments[v] = {
        "tier_3": tier3 == "y",
        "certain": certain == "y",
    }
    last_variable = v

    json.dump(assignments, open(SAVE_FILE, "w"), indent=2)

remaining = 911 - len(already_assigned) - len(assignments)
print(f"\nProgress: {len(assignments)} Tier 3 decisions made, ~{remaining} remaining")
```

    
    ============================================================
    Variable:    C1HSTOWORKNO
    Label:       C1 B32T School doesn't assist students with transition from high school to work
    Instrument:  counselor_BY
    Description: In which of the following ways does the school assist students with the transition from high school to work?
    (Check all that apply.)
      Internships with local employers
      Job fairs
      Career guides or skills assessments such as KUDER, HIRE, What Color is Your Parachute
      School or classroom presentations by local employers
      Career awareness activities
      School courses in career decision making
      Ca
    
    ============================================================
    Variable:    C1G9MMSCNSL
    Label:       C1 C02A Importance of MS counselor recommendation for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Middle school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MHSCNSL
    Label:       C1 C02B Importance of HS counselor recommendation for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    High school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MMSTCHR
    Label:       C1 C02C Importance of MS teacher recommendation for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Middle school teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MMSCOURS
    Label:       C1 C02D Importance of courses taken in MS for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Courses taken in middle school
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                                                        
    
    ============================================================
    Variable:    C1G9MMSACHV
    Label:       C1 C02E Importance of achievement in MS courses for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Achievement in middle school courses
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                                                  
    
    ============================================================
    Variable:    C1G9MENDTST
    Label:       C1 C02F Importance of end-of-year/course test for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Results of district or state end-of-year or end-of-course exams
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MPLACTST
    Label:       C1 C02G Importance of placement tests for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MSTNDTST
    Label:       C1 C02H Importance of standardized tests for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Results of standardized tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MPLAN
    Label:       C1 C02I Importance of career/education plan for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9MSELECT
    Label:       C1 C02J Importance of student/parent choice for 9th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a mathematics course?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMGRADES
    Label:       C1 C04A Importance of prior grades for 10th to 12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Prior grades including grades from a prerequisite class
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" and "A little important" recoded as "Not at all important, a little important, or somewhat important" on the public
    
    ============================================================
    Variable:    C1UPMPLACTST
    Label:       C1 C04B Importance of placement tests for 10th to 12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMTCHR
    Label:       C1 C04C Importance of teacher's recommendation for 10-12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Previous year's teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMSELECT
    Label:       C1 C04D Importance of student/parent choice for 10th-12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMPLAN
    Label:       C1 C04E Importance of career/education plan for 10th-12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPMSCHED
    Label:       C1 C04F Importance of master schedule for 10th to 12th grade math placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 mathematics courses?
    Master schedule considerations
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSCNSL
    Label:       C1 C06A Importance of MS counselor recommendation for grade 9 science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Middle school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SHSCNSL
    Label:       C1 C06B Importance of HS counselor recommendation for grade 9 science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    High school counselor recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSTCHR
    Label:       C1 C06C Importance of MS teacher recommendation for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Middle school teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSCOURS
    Label:       C1 C06D Importance of courses taken in MS for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Courses taken in middle school
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SMSACHV
    Label:       C1 C06E Importance of achievement in MS courses for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Achievement in middle school courses
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SENDTST
    Label:       C1 C06F Importance of end-of-year/course test for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Results of district or state end-of-year or end-of-course exams
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SPLACTST
    Label:       C1 C06G Importance of placement tests for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SSTNDTST
    Label:       C1 C06H Importance of standardized tests for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Results of standardized tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SPLAN
    Label:       C1 C06I Importance of career/education plan for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1G9SSELECT
    Label:       C1 C06J Importance of student/parent choice for 9th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing a typical 9th grade student into a science course?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSGRADES
    Label:       C1 C08A Importance of prior grades for 10th to 12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Prior grades including grades from a prerequisite class
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                          
    
    ============================================================
    Variable:    C1UPSPLACTST
    Label:       C1 C08B Importance of placement tests for 10th to 12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Results of placement tests
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSTCHR
    Label:       C1 C08C Importance of teacher's recommendation for 10th-12th science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Previous year's teacher recommendation
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSSELECT
    Label:       C1 C08D Importance of student/parent choice for 10-12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Student and/or parent or guardian selection
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
    "Not at all important" recoded as "Not at all important or a little important" on the public use file.
    
                                                      
    
    ============================================================
    Variable:    C1UPSPLAN
    Label:       C1 C08E Importance of career/education plan for 10-12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Student career or education plan
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1UPSSCHED
    Label:       C1 C08F Importance of master schedule for 10th to 12th grade science placement
    Instrument:  counselor_BY
    Description: How important is each of the following factors in placing typical students into grades 10 through 12 science courses?
    Master schedule considerations
      Not at all important
      A little important
      Somewhat important
      Very important
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TTEACHING
    Label:       C1 D01A Teachers in this school set high standards for teaching
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    set high standards for teaching.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                            Frequency
    
    ============================================================
    Variable:    C1TLEARNING
    Label:       C1 D01B Teachers in this school set high standards for students' learning
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    set high standards for students' learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                           
    
    ============================================================
    Variable:    C1TBELIEVE
    Label:       C1 D01C Teachers in this school believe all students can do well
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    believe all students can do well.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TGIVEUP
    Label:       C1 D01D Teachers in this school have given up on some students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    have given up on some students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TCARE
    Label:       C1 D01E Teachers in this school care only about smart students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    care only about smart students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency          
    
    ============================================================
    Variable:    C1TEXPECT
    Label:       C1 D01F Teachers in this school expect very little from students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    expect very little from students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1TWORKHARD
    Label:       C1 D01G Teachers in this school work hard to make sure all students learn
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the teachers in your school? Teachers in this school...
    work hard to make sure all students are learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1CLEARNING
    Label:       C1 D02A Counselors in this school set high standards for students' learning
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    set high standards for students' learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                      
    
    ============================================================
    Variable:    C1CBELIEVE
    Label:       C1 D02B Counselors in this school believe all students can do well
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    believe all students can do well.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                            Fre
    
    ============================================================
    Variable:    C1CGIVEUP
    Label:       C1 D02C Counselors in this school have given up on some students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    have given up on some students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency     
    
    ============================================================
    Variable:    C1CCARE
    Label:       C1 D02D Counselors in this school care only about smart students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    care only about smart students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency     
    
    ============================================================
    Variable:    C1CEXPECT
    Label:       C1 D02E Counselors in this school expect very little from students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    expect very little from students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency   
    
    ============================================================
    Variable:    C1CWORKHARD
    Label:       C1 D02F Counselors in this school work hard to make sure all students learn
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about the counselors in your school?  Counselors in this school...
    work hard to make sure all students are learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                               
    
    ============================================================
    Variable:    C1PLEARNING
    Label:       C1 D03A Principal in this school sets high standards for students' learning
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    sets high standards for students' learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                        
    
    ============================================================
    Variable:    C1PBELIEVE
    Label:       C1 D03B Principal in this school believes all students can do well
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    believes all students can do well.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                            Frequ
    
    ============================================================
    Variable:    C1PGIVEUP
    Label:       C1 D03C Principal in this school has given up on some students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    has given up on some students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
                                                                            Frequency             Percent
    
    ============================================================
    Variable:    C1PCARE
    Label:       C1 D03D Principal in this school cares only about smart students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    cares only about smart students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency       
    
    ============================================================
    Variable:    C1PEXPECT
    Label:       C1 D03E Principal in this school expects very little from students
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    expects very little from students.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly agree" recoded as "Strongly agree or agree" on the public use file.
    
                                                                            Frequency     
    
    ============================================================
    Variable:    C1PWORKHARD
    Label:       C1 D03F Principal in this school works hard to make sure all students learn
    Instrument:  counselor_BY
    Description: To what extent do you agree or disagree with each of the following statements about your school's principal?  The principal in this school...
    works hard to make sure all students are learning.
      Strongly agree
      Agree
      Disagree
      Strongly disagree
    
    
    "Strongly disagree" recoded as "Disagree or strongly disagree" on the public use file.
    
                                                                 
    
    ============================================================
    Variable:    C1HIMAJ_STEM
    Label:       C1 D06C Counselor's major for highest level of education STEM code
    Instrument:  counselor_BY
    Description: Major STEM indicators are constructed using the following logic:
    
    array majora P1HIMAJ61 P1BAMAJ61 P1HIMAJ62 P1BAMAJ62 P2HIMAJ61 P2HIMAJ62 
                 M1HIMAJ6 M1BAMAJ6 N1HIMAJ6 N1BAMAJ6 
                 A1HIMAJ6 A1BAMAJ6 A2HIMAJ6 A2BAMAJ6 
                 C1HIMAJ6 C1BAMAJ6 S3FIELD6;
    array majorn P1HIMAJ61n P1BAMAJ61n P1HIMAJ62n P1BAMAJ62n P2HIMAJ61n P2HIMAJ62n 
                 M1HIMAJ6n M1BAMAJ6n N1HIMA
    
    ============================================================
    Variable:    C1BAMAJ_STEM
    Label:       C1 D07C Counselor's major for Bachelor's degree STEM code
    Instrument:  counselor_BY
    Description: Major STEM indicators are constructed using the following logic:
    
    array majora P1HIMAJ61 P1BAMAJ61 P1HIMAJ62 P1BAMAJ62 P2HIMAJ61 P2HIMAJ62 
                 M1HIMAJ6 M1BAMAJ6 N1HIMAJ6 N1BAMAJ6 
                 A1HIMAJ6 A1BAMAJ6 A2HIMAJ6 A2BAMAJ6 
                 C1HIMAJ6 C1BAMAJ6 S3FIELD6;
    array majorn P1HIMAJ61n P1BAMAJ61n P1HIMAJ62n P1BAMAJ62n P2HIMAJ61n P2HIMAJ62n 
                 M1HIMAJ6n M1BAMAJ6n N1HIMA
    
    Progress: 641 Tier 3 decisions made, ~0 remaining



```python
assignments = json.load(open("tier3_assignments.json"))

for variable, decision in assignments.items():
    if decision["certain"]:
        continue
    
    print(f"\n{'='*60}")
    print(f"Variable: {variable}")
    print(f"Current:  tier_3={decision['tier_3']}")
    
    row = codebook_df[codebook_df["variable"] == variable]
    if not row.empty:
        print(f"Label:    {row.iloc[0]['label']}")
        print(f"Description: {row.iloc[0]['description'][:300]}")
    
    tier3 = input("\nTier 3? (y/n/q to quit): ").strip().lower()
    if tier3 == "q":
        break
    
    assignments[variable]["tier_3"] = (tier3 == "y")
    assignments[variable]["certain"] = True
    
    json.dump(assignments, open("tier3_assignments.json", "w"), indent=2)

print("Done")
```

    
    ============================================================
    Variable: S1NOMSACT
    Current:  tier_3=True
    Label:    S1 B04I 9th grader did not participate in any math/science activities listed
    Description: Since the beginning of the last school year (2008-2009), which of the following activities have you participated in?
    (Check all that apply.)
      Math club
      Math competition
      Math camp
      Math study groups or a program where you were tutored in math
      Science club
      Science competition
      Science camp
    
    
    ============================================================
    Variable: S1MUNDERST
    Current:  tier_3=True
    Label:    S1 C02 How often 9th grader thinks he/she really understands math assignments
    Description: When you are working on a math assignment, how often do you think you really understand the assignment?
      Never
      Rarely
      Sometimes
      Often
    
    
                                                                            Frequency             Percent
    Done



```python
tier1 = json.load(open("tier1_assignments.json"))
tier2 = json.load(open("tier2_assignments.json"))
exclusions = json.load(open("exclusions.json"))
tier3 = json.load(open("tier3_assignments.json"))

col_names = open("hsls_column_names.txt").read().strip().split("\n")

tier1_vars = {k for k, v in tier1.items() if v["tier_1"]}
tier2_vars = {k for k, v in tier2.items() if v["tier_2"]}
excluded_vars = {k for k, v in exclusions.items() if v["exclude"]}
tier3_vars = {k for k, v in tier3.items() if v["tier_3"]}

assigned = tier1_vars | tier2_vars | excluded_vars | tier3_vars

tier4_vars = [c for c in col_names if c not in assigned
              and c != "STU_ID" and c != "X4EVERDROP"]

tier4_assignments = {v: {"tier_4": True, "certain": True} for v in tier4_vars}

json.dump(tier4_assignments, open("tier4_assignments.json", "w"), indent=2)
print(f"Tier 4 assignments saved: {len(tier4_assignments)} variables")
```

    Tier 4 assignments saved: 397 variables



```python
tier1 = json.load(open("tier1_assignments.json"))
tier2 = json.load(open("tier2_assignments.json"))
tier3 = json.load(open("tier3_assignments.json"))
tier4 = json.load(open("tier4_assignments.json"))
exclusions = json.load(open("exclusions.json"))

col_names = open("hsls_column_names.txt").read().strip().split("\n")

# Build master assignment for every variable
master = {}
for v in col_names:
    if v in ("STU_ID", "X4EVERDROP"):
        continue
    
    if v in tier1 and tier1[v]["tier_1"]:
        master[v] = 1
    elif v in tier2 and tier2[v]["tier_2"]:
        master[v] = 2
    elif v in exclusions and exclusions[v]["exclude"]:
        master[v] = None  # excluded
    elif v in tier3 and tier3[v]["tier_3"]:
        master[v] = 3
    else:
        master[v] = 4

# Build cumulative feature lists
tier1_features = [v for v, t in master.items() if t == 1]
tier2_features = [v for v, t in master.items() if t in (1, 2)]
tier3_features = [v for v, t in master.items() if t in (1, 2, 3)]
tier4_features = [v for v, t in master.items() if t in (1, 2, 3, 4)]

print(f"Tier 1 features: {len(tier1_features)}")
print(f"Tier 2 features: {len(tier2_features)}")
print(f"Tier 3 features: {len(tier3_features)}")
print(f"Tier 4 features: {len(tier4_features)}")
print(f"Excluded: {sum(1 for t in master.values() if t is None)}")

# Save as JSON
output = {
    "tier1": tier1_features,
    "tier2": tier2_features,
    "tier3": tier3_features,
    "tier4": tier4_features,
}

json.dump(output, open("feature_tiers.json", "w"), indent=2)
print("\nSaved to feature_tiers.json")
```

    Tier 1 features: 130
    Tier 2 features: 192
    Tier 3 features: 436
    Tier 4 features: 833
    Excluded: 78
    
    Saved to feature_tiers.json



```python

```
