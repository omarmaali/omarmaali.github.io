# Deploying this site

Two parts. Part 1 gets you live at `omarmaali.github.io`, which is free and takes about
fifteen minutes. Part 2 is the custom domain, which you can do any time later without
touching the site.

---

## Before you start

Nothing. The contact links in `_config.yml` are filled in and the repository
`omarmaali/omarmaali.github.io` already exists and is empty, so Step 1 below is done
already — skip to Step 2.

---

# Part 1 — Go live on GitHub Pages

## Step 1. Create the repository

The name matters exactly. GitHub treats a repository named `<username>.github.io` as your
**user site** and publishes it at that address automatically. Any other name gets you a
project site at a `/subpath`, which is not what you want.

1. Go to <https://github.com/new>
2. **Repository name:** `omarmaali.github.io`
   (This must match your GitHub username exactly. If your username is not `omarmaali`,
   use whatever it actually is.)
3. **Public.** GitHub Pages needs public for free accounts, and a portfolio site should
   be public anyway.
4. Do **not** tick "Add a README file", "Add .gitignore" or "Choose a license". The repo
   must be empty so your first push isn't fighting an existing commit.
5. Click **Create repository**.

## Step 2. Get the files in

Pick whichever you prefer. Option A needs no tooling.

### Option A — Upload through the browser

1. On the empty repository page, click **uploading an existing file**.
2. Open the unzipped site folder on your machine, select **everything inside it**
   (not the folder itself), and drag it into the browser window.
   Include the folders whose names start with an underscore — `_config.yml`, `_data`,
   `_includes`, `_layouts`, `_posts`, `_drafts`. They are the site.
3. Commit message: `Initial site`
4. Click **Commit changes**.

One catch: Windows Explorer hides files that start with a dot, so `.gitignore` may not
come along. That is harmless — it only affects local previews.

### Option B — Git command line

From inside the unzipped folder:

```bash
git init -b main
git add .
git commit -m "Initial site"
git remote add origin https://github.com/omarmaali/omarmaali.github.io.git
git push -u origin main
```

If it asks for a password, GitHub will not accept your account password. Use a Personal
Access Token instead: **Settings → Developer settings → Personal access tokens → Tokens
(classic) → Generate new token**, tick the `repo` scope, and paste the token as the
password. Or install GitHub Desktop and let it handle authentication.

## Step 3. Check that Pages is switched on

For a `<username>.github.io` repository this usually turns itself on. Verify:

1. In the repository, go to **Settings → Pages**.
2. Under **Build and deployment**, Source should be **Deploy from a branch**, branch
   `main`, folder `/ (root)`.
3. If it isn't, set it and click **Save**.

## Step 4. Watch it build

Click the **Actions** tab. You will see a workflow called `pages build and deployment`
running. It takes one to two minutes.

- **Green tick** — you are live. Open <https://omarmaali.github.io>.
- **Red X** — click into it and read the log. See the troubleshooting list below.

The very first deployment sometimes takes up to ten minutes for DNS to propagate. If you
get a 404 immediately after a green tick, wait five minutes and hard-refresh
(`Ctrl+F5`).

## Step 5. Check it over

On the live site:

- The six diagrams load and open at full size when clicked
- The email button shows your address and opens a mail client
- The LinkedIn and GitHub links go where they should
- `/writing/` lists the post, and the post itself opens
- `/feed.xml` returns XML rather than a 404
- Open it on your phone

## Publishing changes afterwards

Edit, commit, push. That is the whole loop. GitHub rebuilds automatically within a
minute or two. You can also edit any file directly on github.com with the pencil icon,
which is often quicker for fixing a typo.

---

# Part 2 — Custom domain, later

Nothing about the site needs to change. You buy a domain, point it at GitHub, and tell
GitHub about it. The `omarmaali.github.io` address keeps working and redirects.

## Step 1. Buy a domain

Any registrar. Cloudflare and Namecheap are both fine and cheap. Something like
`omarmaali.com` or `omarhussein.dev`.

Keep it consistent with the brand name on the site. That was the point of picking one
name.

## Step 2. Add the DNS records

This is at your registrar, not on GitHub.

**For the apex domain** (`omarmaali.com`, no `www`), add four A records. Same host, four
different values:

| Type | Host | Value |
|---|---|---|
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |

And the four AAAA records for IPv6, which GitHub recommends alongside them:

| Type | Host | Value |
|---|---|---|
| AAAA | `@` | `2606:50c0:8000::153` |
| AAAA | `@` | `2606:50c0:8001::153` |
| AAAA | `@` | `2606:50c0:8002::153` |
| AAAA | `@` | `2606:50c0:8003::153` |

**For `www`**, one CNAME record:

| Type | Host | Value |
|---|---|---|
| CNAME | `www` | `omarmaali.github.io` |

Note the trailing dot some registrars add automatically — that is normal.

If you use Cloudflare, set the proxy status to **DNS only** (grey cloud) for these
records at first. The orange-cloud proxy interferes with GitHub issuing your HTTPS
certificate. You can turn it on afterwards if you want.

These IP addresses are current as of August 2026. Before you paste them, it is worth
thirty seconds to confirm against
<https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site>,
because GitHub has changed them before.

## Step 3. Tell GitHub

1. **Settings → Pages → Custom domain**
2. Type your domain and click **Save**. GitHub adds a `CNAME` file to the repository for
   you — leave it there, it is how the setting survives future builds.
3. GitHub runs a DNS check. It may say "not yet properly configured" for anything from a
   few minutes to a few hours while DNS propagates. That is expected.

## Step 4. Turn on HTTPS

Once the DNS check passes, the **Enforce HTTPS** checkbox becomes available on the same
page. Tick it.

GitHub issues a free Let's Encrypt certificate. This can take up to 24 hours the first
time. Until it completes, the checkbox stays greyed out — that is a waiting problem, not
a broken one.

## Step 5. Update the site's own address

In `_config.yml`, change:

```yaml
url: "https://omarmaali.com"
```

Commit and push. This matters for the RSS feed, the sitemap and the link previews that
LinkedIn generates, all of which build absolute URLs from this value.

---

## When something goes wrong

**Build fails with a YAML error.** Almost always `_config.yml` or `_data/projects.yml`.
YAML is whitespace-sensitive: use spaces not tabs, and keep indentation consistent. If a
value contains a colon followed by a space, wrap the whole value in quotes.

**Site is live but has no styling.** The `url` or `baseurl` in `_config.yml` is wrong.
For a user site, `baseurl` must be an empty string `""`.

**Images don't load.** Check that the `assets/img/` folder actually uploaded, and that
file names match `_data/projects.yml` exactly. GitHub Pages is case-sensitive even
though Windows is not — `Graph-Routed-Memory.webp` and `graph-routed-memory.webp` are
different files as far as the server is concerned.

**A new post doesn't appear.** Three usual causes: the filename is missing its
`YYYY-MM-DD-` prefix, the front matter block is malformed (it needs `---` on its own line
above and below), or the date is in the future. Jekyll hides future-dated posts by
default.

**Everything looks stale.** Hard refresh with `Ctrl+F5`. Browsers cache aggressively and
GitHub's CDN holds assets for a while.

**Nothing in the Actions tab.** The push went to a branch other than the one Pages is
watching. Check **Settings → Pages** and confirm the branch matches what you pushed.
