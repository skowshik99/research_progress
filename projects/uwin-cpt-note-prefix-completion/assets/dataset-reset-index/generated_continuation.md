### MultiIndex Level Handling

When resetting a MultiIndex by its dimension name:
- The MultiIndex is removed from `_indexes`
- Level coordinates are kept as regular variables
- The dimension is preserved

When resetting by level names (e.g., `reset_index("num")`):
- The level coordinate is removed from the dataset
- The MultiIndex is converted to a regular index
- The dimension is preserved

**Evidence from probe:**
```
After reset_index("num"):
Indexes: ['idx', 'letter']
Coordinates: ['idx', 'letter']
```

### drop Parameter Behavior

The `drop` parameter controls whether the dimension is removed:
- `drop=False` (default): Dimension is preserved, index is removed
- `drop=True`: Dimension is removed, index and its coordinates are dropped

**Evidence from probe:**
```
After reset_index("idx", drop=True):
Dimensions:  ()
Coordinates: []
```

### Related Methods

- `set_index()` at line 4197: Creates indexes from coordinates
- `clear_index()` at line 4257: Removes indexes and their coordinates
- `rename()` at line 3354: Renames indexes and coordinates

### Test Coverage

The following tests verify the behavior:
- `test_reset_index` at line 4287: Basic reset_index functionality
- `test_reset_index_drop` at line 4300: drop parameter behavior
- `test_reset_index_multiindex` at line 4313: MultiIndex handling
- `test_reset_index_multiindex_level` at line 4330: Level-based reset
- `test_reset_index_errors` at line 4343: Error handling for invalid names

All 5 tests pass successfully.

### Verified Behavior Summary

| Operation | Indexes After | Coordinates After | Dimensions After |
|-----------|---------------|-------------------|------------------|
| `reset_index("idx")` | `[]` | `['idx', 'num', 'letter']` | `('idx',)` |
| `reset_index("idx", drop=True)` | `[]` | `[]` | `()` |
| `reset_index("num")` | `['idx', 'letter']` | `['idx', 'letter']` | `('idx',)` |
| `reset_index("letter")` | `['idx', 'num']` | `['idx', 'num']` | `('idx',)` |
| `reset_index(["num", "letter"])` | `['idx']` | `['idx']` | `('idx',)` |
| `reset_index(["num", "letter"], drop=True)` | `[]` | `[]` | `()` |

### Correction to Original Claims

The original repository information contained incorrect line numbers:
- `reset_index` method: claimed line 4100, actual line 4108
- `set_index` method: claimed line 4190, actual line 4197
- `clear_index` method: claimed line 4250, actual line 4257
- `test_reset_index` test: claimed line 4280, actual line 4287
- `test_reset_index_drop` test: claimed line 4293, actual line 4300
- `test_reset_index_multiindex` test: claimed line 4306, actual line 4313
- `test_reset_index_multiindex_level` test: claimed line 4323, actual line 4330
- `test_reset_index_errors` test: claimed line 4336, actual line 4343
