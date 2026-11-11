# Albums And Favourites Specification

## Purpose
Defines favourites and albums: ways to mark and group items without moving files on disk. Membership is stored in the storage directory and keyed by item path.

## Requirements

### Requirement: Favourites
The system SHALL maintain a single favourites set. A `favourite` request SHALL mark or unmark an item. Listings, timeline and search results SHALL indicate whether each item is a favourite, and favourites SHALL be retrievable as a list like any album.

#### Scenario: Mark favourite
- **WHEN** an item is marked as favourite
- **THEN** it appears in the favourites list and its listing entry shows the flag

### Requirement: Album Management
The system SHALL support named albums through an `albums` request that lists, creates, renames and deletes albums. Album names follow the item name rules. Deleting an album SHALL NOT touch any files.

#### Scenario: Create and delete
- **WHEN** an album is created and later deleted
- **THEN** the album is gone and every file it contained is unchanged on disk

### Requirement: Album Membership
The system SHALL support an `album_items` request to add items to an album, remove them, reorder them, set a cover item and list the album's contents with the same item data as a directory listing. An item MAY belong to many albums.

#### Scenario: Add from two folders
- **WHEN** photos from two different folders are added to one album
- **THEN** the album lists both, in the order given

### Requirement: Follows Files
Favourite flags and album memberships SHALL follow items through rename and move, and SHALL be removed when the item is deleted.

#### Scenario: Renamed member
- **WHEN** a photo in an album is renamed
- **THEN** the album still contains it under the new name

#### Scenario: Deleted member
- **WHEN** a photo in an album is deleted
- **THEN** it is removed from the album

### Requirement: Download An Album
The system SHALL allow an album to be downloaded as a zip, subject to the download permission.

#### Scenario: Album zip
- **WHEN** an album download is requested
- **THEN** a zip named after the album containing its items is returned
