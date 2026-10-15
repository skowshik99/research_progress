# Verified repository information and learnings

## OPTIONS accessed in file_manager.py

Two global configuration options are accessed in `xarray/backends/file_manager.py`:

### `file_cache_maxsize`
- **Location**: Line 16 in `xarray/backends/file_manager.py`: `FILE_CACHE = LRUCache(maxsize=OPTIONS["file_cache_maxsize"], on_evict=lambda k, v: v.close())`
- **Effect**: Sets the maximum number of open files held in xarray's global LRU cache. When the cache evicts a file, the `on_evict` lambda automatically closes it.
- **Default value**: 128 (defined in `xarray/core/options.py` line 65)
- **Validation**: Must be a positive integer via `_positive_integer` validator. Setting to 0 raises `ValueError`.
- **Usage**:
  - At module load time, `FILE_CACHE` is initialized with this value
  - The setter `xarray.set_options(file_cache_maxsize=<int>)` calls `_set_file_cache_maxsize(value)` which directly assigns `FILE_CACHE.maxsize = value`
  - At module load time, an assertion ensures `FILE_CACHE.maxsize >= 1` (line 18)
- **Test evidence**: `xarray/tests/test_options.py::test_file_cache_maxsize` confirms:
  - Setting `file_cache_maxsize=0` raises `ValueError`
  - Setting `file_cache_maxsize=123` updates `FILE_CACHE.maxsize` to 123
  - The setting is reverted when exiting the context manager

### `warn_for_unclosed_files`
- **Location**: Line 250 in `xarray/backends/file_manager.py` inside `CachingFileManager.__del__`
- **Effect**: Controls whether a `RuntimeWarning` is emitted when a `CachingFileManager` is deallocated but its file is still open. Useful for debugging file handle leaks.
- **Default value**: `False` (defined in `xarray/core/options.py` line 68)
- **Validation**: Must be a boolean (`True` or `False`)
- **Usage**:
  - In `CachingFileManager.__del__`, after decrementing the reference count, if `ref_count == 0` and the file is still in the cache, the manager closes the file
  - If `OPTIONS["warn_for_unclosed_files"]` is `True`, a warning is issued: `"deallocating {}, but file is not already closed. This may indicate a bug."`
- **Test evidence**: `xarray/tests/test_backends_file_manager.py::test_file_manager_autoclose` parameterizes on `[True, False]` and verifies that the warning is emitted only when enabled. Additionally, `xarray/tests/__init__.py` enables this option globally for test runs via `set_options(warn_for_unclosed_files=True)` on line 94.

## Configuration-driven file management summary

The file management subsystem in xarray is driven by two configuration options:
1. **Resource management**: `file_cache_maxsize` limits the global file cache size, controlling memory and file descriptor usage.
2. **Debugging**: `warn_for_unclosed_files` enables warnings for unclosed files during deallocation, aiding in detecting leaks.

## Source references

- `xarray/backends/file_manager.py:16` – `FILE_CACHE` initialization using `file_cache_maxsize`
- `xarray/backends/file_manager.py:250` – `__del__` check using `warn_for_unclosed_files`
- `xarray/backends/file_manager.py:18` – Assertion ensuring `FILE_CACHE.maxsize >= 1` at module load time
- `xarray/backends/file_manager.py:10` – Import of `OPTIONS` from `xarray.core.options`
- `xarray/core/options.py:53-70` – `OPTIONS` dictionary with default values and validators
- `xarray/core/options.py:114-117` – `_SETTERS` mapping `file_cache_maxsize` to `_set_file_cache_maxsize`
- `xarray/core/options.py:77-78` – `_positive_integer` validator function
- `xarray/core/options.py:100-103` – `_set_file_cache_maxsize` function implementation
- `xarray/tests/test_options.py:41-47` – Tests for `file_cache_maxsize`
- `xarray/tests/test_backends_file_manager.py:42-60` – Tests for `warn_for_unclosed_files`
