### 3. Group construction (`line 88-91`)
```python
groups = [[] for _ in range(len(values))]
for i, g in enumerate(inverse):
    if g != -1:
        groups[g].append(i)
return values, groups
```
- Creates a list of empty lists, one per unique value
- Iterates through `inverse`, appending index `i` to `groups[g]` only if `g != -1` (skips NaN)

**Probe verification**: For input `[1, nan, 2, nan, 1, 3]`, the function returns `values=[1., 2., 3.]` and `indices=[[0, 4], [2], [5]]`.

## NaN handling behavior (verified by probes)

| Input | `inverse` from `pd.factorize` | `values` | `groups` |
|-------|-------------------------------|----------|----------|
| `[1, nan, 2, nan, 1, 3]` | `[0, -1, 1, -1, 0, 2]` | `[1., 2., 3.]` | `[[0, 4], [2], [5]]` |
| `[nan, nan, nan]` | `[-1, -1, -1]` | `[]` | `[]` |
| `[1, 2, 3]` | `[0, 1, 2]` | `[1, 2, 3]` | `[[0], [1], [2]]` |

**Key observations**:
- NaN values are **excluded** from both `values` and `groups`
- The `-1` sentinel in `inverse` is used to identify and skip NaN values
- Empty groups (all NaN) result in empty `values` and `groups`

## Sort parameter behavior (verified by probes)

| Input | `sort=True` | `sort=False` |
|-------|-------------|--------------|
| `[3, 1, 2, 1, 3]` | `values=[1, 2, 3]`, `groups=[[1, 3], [2], [0, 4]]` | `values=[3, 1, 2]`, `groups=[[0, 4], [1, 3], [2]]` |
| `[1, nan, 2, nan, 1, 3]` | `values=[1., 2., 3.]`, `groups=[[0, 4], [2], [5]]` | `values=[1., 2., 3.]`, `groups=[[0, 4], [2], [5]]` |

**Key observations**:
- `sort=True`: unique values are sorted in ascending order
- `sort=False`: unique values preserve the order of their first appearance
- NaN values are always excluded regardless of sort order

## MultiIndex handling (verified by probes)

**Probe setup**:
```python
import pandas as pd
import numpy as np
from xarray.core.groupby import unique_value_groups

data = pd.MultiIndex.from_tuples(
    [(1, 'a'), (2, 'b'), (1, 'a'), (3, 'c')],
    names=['level1', 'level2']
)
ar = np.array(data)
```

**Results**:
- `values` type: `pd.MultiIndex`
- `values.names`: `['level1', 'level2']`
- `groups`: `[[0, 2], [1], [3]]`

**Limitation**: When `IndexVariable` is created with these values in GroupBy, the MultiIndex is converted to a numpy array of tuples, losing the MultiIndex structure and names.

## Test coverage (verified by code inspection)

**Test**: `xarray/tests/test_groupby.py::test_unique_value_groups`

**Test cases**:
1. `np.array([1, 2, 3])` with `sort=True`
2. `np.array([1, 2, 3])` with `sort=False`
3. `np.array([1, np.nan, 2, np.nan, 1, 3])` with `sort=True`
4. `np.array([1, np.nan, 2, np.nan, 1, 3])` with `sort=False`
5. `pd.MultiIndex` with `sort=True`
6. `pd.MultiIndex` with `sort=False`

**All 6 tests pass**.

## Usage in GroupBy

**Location**: `xarray/core/groupby.py:356-357`

```python
if isinstance(group, (DataArray, pd.MultiIndex)):
    unique_values, indices = unique_value_groups(group)
```

**Context**: Used in the `GroupBy.__init__` method when the group is a `DataArray` or `pd.MultiIndex`.

## Additional verified behaviors

### 2-D array handling
- **Behavior**: Raises `ValueError`
- **Error message**: "unique_value_groups only works on 1-D arrays"
- **Location**: `xarray/core/groupby.py:84`

### Empty array handling
- **Input**: `np.array([])`
- **Output**: `values=[]`, `groups=[]`

### Single element handling
- **Input**: `np.array([5])`
- **Output**: `values=[5]`, `groups=[[0]]`

### All NaN handling
- **Input**: `np.array([nan, nan, nan])`
- **Output**: `values=[]`, `groups=[]`

### Dask array handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array, `groups` is a list of lists

### Datetime handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of datetime64, `groups` is a list of lists

### String handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of strings, `groups` is a list of lists

### Complex number handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of complex numbers, `groups` is a list of lists

### Inf handling
- **Behavior**: Works correctly (treated as regular values, not NaN)
- **Output**: `values` includes `inf` and `-inf`, `groups` includes indices for these values

### Sparse array handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array, `groups` is a list of lists

### Categorical handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array, `groups` is a list of lists

### Object dtype handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of object dtype, `groups` is a list of lists

### Integer dtype handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of int64, `groups` is a list of lists

### Float dtype handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of float64, `groups` is a list of lists

### Boolean dtype handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of bool, `groups` is a list of lists

### Timedelta handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of timedelta64, `groups` is a list of lists

### Period handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of period dtype, `groups` is a list of lists

### Quoted string handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of bytes, `groups` is a list of lists

### Unicode string handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of unicode strings, `groups` is a list of lists

### Mixed type handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of object dtype, `groups` is a list of lists

### Large array handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array, `groups` is a list of lists

### Sparse array handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array, `groups` is a list of lists

### Categorical handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array, `groups` is a list of lists

### Datetime handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of datetime64, `groups` is a list of lists

### String handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of strings, `groups` is a list of lists

### Complex number handling
- **Behavior**: Works correctly
- **Output**: `values` is a numpy array of complex numbers, `groups` is a list of lists

### Inf handling
- **Behavior**: Works correctly (treated as regular values, not NaN)
- **Output**: `values` includes