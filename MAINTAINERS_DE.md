# Den Community Hub betreuen

[English](MAINTAINERS.md) | **Deutsch**

Die Schritte des Maintainers dieses Repositorys, mit den Werkzeugen des
[LLM Cross-Compiler Framework](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework). Jeder Befehl `python
scripts/zoo_sign_index.py ...` läuft in deinem Framework-Checkout, mit dessen Python-Umgebung (der, in der `llmcf`
läuft), auf deinem eigenen Rechner. Die Beispiele nehmen an, dass dieses Repository daneben als
`../Edge-LLM-Knowledge-Base` geklont ist.

Zwei Schlüssel, die du nie verwechselst:

| Schlüssel | Erzeugt mit | Liegt in | Signiert |
|---|---|---|---|
| **Index-Signaturschlüssel** | `python scripts/zoo_sign_index.py keygen <Schlüsselname>` | dem Schlüsselbund des Betriebssystems auf deinem Rechner (Dienst `lcf-community-index`) | die `index.json` (schreibt `index.json.sig`) |
| **Autorenschlüssel** | entsteht bei deinem ersten `llmcf zoo export` oder `llmcf zoo keys` | dem verschlüsselten Secrets-Speicher des Frameworks | die Packs, die du selbst exportierst, wie bei jedem Beitragenden |

Der Index-Signaturschlüssel kommt nie in die CI, in dieses Repository oder in eine Datei: keine Kopie, kein Secret bei
GitHub.

## Aufbau

| Pfad | Festgelegt von | Zweck |
|---|---|---|
| `contributions/<Art>/` (`target`, `recipe`, `knowledge`) | dem Framework: `--pr` committet `contributions/<Art>/<Datei>`, der Browser-Weg öffnet `upload/main/contributions/<Art>`, der Prüf-Workflow nimmt nur `contributions/<Art>/*.zip` an | per Pull Request eingereichte Packs |
| `packs/<Art>/` | diesem Repository (das Framework legt keinen Ordner fest: Ein Index-Eintrag braucht nur eine https-`url` mit `size` und `sha256`) | veröffentlichte Packs |
| `index.json`, `index.json.sig` | dem Framework: Es holt die `index.json` von `community_index_url` und die Signatur von derselben Adresse mit angehängtem `.sig` | der signierte Index, im Wurzelordner von `main` |

Adressen auf `main`:

- Index: `https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/index.json`
- ein Pack: `https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/packs/<Art>/<Datei>`

Eine veröffentlichte Pack-Datei wird nie geändert, ersetzt oder umbenannt: Ihr Index-Eintrag legt Größe und sha256
fest. Eine neue Version ist eine neue Datei (der Export nennt sie `contribution-<Art>-<ID>-<Version>.zip`). Die
`.gitattributes` hält git davon ab, die Packs und den Index umzuwandeln, damit die Signatur gültig bleibt.

## 1. Einmalig: der Index-Signaturschlüssel

```bash
python scripts/zoo_sign_index.py keygen <Schlüsselname>
```

Wähl einen Schlüsselnamen, der nicht dein persönlicher Name ist, zum Beispiel `lcf-community-2026`: Er steht als
`name` im öffentlichen Eintrag. `keygen` lehnt einen Namen ab, den es schon gibt. Es gibt den Eintrag für die
`configs/community_keys.yml` des Frameworks aus, zwei Jahre gültig. Ersetz dort `keys: []` durch:

```yaml
keys:
  - key_id: <16 Hex-Zeichen, wie ausgegeben>
    pubkey_b64: <44 Zeichen, wie ausgegeben>
    valid_until: <wie ausgegeben>
    name: <Schlüsselname>
```

Committe das im Framework; mit dem nächsten Framework-Release kommt es bei den Benutzern an. Vor `valid_until`
erzeugst du einen neuen Schlüssel mit einem anderen Namen und lieferst den alten und den neuen Eintrag ein Release lang
zusammen aus. Geht der Eintrag im Schlüsselbund verloren, erzeugst du auf dieselbe Weise einen neuen Schlüssel und
lieferst ihn mit einem Release aus.

## 2. Einmalig: die Index-Adresse im Framework

Setz den Standardwert von `community_index_url` in `orchestrator/Core/config_manager.py` des Frameworks (heute `""`)
auf

```text
https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/index.json
```

