# Den Community Hub betreuen

[English](MAINTAINERS.md) | **Deutsch**

Im [LLM Cross-Compiler Framework](https://github.com/Smilez1985/LLM_Cross-Compiler_Framework) auf deinem eigenen
Rechner: **Tools > Community Hub öffnen > Maintainer**. Im Terminal dasselbe mit `llmcf zoo maintainer
key|review|revoke`.

1. **Signaturschlüssel anlegen...** (einmalig): Der Signaturschlüssel des Index kommt in den Schlüsselbund deines
   Rechners und verlässt ihn nie. Speichere den öffentlichen Eintrag, den er zeigt, und trage ihn in
   `configs/community_keys.yml` des Frameworks ein; er erreicht alle mit dem nächsten Framework-Release (zusammen mit
   `community_index_url`, das auf die `index.json` dieses Repositorys zeigt). Der Schlüssel gilt zwei Jahre: Lege
   vorher einen neuen mit anderem Namen an und liefere beide Einträge ein Release lang zusammen aus.
2. **Prüfen und veröffentlichen**: **Beiträge laden**, einen Pull Request wählen, **Auswahl prüfen**, jede Datei des
   Packs lesen, dann **Veröffentlichen...** und „Veröffentlichen?“ mit Ja beantworten. Das Framework mergt den
   Beitrag, verschiebt das Pack nach `packs/<Art>/`, trägt es in die `index.json` ein, signiert sie und mergt diese
   Änderung als Pull Request, sobald die Prüfung `validate` bestanden ist. So veröffentlicht wird nur ein Pull Request
   nach `main`, der nichts als Pack-Dateien unter `contributions/<Art>/` hinzufügt; die Prüfung nennt alles andere,
   was er ändert (sieh dir das auf GitHub an).
3. **Veröffentlichtes Pack zurückziehen...**: Pack, Umfang und Grund wählen; der Rückzug kommt auf demselben Weg in
   den signierten Index.

Was du brauchst: die GitHub CLI, angemeldet (`gh auth login`, und `gh auth setup-git`, damit git mit demselben Login
pusht) mit Schreibrechten an diesem Repository, und git mit eingestelltem Namen und E-Mail. Das Framework nutzt deren
eigenes gespeichertes Login und fragt nie nach einem Token. `main` ist geschützt (Merges nur über Pull Requests mit der
Pflichtprüfung `validate`), also ist jede Änderung ein Pull Request; nach `main` wird nichts gepusht. Jeder Schritt
wird gezeigt und ins Audit-Log des Frameworks geschrieben. **Abbrechen** hält vor dem nächsten Schritt an; nach dem
Push schließt es auch den eigenen Pull Request der Veröffentlichung. Hält ein Lauf an, sagt die Meldung, was auf
GitHub geschehen ist, und nennt den nächsten Schritt; ein gemergter, noch nicht veröffentlichter Beitrag steht wieder
in der Liste.

Aufbau: Beiträge kommen als Pull Requests unter `contributions/<Art>/*.zip`; veröffentlichte Packs liegen in
`packs/<Art>/` und werden nie geändert oder ersetzt (eine neue Version ist eine neue Datei); `index.json` und
`index.json.sig` liegen im Wurzelordner von `main`. Der Prüf-Workflow `.github/workflows/validate-contribution.yml`
nennt das Framework-Release, mit dem er prüft (`LCF_REF`): Setze es auf den Tag jedes neuen Release.

## Anhang: von Hand

Dieselben Schritte ohne die Fenster des Frameworks, mit `scripts/zoo_sign_index.py` in einem Framework-Checkout:
`keygen <Schlüsselname>` (der Signaturschlüssel), `entry <pack.zip> --url <Raw-Adresse>` (ein Index-Eintrag für ein
geprüftes Pack) und `sign index.json --key <Schlüsselname> --previous <alte index.json>` (die Signatur; erhöhe vorher
`version` und `generated`). Dann auf einem neuen Branch committen und einen Pull Request öffnen; nie nach `main`
pushen. Einzelheiten: `docs/Features_DE.md` des Frameworks, Abschnitt „Werkzeuge für Maintainer“.
