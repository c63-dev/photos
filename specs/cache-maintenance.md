# Cache Maintenance Specification

## Purpose
Defines when and how the server removes stale entries from its menu, listing and image caches.

## Requirements

### Requirement: Scheduled Cleaning
The background worker (see `thumbnail-generation.md`) SHALL run cache cleaning on the configured interval in days, without any visitor involvement, and SHALL record when it last ran.

#### Scenario: Interval elapsed with no visitors
- **WHEN** the interval has elapsed and nobody has opened the gallery
- **THEN** the worker cleans the caches anyway

### Requirement: Manual Cleaning
The system SHALL allow `clean_cache` to be requested directly only when manual cleaning is enabled.

#### Scenario: Manual cleaning disabled
- **WHEN** a visitor requests `clean_cache` and manual cleaning is off
- **THEN** a forbidden error is returned

### Requirement: Dry Run
The system SHALL support a test flag that reports what would be deleted without deleting anything.

#### Scenario: Test run
- **WHEN** `clean_cache` is requested with the test flag
- **THEN** the report is prefixed as a test and no files are removed

### Requirement: Menu Cache Cleaning
The system SHALL remove menu cache entries that do not belong to the current configuration and SHALL keep only the most recent entry.

#### Scenario: Orphaned menu cache
- **WHEN** a menu cache entry was written for a root the configuration no longer uses
- **THEN** it is deleted

### Requirement: Listing Cache Cleaning
The system SHALL remove listing cache entries whose configuration is unknown, whose directory no longer exists, or whose recorded modified time no longer matches the directory, keeping at most one valid entry per directory.

#### Scenario: Directory deleted
- **WHEN** a cached directory has been deleted from disk
- **THEN** its listing cache entries are removed

### Requirement: Image Cache Cleaning
The system SHALL remove generated images whose name is malformed, whose size is no longer an allowed size, that have not been accessed within the configured number of days, or whose source file is missing or has changed. Folder previews are valid while newer than their directory.

#### Scenario: Unused thumbnail
- **WHEN** a thumbnail has not been accessed for longer than the configured limit
- **THEN** it is deleted

#### Scenario: Source changed
- **WHEN** a source image's size or modified time differs from what the thumbnail was built from
- **THEN** the thumbnail is deleted

### Requirement: In-Flight Protection
Cleaning SHALL NOT remove a generated image that the background worker is currently producing.

#### Scenario: Cleaning during generation
- **WHEN** cleaning runs while a thumbnail is being generated
- **THEN** that thumbnail is left untouched

### Requirement: Cleaning Report
The system SHALL respond with the number of entries deleted out of the total for each cache type.

#### Scenario: Report
- **WHEN** cleaning completes
- **THEN** the response lists "Deleted N of M" for menu, listing and image caches
