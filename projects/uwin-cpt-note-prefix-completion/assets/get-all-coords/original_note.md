# Verified repository information and learnings

## Task scope
The task was to trace the execution of `Indexes.get_all_coords()` for a coordinate that is part of a multi-index, document the code path, and describe the structure of the returned coordinate dictionary.

## Multi-index storage contract
When a Dataset contains a pandas MultiIndex, the same `PandasMultiIndex` instance is stored under all coordinate names (the dimension name and each level name) in `Indexes._indexes`. Runtime verification confirms all entries in `Indexes._indexes` for a multi-index point to the same object (same memory address).

For example, with a Dataset containing a MultiIndex on dimension 'z' with levels 'level_0' and 'level_1':
```python
ds.indexes._indexes: {
    'z': <PandasMultiIndex object at 0x...>,
    'level_0': <PandasMultiIndex object at 0x...>,  # same object
    'level_1': <PandasMultiIndex object at 0x...>   # same object
}
```

## Internal mapping structure
`Indexes` maintains three internal mappings for tracking coordinate-index relationships:

1. **`_coord_name_id`**: Maps each coordinate name to the memory address (`id`) of its index. For a multi-index, all names map to the same id.
2. **`_id_coord_names`**: Maps each index id to a tuple of all coordinate names sharing that index.
3. **`_id_index`**: Maps each index id to the index object itself (used by `get_unique()` and `is_multi()`).

## Step-by-step code path for `get_all_coords('one')`

The method is located at `xarray/core/indexes.py:1111-1141`:

1. **Entry**: `get_all_coords(key: Hashable, errors: ErrorOptions = "raise") -> dict[Hashable, Variable]`
2. **Validation** (lines 1132-1136):
   - Confirms `errors` is "raise" or "ignore"
   - Checks `key in self._indexes` - for a multi-index, the key is present because the same PandasMultiIndex is stored under all coordinate names
3. **Lookup index id** (line 1137):
   - `all_coord_names = self._id_coord_names[self._coord_name_id[key]]`
   - `_coord_name_id[key]` returns the index's memory address
4. **Build result dictionary** (line 1138):
   - `return {k: self._variables[k] for k in all_coord_names}`
   - Returns a dict mapping each coordinate name to its `Variable`

## Structure of returned coordinate dictionary
```python
{
    'z': Variable(
        dims=('z',),
        values=MultiIndex([(1, 'a'), (2, 'b'), (3, 'c'), (4, 'd')])
    ),
    'level_0': Variable(
        dims=('z',),
        values=np.array([1, 2, 3, 4])
    ),
    'level_1': Variable(
        dims=('z',),
        values=np.array(['a', 'b', 'c', 'd'])
    )
}
```

- **Type**: Standard Python `dict`
- **Keys**: All coordinate names sharing the same index (dimension name + level names)
- **Values**: `xarray.Variable` objects with `dims` (dimension tuple) and `values` (data array)

## Error handling behavior
- `get_all_coords(key, errors="raise")` raises `ValueError: no index found for {key!r} coordinate` if key not found
- `get_all_coords(key, errors="ignore")` returns empty dict `{}` if key not found

## Non-multi-index behavior
For regular (non-multi) indexes, `get_all_coords()` returns only that single coordinate.

## Key source references
- `xarray/core/indexes.py:1111-1141`: `get_all_coords()` method
- `xarray/core/indexes.py:1066-1073`: `_id_coord_names` property
- `xarray/core/indexes.py:1054-1059`: `_coord_name_id` property
- `xarray/core/indexes.py:537-740`: `PandasMultiIndex` class
- `xarray/core/indexes.py:735-768`: `PandasMultiIndex.create_variables()` method

## Test verification
All tests in `xarray/tests/test_indexes.py::TestIndexes` pass (20 tests), including `test_get_all_coords`. All tests in `xarray/tests/test_indexes.py::TestPandasMultiIndex` pass (11 tests), including `test_create_variables`.

## Correction to initial test setup
The first test attempt failed because it constructed `Indexes` directly with `indexes={'z': multi_idx}` and `variables={'one': ..., 'two': ...}`. This does not match how xarray populates `Indexes` for a multi-index. The corrected approach uses a Dataset, which properly initializes the Indexes object with the same PandasMultiIndex stored under all coordinate names.
