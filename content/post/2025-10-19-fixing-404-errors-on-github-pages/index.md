+++
title = "Fixing 404 Errors on Zola GitHub Pages Deployment"
date = 2025-10-19

[extra]
promo_image = "promo-image-11.jpg"

[taxonomies]
tags = ["test", "pagination"]
+++

# Fixing 404 Errors on Zola GitHub Pages Deployment

If you've just deployed your Zola static site to GitHub Pages and are greeted with a frustrating 404 error, you're not alone. This is one of the most common issues newcomers face when deploying Zola sites. Let me walk you through the solution.

## The Problem

You run `zola build`, commit everything to your repository, configure GitHub Pages to deploy from the `main` branch, and... 404. Your site is nowhere to be found.

## Why This Happens

The root cause is a mismatch between where Zola builds your site and where GitHub Pages is looking for it:

- **Zola builds to**: `public/` directory
- **GitHub Pages expects**: Files in the root of your chosen branch

When you set GitHub Pages to deploy from the `main` branch root, it's looking for `index.html` at the repository root, not inside the `public/` folder.

## The Solution: GitHub Actions

The cleanest solution is to use GitHub Actions to automatically build and deploy your site. Here's how:

### Step 1: Create the Workflow File

Create `.github/workflows/deploy.yml` in your repository:

```yaml
name: Deploy Zola site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true
      
      - name: Install Zola
        uses: taiki-e/install-action@v2
        with:
          tool: zola@0.21.0  # Match your local version
      
      - name: Build
        run: zola build
      
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

**Important**: Replace `zola@0.21.0` with your actual Zola version. Check it with:

```bash
zola --version
```

### Step 2: Configure GitHub Pages

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Pages**
3. Under **Source**, select **"GitHub Actions"**
4. Save the changes

### Step 3: Push and Deploy

Commit and push your workflow file:

```bash
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions deployment workflow"
git push
```

Watch the magic happen in the **Actions** tab of your repository. Within a minute or two, your site should be live!

## Additional Checklist

If you're still seeing 404 errors after setting up GitHub Actions, verify these settings:

### 1. Base URL in config.toml

For project sites (not username.github.io), your `config.toml` should have:

```toml
base_url = "https://yourusername.github.io/your-repo-name"
```

**Note**: No trailing slash!

### 2. Theme Submodules

If you're using a theme as a Git submodule, make sure your workflow includes:

```yaml
- uses: actions/checkout@v4
  with:
    submodules: true
```

This ensures the theme is properly checked out during the build process.

### 3. Check the Actions Log

If the deployment fails, click on the failed workflow run in the Actions tab to see detailed error logs. Common issues include:

- Missing dependencies
- Syntax errors in templates
- Invalid frontmatter in content files
- Incorrect theme configuration

## Alternative: Manual gh-pages Branch

If you prefer manual control, you can push the `public/` folder to a separate `gh-pages` branch:

```bash
# Build your site
zola build

# Navigate to public folder
cd public

# Initialize git and push to gh-pages
git init
git add .
git commit -m "Deploy site"
git push -f git@github.com:yourusername/your-repo-name.git main:gh-pages
```

Then in GitHub Pages settings, select the `gh-pages` branch as your source.

However, GitHub Actions is far more convenient for regular updates.

## Conclusion

The 404 error on Zola GitHub Pages deployments is almost always about the mismatch between build output location and deployment source. By using GitHub Actions, you automate the entire build and deployment process, eliminating this class of errors entirely.

Now get back to writing great content for your site!