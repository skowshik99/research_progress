# Verified repository information and learnings

## Function definition and contract

**Location**: `xarray/core/groupby.py:67-92`

**Signature**:
```python
def unique_value_groups(ar, sort: bool = True) -> tuple[np.ndarray | pd.Index, list[list[int]]]:
```

**Purpose**: Groups an array by its unique values, returning unique values and a list of index lists for each group.

**Parameters**:
- `ar`: array-like (must be 1-D; 2-D arrays raise `ValueError`)
- `sort`: bool, default `True`; whether to sort unique values

**Returns**:
- `values`: unique values as `np.ndarray` or `pd.Index` (or `pd.MultiIndex`)
- `indices`: `list[list[int]]` where each inner list contains the indices in `ar` that share that unique value

## Core algorithm (verified by code and probes)

### 1. pd.factorize call (`line 85`)
```python
inverse, values = pd.factorize(ar, sort=sort)
```
- `inverse`: integer codes for each element (0, 1, 2, ...)
- `values`: array of unique values
- **NaN behavior**: pandas assigns `-1` in `inverse` for NaN values; NaN values are **not** included in `values`

**Probe verification**: For input `[1, nan, 2, nan, 1, 3]`, `pd.factorize` returns `inverse=[0, -1, 1, -1, 0, 2]` and `values=[1., 2., 3.]`.

### 2. MultiIndex name preservation (`line 86-87`)
```python
if isinstance(values, pd.MultiIndex):
    values.names = ar.names
```
- If `values` is a `pd.MultiIndex`, the names from the input array `ar` are copied to `values`

**Probe verification**: For `MultiIndex([(1, 'a'), (2, 'b'), (1, 'a'), (3, 'c')], names=['level1', 'level2'])`, `unique_value_groups` returns a `MultiIndex` with names `['level1', 'level2']`.

**Limitation**: When `IndexVariable` is created with these values in GroupBy, the MultiIndex is converted to a numpy array of tuples, losing the MultiIndex structure and names.

### 3. Group construction (`line 88-91`)
```python
groups: list[list[int]] = [[] for _ in range(len(values))]
for n, g in enumerate(inverse):
    if g >= 0:
        groups[g].append(n)
```
- Creates an empty list for each unique value
- Iterates through `inverse`, appending index `n` to `groups[g]` only when `g >= 0`
- **NaN handling**: indices where `g == -1` (NaN values) are skipped

**Probe verification**: For input `[1, nan, 2, nan, 1, 3]`, the function returns `groups=[[0, 4], [2], [5]]`, correctly excluding the NaN indices.

## Usage in xarray GroupBy

**Location**: `xarray/core/groupby.py:430-436`

### NaN handling before `unique_value_groups` (`line 430-431`)
```python
if isinstance(group, DataArray) and group.isnull().any():
    obj = obj.where(group.notnull(), drop=True)
    group = group.dropna(group_dim)
```
- NaN values in the group coordinate are dropped **before** calling `unique_value_groups`
- The underlying object `obj` is also filtered to remove values where the group was NaN
- **Result**: `unique_value_groups` typically never encounters NaN values in normal groupby operations

**Probe verification**: For a DataArray with coordinates `[1, nan, 3, nan, 5]`, GroupBy produces `_group_indices=[[0], [1], [2]]` and `_unique_coord=[1., 3., 5.]`, correctly excluding NaN indices.

### Sort parameter logic (`line 434`)
```python
sort = bins is None and (not isinstance(group_as_index, pd.MultiIndex))
```
- `sort=True` when `bins=None` and group is not a `MultiIndex`
- `sort=False` when group is a `MultiIndex` (to preserve tuple order)

**Probe verification**: For input `[3, 1, 2, 1, 3]`, `sort=True` returns `values=[1, 2, 3]` with groups `[[1, 3], [2], [0, 4]]`, while `sort=False` returns `values=[3, 1, 2]` with groups `[[0, 4], [1, 3], [2]]`.

## Probe evidence summary

| Input type | Example input | `pd.factorize` output | `unique_value_groups` groups | Notes |
|------------|---------------|----------------------|------------------------------|-------|
| Basic numeric | `[1,2,1,3,2,1,4]` | `inverse=[0,1,0,2,1,0,3]`, `values=[1,2,3,4]` | `[[0,2,5],[1,4],[3],[6]]` | Standard grouping |
| With NaN | `[1,nan,2,nan,1,3]` | `inverse=[0,-1,1,-1,0,2]`, `values=[1,2,3]` | `[[0,4],[2],[5]]` | NaN indices skipped |
| Strings | `['a','b','a','c','b','a']` | `inverse=[0,1,0,2,1,0]`, `values=['a','b','c']` | `[[0,2,5],[1,4],[3]]` | Works with object arrays |
| MultiIndex | `MultiIndex([(1,'a'),...]` | `inverse=[0,1,2,3]`, `values=MultiIndex` | `[[0],[1],[2],[3]]` | MultiIndex preserved in return |
| Sort=False | `[3,1,2,1,3]` | `inverse=[0,1,2,1,0]`, `values=[3,1,2]` | `[[0,4],[1,3],[2]]` | Order preserved |
| 2D array | `[[1,2],[1,3],[2,1]]` | **Raises ValueError** | N/A | 2-D arrays are not supported |
| All NaN | `[nan, nan, nan]` | `inverse=[-1,-1,-1]`, `values=[]` | `[]` | Returns empty groups |
| Empty array | `[]` | `inverse=[]`, `values=[]` | `[]` | Returns empty groups |

## GroupBy internal state (probe verification)

- **With NaN in group coordinate**: `_group_indices=[[0], [1], [2]]` (NaN indices excluded), `_unique_coord` contains 3 unique values (NaN dropped)
- **With MultiIndex**: `_unique_coord` is converted to numpy array of tuples, `_group_indices=[0, 1, 2, 3]` (one index per unique tuple)

## Corrections and verified limitations

1. **Type annotation bug**: The return type annotation `tuple[np.ndarray | pd.Index, list[list[int]]]` does not include `pd.MultiIndex`, which is a valid return type when the input is a `pd.MultiIndex`.

2. **Documentation bug**: The docstring states "This will be flattened if it is not already 1-D", but there is no flattening code in the implementation. 2-D arrays raise a `ValueError` from `pd.factorize`.

3. **MultiIndex preservation limitation**: While `unique_value_groups` preserves MultiIndex structure and names in its return value, `IndexVariable` (used in GroupBy) converts it to a numpy array of tuples, losing the MultiIndex structure and names.

4. **Single usage**: `unique_value_groups` is called only once in `xarray/core/groupby.py` at line 436, during GroupBy initialization.

5. **Edge cases**: All-NaN arrays return empty groups; empty arrays return empty groups; GroupBy raises `ValueError: Failed to group data. Are you grouping by a variable that is all NaN?` for all-NaN group coordinates.
