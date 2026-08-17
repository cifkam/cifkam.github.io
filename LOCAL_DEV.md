# Local development notes

Personal notes for running this site locally. See `INSTALL.md` for the upstream
al-folio instructions.

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

### Gem path

The system gem directory (`/usr/lib/ruby/gems`) isn't writable, so a plain
`bundle exec jekyll serve` fails with `Permission denied @ dir_s_mkdir`. Gems
are installed into `vendor/bundle` instead, configured once with:

```bash
bundle config set --local path vendor/bundle
bundle install
```

This is stored in `.bundle/config`, which is gitignored — it's local to this
checkout and persists, so it shouldn't need repeating. On a fresh clone, run
both lines again.

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