und pass die Hinweise „Stand heute“ des Frameworks an, im selben Release wie den Schlüssel aus Schritt 1. Bis zu
diesem Release testest du mit einem Framework-Checkout, dessen `configs/community_keys.yml` deinen Schlüssel enthält,
und gibst die Adresse als Einstellung an (`community_index_url:` in der `config.yml`) oder für einen Befehl:

```bash
LLM_FRAMEWORK_COMMUNITY_INDEX_URL=https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/index.json llmcf zoo list --refresh
```

`community_repo` nennt standardmäßig dieses Repository; ist es in deinen Einstellungen leer, setz es auf
`Smilez1985/Edge-LLM-Knowledge-Base`.

## 3. Einen Beitrag prüfen und mergen

1. **Die Prüfung:** Der Workflow „Validate contribution“ läuft bei jedem Pull Request unter `contributions/`. Bei
   einem ersten Beitrag fragt GitHub dich eventuell zuerst im Pull Request, ob der Lauf starten darf. Merge nur, wenn
   er durchläuft.
2. **Dein Review:** Die Prüfung ersetzt kein Review. Lad das Pack herunter, führ `llmcf zoo verify <Datei>.zip` aus,
   entpack es in einen leeren Ordner und lies jede Datei, bevor irgendetwas davon läuft: `pack.json` (Autor, Lizenz),
   `target.yml` (Boards, Capabilities), das Dockerfile (jedes Image, das es holt, alles, was es herunterlädt, und wie
   es festgelegt ist), `build.sh`, die Modul-Skripte und Hilfsskripte. Bei einem Rezept: das Modell mit seinem festen
   Commit, das Target. Bei einem Wissens-Pack: Quelladresse und Lizenz jeder Seite.
3. **Merge** den Pull Request. Das Pack liegt jetzt in `contributions/<Art>/<Datei>.zip` auf `main`.

## 4. Veröffentlichen: verschieben, Index-Eintrag, Signatur

In diesem Repository, auf einem aktuellen `main`:

```bash
git pull
git mv contributions/<Art>/<Datei>.zip packs/<Art>/<Datei>.zip
```

Bevor du die `index.json` änderst, kopierst du die aktuelle in eine Datei außerhalb des Klons, zum Beispiel
`../index.previous.json` (nicht beim ersten Index). Dann erzeugst du den Eintrag, im Framework-Checkout:

```bash
python scripts/zoo_sign_index.py entry ../Edge-LLM-Knowledge-Base/packs/<Art>/<Datei>.zip \
  --url https://raw.githubusercontent.com/Smilez1985/Edge-LLM-Knowledge-Base/main/packs/<Art>/<Datei>.zip \
  [--trust community|verified] [--title "<Titel>"]
```

`entry` prüft das Pack noch einmal (wie `llmcf zoo verify`) und gibt seinen Eintrag als JSON aus: Art, ID, Version,
Titel, Autor, Lizenz, Vertrauensstufe, URL, Größe und sha256. `--trust` ist standardmäßig `community`; gib `verified`
nur an, wenn jemand anderes als der Autor das Pack nachgebaut hat. Trag den Eintrag in `entries` der `index.json` ein
(du kannst eine `description` mit höchstens 1000 Zeichen ergänzen). Der Index hat diese Form; der erste beginnt mit
`"version": 1`:

```json
{
  "schema_version": 1,
  "version": 1,
  "generated": "2026-10-22T12:00:00Z",
  "entries": [],
  "revoked": []
}
```

Bei jeder Änderung erhöhst du `version` um eins und setzt `generated` auf die aktuelle UTC-Zeit, zum Beispiel aus
`python -c "from datetime import datetime, timezone; print(datetime.now(timezone.utc).strftime('%Y-%m-%dT%H:%M:%SZ'))"`.
Dann signierst du, im Framework-Checkout:

```bash
python scripts/zoo_sign_index.py sign ../Edge-LLM-Knowledge-Base/index.json --key <Schlüsselname> --previous ../index.previous.json
```

Der erste Index wird ohne `--previous` signiert. `sign` prüft die `index.json` gegen das Schema (mit `--previous`
muss ihre `version` höher sein), prüft seine eigene Signatur und schreibt `index.json.sig` daneben. Fass die
`index.json` nach dem Signieren nicht mehr an: Die Signatur deckt ihre genauen Bytes ab. Committe beide Dateien mit
dem verschobenen Pack und pushe:

```bash
git add index.json index.json.sig
git commit -m "Publish <kind> pack <id> <version> (index version <n>)"
git push
```

