# Project Setup and Troubleshooting Summary (January 27, 2026)

This document summarizes the steps taken to clone the "Hux Blog" theme and deploy it as a GitHub Pages website. It also details the numerous issues encountered and their solutions, serving as a guide for future projects.

## 1. Initial Setup

- **Goal:** Clone the `guilyx/guilyx.github.io` repository and set it up as a new GitHub Pages site.
- **Action:**
    1. Cloned the repository into the local project folder.
    2. Removed the original `.git` directory.
    3. Initialized a new Git repository.
    4. Created a new repository on GitHub (`Directly0391/website`).
    5. Pushed the initial commit to the new repository.

## 2. Problem: Unstyled Website on GitHub Pages

The initial deployment resulted in a plain white page with unformatted text.

- **Root Cause:** The theme is a Jekyll site, which requires a build process to generate the final HTML, CSS, and JavaScript. The default GitHub Pages deployment was not building the site correctly.

## 3. Troubleshooting the Local Environment

To debug the build process, we first set up a local development environment.

- **Problem:** Missing dependencies (Ruby and Bundler).
- **Solution:** Installed Ruby and Bundler using RubyInstaller for Windows.

- **Problem:** `bundle install` command failed to run correctly in PowerShell, even with `Gemfile` present.
- **Solution:** We had to use the full, explicit path to the `bundle.bat` executable (`C:\Ruby40-x64\bin\bundle.bat install`) to get it to run. This was a persistent issue related to the local shell environment.

## 4. Troubleshooting the GitHub Actions Workflow

After getting the site to build locally, we switched to a GitHub Actions workflow for deployment to gain more control over the build environment. This led to a series of new errors.

### 4.1. Build Failure: Platform Mismatch

- **Error:** `Your bundle only supports platforms ["x64-mingw-ucrt"] but your local platform is x86_64-linux.`
- **Root Cause:** The `Gemfile.lock` was created on a Windows machine and was locked to the Windows platform. The GitHub Actions runner uses Linux.
- **Solution:** Added the Linux platform to the lockfile by running `bundle lock --add-platform x86_64-linux`.

### 4.2. Build Failure: Ruby Version Incompatibility

- **Error:** `public_suffix-7.0.2 requires ruby version >= 3.2, which is incompatible with the current version, 3.1.7`
- **Root Cause:** A gem in our bundle required a newer version of Ruby than the one specified in the GitHub Actions workflow.
- **Solution:** Updated the Ruby version in `jekyll.yml` from `3.1` to `3.2`.

### 4.3. Build Failure: `rdiscount` Native Extension

- **Error:** `An error occurred while installing rdiscount...`
- **Root Cause:** The `github-pages` gem had a dependency on `rdiscount`, which requires native C extensions and often fails to build on Windows and in some Linux environments.
- **Solution:** Removed the generic `github-pages` gem from the `Gemfile` and replaced it with a specific list of compatible gems (`jekyll`, `jekyll-paginate`, `kramdown`, etc.).

### 4.4. Build Failure: Missing `kramdown-parser-gfm`

- **Error:** `Yikes! It looks like you don't have kramdown-parser-gfm or one of its dependencies installed.`
- **Root Cause:** The site was configured to use GitHub Flavored Markdown (GFM), which requires the `kramdown-parser-gfm` gem, but it was not in our `Gemfile`.
- **Solution:** Added `gem 'kramdown-parser-gfm'` to the `Gemfile`.

### 4.5. Build Failure: `vendor/` Directory Not Excluded

- **Error:** `could not read file ... welcome-to-jekyll.markdown.erb: Invalid date`
- **Root Cause:** The Jekyll build process was trying to build files from inside the `vendor/` directory where gems are installed.
- **Solution:** Added `vendor/` to the `exclude` list in `_config.yml`.

### 4.6. Deploy Failure: Missing `id-token` Permission

- **Error:** `Error: Ensure GITHUB_TOKEN has permission "id-token: write".`
- **Root Cause:** The GitHub Actions workflow did not have the necessary permissions to authenticate with GitHub Pages.
- **Solution:** Added a `permissions` block with `id-token: write` to the `jekyll.yml` workflow file.

## 5. Final Problem: Incorrect Asset Paths (404 Errors)

Even with a successful build and deploy, the live site was still unstyled.

- **Root Cause:** The browser console showed `404 Not Found` errors for all CSS, JavaScript, and image files. This was because the paths to these assets were incorrect on the live site. We had to fix this in multiple places:
    1. The `url` and `baseurl` in `_config.yml` were incorrect for the user's repository name.
    2. The GitHub Actions build command was not correctly applying the `baseurl`.
    3. The theme's templates had hardcoded paths or formatting errors that were not correctly prepending the `baseurl`.
- **Solution:**
    1. Corrected the `url` and `baseurl` in `_config.yml`.
    2. Hardcoded the `--baseurl "/website"` in the `jekyll build` command in the `jekyll.yml` workflow file for maximum certainty.
    3. Manually fixed incorrect asset paths in `_includes/head.html`, `_includes/footer.html`, and `_layouts/default.html` to ensure they correctly used `{{ site.baseurl }}`.

## Key Takeaways for Future Projects

- **Jekyll requires a build step.** A plain HTML site will work on GitHub Pages out of the box, but Jekyll sites need to be built.
- **Local environment setup is critical.** Ensure Ruby and Bundler are installed correctly and that the commands can be run from the project directory.
- **GitHub Actions provide more control.** For complex Jekyll sites, it's better to use a custom GitHub Actions workflow than to rely on the default GitHub Pages build process.
- **`Gemfile.lock` needs to be cross-platform.** When working on Windows and deploying to Linux, always add the Linux platform to your `Gemfile.lock`.
- **Check your asset paths (`baseurl`).** This is the most common cause of "unstyled site" issues after a successful deployment. Ensure `_config.yml` is correct and that all asset links in your templates use `{{ site.baseurl }}`.
