# Xommodo — Tamperproof Prediction Ledger

Ein **append-only, manipulationssicherer Track-Record** von Commodity-Preis-Forecasts
(Flagship: Copper / HG=F). Dieses Repo enthält ausschließlich **veröffentlichte
Vorhersagen und ihre kryptographischen Anker** — keinen Modell- oder Signal-Code.

Zweck: ein extern verifizierbarer, zeitgestempelter Leistungsnachweis. Jede
Vorhersage wird festgeschrieben, *bevor* das Ergebnis bekannt ist, und kann
nachträglich weder verändert, umsortiert noch gelöscht werden, ohne dass die
Verletzung sichtbar wird.

## Struktur

```
ledger.jsonl              Append-only Hash-Chain (eine JSON-Zeile pro Eintrag)
copper/<YYYY-MM-DD>/
  forecast.json           Quantil-Cone (q10/q50/q90) je Horizont + close_t
  metrics.json            Out-of-Sample-Metriken (Coverage, Long-Cash vs BAH, …)
ots/head-<seq>-<hash>.txt(.ots)   OpenTimestamps-Beweise (Bitcoin-Anker)
```

## Drei-Schichten-Integrität

1. **Hash-Chain** — `chain_hash_n = SHA256(eintrag_n | chain_hash_{n-1})`. Jeder
   Eintrag bindet seinen Vorgänger; jede Mutation/Umordnung/Löschung bricht die
   Kette ab dem betroffenen Punkt. `forecast.json`/`metrics.json` sind über ihren
   re-kanonisierten SHA-256 im Ledger gebunden (Feld-Mutation wird erkannt).
2. **OpenTimestamps (Bitcoin)** — der jeweilige HEAD-`chain_hash` wird in die
   Bitcoin-Blockchain gestempelt. Da der Head transitiv die gesamte Kette committet,
   beweist ein Head-Anker, dass der vollständige Verlauf bis dahin zum Block-Zeitpunkt
   existierte — schließt **Tail-Truncation** (das nachträgliche Abschneiden jüngster
   Einträge).
3. **Git/GitHub** — externe, off-box Kopie mit serverseitigen Commit-Zeitstempeln.

## Verifikation (Open-Source-Tooling, kein privater Code nötig)

```bash
# OpenTimestamps-Beweis prüfen (opentimestamps-client):
ots verify ots/head-<seq>-<hash>.txt.ots          # benötigt Bitcoin-Node/Explorer
ots info   ots/head-<seq>-<hash>.txt.ots          # Attestation-Status (pending/Bitcoin)

# Hash-Chain ist mit jedem SHA-256-Werkzeug nachrechenbar (Schema s. ledger.jsonl).
```

Korrekturen werden **nie gelöscht**: ein fehlerhafter Forecast wird durch einen
`invalidation`-Eintrag als INVALIDATED markiert — das Original bleibt in der Kette,
der ehrliche Verlauf bleibt sichtbar.
