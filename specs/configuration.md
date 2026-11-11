# Configuration Specification

## Purpose
Defines how the server assembles its configuration, where it stores it, and which guard rails apply.

## Requirements

### Requirement: Layered Configuration
The system SHALL build its effective configuration by layering, from lowest to highest precedence: built-in defaults, the stored configuration in the storage directory, and a local override file next to the application.

#### Scenario: Local override wins over stored config
- **WHEN** an option is set both in the stored configuration and in the local override file
- **THEN** the value from the local override file is used

### Requirement: Storage Bootstrap
The system SHALL, when serving the main document or the diagnostics page, create the storage directory and its configuration and cache areas if they do not exist, and SHALL generate a stored configuration file listing every option, with only options that differ from the defaults active.

#### Scenario: First run
- **WHEN** the main document is requested and no storage directory exists
- **THEN** the storage directory, its configuration and cache areas, and a configuration file are created

#### Scenario: Application updated
- **WHEN** the application has been updated since the stored configuration was written
- **THEN** the configuration file is regenerated, preserving values that differ from the defaults

#### Scenario: Action request does not bootstrap
- **WHEN** an action request arrives and the storage directory does not exist
- **THEN** no directories or files are created

### Requirement: Storage Can Be Disabled
The system SHALL allow storage to be disabled entirely, in which case all caching and folder preview images are disabled.

#### Scenario: Storage disabled
- **WHEN** the storage location is explicitly set to off
- **THEN** directory caching, image caching and folder previews are all disabled

### Requirement: Root Validation
The system MUST resolve the configured root to an existing, readable directory that is not the storage directory, and MUST refuse to serve any request otherwise.

#### Scenario: Root does not exist
- **WHEN** the configured root does not resolve to a directory
- **THEN** an error naming the invalid root is returned

### Requirement: Allow-All Shortcut
The system SHALL treat the allow-all option as enabling every file action (upload, delete, rename, new folder, new file, duplicate, text edit, zip, unzip, move, copy, download) without enabling settings, diagnostics or tasks.

#### Scenario: Allow-all enabled
- **WHEN** allow-all is on and no individual file action options are set
- **THEN** all file actions are permitted and settings remain disabled

### Requirement: List Options
The system SHALL accept comma-separated strings for list-type options and SHALL trim whitespace around each entry and around every string option value.

#### Scenario: Comma list with spaces
- **WHEN** an option is set to `"jpg, png , gif"`
- **THEN** it is interpreted as the three values `jpg`, `png` and `gif`
