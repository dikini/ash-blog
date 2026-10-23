# Writing repository implementation plan

**Goal:** Establish a content-only workspace for Ash developer diaries.

**Architecture:** Keep topic notes and article folders in `ash-blog`. Let
`ash-site` import a selected committed revision and own presentation and
publication. Local checkout preview is a future site integration.

**Tech stack:** Markdown, YAML front matter, and Git.

## Approved design

Separate notes from articles. Keep each article and its assets together under a
stable slug. Use small authoring metadata; let the site derive its section and
route. Draft commits do not publish content. Publication selects an explicit
source revision.

## Setup checklist

1. Create the repository layout, README, and agent guidance.
2. Add a reusable article template with safe draft defaults and unset dates.
3. Document writing conventions, metadata, and the future publication boundary.
4. Verify paths, template metadata, and whitespace; initialise Git and commit.

## Deferred integration

Implement importing, validation, asset and link handling, local site preview,
and source revision recording in `ash-site` as a separate task.
