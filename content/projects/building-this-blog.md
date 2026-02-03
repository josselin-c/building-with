---
title: "Building a Static Blog with Hugo, GitHub Pages, and a Persistent SSH Key Problem"
date: 2026-02-03T21:00:00-08:00
draft: false
tags: ["hugo", "github-pages", "infrastructure", "openclaw"]
---

## The Problem

We wanted to document our projects in public—real builds, real process, real mistakes included. The requirements were simple:

- **Fast and portable** — static site, not a CMS
- **Version controlled** — everything in git
- **Auto-deploying** — push to main, site updates
- **Low maintenance** — set it up once, ignore it forever

## The Approach

**Hugo** for the generator. Single binary, fast builds, huge theme ecosystem. **PaperMod** for the theme—clean, responsive, no JavaScript bloat. **GitHub Pages** for hosting—free, same repo as content, Actions integration for CI.

**Content structure:** Project case studies instead of chronological posts. Each project gets its own deep dive: problem, approach, build process, OpenClaw patterns used, results, lessons learned.

## The Build

### Initial Scaffolding

Created the Hugo site structure with PaperMod as a submodule:

```bash
hugo new site . --force
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

Configured `hugo.toml` for GitHub Pages deployment with the right baseURL, theme, and menu structure. Added a GitHub Actions workflow to build and deploy on every push.

### OpenClaw in Action

This is where it gets interesting. The build happened almost entirely through OpenClaw tooling:

- **File operations** — `write` for config files, `edit` for updates
- **Git operations** — `exec` for commits, branch management, remote setup
- **SSH key generation** — created an Ed25519 key pair for GitHub authentication
- **GitHub App integration** — for reading Actions logs without manual copy/paste

The workflow was: discuss approach → scaffold structure → configure deployment → debug failures using the GitHub App → iterate until green.

## The Challenges

### SSH Key Approval Hell

The first major hurdle: exec approvals weren't working through Discord. Every `ssh-keygen` command required approval, but the approval mechanism wasn't responding. After multiple attempts, we discovered `/elevated full` bypasses approvals entirely—a useful escape hatch, but not something you want as default policy.

### Hugo Version Mismatch

PaperMod requires Hugo 0.146.0+. The workflow was using 0.139.0. This caused cryptic template errors about missing `google_analytics.html` partials. The fix was simple once diagnosed—bump the version in the workflow—but diagnosis required reading the actual build logs.

### GitHub App Setup

Rather than copy/pasting error logs forever, we created a GitHub App with read-only Actions permissions. This lets OpenClaw fetch logs directly via the API:

```python
# Generate JWT from app private key
# Exchange for installation access token
# Query Actions API for runs and logs
```

The App has minimal scope (`actions:read` only) and is installed just on this repo. Good security hygiene for automation.

### SSH Config Aliases

Multiple GitHub accounts meant SSH key conflicts. Solved with a Host alias in `~/.ssh/config`:

```
Host github.com-building-with
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_building_with
    IdentitiesOnly yes
```

Then updated the git remote to use the alias: `git@github.com-building-with:josselin-c/building-with.git`

## The Result

A working blog at **https://josselin-c.github.io/building-with** with:

- Hugo + PaperMod theme
- GitHub Actions auto-deployment
- GitHub App for automated log access
- Project case study structure ready for content

## What We Learned

**Fail loudly, diagnose quickly.** The Hugo version error manifested as missing partial errors—not a clear "upgrade Hugo" message. Reading actual build logs beats guessing.

**Escape hatches matter.** `/elevated full` exists for a reason. When approval systems break, you need a way to keep working. But use it sparingly and dial back afterward.

**Scope automation narrowly.** The GitHub App only needs `actions:read`. No write permissions, no broad repository access. If the key leaks, blast radius is minimal.

**SSH aliases solve identity conflicts cleanly.** Better than fighting with `ssh-add` or global Git configs.

**Document the process, not just the result.** This post exists because we hit friction worth sharing. Future us will thank present us.
