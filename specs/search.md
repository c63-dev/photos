# Search Specification

## Purpose
Defines library-wide search across all folders, answered from the metadata index.

## Requirements

### Requirement: Search Criteria
The system SHALL, on a `search` request, accept any combination of: free text matched against file name, folder path, descriptive text and keywords; a capture date range; camera make or model; media type; a location and radius; and a folder to restrict the search to. Criteria SHALL combine with AND.

#### Scenario: Text and date
- **WHEN** a search asks for `beach` within 2023
- **THEN** only items captured in 2023 whose name, path, description or keywords contain `beach` are returned

#### Scenario: Camera
- **WHEN** a search asks for a camera model
- **THEN** only items taken with that model are returned

### Requirement: Results
Results SHALL carry the same item data as a directory listing, SHALL be ordered newest first by default with an option to order by name or relevance, and SHALL be paginated.

#### Scenario: Paged results
- **WHEN** a search matches 2,000 items
- **THEN** the first page is returned with a cursor for the next

### Requirement: Hidden Items Excluded
Search SHALL never return hidden items.

#### Scenario: Reserved item
- **WHEN** a search term matches a file in a `_files` directory
- **THEN** that file is not returned
