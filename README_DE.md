# Edge-LLM-Knowledge-Base

[English](README.md) | **Deutsch**

Der Community Hub des [LLM Cross-Compiler Framework](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework).
Hier werden vor allem **Target-Templates** geteilt: Hardware-Templates, zum Beispiel ein Template für einen Orange Pi,
damit das Framework Modelle für Boards bauen kann, die sein Maintainer nicht besitzt. Dazu kommen **Rezepte** (wie ein
Modell für ein Target gebaut wurde) und **Wissens-Packs**. Wie der Name sagt, ist das Repository außerdem eine
Wissensquelle für die Wissensbasis (Qdrant) im LLM Cross-Compiler Framework: Wissens-Packs bringen
Dokumentationsseiten in sie hinein.

Der Model Zoo des Frameworks listet die Packs, die in diesem Repository zum Download bereitliegen, und installiert sie.
Die Liste kommt aus dem signierten Index dieses Repositorys (`index.json` mit seiner Signatur `index.json.sig`).

Alles hier ist Text: Dockerfiles, Build-Skripte, `target.yml`, Rezepte und Seiten. Niemand teilt hier Docker-Images,
Binärdateien oder Modellgewichte, und nichts geht zu Hugging Face: Ein Rezept nennt ein Hugging-Face-Modell mit
festem Commit, und jeder baut es selbst nach.

**Stand:** Noch ist kein Pack veröffentlicht. Das erste wird das Rockchip/Radxa-Target des Maintainers sein, das
einzige Target, das er selbst baut und testet. Die meisten Targets müssen aus der Community kommen: Wenn du ein Board
hast, ist dein Template willkommen.

## Packs nutzen

Im Framework:

- **GUI:** **Tools > Community Hub öffnen** (englische Oberfläche: *Open Community Hub*) listet die Packs des
  signierten Index und deine installierten Packs mit Art, Version, Vertrauensstufe und Autor. **Install** installiert
  ein Pack nach einer Zusammenfassung, die du bestätigst; **Details...** zeigt Lizenz, Download-Adresse, Größe und
  sha256.
- **CLI:** `llmcf zoo list` (mit `--refresh` holt es den Index neu) und `llmcf zoo install <ID> --kind target` (oder
  `recipe`, `knowledge`). `llmcf zoo show <ID>` zeigt ein Pack. `llm-cli` ist dasselbe Programm unter seinem älteren
  Namen.

