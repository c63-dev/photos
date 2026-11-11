# Settings Management Specification

## Purpose
Defines the `settings` action, through which the UI reads and writes the stored configuration. Requires settings to be allowed.

## Requirements

### Requirement: Load Settings
The system SHALL, on a load request, return the stored configuration text.

#### Scenario: Load
- **WHEN** settings are loaded
- **THEN** the response contains the current stored configuration text

### Requirement: Save Settings
The system SHALL, on a save request, validate and store the supplied configuration text as the stored configuration and return the stored text.

#### Scenario: Save
- **WHEN** valid configuration text is saved
- **THEN** it becomes the stored configuration and is returned

### Requirement: Configuration Validation
The system SHALL reject configuration text that is not a valid configuration document, and SHALL reject a root value that does not resolve to a valid root.

#### Scenario: Invalid root
- **WHEN** the saved configuration names a root that does not exist
- **THEN** an "Invalid root dir" error is returned and nothing is stored

### Requirement: Password Protection
The system SHALL store the password in a protected form rather than as entered, unless the author explicitly opts out of protection.

#### Scenario: Plain password entered
- **WHEN** a configuration with a plain password is saved
- **THEN** the stored text contains a protected form of the password, not the original
