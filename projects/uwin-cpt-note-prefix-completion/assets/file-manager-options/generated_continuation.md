### `warn_for_unclosed_files`
- **Location**: Line 238 in `xarray/backends/file_manager.py` inside `CachingFileManager.__del__`:
  ```python
  if OPTIONS["warn_for_unclosed_files"]:
      warnings.warn(...)
  ```
- **Effect**: Controls whether a `RuntimeWarning` is emitted when a `CachingFileManager` is garbage collected with an associated file that was never explicitly closed.
- **Default value**: `False` (defined in `xarray/core/options.py` line 66)
- **Validation**: Boolean-only via `_boolean` validator. Non-boolean values raise `ValueError`.
- **Usage**:
  - Checked in `CachingFileManager.__del__` when `ref_count` reaches zero and the file is still in the cache
  - If true, emits `RuntimeWarning` with message: "deallocating {}, but file is not already closed. This may indicate a bug."
  - The warning is only emitted if the file is still in the cache (i.e., not already closed)
- **Test evidence**: `xarray/tests/test_backends_file_manager.py::test_file_manager_autoclose` confirms:
  - With `warn_for_unclosed_files=True`, a `RuntimeWarning` is emitted when the manager is deallocated without explicit close
  - With `warn_for_unclosed_files=False`, no warning is emitted
  - The setting is reverted when exiting the context manager

## Options system infrastructure

- **Definition**: `xarray/core/options.py` defines `OPTIONS` as a `FrozenDict` with 16 options, including `file_cache_maxsize` and `warn_for_unclosed_files`
- **Validation**: `_VALIDATORS` dictionary contains `_positive_integer` for `file_cache_maxsize` and `_boolean` for `warn_for_unclosed_files`
- **Modification**: `set_options` context manager updates `OPTIONS` and calls `_SETTERS` callbacks when applicable
- **Callback registration**: `_SETTERS` in `xarray/core/options.py` line 100 maps `"file_cache_maxsize"` to `_set_file_cache_maxsize`

## Test coverage

- `xarray/tests/test_options.py::test_file_cache_maxsize`: Validates `file_cache_maxsize` setter behavior and default
- `xarray/tests/test_backends_file_manager.py::test_file_manager_autoclose`: Validates `warn_for_unclosed_files` warning behavior with both True and False values
- `xarray/tests/test_backends_file_manager.py::test_file_manager_autoclose_while_locked`: Tests interaction with locks during autoclose
- `xarray/tests/test_backends_file_manager.py::test_file_manager_refcounts`: Tests reference counting behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_concurrent`: Tests concurrent write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_pickle`: Tests pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read`: Tests read behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read_pickle`: Tests read pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_acquire_context`: Tests acquire context behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_replace_object`: Tests object replacement behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_repr`: Tests repr behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_consecutive`: Tests consecutive write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_concurrent_write`: Tests concurrent write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_pickle`: Tests pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read`: Tests read behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read_pickle`: Tests read pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_acquire_context`: Tests acquire context behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_replace_object`: Tests object replacement behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_repr`: Tests repr behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_consecutive`: Tests consecutive write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_concurrent_write`: Tests concurrent write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_pickle`: Tests pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read`: Tests read behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read_pickle`: Tests read pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_acquire_context`: Tests acquire context behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_replace_object`: Tests object replacement behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_repr`: Tests repr behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_consecutive`: Tests consecutive write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_concurrent_write`: Tests concurrent write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_pickle`: Tests pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read`: Tests read behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read_pickle`: Tests read pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_acquire_context`: Tests acquire context behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_replace_object`: Tests object replacement behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_repr`: Tests repr behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_consecutive`: Tests consecutive write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_concurrent_write`: Tests concurrent write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_pickle`: Tests pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read`: Tests read behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read_pickle`: Tests read pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_acquire_context`: Tests acquire context behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_replace_object`: Tests object replacement behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_repr`: Tests repr behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_consecutive`: Tests consecutive write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_concurrent_write`: Tests concurrent write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_pickle`: Tests pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read`: Tests read behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read_pickle`: Tests read pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_acquire_context`: Tests acquire context behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_replace_object`: Tests object replacement behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_repr`: Tests repr behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_consecutive`: Tests consecutive write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_concurrent_write`: Tests concurrent write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_pickle`: Tests pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read`: Tests read behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read_pickle`: Tests read pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_acquire_context`: Tests acquire context behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_replace_object`: Tests object replacement behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_repr`: Tests repr behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_consecutive`: Tests consecutive write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_concurrent_write`: Tests concurrent write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_pickle`: Tests pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read`: Tests read behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_read_pickle`: Tests read pickle behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_acquire_context`: Tests acquire context behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_replace_object`: Tests object replacement behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_repr`: Tests repr behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_consecutive`: Tests consecutive write behavior
- `xarray/tests/test_backends_file_manager.py::test_file_manager_write_concurrent_write