Bevor etwas installiert wird, prüft das Framework die Signatur des Index, lädt das Pack festgelegt auf Größe und
sha256 herunter, entpackt es in einen Quarantäne-Ordner und führt die Prüfungen seiner Art aus. Vertrauensstufen:
`verified` (geprüft und unabhängig nachgebaut), `community` (geprüft), `local` (eine Datei, die der Index nicht
führt), `revoked` (gesperrt). Details:
[Community-Zoo](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework/blob/main/docs/Features_DE.md#community-zoo).

## Beitragen

1. **Erstell das Template im Framework.** Ein Target: **Tools > Create Hardware Module...** oder
   `llmcf module create`
   ([Leitfaden für Modulentwickler](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework/blob/main/docs/Module_Developer_Guide_DE.md)).
   Ein Target-Template ist der Ordner `targets/<ID>/`: `target.yml` mit seinem `capabilities:`-Block, das
   Dockerfile, die vier Module `source_module.sh`, `config_module.sh`, `convert_module.sh` und `target_module.sh`
   (jedes eine Stufe des Builds, jedes auch einzeln aufrufbar), `build.sh`, das sie in dieser Reihenfolge ausführt,
   und optionale Hilfsskripte. Teste es mit einem echten Build auf deiner Hardware.
2. **Exportier es als Pack.** **Tools > Community Hub öffnen > Exportieren...** oder
   `llmcf zoo export target <ID> --author "<Name>"` (ein Pseudonym genügt); für einen fertigen Build
   `llmcf zoo export recipe <Build-Ordner> --author "<Name>"`, für deine Wissensbasis
   `llmcf zoo export knowledge --author "<Name>"`. Der Export führt die Datenschutz-Bereinigung aus, zeigt jede
   Änderung als Diff, prüft das Pack und signiert es mit deinem Autorenschlüssel. Er schreibt
   `community/contributions/contribution-<Art>-<ID>-<Version>.zip`. Prüf es so, wie dieses Repository es prüfen wird:
   `llmcf zoo verify community/contributions/<Datei>.zip`.
3. **Öffne den Pull Request.** **Open pull request** im Export-Fenster, oder häng `--pr` an den Export-Befehl. Sind
   die GitHub CLI (`gh`, angemeldet) und `git` installiert, forkt das Framework dieses Repository, committet das Pack
   nach `contributions/<Art>/` und öffnet den Pull Request. Sonst öffnet es die Upload-Seite in deinem Browser: Leg
   die `.zip` in
   [contributions/target](https://github.com/Smilez1985/Edge-LLM-Knowledge-Base/upload/main/contributions/target)
   ab (oder `contributions/recipe`, `contributions/knowledge`), und GitHub legt den Fork für dich an. Das Framework
   fragt nie nach einem Token.

Ein Pack pro Pull Request, genau so, wie das Framework es exportiert hat.

## Regeln

- **Nur Packs:** Ein Pull Request legt `.zip`-Packs, die das Framework exportiert hat, in `contributions/<Art>/` ab,
  jedes höchstens 5 MB groß. Sonst nichts.
- **Nur Text:** keine Docker-Images, keine Binärdateien, keine Modellgewichte, keine konvertierten Modelle, nichts,
  was zu Hugging Face hochgeladen wird. Ein Dockerfile holt seine Images festgelegt auf ihren Digest (ausgenommen
  Docker Official Images) und legt jede Version fest, die es herunterlädt (Tags, Commits, Hashes); ein vorgebautes
  llama.cpp wird abgelehnt.
- **Eine Lizenz pro Pack:** Die Lizenz steht in der `pack.json` des Packs (eine SPDX-ID; MIT als Vorgabe für dein
  eigenes Target oder Rezept). Wissens-Packs lassen die Seiten anderer Leute unter deren Bedingungen
  (`NOASSERTION`), mit der Quelladresse jeder Seite und den Lizenztexten und der `NOTICE`, die die Seiten brauchen. Die
  [LICENSE](LICENSE) dieses Repositorys (MIT) gilt für die eigenen Dateien des Repositorys; jedes Pack bringt seine
  eigene Lizenz mit.
- **Keine Secrets, keine persönlichen Daten:** keine API-Schlüssel oder Tokens, keine Home-Ordner, Benutzernamen,
  Hostnamen, IP- oder MAC-Adressen, E-Mail-Adressen. Die Datenschutz-Bereinigung des Exports ersetzt sie; lies ihren
  Diff, bevor du teilst.

Der Prüf-Workflow weist ein Pack ab, das gegen diese Regeln verstößt.

## Nach deinem Pull Request

1. **Automatische Prüfung:** Der Workflow [Validate contribution](.github/workflows/validate-contribution.yml) nimmt
   nur `contributions/<Art>/*.zip` bis 5 MB an und führt `llm-cli zoo verify` für jedes Pack aus: Manifest und Hashes,
   keine Links, keine zusätzlichen oder binären Dateien, die Prüfungen seiner Art, die Autorensignatur und die
   Datenschutz-Bereinigung. Bei einem ersten Beitrag fragt GitHub den Maintainer eventuell, ob der Lauf starten darf.
2. **Review durch den Maintainer:** Der Maintainer liest das Pack und merged den Pull Request.
3. **Veröffentlicht:** Das Pack wandert nach `packs/<Art>/`, sein Eintrag kommt in die `index.json`, und der Index wird
   offline signiert. Der Signaturschlüssel kommt nie in die CI oder in dieses Repository.
4. Danach erscheint das Pack im Community Hub und in `llmcf zoo list`, mit der Vertrauensstufe `community`
   (`verified`, sobald es unabhängig nachgebaut wurde). Ein Pack mit einem Problem kommt in die Sperrliste des Index:
   Installierte Kopien werden markiert und nicht mehr gebaut.

## Aufbau

| Pfad | Was er enthält |
|---|---|
| `contributions/target/`, `contributions/recipe/`, `contributions/knowledge/` | Per Pull Request eingereichte Packs, vor dem Review |
| `packs/target/`, `packs/recipe/`, `packs/knowledge/` | Veröffentlichte Packs; die Einträge des Index zeigen auf diese Dateien |
| `index.json`, `index.json.sig` | Der signierte Index (sobald der erste veröffentlicht ist) |
| `MAINTAINERS_DE.md` | Die Schritte des Maintainers: Signaturschlüssel, Veröffentlichen, Sperren |

Maintainer: [MAINTAINERS_DE.md](MAINTAINERS_DE.md). Die Beitragsanleitung des Frameworks:
[Beiträge zum Community-Zoo](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework/blob/main/CONTRIBUTING_DE.md#beiträge-zum-community-zoo).
