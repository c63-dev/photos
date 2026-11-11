# Request Routing Specification

## Purpose
Defines how action requests are accepted, validated and answered.

## Requirements

### Requirement: Action Whitelist
The system SHALL only accept the following actions and SHALL reject any other value: `files`, `dirs`, `load_text_file`, `delete`, `text_edit`, `unzip`, `rename`, `new_file`, `new_folder`, `zip`, `copy`, `move`, `duplicate`, `get_downloadables`, `upload`, `download_dir_zip`, `preview`, `file`, `download`, `tasks`, `login`, `ping`, `settings`, `tests`, `clean_cache`, `generation_status`, `generation_control`, `timeline`, `search`, `favourite`, `albums`, `album_items`, `duplicates`, `map`.

#### Scenario: Unknown action
- **WHEN** a request names an action not in the list
- **THEN** an "Invalid action" error is returned

### Requirement: Request Method Enforcement
The system SHALL require GET for `download_dir_zip`, `preview`, `file`, `download`, `tasks`, `tests` and `clean_cache`, and SHALL require POST for every other action.

#### Scenario: Wrong method
- **WHEN** `delete` is sent as a GET request
- **THEN** an "Invalid request method" error is returned

### Requirement: POST Parameter Sources
The system SHALL read POST parameters from form fields when present, otherwise from a JSON request body.

#### Scenario: JSON body
- **WHEN** the application posts a JSON body with `dir` and `items`
- **THEN** those values are used as the request parameters

### Requirement: Permission Gating
The system SHALL reject an action when a configuration option named `allow_<action>` exists and is off. All download-related actions (`download`, `get_downloadables`, `download_dir_zip`) SHALL be governed by the single download permission.

#### Scenario: Upload disabled
- **WHEN** uploading is not allowed and an `upload` request arrives
- **THEN** an "upload not allowed" error is returned

#### Scenario: Downloads disabled
- **WHEN** downloads are not allowed and a directory zip download is requested
- **THEN** a "Download not allowed" error is returned

### Requirement: Directory Parameter
The system SHALL validate the `dir` parameter as an existing, non-hidden directory inside the root. The parameter is required for `files`, `copy`, `move`, `upload`, `download_dir_zip` and `preview`, and the directory MUST be writeable for `copy`, `move`, `unzip` and `upload`. An empty value means the root directory.

#### Scenario: Missing dir
- **WHEN** a `files` request has no `dir` parameter
- **THEN** a "Missing dir parameter" error is returned

#### Scenario: Read-only target
- **WHEN** a `move` request targets a directory that is not writeable
- **THEN** a "Dir is not writeable" error is returned

### Requirement: File Parameter
The system SHALL validate the `file` parameter as an existing, readable, non-hidden file inside the root. It is required for `load_text_file`, `file` and `download`.

#### Scenario: Invalid file
- **WHEN** the `file` parameter points outside the root
- **THEN** an "Invalid file path" error is returned

### Requirement: Items Parameter
The system SHALL accept an `items` list, each entry carrying a relative path and a directory flag, SHALL drop entries that fail path validation, and SHALL reject the request when none remain. The list is required for `copy`, `delete`, `duplicate`, `get_downloadables`, `move`, `new_file`, `new_folder`, `rename`, `text_edit`, `unzip` and `zip`. Only `new_file` and `new_folder` may target the root directory.

#### Scenario: Root as item
- **WHEN** a `delete` request lists the root directory as an item
- **THEN** a "Can't delete root directory" error is returned

### Requirement: Name Parameter
The system SHALL, when a `name` parameter is given, require it to be a valid item name, require the parent directory to exist and be writeable, and reject it if an item with that name already exists. For `new_file` and `new_folder` the parent is the selected item; for other actions it is the selected item's parent.

#### Scenario: Name already taken
- **WHEN** a `rename` request names a file that already exists in the same directory
- **THEN** a "File already exists" error is returned

### Requirement: Error Format
The system SHALL return errors as JSON `{"error": "<message>"}` for POST requests and as an HTML page with an appropriate HTTP status (400 by default) for GET requests.

#### Scenario: GET error
- **WHEN** a GET `file` request fails validation
- **THEN** an HTML error page with a 4xx status is returned

### Requirement: Diagnostic Response Header
The system SHALL attach to every response a header describing what the server did (for example "served from cache"), the elapsed time and the peak memory used.

#### Scenario: Cached image
- **WHEN** a resized image is served from cache
- **THEN** the diagnostic header says the image came from cache

### Requirement: Cache Headers
The system SHALL mark served files and images as cacheable for one year and immutable, and SHALL mark all other responses as not cacheable.

#### Scenario: Listing response
- **WHEN** a directory listing is returned
- **THEN** the cache headers mark the response as not cacheable
