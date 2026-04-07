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

Add the reusable workflow file:

```text
.github/workflows/webready-images.yml
```

Commit and push it.

Then create a release tag:

```bash
git add .
git commit -m "feat: add reusable webready image workflow"
git push
git tag -a v1 -m "v1"
git push origin v1
```

You can later create new tags like `v2`, `v3`, etc.

### 2. Add one small workflow file to a consumer repository

In the repository where you want to use this automation, create:

```text
.github/workflows/webready-images.yml
```

Use this content:

```yaml
name: Webready Images

on:
  workflow_dispatch:
  push:
    paths:
      - "assets/img/**"
      - "assets/img/web/**"

permissions:
  contents: write

jobs:
  webready:
    uses: your-user-or-org/github-webready-images/.github/workflows/webready-images.yml@v1
    with:
      source-dir: assets/img
      output-dir: assets/img/web
      quality: 80
      suffix: -web
```

Replace:

- `your-user-or-org`
- `github-webready-images`

with your actual repository path.

### 3. Run it once manually

Open the repository on GitHub:

```text
Actions → Webready Images → Run workflow
```

This will generate the first batch of webready images.

### 4. Done

After that, every push that changes files in `assets/img` will automatically update the generated files in `assets/img/web`.

## Inputs

The reusable workflow supports the following inputs:

### `source-dir`

Path to the source images directory.

Default:

```text
assets/img
```

### `output-dir`

Path where generated JPEG files will be stored.

Default:

```text
assets/img/web
```

### `quality`

JPEG quality from `1` to `100`.

Default:

```text
80
```

### `suffix`

Suffix added before `.jpg`.

Default:

```text
-web
```

## Minimal consumer setup

If you want the least possible work in each repository, this is all you need:

```yaml
name: Webready Images

on:
  workflow_dispatch:
  push:
    paths:
      - "assets/img/**"
      - "assets/img/web/**"

permissions:
  contents: write

jobs:
  webready:
    uses: your-user-or-org/github-webready-images/.github/workflows/webready-images.yml@v1
```

This works as long as the consumer repository uses the default folder structure:

```text
assets/img
assets/img/web
```

## Advanced examples

### Custom source and output folders

```yaml
name: Webready Images

on:
  workflow_dispatch:
  push:
    paths:
      - "public/images/**"
      - "public/images/web/**"

permissions:
  contents: write

jobs:
  webready:
    uses: your-user-or-org/github-webready-images/.github/workflows/webready-images.yml@v1
    with:
      source-dir: public/images
      output-dir: public/images/web
      quality: 80
      suffix: -web
```

### Different suffix

```yaml
jobs:
  webready:
    uses: your-user-or-org/github-webready-images/.github/workflows/webready-images.yml@v1
    with:
      suffix: -optimized
```

This would generate:

```text
hero.png → hero-optimized.jpg
```

### Different JPEG quality

```yaml
jobs:
  webready:
    uses: your-user-or-org/github-webready-images/.github/workflows/webready-images.yml@v1
    with:
      quality: 70
```

## Recommended versioning

For convenience, reference a version tag:

```yaml
uses: your-user-or-org/github-webready-images/.github/workflows/webready-images.yml@v1
```

For maximum stability, reference a full commit SHA instead:

```yaml
uses: your-user-or-org/github-webready-images/.github/workflows/webready-images.yml@<commit-sha>
```

## Permissions

Consumer repositories should include:

```yaml
permissions:
  contents: write
```

This is required because the workflow commits generated files back into the repository.

## Supported image input formats

The workflow is intended to process common raster image formats such as:

- `.png`
- `.jpg`
- `.jpeg`
- `.webp`
- `.avif`
- `.tif`
- `.tiff`
- `.gif`

Generated files are always written as:

```text
.jpg
```

## Notes

### Transparency

If a source image has transparency, it will be flattened onto a white background because JPEG does not support alpha transparency.

### Folder structure is preserved

If your source image is located in a nested folder, the generated file will keep the same relative structure under `assets/img/web`.

Example:

```text
assets/img/posts/2026/banner.png
→ assets/img/web/posts/2026/banner-web.jpg
```

### Original files are never changed

Only new generated files are created in the output folder.

## Troubleshooting

### Nothing happens

Check that your consumer repository has images inside the configured source directory.

Default:

```text
assets/img
```

### Workflow runs but no files are committed

Possible reasons:

- there were no supported source images
- generated files were already up to date
- output path does not match the configured folder
- repository permissions do not allow writing contents

### The workflow does not start on push

Make sure your consumer workflow uses the correct `paths` filter and that your changed files are inside the configured source directory.

### Generated files are committed but workflow does not rerun

That is expected. The generated commit should not recursively trigger the same automation again.

## Suggested setup for many repositories

If you want to roll this out across multiple repositories:

1. Create this central repository once
2. Tag it with `v1`
3. Add the tiny caller workflow into each repository
4. Use the same defaults everywhere
5. Upgrade later by changing only the tag or SHA in each consumer repository

## Example consumer repository layout

```text
.github/
  workflows/
    webready-images.yml
assets/
  img/
    logo.png
    blog/
      header.png
```

After the workflow runs:

```text
.github/
  workflows/
    webready-images.yml
assets/
  img/
    logo.png
    blog/
      header.png
    web/
      logo-web.jpg
      blog/
        header-web.jpg
```