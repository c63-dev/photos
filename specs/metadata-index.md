# Metadata Index Specification

## Purpose
Defines the persistent index of every item in the library, maintained by the background worker, from which timeline, search, albums, duplicates and map queries are answered without reading the disk.

## Requirements

### Requirement: Indexed Fields
The index SHALL hold, for every visible file beneath the root: relative path, name, extension, type (image, video, PDF, other), size, file modified time, dimensions, duration for video, capture date, camera make and model, location coordinates, descriptive text and keywords from embedded metadata, a content fingerprint, the companion group it belongs to (see `directory-listing.md`), and the state of its thumbnails, viewing variants and transcode.

#### Scenario: Photo indexed
- **WHEN** a photo with embedded capture date and location is discovered
- **THEN** its index entry carries that date, those coordinates and its fingerprint

### Requirement: Capture Date
The capture date SHALL be taken from embedded metadata when present, otherwise from a date in the file name when recognisable, otherwise from the file modified time. The index SHALL record which source was used.

#### Scenario: Scanned image without metadata
- **WHEN** a file has no embedded date and its name is `2019-06-14 beach.jpg`
- **THEN** its capture date is 14 June 2019, marked as taken from the name

### Requirement: Built By The Worker
The background worker SHALL build the index on first run and keep it current through its discovery passes. Indexing an item SHALL happen before its thumbnails are generated, so browsing signals move indexing to the front as well.

#### Scenario: Initial build
- **WHEN** the worker first runs over a library of 8,000 files
- **THEN** every file is indexed and the status reports indexing progress

### Requirement: Kept In Sync With File Operations
Every file-management action (upload, delete, rename, move, copy, duplicate, unzip, text edit) and every transcode replacement SHALL update the index immediately, so index queries never return items that no longer exist at the recorded path.

#### Scenario: Move
- **WHEN** a photo is moved to another folder
- **THEN** its index entry, album memberships and favourite flag follow it to the new path

### Requirement: Rebuild
The system SHALL allow a full index rebuild to be triggered through the generation control action. Rebuilding SHALL NOT lose albums or favourites.

#### Scenario: Rebuild
- **WHEN** a rebuild is requested
- **THEN** the index is reconstructed from disk and existing album memberships are reattached by path

### Requirement: Status
Index size, last update time and rebuild progress SHALL be included in the generation status.

#### Scenario: Status
- **WHEN** status is requested
- **THEN** it reports how many items are indexed and when the index was last updated
