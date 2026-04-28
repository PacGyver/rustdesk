# RustDesk-Client mit fest hinterlegten Serverdaten

## Kurzantwort
Wenn du einen Client willst, bei dem `hbbs`/`hbbr` + Key vorkonfiguriert sind, ist der robuste Weg in der aktuellen Codebasis:

1. **Build mit eingebetteten Werten** (Compile-Time-Embedding per Build-Env),
2. **`custom.txt` neben die EXE legen** (wird beim Start geladen), oder
3. **Einmalig Konfiguration per `--config` setzen** (persistente Optionen),

statt den EXE-Namen als Träger der Daten zu verwenden.

## Was der Code aktuell macht
- `load_custom_client()` lädt `custom.txt` beim Start und schreibt Werte in `HARD_SETTINGS`.
- `--config` kann `key`, `custom-rendezvous-server`, `api-server`, `relay-server` setzen.
- Der Verbindungs-/API-/Key-Pfad nutzt die gespeicherten Optionen.

## Praktische Vorgehensweise
### Variante A (eingebacken in die EXE): Build-Umgebungsvariablen
- Beim Build können folgende Variablen gesetzt werden:
  - `RUSTDESK_EMBED_HBBS`
  - `RUSTDESK_EMBED_HBBR`
  - `RUSTDESK_EMBED_API`
  - `RUSTDESK_EMBED_KEY`
- Beispiel:
  - `RUSTDESK_EMBED_HBBS=hbbs.example.com RUSTDESK_EMBED_HBBR=hbbr.example.com RUSTDESK_EMBED_KEY=... cargo build --release`
- Diese Werte werden zur Compile-Zeit in das Binary übernommen.

#### Geht das auch per GitHub Actions?
Ja. Du musst nichts lokal kopieren; die Variablen können im CI-Job gesetzt werden (z. B. aus `secrets`).

Beispiel:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build RustDesk with embedded server config
        env:
          RUSTDESK_EMBED_HBBS: ${{ secrets.RUSTDESK_EMBED_HBBS }}
          RUSTDESK_EMBED_HBBR: ${{ secrets.RUSTDESK_EMBED_HBBR }}
          RUSTDESK_EMBED_API:  ${{ secrets.RUSTDESK_EMBED_API }}
          RUSTDESK_EMBED_KEY:  ${{ secrets.RUSTDESK_EMBED_KEY }}
        run: cargo build --release
```

Im Repository liegt jetzt zusätzlich eine lauffertige Workflow-Datei:
- `.github/workflows/build-embedded-rustdesk.yml`

So nutzt du sie:
1. In GitHub unter **Settings → Secrets and variables → Actions** die Secrets anlegen:
   - `RUSTDESK_EMBED_HBBS`
   - `RUSTDESK_EMBED_HBBR`
   - `RUSTDESK_EMBED_API`
   - `RUSTDESK_EMBED_KEY`
2. Unter **Actions → Build Embedded RustDesk → Run workflow** manuell starten.
3. Nach erfolgreichem Lauf das Artifact `rustdesk-embedded-windows` herunterladen (`rustdesk.exe`).

#### Wenn der Workflow nicht sichtbar ist
- GitHub zeigt Workflow-Namen in der Liste primär aus dem **Default-Branch** an.
- Wenn die Datei nur in einem PR/Feature-Branch liegt, erscheint sie oft erst nach dem Merge in den Default-Branch.
- Prüfe außerdem:
  - **Settings → Actions**: Actions sind für das Repository erlaubt.
  - In der Actions-Ansicht ggf. den **Branch-Filter** auf den Branch mit der Workflow-Datei stellen.

### Variante B (empfohlen): `custom.txt`
- Lege eine gültige `custom.txt` in denselben Ordner wie die EXE.
- Beim Start wird sie automatisch eingelesen.
- Vorteil: funktioniert auch für portable/standalone, ohne Dateinamen-Trick.

### Variante C: `--config` (für Installation/Provisioning)
- Während Provisioning einmal `--config <string>` ausführen.
- Danach sind die Optionen persistiert und werden regulär verwendet.
- **Wichtig:** Die Werte werden **nicht** in die EXE „eingebrannt“, sondern als Konfiguration gespeichert.
- In der aktuellen Implementierung funktioniert `--config` nur im Installationskontext mit Admin-Rechten.

## Wichtiger Hinweis
`custom.txt` wird signaturgeprüft. Eine frei erfundene Datei ohne gültige Signatur wird verworfen.
