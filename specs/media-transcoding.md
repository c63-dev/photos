# Media Transcoding Specification

## Purpose
Defines the automatic conversion of media stored in closed or browser-unfriendly formats (HEIC/HEIF images, HEVC video) into open equivalents, and the safe replacement of the originals once the conversions are verified. Transcoding runs inside the background worker defined in `thumbnail-generation.md` and shares its queue, priorities and controls.

## Requirements

### Requirement: Scope Of Transcoding
The system SHALL transcode every visible file whose format is on the configurable transcode list. The list SHALL default to HEIC and HEIF images and to any video whose video stream is HEVC, regardless of container. Transcoding SHALL be enabled or disabled as a whole and per media type.

#### Scenario: HEVC in an MP4 container
- **WHEN** a `.mp4` file contains an HEVC video stream
- **THEN** it is queued for transcoding

#### Scenario: Already playable
- **WHEN** a `.mp4` file already contains an H.264 video stream
- **THEN** it is not transcoded

### Requirement: Target Formats
The system SHALL convert images to a configurable open image format, defaulting to JPEG at full resolution and high quality. The system SHALL convert video to a configurable container and codec that plays in mainstream browsers without plugins, defaulting to H.264 video in an MP4 container with the audio carried in a browser-playable form, keeping the original resolution and frame rate. Target quality settings SHALL be configurable, with defaults chosen to be visually indistinguishable from the original.

#### Scenario: HEIC to JPEG
- **WHEN** a 4032x3024 HEIC photo is transcoded with default settings
- **THEN** the result is a 4032x3024 JPEG

#### Scenario: HEVC 4K video
- **WHEN** a 3840x2160 60fps HEVC video is transcoded with default settings
- **THEN** the result is 3840x2160 at 60fps H.264 in an MP4 container

### Requirement: Metadata Preservation
The system SHALL carry across to the replacement all embedded metadata the target format can hold, including capture date and time, orientation, location, camera details and descriptive fields, and SHALL set the replacement's file modified time to that of the original. For video, all audio tracks, subtitle tracks and chapter markers SHALL be preserved where the target container supports them.

#### Scenario: Photo with location
- **WHEN** a HEIC photo with GPS coordinates and capture date is transcoded
- **THEN** the JPEG carries the same coordinates and capture date and sorts by the same date

#### Scenario: Video with two audio tracks
- **WHEN** an HEVC video with two audio tracks is transcoded
- **THEN** the replacement has both audio tracks

### Requirement: Naming
The replacement SHALL take the original's base name with the new extension. If a different file with that name already exists in the same directory, the replacement SHALL be given a unique incremented name instead.

#### Scenario: Simple rename
- **WHEN** `IMG_0001.HEIC` is transcoded
- **THEN** the replacement is `IMG_0001.jpg`

#### Scenario: Name clash
- **WHEN** `IMG_0001.HEIC` is transcoded and an unrelated `IMG_0001.jpg` already exists
- **THEN** the replacement is `IMG_0001-2.jpg`

### Requirement: Verification Before Replacement
The system SHALL verify a conversion before touching the original: the output MUST exist, be complete, be decodable, and match the original's dimensions (for images) or duration and resolution within a small tolerance (for video). The original SHALL be discarded only after verification succeeds.

#### Scenario: Truncated output
- **WHEN** a transcode produces a file whose duration is far shorter than the original
- **THEN** the output is deleted, the attempt counts as a failure and the original is kept

### Requirement: Atomic Replacement
The system SHALL write the replacement in a hidden location first and SHALL make it visible and discard the original as one step, so that at no point is the item absent from the directory. If interrupted, the original SHALL remain and any partial output SHALL be discarded on resume.

#### Scenario: Interrupted mid-transcode
- **WHEN** the server stops while a video is half transcoded
- **THEN** on restart the original is untouched, the partial output is removed and the item is queued again

### Requirement: Discard
The system SHALL transcode in place: the replacement is written in the same directory under the same base name with the new extension, and the original is deleted as soon as the replacement is verified.

#### Scenario: Verified replacement
- **WHEN** `clip.mov` containing HEVC is transcoded and verified
- **THEN** `clip.mp4` remains and `clip.mov` is deleted

#### Scenario: Same extension
- **WHEN** `clip.mp4` containing HEVC is transcoded
- **THEN** the replacement is written under a temporary name, verified, then swapped into `clip.mp4` and the original removed

### Requirement: Queue Integration
Transcoding SHALL be a work type in the background queue. Browsing signals SHALL apply: listing a directory moves its pending transcodes to the front of the queue. Thumbnail generation for an item pending transcode SHALL be deferred until the replacement exists, and SHALL then be queued at the front.

#### Scenario: User opens a folder of HEVC videos
- **WHEN** a visitor lists a directory of untranscoded HEVC videos
- **THEN** those transcodes move to the front of the queue

#### Scenario: Thumbnail after transcode
- **WHEN** a transcode completes
- **THEN** the replacement's thumbnails are queued at the front

### Requirement: Pending State In Listings
Directory listings SHALL mark each item whose transcode is queued, in progress or failed, so the UI can show its state. Until a replacement exists, images on the list SHALL still be viewable through on-demand conversion, and videos SHALL be presented as not yet playable.

#### Scenario: Pending video in listing
- **WHEN** a directory containing an untranscoded HEVC video is listed
- **THEN** the item carries a pending transcode state

### Requirement: Changes During Transcoding
If the original is renamed, moved or deleted while its transcode is queued or in progress, the system SHALL discard any output and SHALL re-queue the item at its new location, or drop it if deleted. If the original is modified, the item SHALL be re-queued from the start.

#### Scenario: Moved mid-transcode
- **WHEN** a video is moved to another folder while being transcoded
- **THEN** the in-progress output is discarded and the video is queued again at its new location

### Requirement: Uploads And New Files
Files added by upload, copy, move, unzip or external means that match the transcode list SHALL be queued on discovery like any other media.

#### Scenario: Upload a HEIC
- **WHEN** a HEIC photo is uploaded
- **THEN** it is queued for transcoding

### Requirement: Failure Handling
A failed transcode SHALL be retried once after a delay. After the final failure the item SHALL be recorded as failed with its reason, the original SHALL be left exactly as it was, and the item SHALL NOT be retried until the source changes or a rescan is explicitly requested.

#### Scenario: Undecodable video
- **WHEN** a video cannot be decoded after the retry
- **THEN** it is marked failed, the original remains, and the worker moves on

### Requirement: Preconditions
The system SHALL NOT start a transcode unless the directory is writeable and free space in that location is at least a configurable multiple of the original's size. Items that cannot be transcoded for these reasons SHALL be reported as skipped with the reason and rechecked on later passes.

#### Scenario: Read-only directory
- **WHEN** an HEVC video lives in a directory that is not writeable
- **THEN** it is reported as skipped and no output is produced

### Requirement: Status And Control
The generation status SHALL include transcoding counts: discovered, completed, queued, in progress, failed, skipped, original bytes removed and replacement bytes written. The generation control action SHALL support a report mode that lists what would be transcoded without doing it.

#### Scenario: Report mode
- **WHEN** report mode is requested
- **THEN** the response lists every file that would be transcoded with its current and target format and nothing is changed

### Requirement: Cache Consistency
When a replacement becomes visible, the system SHALL remove cached thumbnails and cached listings that referenced the original.

#### Scenario: Listing after replacement
- **WHEN** a HEIC is replaced by its JPEG
- **THEN** the next listing shows only the JPEG, with no stale entry for the HEIC
