# Maintaining the community hub

**English** | [Deutsch](MAINTAINERS_DE.md)

The steps of the maintainer of this repository, with the tools of the
[LLM Cross-Compiler Framework](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework). Every `python
scripts/zoo_sign_index.py ...` command runs in your framework checkout, with its Python environment (the one `llmcf`
runs in), on your own computer. The examples assume this repository is cloned next to it as
`../Edge-LLM-Knowledge-Base`.

Two keys, never mixed up:

| Key | Made with | Kept in | Signs |
|---|---|---|---|
| **Index signing key** | `python scripts/zoo_sign_index.py keygen <key name>` | the OS keyring of your computer (service `lcf-community-index`) | `index.json` (writes `index.json.sig`) |
| **Author key** | created on your first `llmcf zoo export` or `llmcf zoo keys` | the framework's encrypted secrets store | the packs you export yourself, like every contributor's |

The index signing key never reaches CI, this repository or any file: no copy, no secret in GitHub.

## Layout

| Path | Defined by | Use |
|---|---|---|
| `contributions/<kind>/` (`target`, `recipe`, `knowledge`) | the framework: `--pr` commits `contributions/<kind>/<file>`, the browser fallback opens `upload/main/contributions/<kind>`, the check workflow accepts only `contributions/<kind>/*.zip` | packs submitted by pull request |
| `packs/<kind>/` | this repository (the framework defines no folder: an index entry needs only an https `url` with `size` and `sha256`) | published packs |
| `index.json`, `index.json.sig` | the framework: it fetches `index.json` from `community_index_url` and the signature from the same address plus `.sig` | the signed index, at the root of `main` |

Addresses on `main`:

- index: `https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/index.json`
- a pack: `https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/packs/<kind>/<file>`

A published pack file is never changed, replaced or renamed: its index entry pins size and sha256. A new version is a
new file (the export names it `contribution-<kind>-<id>-<version>.zip`). `.gitattributes` keeps git from converting
the packs and the index, so the signature stays valid.

## 1. Once: the index signing key

```bash
python scripts/zoo_sign_index.py keygen <key name>
```

Choose a key name that is not your personal name, for example `lcf-community-2026`: it appears as `name` in the public
entry. `keygen` refuses a name that exists already. It prints the entry for the framework's
`configs/community_keys.yml`, valid for two years. Replace `keys: []` there with:

```yaml
keys:
  - key_id: <16 hex characters, as printed>
    pubkey_b64: <44 characters, as printed>
    valid_until: <as printed>
    name: <key name>
```

Commit it in the framework; it reaches users with the next framework release. Before `valid_until`, make a new key
with another name and ship the old and the new entry together for one release. If the keyring entry is lost, make a
new key the same way and ship it with a release.

## 2. Once: the index address in the framework

Set the default of `community_index_url` in the framework's `orchestrator/Core/config_manager.py` (it is `""` today)
to

```text
https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/index.json
```

and update the framework's "State today" notes, in the same release as the key of step 1. Until that release, test
with a framework checkout that has your key in `configs/community_keys.yml`, and give the address as a setting
(`community_index_url:` in `config.yml`) or for one command:

```bash
LLM_FRAMEWORK_COMMUNITY_INDEX_URL=https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/index.json llmcf zoo list --refresh
```

`community_repo` names this repository by default; if it is empty in your settings, set it to
`Smilez1985/Edge-LLM-Knowledge-Base`.

## 3. Review and merge a contribution

1. **The check:** the workflow "Validate contribution" runs on every pull request under `contributions/`. For a
   first-time contributor GitHub may ask you to approve the run in the pull request first. Merge only when it
   passes.
2. **Your review:** the check is not a review. Download the pack, run `llmcf zoo verify <file>.zip`, unzip it into an
   empty folder and read every file before anything of it runs: `pack.json` (author, licence), `target.yml` (boards,
   capabilities), the Dockerfile (every image it pulls, everything it downloads and how it is pinned), `build.sh`,
   the module scripts and helper scripts. A recipe: the model and its pinned commit, the target. A knowledge pack: the
   source address and licence of each page.
3. **Merge** the pull request. The pack is now at `contributions/<kind>/<file>.zip` on `main`.

## 4. Publish: move, index entry, signature

In this repository, on an up-to-date `main`:

```bash
git pull
git mv contributions/<kind>/<file>.zip packs/<kind>/<file>.zip
```

Before you change `index.json`, copy the current one to a file outside the clone, for example
`../index.previous.json` (not for the first index). Then make the entry, in the framework checkout:

```bash
python scripts/zoo_sign_index.py entry ../Edge-LLM-Knowledge-Base/packs/<kind>/<file>.zip \
  --url https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/packs/<kind>/<file>.zip \
  [--trust community|verified] [--title "<title>"]
```

