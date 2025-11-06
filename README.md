# SPX User API

Official API documentation for the StarPX (SPX) User API 

## Overview

The SPX User API provides a RESTful interface for astronomical plate solving, allowing you to:

- Authenticate with API keys and receive JWT tokens
- Submit astronomical images for plate solving
- Provide detected star sources or upload full images
- Retrieve job status and astrometric solutions
- Get WCS (World Coordinate System) projection parameters

## Base URL

**Production:** `https://xapi.starpx.com`

## Quick Start

### 1. Authentication

Obtain a JWT token using your API key:

```bash
curl -X POST https://xapi.starpx.com/user/authenticate \
  -H "Content-Type: application/json" \
  -d '{
    "api_key": "xVRczQ6HNh"
  }'
```

**Response:**
```json
{
  "status": "success",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiry": 14400
}
```

The token is valid for **4 hours** (14,400 seconds).

### 2. Create Platesolve Job

There are two ways to create a job:

#### Option A: With Source List (Immediate Processing)

If you have already detected sources/stars in your image, submit them directly:

```bash
curl -X POST https://xapi.starpx.com/user/platesolve/new \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "job_info": {
      "target_image": {
        "file_name": "M31_luminance_300s.fits",
        "creation_date": 1699228800,
        "width": 4096,
        "height": 4096,
        "mime_type": "image/fits",
        "image_size": 67108864,
        "fits_order": "up"
      },
      "hints": {
        "center_ra": 10.6847,
        "center_dec": 41.2687,
        "fov": 1.5,
        "app": 0.52,
        "epoch": "2000.0"
      }
    },
    "source_list": {
      "sources": [
        {"x": 1024.34, "y": 2048.67, "f": 125430.5, "b": 1205.3, "p": 45230.2, "a": 28.6, "t": 8.4},
        {"x": 2156.89, "y": 1834.12, "f": 98234.8, "b": 1198.7, "p": 38920.4, "a": 24.3, "t": 7.9},
        {"x": 3201.45, "y": 3102.88, "f": 156782.3, "b": 1210.2, "p": 52340.6, "a": 32.1, "t": 9.2},
        {"x": 892.67, "y": 1456.23, "f": 72384.1, "b": 1203.9, "p": 28450.8, "a": 21.7, "t": 6.8},
        {"x": 3789.12, "y": 2567.34, "f": 189456.7, "b": 1215.6, "p": 61230.9, "a": 35.4, "t": 10.1}
      ],
      "scale": 1.0,
      "crop": {"left_x": 0, "bot_y": 0}
    }
  }'
```

**Response:**
```json
{
  "status": "success",
  "job_id": "5N2FIUR6EiBITG4H"
}
```

Job starts processing immediately.

#### Option B: With Image Upload (Requires Upload Step)

If you want SPX to detect sources from the image:

```bash
curl -X POST https://xapi.starpx.com/user/platesolve/new \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "job_info": {
      "target_image": {
        "file_name": "NGC7000_Ha_600s.fits",
        "creation_date": 1699228800,
        "width": 4096,
        "height": 4096,
        "mime_type": "image/fits",
        "image_size": 67108864,
        "fits_order": "down"
      },
      "hints": {
        "center_ra": 312.7667,
        "center_dec": 44.2833,
        "fov": 2.1,
        "app": 0.65,
        "epoch": "2000.0"
      }
    },
    "image": {
      "file_name": "NGC7000_Ha_600s_preview.jpg",
      "creation_date": 1699228800,
      "width": 2048,
      "height": 2048,
      "mime_type": "image/jpeg",
      "fits_order": "down",
      "scale": 2.0,
      "crop": {"left_x": 512, "bot_y": 512}
    }
  }'
```

**Response:**
```json
{
  "status": "success",
  "job_id": "rzB3T4viZiFIprKw",
  "image_post_url": "/user/platesolve/rzB3T4viZiFIprKw/upload"
}
```

Job waits for image upload before processing.

### 3. Check Job Status

Poll this endpoint to check job progress:

