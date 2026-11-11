# Directory Listing Specification

## Purpose
Defines the `files` action: what the server returns for a single directory, how file and image metadata is gathered, and how listings are cached.

## Requirements

### Requirement: Directory Data
The system SHALL return, for a requested directory, its name, permissions, readability, writeability, whether it is a symlink, modified time, relative path, URL path, file count, total size and image count.

#### Scenario: Listing a directory
- **WHEN** a `files` request names a readable directory
- **THEN** the response contains the directory's data and a `files` collection

### Requirement: File Data
The system SHALL return, for each visible item in the directory, its name, extension, permissions, type, size, readability, writeability, whether it is a symlink, whether it is a directory, modified time, relative path and URL path. Items that no longer resolve (for example dead symlinks) SHALL be omitted.

#### Scenario: Dead symlink
- **WHEN** a directory contains a symlink whose target no longer exists
- **THEN** the symlink is omitted from the listing

### Requirement: Mime Type
The system SHALL leave the mime type empty by default so the browser infers it from the extension, and SHALL detect it on the server only for files with no extension, for files with an ambiguous extension, or when server-side mime detection is enabled.

#### Scenario: Extensionless file
- **WHEN** a file has no extension
- **THEN** its mime type is detected on the server and included

### Requirement: Image Metadata
The system SHALL, for files that appear to be images, record the width, height, bit depth, channel count and mime type. Formats the server cannot read natively SHALL fall back to the image converter when available. Vector images SHALL be marked as images without dimensions.

#### Scenario: Readable raster image
- **WHEN** a directory contains a readable photo
- **THEN** the item includes its width and height and increments the directory's image count

#### Scenario: HEIC without converter
- **WHEN** a directory contains a HEIC image and no converter is available
- **THEN** the item is marked as an image but has no dimensions

### Requirement: Embedded Metadata
The system SHALL extract a fixed set of descriptive fields from images that carry them (title, headline, description, creator, credit, copyright, keywords, city, sub-location, province/state) and a fixed set of camera fields (aperture, original date and time, exposure time, focal length, camera model, orientation, ISO) plus decimal GPS coordinates. Text values SHALL be clamped to a reasonable length and repaired to valid UTF-8 where possible. Extraction SHALL be skipped when the image is too large to process safely.

#### Scenario: Photo with GPS
- **WHEN** an image carries latitude and longitude with hemisphere references
- **THEN** the item includes signed decimal latitude and longitude

### Requirement: Orientation-Corrected Dimensions
The system SHALL swap the reported width and height when an image's orientation metadata indicates it is displayed rotated by 90 degrees.

#### Scenario: Portrait photo stored landscape
- **WHEN** an image is stored as 4000x3000 with an orientation that rotates it 90 degrees
- **THEN** the listing reports it as 3000x4000

### Requirement: Companion Grouping
The system SHALL present files that belong together as one item with companions rather than as separate entries. Recognised groups are: a RAW file and a developed image with the same base name; a still image and a short video with the same base name (a live photo); and sidecar files (`.xmp`, `.aae` and similar) attached to an image of the same base name. The primary item SHALL be the developed or browser-viewable image; companions SHALL be listed on it with their type and size, and SHALL be counted in the directory size but not the file count.

#### Scenario: RAW plus JPEG
- **WHEN** a directory contains `DSC_1.NEF` and `DSC_1.JPG`
- **THEN** the listing shows one item `DSC_1.JPG` with `DSC_1.NEF` as a RAW companion

#### Scenario: Live photo
- **WHEN** a directory contains `IMG_2.jpg` and a three-second `IMG_2.mov`
- **THEN** the listing shows one item `IMG_2.jpg` with the video as a live-photo companion

### Requirement: Transcode State
The system SHALL mark items whose transcode is queued, in progress or failed (see `media-transcoding.md`).

#### Scenario: Pending item
- **WHEN** a listed video is awaiting transcoding
- **THEN** its entry carries a pending transcode state

### Requirement: Variant And Favourite Flags
Each item SHALL indicate which viewing variants exist (see `viewing-variants.md`) and whether it is a favourite (see `albums.md`).

#### Scenario: Flags present
- **WHEN** a favourited photo with a viewing variant is listed
- **THEN** its entry shows the favourite flag and the available variant

### Requirement: Shortcut Files
The system SHALL read `.url` shortcut files and expose the target address. When the target is a web video, the item SHALL be presented as a playable video whose source is the remote address.

#### Scenario: Shortcut to a video
- **WHEN** a `.url` file points to an `.mp4` address
- **THEN** the item is listed as a video with the remote address as its source

### Requirement: Custom Preview Images
The system SHALL use a sibling image named `_files_<filename>.<image extension>` as the preview for `<filename>` when the directory is publicly reachable and image loading is enabled.

#### Scenario: Custom preview for a document
- **WHEN** a directory contains `report.docx` and `_files_report.docx.jpg`
- **THEN** the listing for `report.docx` includes the custom preview URL

### Requirement: Panorama Variants
The system SHALL, for publicly reachable images wider than 2048 pixels with an exact 2:1 aspect ratio, report any pre-resized variants named `_files_<width>_<filename>` at widths 2048, 4096 and 8192 that are smaller than the original.

#### Scenario: Panorama with variants
- **WHEN** a 16384x8192 panorama has `_files_4096_` and `_files_8192_` variants
- **THEN** the item lists the available variant widths, largest first

### Requirement: Direct Cache Links
The system SHALL, when direct cache linking is enabled, include URLs to already-cached resized versions of each image so the browser can load them without a server request.

#### Scenario: Cached thumbnail exists
- **WHEN** direct cache linking is on and a thumbnail for an image is already cached
- **THEN** the item includes a direct URL for that thumbnail size

### Requirement: Browsing Signal
A directory listing request SHALL signal the background generation worker to move unmade previews in that directory, and the folder previews of its immediate subdirectories, to the front of its queue (see `thumbnail-generation.md`). The listing response SHALL NOT wait for any generation.

#### Scenario: Listing a folder with unmade thumbnails
- **WHEN** a directory with no cached thumbnails is listed
- **THEN** the listing returns immediately and its items move to the front of the generation queue

### Requirement: Sorting
The system SHALL return items with directories first, then files, each group in natural case-insensitive name order.

#### Scenario: Mixed listing
- **WHEN** a directory contains `b.jpg`, `A.jpg`, `img10.jpg`, `img2.jpg` and a folder `zeta`
- **THEN** the order is `zeta`, `A.jpg`, `b.jpg`, `img2.jpg`, `img10.jpg`

### Requirement: Listing Cache
The system SHALL cache each directory listing keyed by the directory, its modified time and the configuration options that affect output, SHALL serve the cached listing when the key still matches, SHALL regenerate it otherwise, and SHALL discard superseded cache entries for the same directory.

#### Scenario: Directory changed
- **WHEN** a file is added to a directory that has a cached listing
- **THEN** the next request regenerates the listing and removes the old cache entry

#### Scenario: Direct cache access
- **WHEN** no login is configured, a listing is cached and the storage directory is publicly reachable
- **THEN** the directory data includes a direct URL to the cached listing

### Requirement: In-Directory Cache Cleanup
The system SHALL, when image caches are stored inside each directory, remove cached previews whose source file no longer exists or has been modified since.

#### Scenario: Source deleted
- **WHEN** in-directory caching is on and an image is deleted
- **THEN** its cached preview is removed on the next listing