`entry` checks the pack again (as `llmcf zoo verify`) and prints its entry as JSON: kind, id, version, title, author,
licence, trust, URL, size and sha256. `--trust` defaults to `community`; give `verified` only when someone other
than the author reproduced the pack. Add the entry to `entries` in `index.json` (you may add a `description` of at
most 1000 characters). The index has this form; the first one starts with `"version": 1`:

```json
{
  "schema_version": 1,
  "version": 1,
  "generated": "2026-10-22T12:00:00Z",
  "entries": [],
  "revoked": []
}
```

With every change, raise `version` by one and set `generated` to the current UTC time, for example from
`python -c "from datetime import datetime, timezone; print(datetime.now(timezone.utc).strftime('%Y-%m-%dT%H:%M:%SZ'))"`.
Then sign, in the framework checkout:

```bash
python scripts/zoo_sign_index.py sign ../Edge-LLM-Knowledge-Base/index.json --key <key name> --previous ../index.previous.json
```

The first index is signed without `--previous`. `sign` checks `index.json` against the schema (with `--previous`
its `version` must be higher), verifies its own signature and writes `index.json.sig` next to it. Do not touch
`index.json` after signing: the signature covers its exact bytes. Commit both files with the moved pack and push:

```bash
git add index.json index.json.sig
git commit -m "Publish <kind> pack <id> <version> (index version <n>)"
git push
```

Check it with `llmcf zoo list --refresh` (index address and key as in step 2). raw.githubusercontent.com can serve
the old files for a few minutes after a push; an `index.json` and `index.json.sig` of different pushes do not verify,
and the framework keeps its cached index until the next refresh.

## 5. Revoke a pack

Add the pack to `revoked` in `index.json`, raise `version`, set `generated`, sign with `--previous` and push as in
step 4:

```json
"revoked": [
  {"kind": "target", "id": "<id>", "version": "<version>", "reason": "<why, at most 300 characters>"}
]
```

Each revocation needs `kind`, `id` and `reason`. With `"sha256": "<sha256 of the file>"` it revokes exactly that
file; without `sha256` it revokes the `version` given, and without `version` every version of that id. Keep the
pack's entry in `entries`: the Community Hub then shows it as revoked with your reason. Installing it is blocked, and
installed copies are flagged by `llmcf zoo list`, `llmcf zoo verify`, `llmcf mojo` and the hub and are no longer
built.

## 6. The check workflow

`.github/workflows/validate-contribution.yml` is a copy of the framework's template
`scripts/community_repo/validate-contribution.yml`, with two changes: `LCF_REF` names the framework commit
`952a504193dd9e9ea6487c1199f3b7caa88cb7de` (the first one with the `--no-deps` install rule the workflow needs), and
the `README.md` of each contribution folder passes the check. Once a framework release with the community zoo is
published, set `LCF_REF` to its tag, and later to the newest one; take over any change of the template at that tag.
Every action stays pinned by its full commit SHA; Dependabot (`.github/dependabot.yml`) proposes new pins weekly.

## 7. The first pack: the Rockchip/Radxa target

In your framework installation, where `targets/Rockchip` is the target you build and test:

```bash
llmcf zoo keys
llmcf zoo export target Rockchip --author "Smilez1985" --license MIT --pr
```

- `llmcf zoo keys` creates your author key on first use and shows its public keys. The export would create it too;
  every exported pack is signed with it. Users pin the first key they see for an author name, so keep the author
  name and the key for later packs.
- The export copies `targets/Rockchip`, runs the privacy scrub and shows every change as a diff, checks the pack as
  an installation would, signs it and writes `community/contributions/contribution-target-Rockchip-1.0.0.zip` (the
  version comes from `metadata.version` in `target.yml`). `--license MIT` is the default, given here explicitly.
- Before you confirm, check the file the way this repository will:
  `llmcf zoo verify community/contributions/contribution-target-Rockchip-1.0.0.zip`.
- `--pr` asks "Submit this pack to the public community repository?" and then opens the pull request with `gh`. If
  the upload page opens instead, choose **Create a new branch for this commit and start a pull request** (not a
  commit to `main`), so the check runs.
- GUI instead: **Tools > Open Community Hub > Export...**, target `Rockchip`, then **Open pull request**.

Then steps 3 and 4 with `contribution-target-Rockchip-1.0.0.zip`, for example with
`--title "Rockchip / Radxa (RK3566, RK3568, RK3576, RK3588)"`, and the first `index.json` (version 1, signed
without `--previous`).

Note: every framework installation ships `targets/Rockchip`. An install never replaces an existing target, so
`llmcf zoo install Rockchip` is refused where that folder exists; the pack is the first published template and the
example for contributors.
