# Edge-LLM-Knowledge-Base

**English** | [Deutsch](README_DE.md)

The community hub of the [LLM Cross-Compiler Framework](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework).
People share **target templates** here first: hardware templates such as a template for an Orange Pi, so that the
framework can build models for boards its maintainer does not own. They also share **recipes** (how a model was
built for a target) and **knowledge packs**. As its name says, the repository is also a knowledge source for the
framework's knowledge base (Qdrant): knowledge packs bring documentation pages into it.

The framework's model zoo lists the packs that are ready for download from this repository and installs them. The
list comes from the signed index of this repository (`index.json` with its signature `index.json.sig`).

Everything here is text: Dockerfiles, build scripts, `target.yml`, recipes and pages. Nobody shares Docker images,
binaries or model weights here, and nothing goes to Hugging Face: a recipe names a Hugging Face model at a pinned
commit, and everyone rebuilds it.

**Status:** no pack is published yet. The first one will be the maintainer's Rockchip/Radxa target, the only target
he builds and tests himself. Most targets have to come from the community: if you have a board, your template is
welcome.

## Using the packs

In the framework:

- **GUI:** **Tools > Open Community Hub** lists the packs of the signed index and your installed packs, with kind,
  version, trust level and author. **Install** installs a pack after a summary you confirm; **Details...** shows its
  licence, download address, size and sha256.
- **CLI:** `llmcf zoo list` (with `--refresh` to fetch the index again) and `llmcf zoo install <id> --kind target`
  (or `recipe`, `knowledge`). `llmcf zoo show <id>` shows one pack. `llm-cli` is the same program under its older
  name.

Before anything is installed, the framework checks the index signature, downloads the pack pinned by its size and
sha256, unpacks it into a quarantine folder and runs the checks of its kind. Trust levels: `verified` (reviewed and
reproduced independently), `community` (reviewed), `local` (a file the index does not list), `revoked` (blocked).
Details: [Community zoo](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework/blob/main/docs/Features.md#community-zoo).

## Contributing

1. **Make the template in the framework.** A target: **Tools > Create Hardware Module...** or `llmcf module create`
   ([Module Developer Guide](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework/blob/main/docs/Module_Developer_Guide.md)).
   A target template is the folder `targets/<id>/`: `target.yml` with its `capabilities:` block, the Dockerfile, the
   modules (`build.sh` and the module scripts the generator writes: source, config, convert and target) and optional
   helper scripts. Test it with a real build on your hardware.
2. **Export it as a pack.** **Tools > Open Community Hub > Export...**, or
   `llmcf zoo export target <id> --author "<name>"` (a pseudonym is fine); for a finished build
   `llmcf zoo export recipe <build folder> --author "<name>"`, for your knowledge base
   `llmcf zoo export knowledge --author "<name>"`. The export runs the privacy scrub, shows every change as a diff,
   checks the pack and signs it with your author key. It writes
   `community/contributions/contribution-<kind>-<id>-<version>.zip`. Check it the way this repository will:
   `llmcf zoo verify community/contributions/<file>.zip`.
3. **Open the pull request.** **Open pull request** in the export window, or add `--pr` to the export command. With
   the GitHub CLI (`gh`, logged in) and `git` installed, the framework forks this repository, commits the pack to
   `contributions/<kind>/` and opens the pull request. Otherwise it opens the upload page in your browser: drop the
   `.zip` into [contributions/target](https://github.com/Smilez1985/Edge-LLM-Knowledge-Base/upload/main/contributions/target)
   (or `contributions/recipe`, `contributions/knowledge`), and GitHub creates the fork for you. The framework never
   asks for a token.

One pack per pull request, exactly as the framework exported it.

## Rules

- **Packs only:** a pull request adds `.zip` packs exported by the framework to `contributions/<kind>/`, each at most
  5 MB. Nothing else.
- **Text only:** no Docker images, no binaries, no model weights, no converted models, nothing uploaded to Hugging
  Face. A Dockerfile pulls its images pinned by digest (Docker Official Images excepted) and pins every version it
  downloads (tags, commits, hashes); a prebuilt llama.cpp is refused.
- **A licence per pack:** the licence is in the pack's `pack.json` (an SPDX ID; MIT by default for your own target
  or recipe). Knowledge packs keep other people's pages under their owners' terms (`NOASSERTION`), with each page's
  source address and the licence texts and `NOTICE` the pages need. The [LICENSE](LICENSE) of this repository (MIT)
  covers the repository's own files; each pack carries its own licence.
- **No secrets, no personal data:** no API keys or tokens, no home folders, user names, host names, IP or MAC
  addresses, e-mail addresses. The export's privacy scrub replaces them; read its diff before you share.

The check workflow rejects a pack that breaks these rules.

## After your pull request

1. **Automatic check:** the workflow [Validate contribution](.github/workflows/validate-contribution.yml) accepts
   only `contributions/<kind>/*.zip` of at most 5 MB and runs `llm-cli zoo verify` on each pack: manifest and
   hashes, no links, no extra or binary files, the checks of its kind, the author signature and the privacy scrub.
   For a first-time contributor, GitHub may ask the maintainer to approve the run first.
2. **Maintainer review:** the maintainer reads the pack and merges the pull request.
3. **Published:** the pack moves to `packs/<kind>/`, its entry is added to `index.json`, and the index is signed
   offline. The signing key never reaches CI or this repository.
4. The pack then appears in the Community Hub and in `llmcf zoo list`, with trust `community` (`verified` once it
   was reproduced independently). A pack with a problem is added to the index's revoked list: installed copies are
   flagged and no longer built.

## Layout

| Path | What it holds |
|---|---|
| `contributions/target/`, `contributions/recipe/`, `contributions/knowledge/` | Packs submitted by pull request, before review |
| `packs/target/`, `packs/recipe/`, `packs/knowledge/` | Published packs; the index entries point to these files |
| `index.json`, `index.json.sig` | The signed index (once the first one is published) |
| `MAINTAINERS.md` | The maintainer's steps: signing key, publishing, revoking |

Maintainers: [MAINTAINERS.md](MAINTAINERS.md). The framework's contributor guide:
[Contributing to the community zoo](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework/blob/main/CONTRIBUTING.md#contributing-to-the-community-zoo).
