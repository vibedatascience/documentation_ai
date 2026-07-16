---
name: vibelogs
description: How to add and edit posts on Rahul's vibelogs site (GitHub Pages blog of AI walkthroughs, repo vibedatascience/documentation_ai, may be renamed to vibedatascience/vibelogs). Use this skill whenever asked to add a post, page, or rule to vibelogs / the blog / documentation_ai, or to deslop its content.
---

# vibelogs skill

Personal blog of AI walkthroughs and reference pages. Live at
https://vibedatascience.github.io/documentation_ai/ (becomes /vibelogs/ if the repo is renamed; GitHub redirects the old URL either way).

## Repo access

Clone with the GitHub PAT from the user's custom instructions (fine-grained, CONTENTS READ/WRITE ONLY):

```
git clone https://x-access-token:<PAT>@github.com/vibedatascience/documentation_ai.git
```

The token CANNOT rename repos, create repos, enable Pages, or call most non-contents API endpoints (403). Pages deploys automatically on push to main via Actions.

## Structure

```
index.html                 homepage: log-entry list + tag filter
about.html
posts/<slug>/index.html    one folder per post, assets live next to index.html
  dated slug (2026-07-16-topic) for walkthrough posts
  bare slug (ai-slop-detection) for living reference pages (stable URL)
```

There is no build step and no shared CSS file. Every page carries its own inline <style> block. When creating a new post, copy the <style> block from an existing post verbatim.

## Design system (do not deviate)

- Palette: paper #FAFAF8, ink #17171C, cobalt #2545FF, muted #6E6E76, rule #E7E7E2, chip #EEF1FF
- Fonts: Space Grotesk (display), IBM Plex Mono (metadata/labels), Inter (body)
- Post metadata line: [YYYY-MM-DD] + author byline "Rahul" + tag chip. Every post page also carries <meta name="author" content="Rahul">.
- Tags: AGENTS, TOOLING, LLM-APPS, DATA-SCIENCE, REFERENCE. New tag = add a filter button on index.html
- Components available in post CSS: pre/code blocks, .gotcha callouts (GOTCHA · LABEL), .pipeline ASCII diagrams, tables

## Adding a post

1. Create posts/<slug>/index.html by copying an existing post and replacing title, meta line, and <article> body.
2. Add one entry block to index.html under the right month heading (newest first):

```html
<a class="entry" data-tags="<tag-lowercase>" href="posts/<slug>/">
  <div class="meta"><span>[YYYY-MM-DD]</span><span class="tag">TAG</span></div>
(inside the post page itself the meta line is: <span>[YYYY-MM-DD]</span><span>Rahul</span><span class="upd">updated YYYY-MM-DD</span><span class="tag">TAG</span>. The first date is the publish date and never changes; bump the "updated" date to today on EVERY content edit to that post.)
  <h2>Title</h2>
  <p>One-line description.</p>
</a>
```

3. Commit, push to main.
4. Verify live: sleep ~60s, then curl the page and grep for new content. Pages can take 1-2 min; a 404 or stale content right after push is normal. If a push happened while Actions was disabled, push an empty commit to trigger deploy.

## Writing rules (MANDATORY)

All content must pass the site's own reference page: posts/ai-slop-detection/index.html. Read it before writing. Highlights the user enforces hard:

- No em dashes or en dashes ANYWHERE, including <title> tags. Use commas, colons, periods, or " -- ".
- No "It's not X, it's Y" constructions.
- No sentence fragments; full sentences always, except inside bullet points.
- No parallel-structure stat ledes.
- No AI vocabulary (delve, leverage, seamless, robust, ...) or ad-speak.
- Minimal language throughout. The user writes tersely and wants the site to read that way.

After editing, run: grep -rc "—\|–" on all html files and expect 0.

## User rules for this repo

- Add ONLY what the user dictates. Do not embellish reference pages with extra items unless asked; when the user shares a source and says "add", confirm which items if ambiguous.
- Never publish internal Pinterest details: no pinadmin.com hosts, internal model names, or proxy call formats. Genericize as "internal LLM proxy".
- Redact API keys in published code samples (YOUR_KEY).
- The user's GitHub PAT, RapidAPI key, and other credentials are in their custom instructions; never commit them.
