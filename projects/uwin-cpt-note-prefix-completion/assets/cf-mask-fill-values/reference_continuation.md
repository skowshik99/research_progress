**Edge cases verified:**
- Single fill value: no warning
- Fill value(s) that are `NaN`: no warning (filtered by `pd.isnull`)
- Only `missing_value` with a single value: no warning
- Both `missing_value` and `_FillValue` with overlapping values: deduplicated by the set, so only distinct values are counted; no warning if deduplicated to one value
- Multiple fill values from both attributes: warning raised with all distinct values

**Example warning messages:**
- No name provided: `variable None has multiple fill values {2, -1}, decoding all values to NaN.`
- With name: `variable 'temperature' has multiple fill values {2, -1}, decoding all values to NaN.`
- Multiple fill values from both attributes: `variable 'temp' has multiple fill values {2, 3, -1}, decoding all values to NaN.`

**Variable class note:**
The `Variable` class (`xarray/core/variable.py`, line 273) does not store a `name` attribute (its `__slots__` are `"_dims", "_data", "_attrs", "_encoding"`). The `name` used in the warning is the optional `name` argument passed to `decode()`. If not provided, the warning shows `variable None`.
