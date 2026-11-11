# Map Specification

## Purpose
Defines location-based browsing from the coordinates held in the metadata index.

## Requirements

### Requirement: Clustered Points
The system SHALL, on a `map` request with a bounding box and zoom level, return clusters of located items for that area, each cluster carrying its centre, item count and a representative item. At high zoom individual items SHALL be returned.

#### Scenario: Country view
- **WHEN** a wide bounding box is requested at low zoom
- **THEN** items are grouped into a small number of clusters with counts

#### Scenario: Street view
- **WHEN** a small bounding box is requested at high zoom
- **THEN** individual items with their coordinates are returned

### Requirement: Near A Point
The system SHALL, on a `map` request with a point and a radius, return items within that radius, nearest first, paginated.

#### Scenario: Photos near here
- **WHEN** a point and a 500 metre radius are requested
- **THEN** items located within 500 metres are returned nearest first

### Requirement: Filters
Map requests SHALL accept the same date range and type filters as the timeline.

#### Scenario: Located videos in a year
- **WHEN** a bounding box is requested with a 2022 date range and the video filter
- **THEN** only videos captured in 2022 in that area are returned