```bash
curl -X GET https://xapi.starpx.com/user/platesolve/rzB3T4viZiFIprKw/status \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

**Response (Waiting for Upload):**
```json
{
  "status": "success",
  "job_status": "waiting_image"
}
```

**Response (Processing):**
```json
{
  "status": "success",
  "job_status": "busy",
  "launch_time": 1762357842
}
```

**Response (Solved):**
```json
{
  "status": "success",
  "job_status": "solved",
  "launch_time": 1762357842,
  "solve_time": 1762357854,
  "solution": {
    "center_ra": 159.56050951686794,
    "center_de": -58.68924318865147,
    "orientation": 257.41421666011445,
    "parity": 1,
    "scale_app": 0.543709614446011,
    "width": 4096,
    "height": 4096,
    "pixel_area": 16,
    "world_area": 0.38269159135410835,
    "type": "tan",
    "bbox": [
      {"ra": 159.11158863158835, "dec": -58.31951168872647},
      {"ra": 158.97995351939014, "dec": -58.621270339694256}
    ],
    "proj_tan": {
      "crval1": 159.374545901,
      "crval2": -58.6401675065,
      "crpix1": 1870.53692627,
      "crpix2": 1353.39221191,
      "cd11": -0.0000329189747641,
      "cd12": 0.000147442865646,
      "cd21": -0.000147344580309,
      "cd22": -0.0000329684994992,
      "flipy": 1,
      "sip_a": [0.0, 0.0, 5.46495259436e-08],
      "sip_ap": [-0.000208454612496, -1.32683935315e-07],
      "sip_b": [0.0, 0.0, -6.54299141219e-07],
      "sip_bp": [-0.00197544868301, 2.00243511212e-06]
    }
  }
}
```

### 4. Upload Image (If Required)

If your job is waiting for an image upload:

```bash
curl -X POST https://xapi.starpx.com/user/platesolve/rzB3T4viZiFIprKw/upload \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -F "file=@/path/to/image.fits"
```

**Response:**
```json
{
  "status": "success"
}
```

After upload, the job automatically starts processing.

## API Reference

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/user/authenticate` | Get JWT token from API key |
| POST | `/user/platesolve/new` | Create new platesolve job |
| GET | `/user/platesolve/{job_id}/status` | Get job status and solution |
| POST | `/user/platesolve/{job_id}/upload` | Upload image for job |

### Job Lifecycle

```
┌─────────────────┐
│  Create Job     │
└────────┬────────┘
         │
         ├── Source list provided? ──> [Job starts immediately]
         │
         └── Image upload needed? ──> [waiting_image] ──> Upload ──> [busy]
                                                                        │
                                                                        ├──> [solved]
                                                                        │
                                                                        └──> [fail]
```

### Job Statuses

- **`waiting_image`** - Job created, waiting for image upload
- **`submitted`** - Job submitted to platesolve queue
- **`busy`** - Job currently processing (1-15 seconds)
- **`solved`** - Successfully plate solved (solution data available)
- **`fail`/`failed`** - Plate solve failed

### Source Fields

Each source in `source_list.sources` must include:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `x` | float | ✓ | X coordinate in pixels |
| `y` | float | ✓ | Y coordinate in pixels |
| `f` | float | ✓ | Flux (sum of pixel values) |
| `b` | float | ✓ | Average background value |
| `p` | float | ✗ | Peak value |
| `a` | float | ✗ | Area in square pixels |
| `t` | float | ✗ | Threshold SNR |

**Limits:**
- Maximum 5,000 sources per request
- Maximum file upload size: 100MB

### Solution Data

When a job is solved, the `solution` object contains:

- **`center_ra`** / **`center_de`** - Image center coordinates (RA/Dec in degrees)
- **`orientation`** - Image rotation in degrees
- **`scale_app`** - Image scale in arcseconds per pixel
- **`bbox`** - Bounding box polygon (RA/Dec corners)
- **`proj_tan`** - WCS TAN projection parameters
  - `crval1`, `crval2` - Reference RA/Dec
  - `crpix1`, `crpix2` - Reference pixel coordinates
  - `cd11`, `cd12`, `cd21`, `cd22` - CD matrix
  - `sip_a`, `sip_ap`, `sip_b`, `sip_bp` - SIP distortion coefficients

## Error Handling

All errors return HTTP status codes with JSON:

```json
{
  "status": "error",
  "error": "error description"
}
```

### Common Errors

| Code | Error | Cause |
|------|-------|-------|
| 400 | `api_key is required` | Missing API key in authentication |
| 401 | `invalid API key` | Incorrect API key |
| 401 | `invalid or expired token` | JWT token expired or invalid |
| 403 | `access denied` | Attempting to access another user's job |
| 404 | `job not found` | Job ID doesn't exist |
| 400 | `source_list.sources cannot exceed 5000 items` | Too many sources |

## OpenAPI Specification

Full API specification available in OpenAPI 3.0 format:

**[openapi/spec.yaml](./openapi/spec.yaml)**

## License

Copyright © StarPX. All rights reserved.
