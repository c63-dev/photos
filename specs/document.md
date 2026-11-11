# Main Document Specification

## Purpose
Defines what the server delivers when the gallery itself is requested: the starting location, preloaded data, configuration handed to the browser, localisation and customisation hooks.

## Requirements

### Requirement: Main Document
The system SHALL return the application shell when a request has no `action`, with the root directory's data embedded so the browser can render without a further request.

#### Scenario: Open the gallery
- **WHEN** the gallery URL is opened with no parameters
- **THEN** the shell is returned with the root directory preloaded

### Requirement: Start Location From URL
The system SHALL treat the first bare query segment (one with no `=`) as the directory to open. If that directory is valid it SHALL be preloaded; if not, the browser SHALL be told the requested path is invalid.

#### Scenario: Deep link
- **WHEN** the gallery is opened as `?2024/summer`
- **THEN** the shell preloads `2024/summer` and opens there

#### Scenario: Invalid deep link
- **WHEN** the query path does not exist
- **THEN** the shell is returned with a flag marking the path invalid

### Requirement: Configured Start Location
The system SHALL, when no URL path is given and a start path is configured, open there. The configured start path MUST exist, lie inside the root and not be hidden, or the request fails with an error.

#### Scenario: Bad configured start path
- **WHEN** the configured start path is hidden or outside the root
- **THEN** an "Invalid start_path" error is returned

### Requirement: Browser Configuration
The system SHALL embed the effective configuration in the document, omitting sensitive or server-only options (root, storage location, credentials, cache internals, external tool locations), and SHALL add computed values including: whether a menu exists, cache keys, the start path, preloaded directories, which image formats can be resized or converted, whether video, converter, PDF and archive support are available, the upload size limit, the resize memory limit, the version, login state, the session token, the assets location, whether it is time to clean the cache, and a summary of background generation progress.

#### Scenario: Secrets not exposed
- **WHEN** the document is delivered
- **THEN** the embedded configuration contains no password, root path or storage path

### Requirement: Internationalisation
The system SHALL support a configurable default language and automatic selection from the browser's preferred languages, and SHALL make custom language packs placed in the storage directory available to the browser. The login page SHALL be localised the same way.

#### Scenario: Custom language pack
- **WHEN** a language file is placed in the storage directory's language folder
- **THEN** the document exposes it under its language code

### Requirement: Custom Stylesheet And Script
The system SHALL include a custom stylesheet and a custom script when they exist in the storage directory and the directory is publicly reachable. Links SHALL be cache-busted whenever the files change.

#### Scenario: Custom stylesheet
- **WHEN** a custom stylesheet exists in the storage directory
- **THEN** the document links to it

### Requirement: Assets Location
The system SHALL load frontend scripts, styles and language packs from a public CDN pinned to the current version, or from a configured self-hosted location.

#### Scenario: Self-hosted assets
- **WHEN** an assets location is configured
- **THEN** all frontend resources are referenced from that location
