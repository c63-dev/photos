# Path Security Specification

## Purpose
Defines how relative paths from the browser are resolved and validated, which items are hidden, and how public URLs for files are derived.

## Requirements

### Requirement: Relative Path Resolution
The system SHALL interpret every path received from the browser as relative to the root. A path MUST NOT start or end with a slash, MUST resolve to an existing item of the expected kind (file or directory), MUST be readable, and MUST NOT be hidden.

#### Scenario: Leading slash
- **WHEN** a request path begins with `/`
- **THEN** the path is rejected as invalid

#### Scenario: File where directory expected
- **WHEN** a `dir` parameter resolves to a file
- **THEN** the path is rejected as invalid

### Requirement: Traversal Protection
The system MUST reject a path that resolves outside the root when the path contains parent-directory segments or a drive-style colon in its directory part. Symbolic links inside the root that point outside it remain allowed when symlinks are permitted.

#### Scenario: Parent traversal
- **WHEN** a request path contains `../` and resolves outside the root
- **THEN** the path is rejected

#### Scenario: Symlink outside root
- **WHEN** a directory inside the root is a symlink to a location outside the root and symlinks are allowed
- **THEN** the path is accepted

### Requirement: Hidden Items
The system SHALL hide from all listings and reject in all requests: any item whose relative path contains a segment starting with `_files`, the application itself, the local configuration override file, and anything inside the storage directory.

#### Scenario: Reserved prefix
- **WHEN** a directory contains a file named `_files_notes.txt`
- **THEN** it does not appear in the listing and cannot be requested

### Requirement: Symlink Policy
The system SHALL hide symlinked items when symlinks are not allowed.

#### Scenario: Symlinks disabled
- **WHEN** symlinks are not allowed and a directory contains a symlinked file
- **THEN** the symlinked file is omitted from the listing

### Requirement: Public URL Resolution
The system SHALL determine whether files in the root are directly reachable by URL and, if so, SHALL expose a URL path for each item. A custom root URL path may be configured. When the root is reachable only through the application (for example it lies outside the web-served area), items carry no URL path and must be served through the application instead.

#### Scenario: Root inside web area
- **WHEN** the root lies inside the area the web server publishes
- **THEN** each listed item includes a URL path that the browser can load directly

#### Scenario: Root outside web area
- **WHEN** the root lies outside the published area and no custom root URL path is configured
- **THEN** listed items carry no URL path and the browser loads them through the application

#### Scenario: Symlinked root resolving outside
- **WHEN** the configured root is a relative symlink inside the application directory that resolves outside the web area
- **THEN** the system derives a URL path from the symlink's own location
