# File Delivery Specification

## Purpose
Defines how the server serves, proxies and downloads files and archives.

## Requirements

### Requirement: Serve Through Application
The system SHALL, on a `file` request with no processing flag, serve the file's contents through the application. This SHALL be permitted only when proxying is enabled or when the file has no direct URL. The response SHALL carry the file's mime type (or a generic binary type when unknown) and be cacheable.

#### Scenario: Proxying disabled for public file
- **WHEN** proxying is off and the requested file is directly reachable by URL
- **THEN** a "File can't be proxied" error is returned

#### Scenario: Private root
- **WHEN** the root is not reachable by URL
- **THEN** files are served through the application regardless of the proxy setting

### Requirement: Partial Content
The system SHALL honour single-range partial content requests when serving files, so that large media can be seeked. Multi-range requests SHALL be rejected.

#### Scenario: Video seek
- **WHEN** the browser requests a byte range of a video served through the application
- **THEN** only that range is returned with a partial-content status

### Requirement: Force Download
The system SHALL, on a `download` request, serve the file as an attachment named after the file, when downloads are allowed.

#### Scenario: Download a file
- **WHEN** a `download` request names a readable file
- **THEN** the file is returned as an attachment with its own name

### Requirement: Load Text File
The system SHALL, on a `load_text_file` request, return the file's contents as plain UTF-8 text.

#### Scenario: Open a text file for editing
- **WHEN** the application requests a text file's contents
- **THEN** the raw contents are returned as plain text

### Requirement: Downloadable File Discovery
The system SHALL, on a `get_downloadables` request listing directories, return every readable, non-hidden file beneath those directories, each with its relative path, URL path, name, extension and size. Downloads MUST be allowed.

#### Scenario: Nested files
- **WHEN** the request lists a directory with files in subdirectories
- **THEN** all files at every depth are returned, directories themselves are not

### Requirement: Directory Zip Download
The system SHALL, on a `download_dir_zip` request, build a zip archive of the directory on demand, return it as an attachment named after the directory, and delete the archive afterwards, even if the download is cancelled. This requires the directory download mode to be set to zip, downloads to be allowed, and archive support to be available.

#### Scenario: Download a folder
- **WHEN** a directory zip is requested and all conditions are met
- **THEN** a zip named `<directory>.zip` containing the directory's visible contents is returned

#### Scenario: Zip mode not enabled
- **WHEN** the directory download mode is not zip
- **THEN** a "download_dir zip disabled" error is returned

