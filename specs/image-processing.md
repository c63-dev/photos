# Image Processing Specification

## Purpose
Defines how the server produces resized images, format conversions, video and PDF thumbnails, and folder preview images, and how those results are cached.

## Requirements

### Requirement: Resize Request
The system SHALL serve a resized copy of an image when the `file` action carries a `resize` size. Image loading and resizing MUST both be enabled, the size MUST be the standard thumbnail size, the high-density size, or one of the explicitly allowed sizes, and the file MUST be an image.

#### Scenario: Allowed size
- **WHEN** a photo is requested with the standard thumbnail size
- **THEN** a JPEG no larger than that size on its longest side is returned

#### Scenario: Disallowed size
- **WHEN** a photo is requested with a size that is not configured as allowed
- **THEN** an error stating the size is not allowed is returned

### Requirement: High-Density Size
The system SHALL offer a second, larger thumbnail size for high-density screens when it is configured larger than the standard size.

#### Scenario: Retina size configured
- **WHEN** the high-density size is larger than the standard size
- **THEN** both sizes are accepted for resize requests and the main document advertises the high-density size

### Requirement: Resize Output
The system SHALL scale the image so its longest side equals the requested size, preserve aspect ratio, honour orientation metadata so the result displays upright, sharpen the result, and encode it as JPEG at the configured quality.

#### Scenario: Rotated source
- **WHEN** a source photo carries orientation metadata rotating it 90 degrees
- **THEN** the resized result is rotated to display upright

### Requirement: Serve Original When Close
The system SHALL serve the original file instead of resizing when the original's longest side is less than the configured minimum ratio times the requested size and its file size does not exceed the configured maximum for direct loading.

#### Scenario: Small source image
- **WHEN** a 400px-wide image is requested at 320px, the minimum ratio is 1.5 and the file is small
- **THEN** the original image is served unchanged

### Requirement: Resource Limits
The system SHALL refuse to resize an image whose pixel count exceeds the configured maximum, SHALL estimate the memory needed and attempt to raise the available memory up to the configured limit, and SHALL return an error naming the required and available memory when it is insufficient.

#### Scenario: Too many pixels
- **WHEN** an image exceeds the configured pixel maximum
- **THEN** an error naming the pixel count and limit is returned

### Requirement: Alternate Resizer
The system SHALL use the external image converter instead of the built-in resizer when configured to prefer it, or when the file's format is one the built-in resizer cannot read. Results produced by the external converter MUST be cached.

#### Scenario: HEIC thumbnail
- **WHEN** a HEIC file is requested at thumbnail size and the converter is available
- **THEN** a JPEG thumbnail is produced via the converter and cached

#### Scenario: Converter unavailable
- **WHEN** a format requires the converter and it is not available
- **THEN** an error pointing to the diagnostics page is returned

### Requirement: Format Conversion
The system SHALL, on a `file` request with the `convert` flag, return a full-size JPEG rendition of an image in a non-browser format (such as HEIC, TIFF, PSD or DNG).

#### Scenario: Convert a TIFF
- **WHEN** a TIFF file is requested with the convert flag
- **THEN** a JPEG of the same dimensions is returned

#### Scenario: Image awaiting transcode
- **WHEN** a HEIC image queued for background transcoding is requested with the convert flag
- **THEN** it is converted on demand as usual, and the background transcode remains queued

### Requirement: Video Thumbnails
The system SHALL, on a `file` request with the `video_thumb` flag, return a JPEG frame taken a few seconds into the video, falling back to the first frame for very short videos. Shortcut files pointing to remote web videos SHALL also be supported.

#### Scenario: Short clip
- **WHEN** a thumbnail is requested for a one-second video
- **THEN** the first frame is returned

### Requirement: PDF Thumbnails
The system SHALL, on a `file` request with the `pdf_thumb` flag, return a JPEG of the first page rendered on a white background, when PDF rendering is available.

#### Scenario: PDF first page
- **WHEN** a PDF thumbnail is requested and rendering is available
- **THEN** a JPEG of page one is returned

### Requirement: Generated Image Cache
The system SHALL cache every generated image keyed by the source file, its size, its modified time and the requested operation, and SHALL serve the cached result on subsequent requests. Caches MAY live in a central cache directory or alongside the source in a hidden `_files` subdirectory, as configured. A generated result that is empty SHALL be replaced by a one-pixel placeholder.

#### Scenario: Repeat request
- **WHEN** the same thumbnail is requested twice
- **THEN** the second response is served from cache

#### Scenario: Source modified
- **WHEN** the source image is modified after its thumbnail was cached
- **THEN** the next request generates and caches a fresh thumbnail

### Requirement: Coordination With Background Generation
On-demand generation SHALL share its cache identity and output format with the background worker (see `thumbnail-generation.md`), SHALL wait for an in-progress background generation of the same item rather than duplicating it, and SHALL raise the item's priority as described there.

#### Scenario: Already generated in background
- **WHEN** the worker has already produced a thumbnail for the requested item
- **THEN** the request is served from that cache entry

### Requirement: Folder Preview
The system SHALL, on a `preview` request for a directory, return a thumbnail representing that directory, provided folder previews, image loading, resizing and image caching are all enabled. The preview SHALL be a resized copy of a designated default preview image if one exists in the directory; otherwise a thumbnail of the first eligible image, video or PDF found; otherwise a one-pixel placeholder. The preview SHALL be cached against the directory and regenerated when the directory changes.

#### Scenario: Default preview image present
- **WHEN** the directory contains the configured default preview image
- **THEN** that image, resized, is the preview

#### Scenario: No eligible files
- **WHEN** the directory contains no images, videos or PDFs
- **THEN** a one-pixel placeholder is returned and cached

#### Scenario: Directory modified
- **WHEN** a file is added to a directory whose preview is cached
- **THEN** the next preview request regenerates the preview