Prüf es mit `llmcf zoo list --refresh` (Index-Adresse und Schlüssel wie in Schritt 2). raw.githubusercontent.com kann
nach einem Push noch einige Minuten die alten Dateien ausliefern; eine `index.json` und eine `index.json.sig` aus
verschiedenen Pushes verifizieren nicht, und das Framework behält bis zum nächsten Refresh seinen zwischengespeicherten
Index.

## 5. Ein Pack sperren

Nimm das Pack in `revoked` der `index.json` auf, erhöh `version`, setz `generated`, signier mit `--previous` und pushe
wie in Schritt 4:

```json
"revoked": [
  {"kind": "target", "id": "<ID>", "version": "<Version>", "reason": "<Grund, höchstens 300 Zeichen>"}
]
```

Jede Sperre braucht `kind`, `id` und `reason`. Mit `"sha256": "<sha256 der Datei>"` sperrt sie genau diese Datei;
ohne `sha256` die angegebene `version`, und ohne `version` jede Version dieser ID. Lass den Eintrag des Packs in
`entries` stehen: Der Community Hub zeigt es dann als gesperrt mit deinem Grund. Installieren ist blockiert, und
installierte Kopien werden von `llmcf zoo list`, `llmcf zoo verify`, `llmcf mojo` und dem Hub markiert und nicht mehr
gebaut.

## 6. Der Prüf-Workflow

`.github/workflows/validate-contribution.yml` ist eine Kopie der Framework-Vorlage
`scripts/community_repo/validate-contribution.yml`, mit zwei Änderungen: `LCF_REF` nennt den Framework-Commit
`952a504193dd9e9ea6487c1199f3b7caa88cb7de` (den ersten mit der `--no-deps`-Installationsregel, die der Workflow
braucht), und die `README.md` jedes Beitragsordners besteht die Prüfung. Sobald ein Framework-Release mit dem
Community-Zoo veröffentlicht ist, setzt du `LCF_REF` auf dessen Tag, später auf den neuesten; übernimm dabei jede
Änderung der Vorlage an diesem Tag. Jede Action bleibt auf ihren vollen Commit-SHA festgelegt; Dependabot
(`.github/dependabot.yml`) schlägt wöchentlich neue Pins vor.

## 7. Das erste Pack: das Rockchip/Radxa-Target

In deiner Framework-Installation, in der `targets/Rockchip` das Target ist, das du baust und testest:

```bash
llmcf zoo keys
llmcf zoo export target Rockchip --author "Smilez1985" --license MIT --pr
```

- `llmcf zoo keys` erzeugt beim ersten Aufruf deinen Autorenschlüssel und zeigt seine öffentlichen Schlüssel. Der
  Export würde ihn ebenfalls erzeugen; jedes exportierte Pack wird damit signiert. Benutzer merken sich den ersten
  Schlüssel, den sie für einen Autorennamen sehen, also behalte Autorennamen und Schlüssel für spätere Packs.
- Der Export kopiert `targets/Rockchip`, führt die Datenschutz-Bereinigung aus und zeigt jede Änderung als Diff,
  prüft das Pack wie eine Installation, signiert es und schreibt
  `community/contributions/contribution-target-Rockchip-1.0.0.zip` (die Version kommt aus `metadata.version` in der
  `target.yml`). `--license MIT` ist die Vorgabe, hier ausdrücklich angegeben.
- Bevor du bestätigst, prüf die Datei so, wie dieses Repository es tun wird:
  `llmcf zoo verify community/contributions/contribution-target-Rockchip-1.0.0.zip`.
- `--pr` fragt „Submit this pack to the public community repository?“ und öffnet dann den Pull Request mit `gh`.
  Öffnet sich stattdessen die Upload-Seite, wähl **Create a new branch for this commit and start a pull request**
  (keinen Commit auf `main`), damit die Prüfung läuft.
- Stattdessen in der GUI: **Tools > Community Hub öffnen > Exportieren...**, Target `Rockchip`, dann
  **Open pull request**.

Danach die Schritte 3 und 4 mit `contribution-target-Rockchip-1.0.0.zip`, zum Beispiel mit
`--title "Rockchip / Radxa (RK3566, RK3568, RK3576, RK3588)"`, und der ersten `index.json` (Version 1, ohne
`--previous` signiert).

Hinweis: Jede Framework-Installation bringt `targets/Rockchip` mit. Eine Installation ersetzt nie ein vorhandenes
Target, deshalb wird `llmcf zoo install Rockchip` dort abgelehnt, wo dieser Ordner existiert; das Pack ist das erste
veröffentlichte Template und das Beispiel für Beitragende.
