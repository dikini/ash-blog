# Ash developer diaries

This repository holds writing and article assets. `ash-site` owns rendering,
navigation, search, routes, and deployment.

## Working here

- Read `docs/authoring.md` before editing articles or their metadata.
- Keep loose working material in `notes/` and articles in
  `entries/<slug>/index.md`. Keep article assets beside the article.
- Do not invent experiences, results, dates, author names, or language guarantees.
  Ask for missing facts or leave an explicit draft placeholder.
- Preserve the author's personal voice. Explain uncertainty and distinguish
  experiments from implemented or specified behaviour.
- Keep templates and new entries marked `draft: true`. Do not change publication
  state or dates unless the user has authorised publication preparation.
- Never add site-specific layout, generated HTML, dependencies, or deployment
  tooling as part of ordinary writing work.
- Relative article links should target source Markdown; relative image links
  should target assets within the article folder. The future importer must
  translate these into website URLs.
- Before committing, spell-check changed prose, including titles and summaries,
  run `git diff --check`, inspect changed metadata, and check that local links
  and asset paths resolve. Follow `docs/authoring.md` for the spelling pass.
  There is no automated test suite.
