# Duplicate Detection Specification

## Purpose
Defines how the system finds duplicate media across the library so the owner can remove them. Detection never deletes anything by itself.

## Requirements

### Requirement: Exact Duplicates
The system SHALL treat files with identical content fingerprints as exact duplicates, regardless of name or folder.

#### Scenario: Same photo in two folders
- **WHEN** the same photo exists as `IMG_1.jpg` in two folders
- **THEN** both are reported in one exact-duplicate group

### Requirement: Likely Duplicates
The system SHALL treat images with the same capture date and time, the same dimensions and the same camera as likely duplicates, and SHALL report them separately from exact duplicates.

#### Scenario: Re-exported photo
- **WHEN** two JPEGs have identical capture time, dimensions and camera but different content
- **THEN** they are reported in one likely-duplicate group

### Requirement: Duplicate Report
The system SHALL, on a `duplicates` request, return groups of duplicates with each member's item data and a suggested keeper (the oldest file modified time, then the shortest path). Results SHALL be paginated.

#### Scenario: Report
- **WHEN** duplicates are requested
- **THEN** each group lists its members and marks one as the suggested keeper

### Requirement: Resolution
Removing duplicates SHALL be done by the owner through the ordinary delete action; the report SHALL refresh after deletion.

#### Scenario: Delete a duplicate
- **WHEN** one member of a group is deleted
- **THEN** the next report no longer shows that group if only one member remains
