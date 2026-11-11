# Thumbnail Generation Specification

## Purpose
Defines the continuous background job that generates preview images for all media ahead of demand, and how user browsing moves work to the front of the queue. On-demand generation (see `image-processing.md`) remains available and produces identical results; this job exists so that visitors rarely have to wait for it.

## Requirements

### Requirement: Work Types
The worker SHALL handle these work types sharing one queue: indexing (`metadata-index.md`), preview generation (this spec), viewing variants (`viewing-variants.md`), media transcoding (`media-transcoding.md`) and scheduled cache cleaning (`cache-maintenance.md`). For a given item the order is index, transcode if needed, thumbnails, then variants. A single configurable concurrency, defaulting to one, SHALL apply to the worker as a whole.

#### Scenario: Default concurrency
- **WHEN** no concurrency is configured
- **THEN** the worker processes one item at a time

### Requirement: Continuous Worker
The system SHALL run a single background generation worker continuously, independent of visitor traffic. Only one worker instance SHALL be active at a time. The worker SHALL survive restarts and resume from where it left off without repeating completed work.

#### Scenario: Server restarts mid-backlog
- **WHEN** the server restarts while 5,000 items remain queued
- **THEN** the worker resumes with those 5,000 items and does not regenerate the ones already completed

#### Scenario: Second worker attempted
- **WHEN** a second worker is started while one is already running
- **THEN** the second exits immediately without doing any work

### Requirement: Scope Of Work
The worker SHALL generate, for every visible image, convertible-format image, video and PDF beneath the root: the standard thumbnail size and, when configured, the high-density size. It SHALL also generate the folder preview for every visible directory. Generated results SHALL be indistinguishable from on-demand results and stored under the same cache identity, so that either path can serve the other's output.

#### Scenario: Worker output served on demand
- **WHEN** the worker has generated a video thumbnail and a visitor later requests it
- **THEN** the request is served from cache with no generation

#### Scenario: Hidden media
- **WHEN** a video lives in a directory whose name starts with `_files`
- **THEN** the worker never generates a thumbnail for it

### Requirement: Discovery
The worker SHALL perform a full scan of the root on first run and SHALL thereafter detect new and modified media incrementally, using directory modified times and a configurable periodic rescan. Media whose source has been deleted SHALL have its cached previews removed.

#### Scenario: Initial import
- **WHEN** 8,000 new movie files are placed under the root before the worker first runs
- **THEN** all 8,000 are discovered and queued for background generation

#### Scenario: File added later
- **WHEN** a file is added to an existing directory after the initial scan
- **THEN** it is discovered on the next incremental pass and queued

#### Scenario: Source removed
- **WHEN** a file with cached thumbnails is deleted
- **THEN** its cached thumbnails are removed on the next pass

### Requirement: Queue Order
The queue SHALL be processed newest first by modified time, then by path. Browsing (below) moves items to the front. An item SHALL appear in the queue at most once; moving it to the front does not duplicate it.

#### Scenario: Duplicate enqueue
- **WHEN** an item already queued is moved to the front by browsing
- **THEN** it remains queued once, now at the front

### Requirement: Browsing Signals
When a visitor lists a directory, the system SHALL move every unmade preview in that directory, and the folder previews of its immediate subdirectories, to the front of the queue. A direct request for a specific item's thumbnail SHALL generate it immediately on demand.

#### Scenario: User opens a folder of movies
- **WHEN** a visitor lists a directory containing 200 videos with no thumbnails yet
- **THEN** those 200 thumbnails are moved to the front of the queue

#### Scenario: User scrolls to one video
- **WHEN** the browser requests the thumbnail of a single video that is still queued
- **THEN** it is generated on demand and the queued entry is dropped

### Requirement: On-Demand Coordination
When a visitor requests a thumbnail that the worker is currently generating, the system SHALL wait for that generation to finish, up to a bounded time, and serve the result rather than generating a second copy. Interactive requests MUST never be blocked by background work.

#### Scenario: Requested while in progress
- **WHEN** a thumbnail is requested while the worker is halfway through generating it
- **THEN** the request waits for completion and serves the worker's output

### Requirement: Control
The system SHALL provide a control action, available only when generation control is allowed, to pause, resume, trigger a rescan and trigger an index rebuild.

#### Scenario: Paused
- **WHEN** the worker is paused and a visitor requests a specific thumbnail
- **THEN** the thumbnail is still generated on demand

### Requirement: Failure Handling
A failed generation SHALL be retried once after a delay. After that the item SHALL be recorded as failed with its reason and SHALL NOT be retried until its source changes or a rescan is explicitly requested. Failed items SHALL be served as a placeholder on demand. A failure SHALL never stop the worker.

#### Scenario: Corrupt video
- **WHEN** a video cannot be decoded after the retry
- **THEN** it is marked failed with the reason, a placeholder is served on demand, and the worker moves on

#### Scenario: Source replaced
- **WHEN** a failed file is replaced with a new copy
- **THEN** it is queued again

### Requirement: Resource Safety
The worker SHALL respect the same pixel and memory limits as on-demand generation, SHALL abandon any item that exceeds a configurable per-item time limit and treat it as a failure, and SHALL stop starting new work when free disk space in the cache location falls below a configured threshold.

#### Scenario: Low disk
- **WHEN** free space in the cache location drops below the threshold
- **THEN** no new generations start and the status reports the condition

#### Scenario: Item too slow
- **WHEN** a single video takes longer than the per-item limit
- **THEN** that generation is abandoned and counted as a failure

### Requirement: Progress And Status
The system SHALL provide a status action reporting: items discovered, completed, queued, in progress, failed, recent throughput, whether the worker is running or paused, and an estimated time to finish the backlog. A short summary of the same SHALL be included in the main document so the UI can show progress.

#### Scenario: Status during import
- **WHEN** status is requested with 3,000 of 8,000 items complete
- **THEN** the response reports 3,000 completed, 5,000 queued, the current rate and an estimate for the remainder

### Requirement: Cache Maintenance Interaction
Cache maintenance SHALL NOT remove an entry that is currently being generated. Entries removed by maintenance because they were never accessed SHALL NOT be regenerated in the background; they SHALL be regenerated only when browsing raises them again.

#### Scenario: Expired unused thumbnail
- **WHEN** a worker-generated thumbnail expires because nobody viewed it
- **THEN** the worker does not rebuild it until a visitor browses its directory

### Requirement: Startup Responsiveness
During a large initial backlog the system SHALL remain fully responsive to visitors: listings, browsing and on-demand thumbnails MUST behave as they would with no backlog, apart from the browsed items being moved to the front.

#### Scenario: Browsing during import
- **WHEN** a visitor browses while 8,000 items are queued
- **THEN** listings return at normal speed and the browsed folder's thumbnails appear ahead of the backlog
