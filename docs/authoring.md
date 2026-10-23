# Writing and publication

## Notes and articles

Keep early material in `notes/`, grouped by subject. A note can hold questions,
links, code fragments, and partial explanations. Move material into
`entries/<slug>/index.md` when it becomes an article. A slug names the subject;
use lowercase words separated by hyphens. Avoid renaming it after publication.

Keep images beside the article and use relative paths, for example
`![Diagnostic showing the source location](diagnostic.png)`. Provide useful alt
text. Link another article as `../other-slug/index.md`. Website URL rewriting
belongs to the future site importer.

Topics belong in metadata rather than directory hierarchies. Entries can share
a topic or form a series without changing their file paths.

## Metadata

```yaml
title: "Making errors readable"
description: "A short summary of the problem and what the entry explores."
author: "Your name"
draft: true
published: null
updated: null
topics: []
```

- `title`, `description`, and `author` describe the article. Replace placeholders
  before marking it ready.
- `draft` defaults to `true` in the starter. Only explicit `false` should make an
  entry eligible for publication.
- `published` is the intended first publication date, quoted as `YYYY-MM-DD`.
  Leave it `null` while drafting. Set the actual date when preparing publication;
  keep it unchanged during later edits.
- `updated` is the latest substantive revision date, quoted as `YYYY-MM-DD`.
  Leave it `null` while drafting. For a publishable entry it must be present and
  at or after `published`. Updating it must not move the entry up the diary index.
- `topics` is an optional list of subjects. It does not yet create website tags
  or topic pages.

These are authoring conventions for now; this repository has no metadata
validator. They describe the contract the site importer should enforce.

## Voice

Write in the first person when describing your own work. Start with a concrete
problem, observation, or question. Describe what you tried, the evidence you
found, and what remains uncertain. An unfinished investigation is a valid entry.
The starter's prompts are optional; remove them or replace the structure as the
article develops.

Use precise terms, strong verbs, sentence case, and descriptive links. Avoid
promotional claims. Link relevant code, commits, specifications, or experiments.
Identify the revision when a claim depends on a particular language checkpoint.
Distinguish an idea, an experiment, and implemented behaviour.

For public headings and summaries, follow the Ash site's voice guide at
`ash-site/docs/site-voice.md`. Personal narrative can retain the author's rhythm;
it does not need to read like a website landing page.

## Spelling review

Spell-check new or changed prose after drafting or editing and before committing.
Include titles, descriptions, headings, captions, and image alt text. Use British
English, consistent with the site voice guide.

Correct misspellings, accidental spacing, and obvious capitalisation or
hyphenation errors. Preserve the author's phrasing, rhythm, deliberate fragments,
and rhetorical exaggeration. Keep broader grammar and style suggestions separate
from spelling corrections.

Review suggestions in context: language names, technical terms, identifiers, code,
and URLs must not be changed merely because a dictionary does not recognise them.
A manual spelling pass is part of this workflow; no automated spell checker is
configured.

## Intended publication handoff

1. Write and review the article here. A Markdown editor provides basic preview.
2. When the site integration exists, preview the local checkout through
   `ash-site`, including drafts only in an explicit local preview mode.
3. Before publication, fill in real metadata, set the publication and update
   dates, remove template prompts, check links, and set `draft: false`.
4. Commit the reviewed content. Select its full Git revision in the site's
   source configuration.
5. Build and verify `ash-site`, then use its explicit publication workflow.

The future importer should read only `entries/*/index.md`, exclude drafts from
public output, validate metadata and slugs, copy article assets, rewrite local
links, and reject broken links or route collisions. It should supply the site's
`Developer Diaries` section and derive `diaries/<slug>` routes. It should record
the resolved source revision in the build manifest and support a local checkout
for authoring preview without mistaking that preview for a pinned publication.

`notes/`, `templates/`, and repository documentation are never publication inputs.
The website integration is a separate change in `ash-site`; none of the handoff
steps fetch, build, or deploy from this repository today.
