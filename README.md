# ScipionWeb Documentation

This repository contains the MkDocs source for the ScipionWeb documentation.

## Local development

Install dependencies:

    pip install -r requirements.txt

Run the local development server:

    mkdocs serve

Open:

    http://127.0.0.1:8000/

## Build

Build the site locally with strict validation:

    mkdocs build --strict

The generated site is written to the `site/` directory.

## Deployment

The GitHub Actions workflow builds the documentation and deploys it to GitHub Pages.

## Editing guidelines

- Keep pages concise and task-oriented.
- Prefer concrete commands and verification steps.
- Keep installation, configuration, operations, and user workflows clearly separated.
- Add recurring problems to `docs/support/known-issues.md`.
- Run `mkdocs build --strict` before merging documentation changes.