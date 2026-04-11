```python
import pandas as pd
import numpy as np
from sklearn.model_selection import train_test_split
import json

df = pd.read_csv("hsls_clean.csv")
print(df.shape)
print(df["X4EVERDROP"].value_counts(normalize=True))
```

    (15818, 913)
    X4EVERDROP
    0    0.850803
    1    0.149197
    Name: proportion, dtype: float64



```python
train_idx, test_idx = train_test_split(
    df.index,
    test_size=0.2,
    random_state=42,
    stratify=df["X4EVERDROP"]
)

print(f"Train: {len(train_idx)} students")
print(f"Test:  {len(test_idx)} students")
print(f"\nTrain dropout rate: {df.loc[train_idx, 'X4EVERDROP'].mean():.3f}")
print(f"Test dropout rate:  {df.loc[test_idx, 'X4EVERDROP'].mean():.3f}")
```

    Train: 12654 students
    Test:  3164 students
    
    Train dropout rate: 0.149
    Test dropout rate:  0.149



```python
split = {
    "train": train_idx.tolist(),
    "test": test_idx.tolist(),
    "random_state": 42,
    "test_size": 0.2,
    "train_dropout_rate": float(df.loc[train_idx, "X4EVERDROP"].mean()),
    "test_dropout_rate": float(df.loc[test_idx, "X4EVERDROP"].mean()),
}

json.dump(split, open("train_test_split.json", "w"))
print("Saved train_test_split.json")
```

    Saved train_test_split.json



```python

```
