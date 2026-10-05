# Local development notes

Personal notes for running this site locally. See `INSTALL.md` for the upstream
al-folio instructions.

## First-time setup

### Prerequisites

| Tool                | Needed for                                                       | Check                     |
| ------------------- | ---------------------------------------------------------------- | ------------------------- |
| Ruby + Bundler      | Jekyll itself                                                    | `ruby -v`, `bundle -v`    |
| ImageMagick         | responsive WebP images (`imagemagick: enabled` in `_config.yml`) | `magick -version`         |
| Node.js + npm       | Prettier formatting only                                         | `node -v`                 |
| `jupyter-nbconvert` | only if a post embeds a notebook                                 | `which jupyter-nbconvert` |

On Arch: `sudo pacman -S ruby ruby-bundler imagemagick nodejs npm`. The Jupyter
one is optional — without it the build just prints a `jupyter-nbconvert not
found` warning and carries on.

### Steps

From a fresh clone:

```bash
bundle config set --local path vendor/bundle   # once per clone
bundle install                                 # installs into ./vendor/bundle
bundle exec jekyll serve --livereload
```

The first `bundle install` takes a couple of minutes; the first build another
~20 s while ImageMagick generates WebP variants.

Optionally, for the formatter (see below):

```bash
npm install
```

### Why the `bundle config` line is needed

The system gem directory (`/usr/lib/ruby/gems`) is root-owned, so a plain
`bundle install` fails with:

```
Permission denied @ dir_s_mkdir - /usr/lib/ruby/gems/3.4.0/cache (Errno::EACCES)
```

Pointing Bundler at `vendor/bundle` sidesteps it without needing `sudo`. The
setting is stored in `.bundle/config`, which is gitignored, so it's local to
this checkout and persists — but a fresh clone needs it again.

**To fix this once for every Jekyll repo** instead of per-clone, set it
globally:

```bash
bundle config set --global path ~/.gem/bundle
```

That writes to `~/.bundle/config` and applies everywhere, so new projects work
without any per-repo step.

### Troubleshooting

| Symptom                                                | Cause and fix                                                                                                                                                           |
| ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Permission denied @ dir_s_mkdir … /usr/lib/ruby/gems` | Bundler is trying to write to the system gem dir. Run the `bundle config set` line above.                                                                               |
| `Could not find gem …` / `Run 'bundle install'`        | Gems not installed for this clone. Run `bundle install`.                                                                                                                |
| `Your Ruby version is X, but your Gemfile specified Y` | Wrong Ruby active. Note rvm is on `PATH` here but does not manage the shell by default, so `/usr/bin/ruby` wins. `rvm use system` or open a shell without rvm.          |
| `bundler: command not found: jekyll`                   | `bundle install` hasn't run, or ran under a different Ruby.                                                                                                             |
| Bundler and Ruby versions disagree                     | `which bundle` currently resolves to a `ruby/3.3.0` path while `ruby -v` is 3.4.8. If things behave oddly, `gem install bundler` under the active Ruby to realign them. |
| `Address already in use - bind(2) for 127.0.0.1:4000`  | A server is already running. Reuse it, or pass `--port 4001`.                                                                                                           |

## Running the site

From the repo root:

```bash
bundle exec jekyll serve --livereload
```

Then open <http://127.0.0.1:4000>. `--livereload` rebuilds and refreshes the
browser on every file change; drop it to build once and leave it alone.

Useful variations:

| Command                    | Why                                                                                                                                                       |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--incremental`            | Skips unchanged files. Much faster — a full build is ~20 s, mostly ImageMagick regenerating WebP variants and jekyll-scholar processing the bibliography. |
| `--port 4001`              | If port 4000 is already taken.                                                                                                                            |
| `bundle exec jekyll build` | Build into `_site/` without serving.                                                                                                                      |

## Formatting (Prettier)

The `Prettier code formatter` GitHub Action runs `npx prettier . --check` on
every push to `main` and fails the build on any unformatted file. To run the
same check locally, install the toolchain once:

```bash
npm install
```

Then:

```bash
npx prettier . --check    # list offending files, same as CI
npx prettier . --write    # reformat them in place
```

A single file works too: `npx prettier _includes/head.liquid --write`.

Config lives in `.prettierrc` (the `@shopify/prettier-plugin-liquid` plugin is
what lets it parse `.liquid` templates) and `.prettierignore`.

**Version caveat.** `package.json` pins prettier 3.1.1, but the workflow
installs the _latest_ release with `npm install --save-dev --save-exact prettier
@shopify/prettier-plugin-liquid`. If CI ever flags a file that passes locally,
a version skew between the two is the likely cause.

Note `npm install` rewrites the `name` field in `package-lock.json` from
`al-folio` to the directory name, since `package.json` declares no `name`.
That change is harmless but unrelated to formatting — `git checkout
package-lock.json` to drop it.

## Contact form

The email address is deliberately **not** in the repo. `/contact/` posts to
[Web3Forms](https://web3forms.com), which holds the destination address and
relays messages to the inbox. The access key in `_config.yml`
(`web3forms_access_key`) is public by design — it identifies the destination
without revealing it.

Relevant files:

| File                        | Role                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `_pages/contact.md`         | The form, its styling, and the `fetch` that submits it.      |
| `_config.yml`               | `web3forms_access_key` setting.                              |
| `_data/socials.yml`         | `contact_url: /contact/` in place of the old `email:` entry. |
| `_includes/social.liquid`   | Renders the envelope icon linking to `/contact/`.            |
| `_scripts/search.liquid.js` | Same, for search results.                                    |

### Testing it

Only a real browser works. Web3Forms rejects server-side POSTs on the free tier
with `403 — "Use our API in client side"`, so `curl` can't verify the relay.
Serve the site and submit the form by hand.

Each test sends a real email and counts against the free tier's 250
submissions/month, so test sparingly rather than on every reload.

### Spam

The access key is public, so anyone can post to it. The form includes a
`botcheck` honeypot field that catches naive bots. If spam becomes a problem,
Web3Forms offers optional hCaptcha.

## Still exposed

The old email address remains in this repo's git history (commit `a9e6968`) and
in the built HTML on the `gh-pages` branch. Removing it there means rewriting
history on both branches and force-pushing.
