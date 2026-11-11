# File Management Specification

## Purpose
Defines the write operations the UI can trigger on files and folders and the rules they share.

## Requirements

### Requirement: Operation Response
The system SHALL answer each file-management action with a success count and a failure count (or a single success flag for single-item actions), and SHALL answer with an error when nothing succeeded.

#### Scenario: Partial success
- **WHEN** three items are deleted and one cannot be removed
- **THEN** the response reports two successes and one failure

### Requirement: Item Name Rules
The system SHALL reject any new name that is empty or only whitespace, contains any of `< > : " ' / \ | ? * #`, contains `..`, or ends with a dot.

#### Scenario: Name with slash
- **WHEN** a rename request supplies `photos/2024`
- **THEN** an "Invalid name" error is returned

### Requirement: Unique Naming
The system SHALL, when it must create an item whose name is already taken, append an incrementing `-N` suffix before the extension, continuing an existing numeric suffix rather than nesting one.

#### Scenario: Duplicate of a numbered file
- **WHEN** `photo-3.jpg` is duplicated and `photo-3.jpg` exists
- **THEN** the copy is named `photo-4.jpg`

### Requirement: Delete
The system SHALL delete the listed files and folders, removing folder contents recursively, and SHALL NOT follow symlinks into their targets.

#### Scenario: Delete a symlinked folder
- **WHEN** a symlinked folder is deleted
- **THEN** only the link is removed and the target is untouched

### Requirement: Rename
The system SHALL rename a single item to the supplied name within its current folder.

#### Scenario: Rename without a name
- **WHEN** a rename request has no name
- **THEN** a "Missing name parameter" error is returned

### Requirement: New File And New Folder
The system SHALL create an empty file or an empty folder inside the selected directory, using the supplied name or, when none is given, `untitled-file.txt` or `untitled-folder` made unique.

#### Scenario: Unnamed new folder
- **WHEN** a new folder is requested without a name and `untitled-folder` exists
- **THEN** `untitled-folder-2` is created

### Requirement: Duplicate
The system SHALL copy the listed items in place. A single item with a supplied name is copied to that name; otherwise each item is copied under an automatically uniqued name. Folders are copied recursively.

#### Scenario: Duplicate several items
- **WHEN** two files are duplicated without a name
- **THEN** each gets a `-2` suffixed copy beside it

### Requirement: Copy And Move
The system SHALL copy or move the listed items into the target directory. An item MUST NOT be copied or moved into itself or its own descendant. An existing item at the destination that is newer than the source SHALL NOT be overwritten. Copied files keep their original modified time. Symlinks are recreated at the destination rather than having their targets copied.

#### Scenario: Move into self
- **WHEN** a folder is moved into one of its own subfolders
- **THEN** a "can't move into self" error is returned

#### Scenario: Newer destination
- **WHEN** a file is copied over a destination file that is newer
- **THEN** the destination is left unchanged and the item counts as failed

### Requirement: Zip
The system SHALL create a zip archive from the listed items beside the first item, named with the supplied name, or `<item>.zip` for a single item, or `archive.zip` for several, made unique. Hidden and unreadable items are omitted. An archive with no entries is a failure.

#### Scenario: Zip a folder
- **WHEN** a folder `trip` is zipped without a name
- **THEN** `trip.zip` is created beside it containing the folder's visible contents

### Requirement: Unzip
The system SHALL extract a single non-empty zip file into the given target directory, or into the zip file's own directory when no target is given. The target MUST be writeable.

#### Scenario: Extract beside the archive
- **WHEN** a zip is extracted with no target directory
- **THEN** its contents land in the same directory as the archive

### Requirement: Text Edit
The system SHALL overwrite a single writeable file with the supplied text and SHALL invalidate the parent directory's cached listing.

#### Scenario: Save edited text
- **WHEN** new text is posted for a writeable text file
- **THEN** the file now contains exactly that text and the next listing reflects the change

### Requirement: Upload
The system SHALL accept one uploaded file per request into the target directory. The upload MUST have a non-zero size, MUST NOT exceed the configured maximum size, and its name MUST NOT contain slashes. Upload transport errors SHALL be reported with a human-readable message. The response SHALL include the final file name and its URL.

#### Scenario: Oversized upload
- **WHEN** an uploaded file exceeds the configured maximum
- **THEN** an error naming the size and the limit is returned

### Requirement: Upload Type Restrictions
The system SHALL, when allowed upload types are configured, accept a file only if its extension or mime type matches one of the entries (mime entries may use wildcards such as `image/*`). Files claiming an image extension MUST additionally be verified as real images.

#### Scenario: Disguised executable
- **WHEN** allowed types include `.jpg` and a non-image file named `x.jpg` is uploaded
- **THEN** an "Invalid image type" error is returned

### Requirement: Upload Folder Structure
The system SHALL, when the upload carries a relative path, recreate that folder structure beneath the target directory.

#### Scenario: Drag and drop a folder
- **WHEN** a file is uploaded with relative path `album/day1/a.jpg`
- **THEN** it is stored at `<target>/album/day1/a.jpg`, creating folders as needed

### Requirement: Companions Move Together
Rename, move, copy, duplicate and delete on a primary item SHALL apply to all of its companions (see `directory-listing.md`), keeping their shared base name. Downloads of a primary item SHALL offer its companions.

#### Scenario: Rename a RAW pair
- **WHEN** `DSC_1.JPG` with companion `DSC_1.NEF` is renamed to `sunset.JPG`
- **THEN** the companion becomes `sunset.NEF`

### Requirement: Index Updates
Every file-management action SHALL update the metadata index (see `metadata-index.md`) before responding.

#### Scenario: Upload indexed
- **WHEN** an upload completes
- **THEN** the new file is present in the index

### Requirement: Operations On Transcoding Items
Rename, move and delete SHALL succeed on an item whose transcode is queued or in progress; the pending work SHALL follow the item or be dropped as described in `media-transcoding.md`.

#### Scenario: Delete during transcode
- **WHEN** a video is deleted while its transcode is in progress
- **THEN** the delete succeeds and the transcode is abandoned

### Requirement: Upload Collision Policy
The system SHALL handle an upload whose name already exists according to configuration: increment the name, fail with an error, or overwrite.

#### Scenario: Increment policy
- **WHEN** `a.jpg` exists and the policy is increment
- **THEN** the upload is stored as `a-2.jpg` and the response reports that name
