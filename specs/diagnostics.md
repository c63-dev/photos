# Diagnostics Specification

## Purpose
Defines the `tests` action, a human-readable health check page. Requires diagnostics to be allowed.

## Requirements

### Requirement: Health Report
The system SHALL return a page reporting the application version, server details, whether the root, storage directory and application file exist and are writeable, which optional capabilities are available (image resizing, image conversion, PDF rendering, video frame extraction, archive support, metadata extraction, mime detection, sessions), and relevant server limits (memory, upload sizes).

#### Scenario: Missing capability
- **WHEN** the video frame extractor is not available
- **THEN** the report marks it as failing

### Requirement: Bootstrap Report
The system SHALL perform the storage bootstrap when the diagnostics page is requested and SHALL list any storage items it created.

#### Scenario: Diagnostics on first run
- **WHEN** diagnostics are requested before the storage directory exists
- **THEN** the report lists the storage directory and files it just created

### Requirement: Masked Configuration
The system SHALL show the effective configuration on the page with sensitive values (root, storage path, start path, credentials) masked.

#### Scenario: Password masked
- **WHEN** a default password is configured
- **THEN** the configuration display shows `***` in its place

### Requirement: Login Behaviour
The system SHALL show the login form, rather than an error, when diagnostics are requested without a session and login is required.

#### Scenario: Unauthenticated diagnostics
- **WHEN** login is required and diagnostics are requested anonymously
- **THEN** the login form is shown
