# Ash developer diaries

Working notes, drafts, and articles about developing Ash. Write here; publish
selected articles through `ash-site`.

## Layout

- `notes/` — loose ideas, observations, and research organised by topic.
- `entries/<slug>/index.md` — article drafts and finished entries.
- `entries/<slug>/` — images and other assets belonging to that article.
- `templates/entry/` — starter folder for a new article.
- `docs/authoring.md` — writing, metadata, and publication workflow.

## Start writing

From this repository, copy the starter to a descriptive, lowercase slug:

```sh
cp -R templates/entry entries/making-errors-readable
```

Edit `entries/making-errors-readable/index.md`. Replace the title and summary,
set the author, and write. Keep `draft: true` while the article is in progress.
Use a Markdown editor's preview for prose and images.

Use `notes/` for material that does not yet have an article shape. Notes and
article drafts are not private: anyone with repository access can read them.

## Publication

This repository has no website build or deployment. The intended integration
lets `ash-site` preview a local checkout and publish from an explicitly selected
committed revision. That importer and site preview are not implemented yet.

Until then, write and commit here. Changing `draft` to `false` marks an entry as
ready for publication; it does not deploy anything. See
[the authoring guide](docs/authoring.md) for the metadata contract and handoff.
