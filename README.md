# omarmaali.github.io

Personal site. Static, built by Jekyll, hosted free on GitHub Pages.

Deployment steps are in **[DEPLOY.md](DEPLOY.md)**. This file is about editing it once
it is live.

---

## Settings

Contact links and the site title live in `_config.yml`. They are filled in and correct
as of the first push; change them there and nowhere else.

---

## Where everything lives

```
_config.yml           Site title, URL, contact links. Most edits start here.
index.html            The home page: hero, expertise, work, writing, background, contact.
writing.html          The blog index at /writing/
_data/projects.yml    The six project cards. Edit content here, not in index.html.
_posts/               Published blog posts. One markdown file each.
_drafts/              Unpublished drafts. Ignored by the build.
_layouts/             Page shells: default.html wraps everything, post.html wraps articles.
_includes/footer.html Footer plus the script that assembles the email address.
assets/css/style.css  All the styling. One file.
assets/img/           Project diagrams. <name>-card.webp on the card, <name>.webp full size.
assets/fonts/         Inter, self-hosted, latin subset only.
404.html              Not-found page.
```

## Writing a post

Create a file in `_posts/` named `YYYY-MM-DD-some-slug.md`:

```markdown
---
layout: post
title: "The title, in quotes"
date: 2026-09-02
reading_time: "6 min read"
summary: >-
  Two or three lines. This shows on the blog index and under the title,
  and it is what search engines and LinkedIn previews use.
---

Your first paragraph. Plain markdown from here down.

## A subheading

More text. `Inline code` works, and so do fenced code blocks, lists,
> blockquotes,

and links.
```

Commit and push. The post is live in about a minute at
`/writing/some-slug/`, appears on the blog index and the home page, and goes
into the RSS feed automatically.

Two drafts are already in `_drafts/` with outlines. To publish one, move it into
`_posts/` and add a date to the front of the filename.

## Editing a project card

Everything is in `_data/projects.yml`. Each entry takes a `title`, a one-line `kicker`,
the `image` file stem, an `alt` description, a list of `points`, an optional `note`, a
list of `links`, and a list of `tags`. HTML is allowed inside `points`, so `<b>` and
`<code>` work.

To add a diagram: drop `something.webp` and `something-card.webp` in `assets/img/` and
set `image: something`.

## Previewing locally (optional)

You do not need this. GitHub builds the site for you. But if you want to see changes
before pushing:

```bash
gem install bundler jekyll
bundle install
bundle exec jekyll serve
```

Then open <http://127.0.0.1:4000>.

## What this site deliberately does not do

It is a portfolio and a place to publish, not a sales page. No rates, no availability
signals, no marketplace links, no employer names. The call to action is a conversation.
Keep it that way unless you decide otherwise on purpose.
