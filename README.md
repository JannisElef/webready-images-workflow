# GitHub Webready Images

Reusable GitHub Actions workflow that converts original images from `assets/img` into webready JPEG copies and stores them in `assets/img/web`.

The workflow is designed to be used centrally from one repository and called from many other repositories with only a tiny local workflow file.

## What it does

- Watches for changes in `assets/img`
- Converts supported source images into JPEG
- Uses a `-web` suffix for generated files
- Writes generated images into `assets/img/web`
- Preserves subfolder structure
- Uses JPEG quality `80` by default
- Skips unchanged files
- Removes stale generated files if the original image was deleted
- Can be triggered manually for the initial run

### Example

Source:

```text
assets/img/hero.png
assets/img/blog/header/cover.webp
```

Generated output:

```text
assets/img/web/hero-web.jpg
assets/img/web/blog/header/cover-web.jpg
```

## Why use this repo

Instead of copying image conversion logic into every repository, you keep the workflow here once and reuse it everywhere.

This means:

- only **one tiny workflow file** in each consumer repository
- central maintenance
- same behavior across all repositories
- easy updates by changing just the version tag or SHA

## Repository structure

```text
.github/
  workflows/
    webready-images.yml
README.md
```

## Requirements for consumer repositories

Your repository should contain original images in:

```text
assets/img
```

Generated files will be written to:

```text
assets/img/web
```

If your repository uses a different structure, you can override the paths through workflow inputs.

## How it works

The central workflow is a reusable workflow based on `workflow_call`.

Consumer repositories add a very small workflow file that:

- triggers on `push` to `assets/img/**`
- excludes `assets/img/web/**`
- allows manual execution with `workflow_dispatch`
- calls this central workflow

The central workflow then:

- checks out the consumer repository
- installs `sharp`
- generates webready JPEG copies
- commits and pushes the generated files back if needed

## Quick start

### 1. Create and publish this central repository

Create a repository, for example:

```text
your-user-or-org/github-webready-images
```

