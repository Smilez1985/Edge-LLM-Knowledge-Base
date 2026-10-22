# Maintaining the community hub

**English** | [Deutsch](MAINTAINERS_DE.md)

In the [LLM Cross-Compiler Framework](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework) on your own computer:
**Tools > Open Community Hub > Maintainer**. In a terminal the same is `llmcf zoo maintainer key|review|revoke`.

1. **Create signing key...** (once): the index signing key goes into your computer's keyring and never leaves it. Save
   the public entry it shows and add it to the framework's `configs/community_keys.yml`; it reaches users with the
   next framework release (together with `community_index_url` pointing to this repository's `index.json`). The key
   is valid for two years: before that, create a new one with another name and ship both entries for one release.
2. **Review and publish**: **Load contributions**, choose a pull request, **Review selected**, read every file of the
   pack, then **Publish...** and answer "Publish?" with Yes. The framework merges the contribution, moves the pack to
   `packs/<kind>/`, adds its entry to `index.json`, signs it and merges that change as a pull request once the check
   `validate` has passed. Only a pull request into `main` that adds nothing but pack files under
   `contributions/<kind>/` is published this way; the review names anything else it changes (look at that on GitHub).
3. **Revoke a published pack...**: choose the pack, the scope and the reason; it is added to the signed index the
   same way.

What you need: the GitHub CLI logged in (`gh auth login`, and `gh auth setup-git` so git pushes with the same login)
with write access to this repository, and git with your name and e-mail set. The framework uses their own stored
login and never asks for a token. `main` is protected (merges only through pull requests with the required check
`validate`), so every change is a pull request; nothing is pushed to `main`. Every step is shown and written to the
framework's audit log. **Cancel** stops before the next step; after the push it also closes the publication's own pull
request. When a run stops, the message says what happened on GitHub and the next step; a contribution that was merged
but not published is listed again.

Layout: contributions arrive as pull requests under `contributions/<kind>/*.zip`; published packs live in
`packs/<kind>/` and are never changed or replaced (a new version is a new file); `index.json` and `index.json.sig` sit
at the root of `main`. The check workflow `.github/workflows/validate-contribution.yml` names the framework release it
checks with (`LCF_REF`): set it to the tag of each new release.

## Appendix: by hand

The same steps without the framework's windows, with `scripts/zoo_sign_index.py` in a framework checkout:
`keygen <key name>` (the signing key), `entry <pack.zip> --url <raw address>` (an index entry for a checked pack) and
`sign index.json --key <key name> --previous <old index.json>` (the signature; raise `version` and `generated` first).
Then commit on a new branch and open a pull request; never push to `main`. Details: the framework's
`docs/Features.md`, section "Maintainer tools".
