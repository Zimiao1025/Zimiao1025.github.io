# Writing a new blog post

1. Copy `_drafts/post-template.md` into `_posts/`.
2. Rename it using `YYYY-MM-DD-short-title.md`, for example `2026-11-08-protein-dynamics.md`.
3. Update the metadata at the top and write the article in Markdown.
4. Commit and push. GitHub Pages will rebuild the site automatically.

Posts are published newest first at `/blog/`. Keep unfinished work in `_drafts/`; GitHub Pages will not publish drafts.

## Local preview

If Ruby and Bundler are installed, run the following from this directory:

```bash
bundle exec jekyll serve
```

The design does not require a database or JavaScript framework.

