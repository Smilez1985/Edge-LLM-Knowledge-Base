# Recipe packs (contributions)

Pull requests put recipe packs here: `contribution-recipe-<id>-<version>.zip`, exported from a finished build with
`llmcf zoo export recipe <build folder> --author "<name>" --pr` or uploaded here in the browser. A recipe pins the
Hugging Face model's commit and the target; it carries no weights. Only pack files of at most 5 MB; the check
workflow runs `llm-cli zoo verify` on each. After review the maintainer moves an accepted pack to
[packs/recipe](../../packs/recipe) and adds it to the signed index. How to contribute: [README](../../README.md#contributing).

Hier legen Pull Requests Rezept-Packs ab; nach dem Review wandert ein angenommenes Pack nach `packs/recipe/` und in den signierten Index ([README_DE](../../README_DE.md#beitragen)).
