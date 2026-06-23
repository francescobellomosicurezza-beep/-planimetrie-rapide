# Changelog V4

> Nota: per le modifiche della release corrente vedere `CHANGELOG_V4_1.md`.

## Architettura

- Introdotto il modello progetto multipiano.
- Ogni piano possiede muri, muri REI, finestre, porte, scale, aree, presidi, percorsi e foto propri.
- Aggiunta migrazione automatica dei file JSON V3 monopiano.
- Introdotto autosalvataggio IndexedDB con fallback localStorage senza fotografie.

## Disegno

- Aggiunte finestre.
- Aggiunte porte con verso di apertura e opzione REI.
- Aggiunte scale orientabili e collegabili tra piani.
- Aggiunto pittogramma ascensore.
- Aggiunte quote permanenti attivabili.
- Aggiunta gestione layer.

## Produttività

- Aggiunti annulla e ripristina.
- Aggiunta duplicazione dell'ultimo presidio.
- Aggiunta duplicazione completa di un piano.
- Migliorata la gestione delle fotografie con ridimensionamento e compressione.

## Esportazione

- PDF multipiano, con una pagina planimetrica per ogni livello.
- Nome del piano riportato nell'intestazione.
- Legenda distinta per piano.
- Tabella misurazioni cumulativa.
- Istruzioni di emergenza stampate una sola volta.
- Esportazione PNG del piano attivo.
- Nomi file con revisione e data.

## Affidabilità

- Controllo preliminare per planimetria di emergenza esteso all'intero progetto.
- Service worker per cache dell'app e della libreria PDF.
- jsPDF aggiornato alla versione 4.2.1.
