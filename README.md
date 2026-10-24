# ACE3P GUI user documentation

The user guide published at <https://mothmanslac.github.io/ace3p-gui-docs/>.
It is a [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) site.

```
mkdocs.yml          site configuration and navigation
requirements.txt    MkDocs packages
docs/               the pages (Markdown)
docs/images/        screenshots
```

## Preview locally

```bash
python -m pip install -r requirements.txt
mkdocs serve --dev-addr 127.0.0.1:8100
```

Open <http://127.0.0.1:8100>. Pages reload on save.

Check the site the way the publishing workflow does; it fails on broken links
and missing images:

```bash
mkdocs build --strict
```

## Publish

`.github/workflows/docs.yml` builds the site and deploys it to GitHub Pages on
every push to `main`. It can also be started by hand from the **Actions** tab.

One-time setup: **Settings → Pages → Build and deployment → Source: GitHub
Actions**.

## Writing rules

- Write direct instructions: "Press **Run Simulation**", not "you might want
  to consider pressing".
- Name controls exactly as the app labels them, in bold.
- One task per numbered list. One action per step.
- Put a screenshot after the step it illustrates. Store it in `docs/images/`
  as PNG, numbered in reading order.
- Update the version in `docs/index.md` and `docs/reference/settings.md` when
  screenshots are retaken from a new release.
- A new page must be added to `nav` in `mkdocs.yml`.
