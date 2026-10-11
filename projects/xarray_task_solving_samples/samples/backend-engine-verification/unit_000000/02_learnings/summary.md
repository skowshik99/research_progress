Repository information and learnings

1. Task and scope
The study traced how xarray automatically selects a backend engine for a given file path via `_get_default_engine()` in `xarray/backends/api.py`. It documented the decision tree for remote URIs, `.gz` files, and standard netCDF files, and identified which Python dependencies are checked and what error messages are raised when they are missing. The investigation verified the behavior with unit tests and focused probes that simulated missing dependencies.

2. Repository information and learnings
- Core function and signature
  - `xarray.backends.api._get_default_engine(path: str, allow_remote: bool = False)` returns a literal engine name (`"netcdf4"` or `"scipy"` for local files; `"netcdf4"` or `"pydap"` for remote URIs).
  - Decision logic:
    - If `allow_remote=True` and `is_remote_uri(path)` is true, it calls `_get_default_engine_remote_uri()`.
    - Else if `path.endswith(".gz")`, it calls `_get_default_engine_gz()`.
    - Otherwise, it calls `_get_default_engine_netcdf()`.

- Remote URIs
  - Detection: `xarray.core.utils.is_remote_uri(path)` uses the regex `r"^[a-z][a-z0-9]*(\://|:\:)"`. This matches schemes like `http://`, `https://`, `ftp://`, `opendap://`, `s3://`, `gs://`, `file://`, and even `http::/` (double colon).
  - Engine selection:
    - Tries to import `netCDF4`; if successful, returns `"netcdf4"`.
    - Fallback: tries to import `pydap`; if successful, returns `"pydap"`.
    - If both fail, raises `ValueError("netCDF4 or pydap is required for accessing remote datasets via OPeNDAP")`.
  - Evidence: Probes confirmed `"netcdf4"` when `netCDF4` is present, `"pydap"` when `netCDF4` is absent but `pydap` is present, and the ValueError when both are absent.

- `.gz` files
  - Trigger: Any path ending with `.gz`.
  - Engine selection:
    - Requires `scipy`. If present, returns `"scipy"`.
    - If missing, raises `ValueError("scipy is required for accessing .gz files")`.
  - Evidence: Probe with `scipy` mocked absent raised the above ValueError.

- Standard netCDF files
  - Trigger: Paths that are neither remote URIs nor `.gz`.
  - Engine selection:
    - Tries to import `netCDF4`; if present, returns `"netcdf4"`.
    - Fallback: tries to import `scipy.io.netcdf`; if present, returns `"scipy"`.
    - If both fail, raises `ValueError("cannot read or write netCDF files without netCDF4-python or scipy installed")`.
  - Evidence: Probe with `netCDF4` absent but `scipy` present returned `"scipy"`. Probe with both absent raised the ValueError.

- Available engines in the codebase
  - `xarray/backends/api.py` defines `ENGINES` mapping engine names to backend entry points:
    - `"netcdf4"`, `"scipy"`, `"pydap"`, `"h5netcdf"`, `"pynio"`, `"pseudonetcdf"`, `"cfgrib"`, `"zarr"`.
  - `_get_default_engine()` only considers `netcdf4`, `scipy`, and `pydap` depending on file type; other engines are not part of automatic selection.

- Test coverage
  - `xarray/tests/test_backends_api.py::test__get_default_engine` asserts:
    - Remote URI with `allow_remote=True` returns `"netcdf4"`.
    - `.gz` file returns `"scipy"`.
    - Standard file returns `"netcdf4"`.
  - The test passed in the environment.

3. Corrections and limits
- No corrections were needed: the earlier conclusions about the decision tree and error messages match the source code and probe results.
- Unknowns/limitations:
  - The study did not inspect how `open_dataset` or `open_dataarray` use `_get_default_engine` when `engine=None`; the integration path (e.g., how `allow_remote` is passed or defaulted) was not traced.
  - The behavior for compressed non-`.gz` files (e.g., `.bz2`, `.zip`) is not covered by `_get_default_engine`; their engine selection strategy is not documented here.
  - The study did not verify whether `allow_remote` is ever set to `True` by default in user-facing open functions; only the behavior when explicitly provided was observed.

4. Useful next checks
- Trace how `allow_remote` is determined inside `open_dataset`/`open_dataarray` when `engine=None`, including whether it defaults to `False` or is derived from the path.
- Determine how non-`.gz` compressed formats are handled in engine selection (if at all) and whether they rely on magic-number detection or explicit engine specification.
- Confirm whether `is_remote_uri` is used consistently across all open functions and whether any additional protocol prefixes (e.g., `https`) require special handling beyond the current regex.