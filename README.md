# Building With

Building in public. Documenting real projects built with OpenClaw.

## Structure

- `content/projects/` - Case studies of each project
- `content/posts/` - Shorter thoughts, tips, discoveries
- `archetypes/projects.md` - Template for new project posts
- `static/` - Images, assets

## Local Development

### Install Hugo

```bash
# macOS
brew install hugo

# Ubuntu/Debian
sudo apt-get install hugo

# Or download from https://github.com/gohugoio/hugo/releases
```

### Run locally

```bash
cd /home/ash/projects/openclaw-blog
hugo server -D
```

Visit http://localhost:1313

## Creating Content

### New project case study:

```bash
hugo new content projects/my-project.md
```

Then fill in the sections: Problem, Approach, Build, OpenClaw patterns, Results, Lessons.

## Deploying

Push to `main` branch. GitHub Actions automatically builds and deploys to GitHub Pages at:

`https://josselin-c.github.io/building-with`

## First Time Setup (GitHub)

1. Create a new repo on GitHub named `building-with`
2. Enable GitHub Pages in repo settings (Source: GitHub Actions)
3. Push this repo:

```bash
git branch -m main
git remote add origin https://github.com/josselin-c/building-with.git
git add .
git commit -m "Initial commit: Hugo + PaperMod"
git push -u origin main
```

4. GitHub Actions will deploy automatically

