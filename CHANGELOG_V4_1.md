# Changelog V4.1

## Prodotto e terminologia

- Aggiornata la versione visibile a V4.1.
- Sostituita la formulazione "Planimetria PEE" con "Planimetria di emergenza allegabile al PEE".
- Rafforzata l'indicazione "Scala indicativa" quando il piano non e calibrato.

## Interfaccia

- Nuovo filtro contesto: Completo, Emergenza, Misurazioni, DVR.
- Toolbar riorganizzata con strumenti Struttura ed Elaborati.
- Catalogo elementi ordinato in sezioni funzionali invece di lista piatta.
- Catalogo antincendio esteso con disponibilita funzionale ISO 7010 F001-F019.
- Ricerca rapida nel catalogo elementi.
- Nuova modalita Seleziona con evidenziazione, trascinamento e pannello proprieta.
- Azioni rapide sul canvas per l'elemento selezionato: Proprieta, Duplica, Elimina.
- Pannello Informazioni con versione, funzionamento locale, dati sul dispositivo e avvertenza professionale.
- Indicatore salvataggio: Salvataggio, Salvato, errore.

## Modello dati

- Aggiunti `elevators` e `texts` per piano.
- Vani scala estesi con larghezza/profondita.
- Migrazione dei vecchi marker `scala` verso `stairs`.
- Migrazione dei vecchi marker `ascensore` verso `elevators`.
- Conservazione dei dati V3/V4 tramite normalizzazione difensiva.

## Disegno e modifica

- Inserimento dedicato di vano scala.
- Inserimento dedicato di vano ascensore.
- Inserimento di testi e note.
- Modifica da pannello proprieta per muri, REI/EI, porte, finestre, aree, marker, vie, scale, ascensori e testi.
- Colore del codice/testo sotto i simboli modificabile per ogni marker, utile su basi bianche.
- Duplica/elimina selezione.
- Scorciatoie: S seleziona, M muri, P sposta vista, Cmd/Ctrl+D duplica, Delete elimina.

## Export

- Controlli pre-export distinti per emergenza, misure, DVR e completo.
- PDF con branding "Sicuro 360 - Planimetrie" e nota di responsabilita corretta: elaborato redatto dall'utilizzatore, piattaforma locale di editing grafico, contenuti e conformita in capo al redattore/committente.
- Il piano misurazioni puo mostrare anche macchine, postazioni, depositi e altri elementi DVR come contesto dei punti di misura.
- Tabella misurazioni estesa con codice, tipo, valore, unita, altezza, durata, data e note.
- Legenda aggiornata per vani scala, ascensori, testi e percorsi.
- Service worker aggiornato e manifest PWA aggiunto.
