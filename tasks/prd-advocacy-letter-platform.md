# PRD: Static-First Advocacy Letter Platform Without a Database

## Summary
Build a reusable advocacy letter platform optimized for GitHub Pages and similar low-cost hosting, with no full database in the core design. The system should let an operator publish a letter, attach references, collect supporter sign-ons, moderate them, and publish approved signers by area, while remaining easy for other groups to fork or spawn for their own campaigns.

The key design constraint is content- and blob-first storage: campaign content, pending submissions, moderation state, and published signer indexes should live in files or object storage, not Postgres or another traditional DB.

## Key Changes
- Hosting and runtime:
  - Public site is fully static and deployable to GitHub Pages.
  - Dynamic behavior is optional and minimal: form intake and moderation triggers can use serverless functions, but the public experience cannot depend on an always-on backend.
- Storage model:
  - No full database.
  - Canonical campaign data lives as versioned Markdown/JSON files.
  - Signature submissions are stored as append-only JSON/NDJSON objects in blob storage or Git-managed files.
  - Moderation state is stored as explicit status files or manifests, separate from published signer data.
  - Public pages only consume approved, prebuilt JSON artifacts.
- Core product:
  - Campaign config with branding, locale, signer field settings, legal copy, and share metadata.
  - Letter authoring with references and optional supporting resources.
  - Public sign-on form.
  - Moderation workflow with approve/reject/edit before publish.
  - Static signer lists grouped by area/region and total counts.
  - Share/export tools for outreach and campaign distribution.
- Reusability:
  - Structure the project as a starter platform rather than one campaign instance.
  - A new group should be able to fork the repo, change workspace config/content, connect a storage target, and publish.
- Extension model:
  - Keep graphs, constituency/MP data, enrichment, and analytics out of core.
  - Ship optional `/skills` templates/prompts that generate Python or TypeScript code into defined content/data directories.

## Implementation Changes
- App structure:
  - Use TanStack as the main app framework for operator tooling, routing, and local preview.
  - Separate:
    - `content/` for authored campaigns and references
    - `data/` for generated public artifacts
    - `submissions/` or blob-backed equivalents for pending moderation records
    - `skills/` for optional generators and prompt packs
- Submission pipeline:
  - Form submissions write a single record per submission into blob storage or a GitHub-backed file queue.
  - Moderation reads pending records, writes status updates, and regenerates published signer indexes.
  - Approval triggers a static rebuild/deploy via GitHub Actions or equivalent.
- Interfaces:
  - `Campaign` manifest: slug, title, body source, references, status, branding, share config, field config.
  - `SignatureSubmission` record: id, campaign slug, submitted fields, consent flags, anti-spam metadata, moderation status, timestamps.
  - `PublishedSignerIndex` artifact: approved signer display fields grouped for static rendering.
  - `StorageAdapter` contract: put/get/list/delete or archive objects for content, submissions, and generated artifacts.
- Admin path:
  - Recommended default is a lightweight operator UI plus CLI fallback.
  - CLI moderation must remain a first-class path so the platform still works without any dedicated admin hosting.

## Test Plan
- Static output:
  - campaign pages render correctly from content plus generated signer artifacts
  - public site works with no runtime database or authenticated API
- Submission flow:
  - new sign-on creates a pending submission record
  - approved sign-on appears in published artifacts after rebuild
  - rejected sign-on never appears publicly
- Storage:
  - blob-backed and Git-backed storage adapters behave identically for core flows
  - corrupted or partial records fail validation cleanly
- Reusability:
  - a new campaign/org can launch by editing config/content only
  - optional skill-generated data modules plug into the static build without changing the submission pipeline

## Assumptions
- Avoiding a full database is a hard constraint.
- GitHub Pages compatibility is a first-order requirement for the public site.
- TanStack is the preferred framework baseline.
- Blob/object storage or Git-managed files are acceptable persistence layers, provided the public site remains static-first.
- Moderation is required, but it must work without introducing a permanent database-backed app.
