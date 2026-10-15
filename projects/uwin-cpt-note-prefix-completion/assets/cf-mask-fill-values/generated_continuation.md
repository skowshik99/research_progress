**Test coverage:**
- `xarray/tests/test_coding.py::test_CFMaskCoder_decode` (lines 40–50) verifies the warning is raised when multiple fill values are present.
- `xarray/tests/test_coding.py::test_CFMaskCoder_decode_dask` (lines 52–62) verifies the same behavior for dask-backed arrays.

**Verified probe results:**
- With `name=None`: Warning message is `variable None has multiple fill values {2, -1}, decoding all values to NaN.`
- With `name='temperature'`: Warning message is `variable 'temperature' has multiple fill values {2, -1}, decoding all values to NaN.`
- With single fill value: No warning is raised.
- With no fill values: No warning is raised.
- With only `missing_value` attribute: No warning is raised.
- With only `_FillValue` attribute: No warning is raised.
- With both attributes containing the same value: No warning is raised (deduplicated by the set).
- With both attributes containing different values: Warning is raised with the deduplicated set of values.
