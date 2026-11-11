# Timeline Specification

## Purpose
Defines date-based browsing across the whole library, independent of folder layout, answered from the metadata index.

## Requirements

### Requirement: Date Summary
The system SHALL, on a `timeline` request with no range, return counts of items per year, per month and per day so the UI can draw a date picker or scrubber.

#### Scenario: Summary
- **WHEN** the timeline summary is requested
- **THEN** the response lists each year, month and day that has items, with the item count for each

### Requirement: Items By Date Range
The system SHALL, on a `timeline` request with a start and end date, return items whose capture date falls in that range, newest first, grouped by day, with the same item data as a directory listing. Results SHALL be paginated.

#### Scenario: One month
- **WHEN** July 2023 is requested
- **THEN** every item captured in July 2023 is returned grouped by day, regardless of folder

#### Scenario: Large range
- **WHEN** a range containing 5,000 items is requested
- **THEN** the first page is returned with a cursor for the next

### Requirement: Type Filter
Timeline requests SHALL accept an optional type filter (images, videos, or both).

#### Scenario: Videos only
- **WHEN** a range is requested with the video filter
- **THEN** only videos are returned
