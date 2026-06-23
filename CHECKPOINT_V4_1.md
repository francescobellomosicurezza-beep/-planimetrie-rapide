# Checkpoint stabile V4.1

Data: 18 giugno 2026

## Completato realmente

- Audit iniziale della struttura statica.
- Versione visibile V4.1.
- Toolbar per contesto: Completo, Emergenza, Misurazioni, DVR.
- Modalita Seleziona con evidenziazione, trascinamento, pannello proprieta, duplica ed elimina.
- Azioni rapide su canvas per elemento selezionato: Proprieta, Duplica, Elimina.
- Vano scala come elemento strutturale dedicato, salvato in JSON e migrato da vecchi marker `scala`.
- Ascensore come elemento strutturale dedicato, salvato in JSON e migrato da vecchi marker `ascensore`.
- Testi/note libere.
- Catalogo elementi ordinato in sezioni e filtrato per contesto.
- Ricerca rapida nel catalogo.
- Catalogo antincendio con disponibilita funzionale ISO 7010 F001-F019.
- Pittogrammi schematici proprietari per i nuovi simboli antincendio.
- Colore del codice/testo sotto i marker modificabile.
- Piano misurazioni con elementi DVR visibili come contesto.
- Branding PDF: `Sicuro 360 - Planimetrie`.
- Nota PDF di responsabilita: elaborato redatto dall'utilizzatore, contenuti e conformita in capo a redattore/committente.
- Manifest PWA e icona locale.
- Demo separata `demo_officina_v4_1.json`.

## Parziale

- Seleziona/Modifica: funzionante per selezione, spostamento, proprieta, duplicazione ed eliminazione. Non ha ancora maniglie complete per ogni vertice.
- Vano scala: inserimento, modifica base, dimensioni, rotazione, collegamento piano e migrazione. Non testato in browser reale dopo le ultime modifiche.
- Ascensore strutturale: inserimento, tipologia, dimensioni, note e migrazione. Non testato in browser reale dopo le ultime modifiche.
- Porte e finestre: creazione, proprieta e modifica coordinate. Manca snap automatico al muro piu vicino.
- Misurazioni: punti, codici, campi estesi, tabella PDF e contesto macchine. Manca rinumerazione automatica dedicata e filtri avanzati per tipologia.
- DVR: catalogo esteso e visibilita nel piano misure. Mancano forme DVR avanzate con maniglie complete.
- Export PDF/PNG: pipeline esistente mantenuta e aggiornata. Avvio export verificabile solo da browser; non e stato eseguito PDF reale in questa sessione.
- Mobile/responsive: CSS e bottom sheet presenti; verifica visuale reale non eseguita.
- Offline/service worker: cache aggiornata e asset statici serviti; test offline reale via DevTools non eseguito.

## Non ancora iniziato o non completato

- Maniglie grafiche complete per ridimensionamento/rotazione/vertici di tutti gli elementi.
- Snap automatico porte/finestre al muro.
- Crop/rotazione immagine di base.
- Import PDF/DWG/DXF.
- Export SVG.
- Allinea/distribuisci/porta avanti/porta indietro.
- Test browser automatici completi con Playwright/Puppeteer.
- Collaudo visuale completo su desktop, tablet e smartphone.
- Verifica PDF reale di tutte e quattro le esportazioni dopo le ultime modifiche.
- Verifica offline reale con service worker dopo clear cache.

## Smoke test richiesto

Eseguito realmente:

- caricamento applicazione: verificato HTTP 200 su `http://localhost:8081/`;
- sintassi `index.html`: OK;
- sintassi `sw.js`: OK;
- parsing JSON demo e manifest: OK;
- presenza codice per creazione muro, catalogo, ricerca, inserimento simbolo, salvataggio JSON, riapertura JSON e export PDF: verificata staticamente.

Non eseguito realmente per assenza automazione browser installata:

- click/drag canvas per creazione muro;
- apertura catalogo in browser;
- ricerca simbolo in UI browser;
- inserimento simbolo in UI browser;
- salvataggio JSON via download browser;
- riapertura JSON via file picker;
- avvio export PDF con jsPDF in browser.

## Ripresa lavori consigliata

1. Installare o rendere disponibile Playwright.
2. Eseguire smoke test browser reale su `http://localhost:8081/`.
3. Verificare console e Network.
4. Generare i quattro PDF con `demo_officina_v4_1.json`.
5. Controllare leggibilita pittogrammi, legenda, tabelle e footer.
6. Testare smartphone 390x844.
7. Solo dopo il collaudo, proseguire con maniglie, snap porte/finestre e foto base.
