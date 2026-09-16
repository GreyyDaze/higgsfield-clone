# Higgsfield Clone — 24-Hour Build PRD

## 1\. Product Goal

Build the **smallest meaningful version of Higgsfield** that demonstrates strong product judgment, fast execution, and a polished user experience.

The goal is not to clone the entire Higgsfield platform. The goal is to build a coherent creative workflow where a user can:

**Start with an idea/image → generate an image → modify it → turn it into a video → save and reuse the result.**

Every core feature should be genuinely functional, not just a UI mockup.

 *

## 2\. Target User

A creator who wants to quickly turn an idea, prompt, or existing image into polished visual content.

### Main use case

1. Start with a prompt or image.
2. Generate a visual.
3. Modify or iterate on it.
4. Animate it into a video.
5. Save, revisit, download, or reuse the result.

 *

# 3\. Product Surface

The application has two main areas:

### Create

The main workspace for generating content.

Modes:

*   **Image**
*   **Edit**
*   **Image → Video**
*   **Video** — P1, only if the core flows are stable

### My Creations

A persistent library of generated images and videos.

Users can:

*   View creations
*   Open a creation
*   Download
*   Delete
*   Reuse as an input
*   See related outputs

 *

# 4\. P0 — Core Features

## A. AI Image Generation

User enters:

*   Prompt
*   Aspect ratio

Then:

**Generate → Loading → Real AI result → Save → Download**

Requirements:

*   Real image generation API
*   Loading state
*   Error state
*   Retry
*   Regenerate
*   Download
*   Persist result
*   Prevent duplicate submissions

This is the first vertical slice and should be working before moving to the next feature.

 *

## B. Image → Video

User selects:

*   A generated image
*   Or an uploaded image

Then enters a motion instruction.

Example:

> Slow cinematic camera movement, wind moving through the subject’s hair.

Flow:

**Image → Motion prompt → Generate → Progress → Real video → Play → Save → Download**

Requirements:

*   Real I2V generation
*   Async generation handling
*   Progress/status state
*   Video player
*   Error/retry
*   Save result
*   Download result

 *

## C. Image Editing / Variation

User selects an existing image and describes the change.

Example:

> Change the background to a modern Tokyo street at night.

Flow:

**Existing image → Edit instruction → Generate → New image → Save**

The original image remains available.

Requirements:

*   Real image editing/generation
*   Original → edited relationship
*   Loading state
*   Error/retry
*   Download
*   Save
*   Ability to continue editing the new result

 *

# 5\. P1 — Optional Feature

## Text → Video

Only build this if the three P0 flows are already stable.

Flow:

**Prompt → Video → Play → Save → Download**

It should reuse the same video-generation infrastructure as Image → Video.

If adding it would reduce the quality or reliability of P0, **do not build it**.

 *

# 6\. Shared Generation System

All generation features should use a common generation model internally.

Each generation should track:

```
Generation
├── id
├── type
├── status
├── prompt / instruction
├── input asset
├── output asset
├── parent generation
├── created_at
└── error
```

### Statuses

*   queued
*   processing
*   completed
*   failed

This allows the product to support relationships such as:

```
Image #1
   ↓
Edited Image #2
   ↓
Video #3
```

This makes the product feel like a real creative tool rather than three disconnected AI demos.

 *

# 7\. My Creations

The library should persist across refreshes.

Each creation can contain:

*   Preview
*   Type
*   Prompt/instruction
*   Created time
*   Parent creation
*   Output asset

Actions:

*   Open
*   Download
*   Delete
*   Reuse
*   View related generations

The reviewer should be able to:

**Generate → refresh the browser → return to My Creations → still see the result.**

 *

# 8\. UX / UI Direction

The product should be **Higgsfield-inspired**, not a pixel-for-pixel clone.

Focus on:

*   Strong visual hierarchy
*   Large media previews
*   Simple generation controls
*   Clear primary actions
*   Good spacing and typography
*   Responsive layout
*   Fast interaction
*   Consistent components
*   Minimal unnecessary UI

### Important states

Every important action should have:

*   Empty state
*   Loading state
*   Success state
*   Error state
*   Retry state
*   Disabled state where appropriate

Avoid:

*   Fake progress
*   Hardcoded generated results
*   Buttons that don’t work
*   Dead navigation
*   Duplicate submissions
*   Results disappearing after refresh

 *

# 9\. AI Architecture

Use a provider abstraction so the application isn’t tightly coupled to one provider.

### Image generation / editing

**Cloudflare Workers AI**

Use eligible free Workers AI models for image generation/editing.

### Video generation

**Pixazo LTX**

Use the free LTX endpoints for:

*   Text → Video
*   Image → Video

Video generation should be handled asynchronously:

```
FastAPI
   ↓
Create video job
   ↓
Poll provider
   ↓
Completed
   ↓
Store video
   ↓
Return result
```

 *

# 10\. Technology Stack

### Frontend

*   Next.js
*   React
*   TypeScript
*   Tailwind CSS
*   shadcn/ui

