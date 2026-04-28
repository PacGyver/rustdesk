# Untersuchung: Serverdaten (hbbs/hbbr) "beim Kompilieren fest einbauen"

## Ergebnis
Es gibt jetzt einen Mechanismus in `build.rs`, um hbbs/hbbr/api/key über Build-Umgebungsvariablen als `rustc-env` in das Binary einzubetten.

Zusätzlich existieren mehrere **Laufzeit-Mechanismen**:

1. **Custom-Client per `custom.txt`** (signierte Konfiguration) wird beim Start geladen (`load_custom_client`).
2. **Windows: Konfiguration über EXE-Namen** (`host=`, `key=`, `api=`, `relay=` bzw. codierte Form) wird geparst.
3. **CLI `--config`** kann dieselben Werte aus einem String übernehmen und als Optionen setzen.
4. **`naming`-Binary** erzeugt weiterhin einen codierten EXE-Namen für diese Verteilung.

Damit sind beide Wege möglich: **Compile-Time-Embedding** (Build-Variablen) und **Packaging/Runtime-Konfiguration**.

## Belegstellen im Code
- `build.rs` übernimmt Build-Variablen in `cargo:rustc-env` (`RUSTDESK_EMBED_*`).  
- `src/common.rs`: `load_custom_client()` lädt `custom.txt`; `read_custom_client()` schreibt Werte in `HARD_SETTINGS`.  
- `src/common.rs`: Verbindungs-/API-/Key-Auflösung berücksichtigt eingebettete Build-Werte.
- `src/custom_server.rs`: Parser für EXE-Namen mit `host`, `key`, `api`, `relay`.  
- `src/platform/windows.rs`: `get_license_from_exe_name()` + `bootstrap()` nutzt diese Daten direkt beim Start.  
- `src/core_main.rs`: `--config` setzt `custom-rendezvous-server`, `api-server`, `relay-server`, `key`.  
- `src/naming.rs`: erzeugt codierten Namen (`rustdesk-custom_serverd-...exe`).
