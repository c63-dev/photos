# Menu Specification

## Purpose
Defines the `dirs` action, which returns the directory tree used for the sidebar menu, and how that tree is cached.

## Requirements

### Requirement: Directory Tree
The system SHALL return every visible directory beneath the root, recursively, each with the same directory data as a listing. Excluded directories SHALL be skipped along with their descendants.

#### Scenario: Nested folders
- **WHEN** the root contains `2024/jan` and `2024/feb`
- **THEN** the response includes `2024`, `2024/jan` and `2024/feb`

### Requirement: Depth Limit
The system SHALL stop descending once the configured maximum depth is reached.

#### Scenario: Depth exceeded
- **WHEN** the maximum depth is 2 and a directory exists at depth 3
- **THEN** the depth-3 directory is not included

### Requirement: Symlinked Directories
The system SHALL include symlinked directories but SHALL NOT descend into them when recursive symlink following is disabled.

#### Scenario: Symlink recursion off
- **WHEN** recursive symlink following is off and a symlinked directory has subdirectories
- **THEN** the symlinked directory is listed but its subdirectories are not

### Requirement: Menu Sorting
The system SHALL sort sibling directories by name or by modified date, ascending or descending, as configured.

#### Scenario: Newest first
- **WHEN** the menu sort is date descending
- **THEN** the most recently modified sibling appears first

### Requirement: Full Preload
The system SHALL, when full menu loading is enabled, include each directory's file listing in the tree response.

#### Scenario: Full load enabled
- **WHEN** full menu loading is on
- **THEN** every directory in the tree carries its `files` collection

### Requirement: Menu Cache
The system SHALL cache the tree under a key supplied by the browser that encodes the root, the relevant options and the latest modified time of the root and its immediate subdirectories. A malformed key SHALL be rejected. When the browser reports it already holds the tree for that key, the system SHALL confirm validity instead of resending the tree.

#### Scenario: Browser already has the tree
- **WHEN** the browser posts a valid key with a flag saying the tree is in local storage, and the cache is valid
- **THEN** the response is a confirmation rather than the tree

#### Scenario: Malformed key
- **WHEN** the posted key does not have the expected shape
- **THEN** an "Invalid menu cache hash" error is returned

### Requirement: Deep Cache Validation
The system SHALL, when deep validation is enabled, treat a cached tree as valid only if the modified time of every directory below the first level still matches. When deep validation is disabled, the system SHALL expose a direct URL to the cached tree in the main document so the browser can load it without a request.

#### Scenario: Deep folder changed
- **WHEN** deep validation is on and a second-level directory has been modified since the cache was written
- **THEN** the tree is regenerated

#### Scenario: Validation disabled
- **WHEN** deep validation is off, caching is on and the storage directory is publicly reachable
- **THEN** the main document carries a direct URL to the cached tree

### Requirement: Stale Menu Cleanup
The system SHALL remove superseded cache entries for the same root and options when regenerating the tree.

#### Scenario: Regeneration
- **WHEN** the tree is regenerated for a root that has older cache entries
- **THEN** the older entries are deleted
