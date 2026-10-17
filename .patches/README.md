# Patches

Unified diff files in this directory are applied at build time, after
the fork is synced from upstream and before the Docker image is built.

This lets us carry local changes against `nhyyebo/aiopicks` without
maintaining a long-lived divergent branch. Patches are applied in
lexical order, so prefix filenames with a 4-digit number.

## Active patches

### `0001-discover-search-capable-catalog-from-manifest.patch`

Upstream hardcodes the metadata add-on search path as
`/catalog/{type}/top/search={query}.json`. This works against Cinemeta
(whose `top` catalog supports the `search` extra) but fails against
metadata add-ons that follow the spec more strictly, such as
AIOMetadata, which exposes `search.movie` / `search.series` instead
and returns an empty `metas` array for any other catalog ID.

The patch makes `MetadataAddonClient` fetch `manifest.json` from the
configured metadata add-on once per `(base_url, content_type)` pair,
locate a catalog whose `extra` array contains `"search"`, and use its
ID. Result is cached for the lifetime of the client. Falls back to
`top` if the manifest cannot be fetched or no search-capable catalog
is declared, which preserves Cinemeta behavior.

Upstream tracking: not yet submitted. When upstream merges an
equivalent fix, delete this file.

### `0002-fetch-meta-endpoint-for-rich-poster-art.patch`

Builds on `0001`. After the search endpoint resolves a title to an
IMDb ID, this patch fetches `/meta/{type}/{id}.json` from the same
metadata add-on and overrides the poster and background fields when
the meta response contains them.

Motivation: many metadata add-ons (notably AIOMetadata with RPDB
enabled) only attach enriched poster URLs — rating overlays,
language-aware fanart, proxied art — to their meta endpoint.
Search responses contain only the raw upstream image URL. Without
this patch, AIOPicks catalogs render with plain TMDB posters even
though the user has configured rich art elsewhere in their stack.

Failures and missing fields are cached per-id so a broken meta
endpoint does not slow down every search. If meta returns nothing
useful, the patch leaves the search-derived poster in place
(no regression).

Upstream tracking: not yet submitted. Same lifecycle as `0001`.

## Workflow

1. Edit code locally against a checkout of upstream.
2. `git diff > .patches/NNNN-short-description.patch` from inside the
   upstream checkout.
3. Commit the patch file to this fork.
4. Push. The workflow applies it on the next run.

## When a patch breaks

If upstream rewrites code that a patch touches, `git apply --check`
fails in the workflow and the build stops. Update the patch against
the new upstream and push again.# Patches

Unified diff files in this directory are applied at build time, after
the fork is synced from upstream and before the Docker image is built.

This lets us carry local changes against `nhyyebo/aiopicks` without
maintaining a long-lived divergent branch. Patches are applied in
lexical order, so prefix filenames with a 4-digit number.

## Active patches

### `0001-discover-search-capable-catalog-from-manifest.patch`

Upstream hardcodes the metadata add-on search path as
`/catalog/{type}/top/search={query}.json`. This works against Cinemeta
(whose `top` catalog supports the `search` extra) but fails against
metadata add-ons that follow the spec more strictly, such as
AIOMetadata, which exposes `search.movie` / `search.series` instead
and returns an empty `metas` array for any other catalog ID.

The patch makes `MetadataAddonClient` fetch `manifest.json` from the
configured metadata add-on once per `(base_url, content_type)` pair,
locate a catalog whose `extra` array contains `"search"`, and use its
ID. Result is cached for the lifetime of the client. Falls back to
`top` if the manifest cannot be fetched or no search-capable catalog
is declared, which preserves Cinemeta behavior.

Upstream tracking: not yet submitted. When upstream merges an
equivalent fix, delete this file.

## Workflow

1. Edit code locally against a checkout of upstream.
2. `git diff > .patches/NNNN-short-description.patch` from inside the
   upstream checkout.
3. Commit the patch file to this fork.
4. Push. The workflow applies it on the next run.

## When a patch breaks

If upstream rewrites code that a patch touches, `git apply --check`
fails in the workflow and the build stops. Update the patch against
the new upstream and push again.
