---
name: catalog-funnel
description: |
  Build and manage marketing catalogs, landing pages, and multi-step funnels with your AI agent. Create catalogs from JSON schemas, publish them instantly, run A/B split tests, and track visitor analytics — all through conversation.
  Use when: (1) Creating or updating a catalog/funnel/landing page, (2) Checking analytics like visitors, conversions, and drop-off rates, (3) Running A/B tests on different catalog versions, (4) Managing API keys for team access, (5) Uploading videos for catalogs, (6) Viewing individual visitor journeys, (7) Reviewing response distributions for form fields.
  Triggers: catalog funnel, catalog builder, funnel builder, landing page, lead capture, create catalog, catalog analytics, conversion funnel, form builder, split test, ab test, catalog api
---

# Catalog Funnel

Build and manage marketing catalogs, landing pages, and multi-step funnels — directly through your AI agent. Create catalogs with 56+ component types, publish them to your custom subdomain, run A/B split tests, and monitor conversion analytics in real time.

> **Install on OfficeX:** [officex.app/store/en/app/catalog-funnel](https://officex.app/store/en/app/catalog-funnel)

## What You Can Do

- **Create catalogs** — build lead capture forms, product catalogs, multi-step funnels from a JSON schema
- **Publish instantly** — catalogs go live at `https://yourname.catalogs.cloud.zoomgtm.com/your-slug`
- **Check analytics** — see visitors, conversions, page drop-off, field completions, referrer sources, and revenue
- **Run A/B tests** — split traffic between catalog variants to find what converts best
- **View visitor journeys** — trace exactly what each visitor did step by step
- **Manage access** — create API keys for team members or integrations
- **Upload videos** — add video content with automatic HLS transcoding

## Getting Started

After installing Catalog Funnel on OfficeX, you receive credentials automatically. You can also sign up at the dashboard and create API keys from Settings.

```bash
# Your API key (created from Settings page or received on install)
CF_API_KEY="cfk_..."

# Production API
CF_API_URL="https://catalog-funnel-api.cloud.zoomgtm.com"
```

### Authentication

Pass your API key as a Bearer token on all requests:

```bash
curl -H "Authorization: Bearer cfk_..." \
  https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs
```

If you installed via OfficeX, you can also use your install credentials:

```bash
TOKEN=$(echo -n "${OFFICEX_INSTALL_ID}:${OFFICEX_INSTALL_SECRET}" | base64)
curl -H "Authorization: Bearer $TOKEN" \
  https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs
```

---

## Managing Catalogs

### List your catalogs

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs
```

**Response:**
```json
{
  "ok": true,
  "data": [
    {
      "catalog_id": "01HXY...",
      "slug": "my-funnel",
      "name": "My Funnel",
      "status": "published",
      "visibility": "public",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  ]
}
```

### Create a catalog

```
POST https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs
```

```json
{
  "slug": "spring-sale",
  "name": "Spring Sale Landing Page",
  "schema": { ... },
  "status": "published",
  "visibility": "public"
}
```

- `slug` — URL-friendly name (lowercase, hyphens). Your catalog will be live at `https://yourname.catalogs.cloud.zoomgtm.com/spring-sale`
- `status` — `"published"` (live) or `"draft"` (hidden). Default: `"published"`
- `visibility` — `"public"` (listed) or `"unlisted"` (link-only). Default: `"unlisted"`

**Response (201):**
```json
{
  "ok": true,
  "data": {
    "catalog_id": "01HXY...",
    "slug": "spring-sale",
    "name": "Spring Sale Landing Page",
    "status": "published",
    "visibility": "public",
    "url": "https://yourname.catalogs.cloud.zoomgtm.com/spring-sale"
  }
}
```

### View a catalog

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id
```

Returns the full catalog including its schema.

### Update a catalog

```
PUT https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id
```

All fields are optional — only send what you want to change:

```json
{
  "name": "Updated Name",
  "schema": { ... },
  "status": "draft",
  "visibility": "public",
  "slug": "new-slug",
  "old_slug_action": "redirect"
}
```

When changing the slug, `old_slug_action` controls what happens to the old URL:
- `"redirect"` (default) — old URL redirects to the new one
- `"release"` — old URL becomes available for reuse

### Delete a catalog

```
DELETE https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id
```

---

## Analytics & Results

All analytics endpoints require authentication. Each analytics call costs **1 credit**. Event tracking (visitor activity) is **free**.

### Overview metrics

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id
```

**Query params:** `start`, `end` (ISO dates, e.g. `2024-01-01`)

Returns aggregate metrics: unique visitors, total page views, form submissions, conversion rate, page-level views, variant breakdown, referrer sources, checkout stats, and revenue.

### Timeseries (daily/hourly trends)

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id/timeseries
```

**Query params (required):** `start`, `end` (ISO dates), `interval` (`day` or `hour`)

```json
{
  "ok": true,
  "data": [
    { "date": "2024-01-01", "page_views": 150, "sessions": 80, "form_submits": 25, "checkout_completes": 5, "revenue_cents": 4900 }
  ]
}
```

### Drop-off analysis

See exactly where visitors abandon your funnel:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id/dropoff
```

**Query params:** `start`, `end` (ISO dates)

```json
{
  "ok": true,
  "data": {
    "total_visitors": 500,
    "pages": [
      { "page_id": "intro", "visitors": 500, "drop_off_rate": 0 },
      { "page_id": "questions", "visitors": 350, "drop_off_rate": 30 }
    ],
    "fields": [
      { "field_id": "questions/email", "completions": 300, "completion_rate": 85.7 }
    ]
  }
}
```

### Response distributions

See how visitors answered each question or form field:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id/responses
```

**Query params:** `start`, `end`, `page_id`, `component_id` (all optional)

```json
{
  "ok": true,
  "data": {
    "components": {
      "questions/q1": {
        "total_responses": 200,
        "distribution": {
          "Option A": { "count": 112, "percent": 56 },
          "Option B": { "count": 28, "percent": 14 },
          "Option C": { "count": 60, "percent": 30 }
        }
      }
    }
  }
}
```

### Raw events

Browse individual visitor events with filtering:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/catalogs/:id/events
```

**Query params:** `start`, `end`, `cursor`, `limit` (default 100, max 500), `event_type`, `page_id`, `component_id`, `variant_slug`, `utm_source`, `utm_medium`, `utm_campaign`, `referrer`

Response includes a `cursor` for pagination (null when done).

### Visitor journey

Trace a single visitor's complete journey through your catalog:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/analytics/tracers/:tracerId
```

Returns every event in chronological order with a summary: total events, first/last seen, pages viewed, and whether they submitted.

---

## A/B Split Tests

Test different versions of your catalog to find what converts best. Split tests route visitors to different catalog variants based on weighted traffic distribution.

### Create a split test

```
POST https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/split-tests
```

```json
{
  "slug": "spring-sale",
  "name": "Spring Sale A/B Test",
  "destinations": [
    { "slug": "spring-sale-v1", "weight": 50, "label": "Control" },
    { "slug": "spring-sale-v2", "weight": 50, "label": "New headline" }
  ]
}
```

The `slug` is the URL visitors see. They get routed to one of the destination catalogs based on weights. Visitors are sticky — they always see the same variant on return visits.

### Other split test operations

- `GET /api/v1/split-tests` — List all split tests
- `GET /api/v1/split-tests/:slug` — Get one split test
- `PUT /api/v1/split-tests/:slug` — Update (change `name`, `destinations`, or `status`: `"active"` / `"paused"`)
- `DELETE /api/v1/split-tests/:slug` — Delete a split test

---

## Schema Introspection

Get a map of all pages and components in a catalog — useful for understanding the structure before querying analytics:

```
GET https://catalog-funnel-api.cloud.zoomgtm.com/api/v1/catalogs/:id/schema/ids
```

```json
{
  "pages": {
    "landing": { "title": "Get Started", "index": 0 },
    "details": { "title": "Your Details", "index": 1 }
  },
  "components": {
    "landing/email": { "type": "email", "label": "Your Email", "required": true },
    "landing/company": { "type": "short_text", "label": "Company Name" }
  },
  "routing_entry": "landing"
}
```

---

## API Keys

Manage API keys for team members or integrations.

- `POST /api/v1/api-keys` — Create a key (roles: `reader`, `editor`, `admin`, `custom`). Returns the secret once — store it securely.
- `GET /api/v1/api-keys` — List all keys (secrets redacted)
- `DELETE /api/v1/api-keys/:keyId` — Revoke a key
- `POST /api/v1/api-keys/:keyId/rotate` — Rotate: revokes old key, creates new one with same config

---

## Videos

Upload video content to use in your catalogs with automatic HLS transcoding:

- `POST /api/v1/videos/upload` — Upload a video file
- `GET /api/v1/videos/:videoId/status` — Check transcoding progress
- `GET /api/v1/videos/:videoId/hls_url` — Get the playback URL

---

## Webhooks

If your catalog has a `webhook_url` configured in its schema, all visitor events are forwarded there in real time. Each webhook payload includes an `event_id` (ULID) for deduplication and `schema_ref` with human-readable page/component context.

---

## Variant Analytics

Every catalog gets an automatic `catalog:{catalog_id}` tag. To compare analytics across catalog variants (e.g. for A/B tests), add the base catalog's `catalog:{base_id}` tag to each variant's `schema.tags`. API keys scoped with matching `tag_patterns` can then query analytics across all tagged variants.

---

## Catalog Schema Reference

A catalog schema defines your entire funnel as JSON. Here's a minimal lead capture example:

```json
{
  "slug": "lead-capture",
  "pages": [
    {
      "id": "landing",
      "title": "Get Started",
      "components": [
        { "id": "name", "type": "short_text", "label": "Your Name", "required": true },
        { "id": "email", "type": "email", "label": "Email", "required": true }
      ],
      "submit_label": "Submit"
    }
  ],
  "routing": { "entry": "landing", "edges": [] }
}
```

### Component Types (56 total)

**Input (27):** `short_text`, `long_text`, `rich_text`, `email`, `phone`, `url`, `password`, `number`, `currency`, `date`, `datetime`, `time`, `date_range`, `dropdown`, `multiselect`, `multiple_choice`, `checkboxes`, `picture_choice`, `star_rating`, `slider`, `file_upload`, `signature`, `address`, `location`, `switch`, `checkbox`, `choice_matrix`, `ranking`, `opinion_scale`

**Display (11):** `heading`, `paragraph`, `banner`, `image`, `video`, `pdf_viewer`, `social_links`, `html`, `divider`, `faq`, `testimonial`, `pricing_card`

**Layout (3):** `section_collapse`, `table`, `subform`

**Page features:** `payment`, `captcha`

### Multi-Page Routing

Route visitors through different pages based on their answers:

```json
{
  "routing": {
    "entry": "landing",
    "edges": [
      {
        "from": "landing",
        "to": "enterprise",
        "conditions": {
          "match": "all",
          "rules": [{ "field": "company_size", "operator": "greater_than", "value": 100 }]
        }
      },
      { "from": "landing", "to": "standard", "is_default": true }
    ]
  }
}
```

**Condition operators:** `equals`, `not_equals`, `contains`, `not_contains`, `greater_than`, `less_than`, `greater_than_or_equal`, `less_than_or_equal`, `starts_with`, `ends_with`, `regex`, `in`, `not_in`, `is_empty`, `is_not_empty`, `between`

### Quiz Scoring

Add quiz scoring to any multiple choice or input component:

```json
{
  "id": "q1",
  "type": "multiple_choice",
  "label": "What does CTA stand for?",
  "options": ["Click To Act", "Call To Action", "Create The Ad"],
  "quiz": { "correct_answer": "Call To Action", "points": 10, "explanation": "CTA = Call To Action" }
}
```

Score-based routing: `{ "score": "percent", "operator": "greater_than", "value": 80 }`

### Popups

Trigger popups based on visitor behavior:

```json
{
  "popups": [
    {
      "id": "exit-popup",
      "trigger": { "type": "exit_intent", "delay_ms": 3000 },
      "pages": ["landing"],
      "mode": "modal",
      "content": { "title": "Wait!", "body": "Get 10% off before you go" }
    }
  ]
}
```

**Trigger types:** `exit_intent`, `scroll_depth`, `inactive`, `timed`, `page_count`, `custom`, `video_progress`, `video_chapter`

---

## CLI

Manage catalogs from the command line:

```bash
npx catalogs catalog push schema.json --publish    # Create or update a catalog from a JSON file
npx catalogs catalog list                           # List all your catalogs
npx catalogs video upload ./intro.mp4               # Upload a video
npx catalogs video status VIDEO_ID                  # Check transcoding progress
```

---

## Event Tracking (Free)

Visitor events are tracked automatically by the catalog frontend. You can also send custom events:

```
POST https://catalog-funnel-api.cloud.zoomgtm.com/events
```

**Valid event types:** `page_view`, `field_change`, `field_complete`, `form_submit`, `action_click`, `exit_intent`, `session_start`, `session_resume`, `cart_add`, `cart_remove`, `checkout_start`, `checkout_skip`, `checkout_complete`, `payment_info_added`, `offer_declined`, `lead_captured`, `video_play`, `video_pause`, `video_progress`, `video_complete`, `video_chapter`, `video_seek`

Batch up to 25 events: `POST /events/batch` with `{ "events": [...] }`