### Backend

*   FastAPI
*   Python
*   Pydantic
*   httpx

### Database

*   **Supabase PostgreSQL**
*   SQLAlchemy
*   Alembic

### File Storage

*   **Supabase Storage**

Store generated:

*   Images
*   Videos
*   User uploads

The database stores the metadata and references to those assets.

### Deployment

*   **Frontend:** Vercel
*   **Backend:** FastAPI Cloud
*   **Database:** Supabase
*   **Storage:** Supabase Storage

### AI

*   **Cloudflare Workers AI** — image generation/editing
*   **Pixazo LTX** — video generation

 *

# 11\. Architecture

```
                    ┌──────────────────┐
                    │     Next.js      │
                    │      Vercel      │
                    └────────┬─────────┘
                             │
                         REST API
                             │
                             ▼
                    ┌──────────────────┐
                    │     FastAPI      │
                    │   FastAPI Cloud  │
                    └───────┬──────────┘
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
          ▼                 ▼                 ▼
    ┌───────────┐    ┌─────────────┐   ┌───────────┐
    │ Supabase  │    │  Cloudflare │   │  Pixazo   │
    │ DB+Storage│    │  Workers AI │   │    LTX    │
    └───────────┘    └─────────────┘   └───────────┘
```

FastAPI owns the application logic and provider integrations.

Next.js owns the user experience.

Supabase owns persistence and media storage.

 *

# 12\. What Is Explicitly Out of Scope

Do **not** attempt to recreate the entire Higgsfield platform.

Out of scope:

*   Canvas / node-based editor
*   Full Cinema Studio
*   Soul ID training
*   Character-training system
*   Collaboration
*   Team/workspace management
*   Billing
*   Credits system
*   Full model marketplace
*   Advanced timeline editor
*   Social publishing
*   Dozens of specialized AI tools
*   Complex project management
*   Every Higgsfield app

These features are deliberately excluded so the core product can actually work.

 *

# 13\. Definition of “Working”

A feature is **not complete because the UI exists**.

The reviewer must be able to complete the full workflow on the deployed application.

### Image

```
Prompt
  ↓
Real AI generation
  ↓
Image displayed
  ↓
Saved
  ↓
Downloadable
```

### Edit

```
Image
  ↓
Edit instruction
  ↓
Real AI generation
  ↓
New image
  ↓
Original preserved
  ↓
Saved + downloadable
```

### Image → Video

```
Image
  ↓
Motion prompt
  ↓
Real video generation
  ↓
Video playable
  ↓
Saved + downloadable
```

### Creations

```
Generate
  ↓
Refresh browser
  ↓
Open My Creations
  ↓
Result still exists
  ↓
Reuse / download
```

 *

# 14\. Build Priority

## Phase 1 — Foundation

*   Agent capture setup
*   Next.js application
*   FastAPI application
*   Frontend ↔ backend connection
*   App shell
*   Create page
*   My Creations page
*   Supabase setup
*   Generation data model
*   Storage setup
*   Provider abstraction
*   Initial deployment

## Phase 2 — Image Generation

Build the complete vertical slice:

**Prompt → AI image → database → storage → UI → download**

Do not move on until this works end-to-end.

## Phase 3 — Image → Video

Add:

**Image → motion prompt → async video generation → storage → playback**

## Phase 4 — Image Editing

Add:

**Existing image → edit instruction → new image → persistence**

## Phase 5 — Creations

Connect all generations to:

*   History
*   Relationships
*   Reuse
*   Delete
*   Download
*   Refresh persistence

## Phase 6 — Polish

Focus on:

*   Responsive behavior
*   Loading states
*   Error handling
*   Retry
*   Empty states
*   Visual consistency
*   Interaction quality
*   Performance
*   Removing rough edges

## Phase 7 — Optional T2V

Only if everything above is stable.

 *

# 15\. Assignment Success Criteria

The assignment explicitly judges:

### Speed

**How much working product was shipped in the available time.**

Therefore, prioritize complete vertical slices over infrastructure perfection or feature count.

### Product Judgment

Show that you understood the product and deliberately chose what to build first.

The product should feel coherent even though major Higgsfield features are missing.

### UX / UI

The shipped product should be good to use:

*   Clear
*   Fast
*   Polished
*   Responsive
*   Predictable
*   Good loading/error states
*   Real outputs

 *

# 16\. Final Reviewer Experience

The ideal reviewer journey is:

```
Open app
   ↓
Generate an image
   ↓
See a real AI result
   ↓
Edit the image
   ↓
See the edited result
   ↓
Animate the image
   ↓
Watch the real video
   ↓
Open My Creations
   ↓
See the complete history
   ↓
Download or reuse a result
```

That is the **smallest meaningful Higgsfield-like product** we are targeting.

The priority is not to demonstrate how many features can be copied.

The priority is to demonstrate:

**“I can take a product, identify its core value, build the important parts quickly, make the experience good, and ship something that actually works.”**