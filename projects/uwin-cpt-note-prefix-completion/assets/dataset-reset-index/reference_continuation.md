### MultiIndex Special Case

For `PandasMultiIndex` where the name is not in `self.dims` (i.e., it's a level name):

```python
if isinstance(index, PandasMultiIndex) and name not in self.dims:
    # special case for pd.MultiIndex (name is an index level)
    level_names = index.index.names
    level_vars = {
        k: self._variables[k]
        for k in level_names
        if k not in dims_or_levels
    }
    if level_vars:
        idx = index.keep_levels(level_vars)
        idx_vars = idx.create_variables(level_vars)
        new_indexes.update({k: idx for k in idx_vars})
        new_variables.update(idx_vars)
```

This uses **`PandasMultiIndex.keep_levels()`** (defined in `xarray/core/indexes.py` at **line 705**):

```python
def keep_levels(
    self, level_variables: Mapping[Any, Variable]
) -> PandasMultiIndex | PandasIndex:
    """Keep only the provided levels and return a new multi-index with its
    corresponding coordinates."""
    index = self.index.droplevel(
        [k for k in self.index.names if k not in level_variables]
    )
    
    if isinstance(index, pd.MultiIndex):
        level_coords_dtype = {k: self.level_coords_dtype[k] for k in index.names}
        return self._replace(index, level_coords_dtype=level_coords_dtype)
    else:
        return PandasIndex(
            index, self.dim, coord_dtype=self.level_coords_dtype[index.name]
        )
```

**Evidence from probe:**
```
Index object type: <class 'xarray.core.indexes.PandasMultiIndex'>
Index names: ['num', 'letter']

New index type: <class 'xarray.core.indexes.PandasIndex'>
New index names: ['num']

New variables: ['num']
```

### The drop Parameter

When `drop=True`:
- The dimension name is added to `drop_variables`, removing its coordinate variable
- The data variable with that dimension is also removed
- The dimension itself is kept (not removed)

**Evidence from probe:**
```
After reset_index("idx", drop=True):
Coordinates:
    num      (idx) int64 1 1 2 2
    letter   (idx) object 'a' 'b' 'a' 'b'
Data variables:
    *empty*
```

### Call Chain Summary

```
Dataset.reset_index()
    ↓
Dataset._replace(variables, coord_names, indexes)
    ↓
Dataset._construct_direct(...)  # Reconstructs the Dataset
```

For MultiIndex level drop:
```
PandasMultiIndex.keep_levels(level_variables)
    ↓
pd.MultiIndex.droplevel(...)  # Pandas operation
    ↓
PandasMultiIndex._replace(index, level_coords_dtype)
    ↓
PandasIndex or PandasMultiIndex
    ↓
PandasIndex.create_variables()  # Creates coordinate variables
```

### Relationship to set_index

The `set_index` method (line 3968) is the inverse operation:
- `set_index`: Converts regular coordinates to an index
- `reset_index`: Converts an index back to regular coordinates

Both methods use `self._replace()` to reconstruct the dataset.

**Evidence from probe:**
```
After stack:
Dimensions:  (idx: 6)
Coordinates:
  * idx      (idx) object MultiIndex
  * x        (idx) int64 10 10 10 20 20 20
  * y        (idx) int64 100 200 300 100 200 300

After reset_index("idx"):
Dimensions:  (idx: 6)
Coordinates:
    idx      (idx) object MultiIndex
    x        (idx) int64 10 10 10 20 20 20
    y        (idx) int64 100 200 300 100 200 300

After reset_index("idx", drop=True):
Dimensions:  (idx: 6)
Coordinates:
    x        (idx) int64 10 10 10 20 20 20
    y        (idx) int64 100 200 300 100 200 300
Data variables:
    *empty*
```

### Key Constraints

1. **Cannot call reset_index on non-index names**: After `reset_index('idx')`, the level coordinates 'num' and 'letter' are no longer indexes, so calling `reset_index('num')` raises `ValueError: ('num',) are not coordinates with an index`.

2. **Index removal is by name**: The filtering `indexes = {k: v for k, v in self._indexes.items() if k not in drop_indexes}` removes indexes whose names appear in `drop_indexes`.

3. **Variable filtering is separate**: The `drop_variables` list controls which coordinate variables are removed, independent of index removal.

4. **reset_index and set_index are not perfect inverses**: After `reset_index`, the coordinate variables retain their original dimension name (e.g., 'idx'), so calling `set_index` with the original dimension names will fail due to dimension mismatch.

### Verified Behavior with Dask Arrays

**Evidence from probe:**
```
Original dask dataset:
Indexes: ['x', 'y']

After reset_index("x"):
Indexes: ['y']
Chunks: ((1, 1), (1, 1))
```

The `reset_index` operation preserves dask array chunks.

### Test Failures

The existing tests in `xarray/tests/test_dataset.py` for `reset_index` are failing:
- `test_reset_index`: Expects MultiIndex to be completely removed, but actual behavior keeps it as a coordinate
- `test_reset_index_drop_dims`: Expects dimension to be removed with `drop=True`, but actual behavior keeps the dimension

These test failures suggest the tests may be outdated or the implementation has changed since the tests were written.
