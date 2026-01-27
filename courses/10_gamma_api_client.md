# PRD: Gamma Slide Generation API Client

## 1. Goal

Provide a reusable client and integration flow to:

- Generate slide decks for each course (and optionally for each module) using Gamma.
- Save the resulting deck as a PDF.
- Attach that PDF URL to the corresponding course record.
- Keep logic centralized and testable.

## 2. Assumptions

- Backend service (Node/TypeScript or similar) exists for admin operations.
- There is a storage layer for static assets (e.g. S3, Backblaze, or your current media storage).
- Gamma provides:
  - an HTTP API to create slide decks from a prompt (e.g. `POST /v1/presentations`),
  - and either:
    - an immediate PDF URL, or
    - a job ID that can be polled.

(This PRD stays API-agnostic; adapt endpoint names to the real Gamma docs.)

## 3. Core Requirements

### 3.1 Course Slide Generation

For each course:

- Admin can trigger slide generation via a protected endpoint.
- System:
  - Builds a course-specific prompt.
  - Sends it to Gamma.
  - Receives a deck.
  - Converts or downloads it as PDF.
  - Stores PDF.
  - Updates `Course.resources.slideDeckPdfUrl` and `slideDeckGammaJobId` (if any).

### 3.2 Idempotency

- If a course already has `slideDeckPdfUrl` and admin does not pass `force=true`, the call must:
  - return 409 Conflict, or
  - return existing resource.
- Admin can pass `force=true` to regenerate (and overwrite) the slides.

### 3.3 Error Handling

- If Gamma call fails:
  - Log error with full context (courseId, prompt snippet).
  - Return clear error to admin: “Gamma generation failed”.
- If file storage fails:
  - Do not update course record.
  - Return 500 with message.

## 4. API Design

### 4.1 Endpoint: Trigger Course Slide Generation

**Method:** `POST`  
**Path:** `/admin/courses/:courseId/gamma/slides`  

**Auth:**  
- Must require admin role or internal service key.

**Request body:**

```json
{
  "forceRegenerate": false,
  "target": "course" 
}
````

`target` can later support `"module"` to generate module-specific decks.

**Response – success:**

```json
{
  "courseId": "c1",
  "slideDeckPdfUrl": "https://cdn.example.com/courses/c1/slides.pdf",
  "gammaJobId": "gamma-12345",
  "status": "completed"
}
```

**Response – already exists (no force):**

```json
{
  "status": "already_exists",
  "courseId": "c1",
  "slideDeckPdfUrl": "https://cdn.example.com/courses/c1/slides.pdf"
}
```

### 4.2 Internal Gamma Client Interface

Implement a shared client:

```ts
export interface GammaSlideRequest {
  title: string;
  prompt: string;
}

export interface GammaSlideResponse {
  jobId: string;
  pdfUrl?: string;
}

export interface GammaClient {
  createPresentation(req: GammaSlideRequest): Promise<GammaSlideResponse>;
  getPresentation(jobId: string): Promise<GammaSlideResponse>;
}
```

### 4.3 Storage Interface

Abstract file storage:

```ts
export interface FileStorage {
  saveFromUrl(options: {
    sourceUrl: string;
    destPath: string;
    contentType?: string;
  }): Promise<{ publicUrl: string }>;
}
```

## 5. Flow

1. **Admin calls:** `POST /admin/courses/:courseId/gamma/slides`.
2. **Backend logic:**

   * Load course metadata.
   * If slides exist and `forceRegenerate` is false → return “already_exists”.
   * Build prompt string using:

     * course title,
     * audience,
     * module structure and key points (from course PRD).
3. **Gamma client:**

   * Call `createPresentation`.
   * If `pdfUrl` is immediate:

     * Use `FileStorage.saveFromUrl` to store.
     * Update course record with `slideDeckPdfUrl` and `gammaJobId`.
   * If only `jobId`:

     * Store jobId.
     * Either:

       * immediately poll until `pdfUrl` appears, or
       * enqueue background job to poll.
4. **Return JSON response** with final `slideDeckPdfUrl` if available.

## 6. Data Model Extension

Extend `CourseResources`:

```ts
export interface CourseResources {
  pocketAidPdfUrl?: string;
  slideDeckPdfUrl?: string;
  slideDeckGammaJobId?: string;
  lastSlidesGeneratedAt?: string;
}
```

## 7. Non-Functional Requirements

* Proper timeouts and retries on Gamma calls.
* Clear logging with correlation IDs.
* Respect Gamma rate limits (throttle or queue requests).
* Safe for future extension to:

  * module-specific decks,
  * localized decks (prompt includes language).

